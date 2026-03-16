---
title: Updating
layout: home
parent: Pluto
nav_order: 3
---

If you've already built Pluto before, you can update it without recompiling.

Enter your Pluto folder:
```
cd Pluto
```
Pull the latest commit from GitHub and recompile the changed files:
```
git pull
make
```

If for any reason you need to start over from scratch, you can run `make clean` and then `make` for a fresh compile.