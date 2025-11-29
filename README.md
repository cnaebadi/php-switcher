# php-switcher — Lightweight PHP version switcher for macOS (Homebrew)

A tiny, reliable helper to switch the active Homebrew PHP version on macOS by updating only the PHP PATH line in `~/.zshrc`.  
Designed for local development workflows (Laravel Valet, local CLI), **not** for multi-service containerized environments.

## Why this exists
Docker is great, but not everyone uses it for small single-service projects. This small tool lets you quickly switch the system PHP version installed via Homebrew without accidentally modifying your shell functions.

## Features
- Detects current `php` major.minor version (e.g. `8.3`) from `php -v`.
- Switches to a specified `php@<major.minor>` installed by Homebrew.
- Safe checks: warns if the target version is not installed.
- Replaces **only** the PHP PATH line in `~/.zshrc` to avoid unnecessary changes or breaking your switch function.

## Install / Usage
Place the function in your `~/.zshrc` (or source `switch.sh`), reload shell, then:

```bash
# example
switch php 8.2
switch php 8.3
```

## Example function (best current version)
```bash
switch() {
  if [ "$1" = "php" ] && [ -n "$2" ]; then
    TARGET_VERSION="$2"
    TARGET_FORMULA="php@$TARGET_VERSION"

    echo "Detecting current PHP version..."

    # Extract current major.minor version (e.g., 8.3)
    CURRENT_VERSION=$(php -v | head -n 1 | grep -oE '[0-9]+\.[0-9]+')
    CURRENT_FORMULA="php@$CURRENT_VERSION"

    echo "Current PHP: $CURRENT_FORMULA"
    echo "Target PHP:  $TARGET_FORMULA"

    # Check if target is installed
    if [ ! -d "/opt/homebrew/Cellar/$TARGET_FORMULA" ]; then
      echo "Target PHP version is not installed: $TARGET_FORMULA"
      echo "Install it with: brew install $TARGET_FORMULA"
      return 1
    fi

    echo "Switching PHP version..."

    brew unlink php >/dev/null 2>&1
    brew unlink "$CURRENT_FORMULA" >/dev/null 2>&1

    brew link "$TARGET_FORMULA" --force --overwrite

    # Replace version (e.g., 8.3 → 8.2)
    sed -i '' "/opt\\/homebrew\\/opt\\/php@/s|php@$CURRENT_VERSION|php@$TARGET_VERSION|" ~/.zshrc

    echo "Reloading shell..."
    source ~/.zshrc

    php -v
  else
    echo "️ Usage: switch php <version>"
    echo "   Example: switch php 8.2"
  fi
}
```

## Contributing
Pull requests are welcome.  
Feel free to open an issue if you have ideas or improvements.

## License
MIT