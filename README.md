# zypperoni
Speed up openSUSE's package manager zypper's refresh and download commands.

## Synopsis
Zypperoni 🍕 a single page python program without any external dependencies that can be used to massively speed up
zypper's most used and time consuming commands, namely repository refreshing and downloading packages.
It does this by using well-known async and chroot concepts to safely parallelize what would otherwise be a tedious sequential task.

## Installation
```
git clone https://github.com/pavinjosdev/zypperoni.git
chmod 755 zypperoni/zypperoni
cp zypperoni/zypperoni /usr/bin
```

## Usage
Type in `zypperoni` without any arguments for help.

```
Usage: zypperoni [options] command

zypperoni provides parallel operations
for zypper's oft-used time consuming commands.

Commands:
  refresh - Refresh all enabled repos
  force-refresh - Force refresh all enabled repos
  dup-download - Download all packages required for distribution upgrade

Options:
  --debug - Enable debug output
  --max-jobs - Maximum number of parallel operations [default: 10 / max: 20]
```

## Uninstallation
```
rm /usr/bin/zypperoni
```
