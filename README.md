# zypperoni
Speed up [openSUSE's](https://en.wikipedia.org/wiki/OpenSUSE) package manager [zypper's](https://en.wikipedia.org/wiki/ZYpp) refresh and download commands.

## Synopsis
Zypperoni (a portmanteau of zypper and pepperoni 🍕) is a single page python program without any external dependencies that can be used to massively speed up
zypper's most used and time consuming commands, namely repository refreshing and downloading packages.
It does this by using well-known async and chroot concepts to safely parallelize what would otherwise be a tedious sequential task.

## Installation
```
git clone https://github.com/pavinjosdev/zypperoni.git
chmod 755 zypperoni/zypperoni
sudo cp zypperoni/zypperoni /usr/bin
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

## Performance tests

Tested with `4 CPU / 4GB RAM / 200 Mbps network` VM running fresh installation of Tumbleweed `20240116` on 2024-02-07 using default settings.
Repos enabled: repo-oss, repo-non-oss, repo-update, repo-openh264, google-chrome (external).

| Test                          | zypper    | zypperoni (10 jobs) | zypperoni (20 jobs) |
|-------------------------------|-----------|---------------------|---------------------|
| force refresh repos           | 40.55s    | 9.16s               | 9.36s               |
| refresh repos                 | 2.58s     | 2.55s               | 2.35s               |
| download dup packages (2048)  | 34m26s    | 9m54s               | 8m17s               |

## Uninstallation
```
sudo rm /usr/bin/zypperoni
```

## Troubleshooting
Specify the `--debug` option for troubleshooting.
Zypperoni is intended to catch SIGINT (Ctrl+C) and properly cleanup.
If for some reason it does not cleanup such as when receiving SIGTERM or SIGKILL, future operations should not be affected.
Zypperoni keeps its working directory in `/tmp/zypperoni`, so a reboot would always cleanup.
Should the worst happen and zypperoni somehow messes up, it's very simple to clear whatever it has done wrong by doing:
```
sudo rm -rI /var/cache/zypp
sudo zypper refresh --force
```
