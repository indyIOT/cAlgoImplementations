# cAlgoImplementations
Implementations of some basic libraries that I like to twiddle on.

## Submodules

This repo uses git submodules. Use the following commands to get everything set up.

**Cloning fresh:**
```bash
git clone --recurse-submodules git@github.com:tbandtg/cAlgoImplementations.git
```

**Pulling updates (including submodules):**
```bash
git pull && git submodule update --remote --merge
```

**Adding a new submodule:**
```bash
git submodule add git@github.com:tbandtg/<repo-name>.git <folder-name>
git commit -m "Add <repo-name> as submodule"
```
