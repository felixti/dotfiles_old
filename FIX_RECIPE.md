# Neovim Plugin Update Recipe

Quick fix guide for common Neovim 0.11+ compatibility issues with AstroNvim.

## Issues Fixed

1. **aerial.nvim**: `attempt to call method 'start' (a nil value)`
2. **vim.lsp.with()**: Deprecation warning at startup
3. **nvim-treesitter**: `gzip: stdin: not in gzip format` error for jsonc parser

---

## Quick Fix (One Command)

```bash
cd ~/.config/nvim && \
sed -i 's/"5c4e2da4486da5f9b798ea9a0f1fc5c6bcd3d9cf"/"645d108a5242ec7b378cbe643eb6d04d4223f034"/g' lazy-lock.json && \
nvim --headless -c 'Lazy! restore' -c 'qa'
```

---

## Manual Step-by-Step Fix

### Step 1: Update aerial.nvim in lazy-lock.json

Edit `~/.config/nvim/lazy-lock.json` and change:

```json
# FROM:
"aerial.nvim": { "branch": "master", "commit": "5c4e2da4486da5f9b798ea9a0f1fc5c6bcd3d9cf" },

# TO:
"aerial.nvim": { "branch": "master", "commit": "645d108a5242ec7b378cbe643eb6d04d4223f034" },
```

### Step 2: Apply the Update

```bash
nvim --headless -c 'Lazy! restore' -c 'qa'
```

Or inside Neovim:
```
:Lazy restore
```

---

## Alternative: Update All Plugins

If you prefer to update everything (may take longer):

```bash
nvim --headless -c 'Lazy! sync' -c 'qa'
```

Or inside Neovim:
```
:Lazy sync
```

---

## Fix Treesitter Parser Issues

If you see `gzip: stdin: not in gzip format` for jsonc or other parsers:

```bash
# Clear parser cache
rm -rf ~/.local/share/nvim/site/parser/jsonc.so
rm -rf ~/.local/share/nvim/lazy/nvim-treesitter/parser/jsonc.so

# Reinstall parsers in Neovim
nvim -c 'TSInstall! jsonc' -c 'qa'
```

Or manually install jsonc:
```bash
mkdir -p ~/.local/share/nvim/site/parser
cd /tmp
git clone --depth 1 https://github.com/tree-sitter/tree-sitter-json.git
cd tree-sitter-json
cargo build --release 2>/dev/null || make
cp libtree-sitter-json.so ~/.local/share/nvim/site/parser/jsonc.so
```

---

## Verify the Fix

1. Open Neovim: `nvim`
2. Open any file: `:e test.lua`
3. Check for errors - you should see NO red error messages
4. Test aerial: `:AerialToggle`
5. Check health: `:checkhealth vim.deprecated`

---

## What Was Fixed

| Plugin | Old Version | New Version | Fix |
|--------|-------------|-------------|-----|
| aerial.nvim | v2.7.0 | v3.1.0 | Neovim 0.11 compatibility |
| nvim-treesitter | varies | latest | Parser download fixes |

---

## Troubleshooting

### Issue: "fatal: remote error: upload-pack: not our ref"

**Cause**: Invalid commit hash in lazy-lock.json

**Fix**: Use `Lazy sync` instead of `Lazy restore`:
```bash
nvim --headless -c 'Lazy! sync' -c 'qa'
```

### Issue: "attempt to call method 'start' (a nil value)" persists

**Cause**: Plugin not actually updated

**Fix**: Force reinstall aerial.nvim:
```bash
rm -rf ~/.local/share/nvim/lazy/aerial.nvim
nvim --headless -c 'Lazy! sync' -c 'qa'
```

### Issue: vim.lsp.with() deprecation warning still shows

**Cause**: Other plugins still use deprecated API

**Fix**: Update all AstroNvim plugins:
```bash
nvim --headless -c 'Lazy! sync' -c 'qa'
```

---

## Full Reset (Nuclear Option)

If nothing works, reset everything:

```bash
# Backup first
mv ~/.config/nvim ~/.config/nvim.bak.$(date +%Y%m%d)
mv ~/.local/share/nvim ~/.local/share/nvim.bak.$(date +%Y%m%d)
mv ~/.local/state/nvim ~/.local/state/nvim.bak.$(date +%Y%m%d)
mv ~/.cache/nvim ~/.cache/nvim.bak.$(date +%Y%m%d)

# Reinstall your config (adjust URL as needed)
git clone https://github.com/yourusername/nvim-config.git ~/.config/nvim

# First start
nvim
```

---

## Copy-Paste Script

Save as `fix-nvim.sh`:

```bash
#!/bin/bash
set -e

echo "Fixing Neovim plugins..."

# Update aerial.nvim commit in lazy-lock.json
if [ -f ~/.config/nvim/lazy-lock.json ]; then
    sed -i 's/"5c4e2da4486da5f9b798ea9a0f1fc5c6bcd3d9cf"/"645d108a5242ec7b378cbe643eb6d04d4223f034"/g' ~/.config/nvim/lazy-lock.json
    echo "Updated aerial.nvim commit hash"
fi

# Apply changes
echo "Restoring plugins..."
nvim --headless -c 'Lazy! restore' -c 'qa' 2>/dev/null || true

# Clean treesitter cache
rm -rf ~/.local/share/nvim/site/parser/jsonc.so 2>/dev/null || true
rm -rf ~/.local/share/nvim/lazy/nvim-treesitter/parser/jsonc.so 2>/dev/null || true

echo "Done! Open nvim and test."
```

Make executable: `chmod +x fix-nvim.sh`
Run: `./fix-nvim.sh`

---

## References

- [aerial.nvim releases](https://github.com/stevearc/aerial.nvim/releases)
- [nvim-treesitter issues](https://github.com/nvim-treesitter/nvim-treesitter/issues)
- [AstroNvim documentation](https://docs.astronvim.com/)
