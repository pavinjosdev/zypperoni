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

Tested on VM running fresh installation of Tumbleweed `20240116` on 2024-02-07 using default settings.
- VM specs: 4 CPU, 4GB RAM, 200 Mbps bandwidth, 200ms latency to default mirror.
- Repos enabled: repo-oss, repo-non-oss, repo-update, repo-openh264, google-chrome (external).

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

## Optional changes

1. For faster connections to the official openSUSE repos without hardcoding it to a local mirror, change the repo URLs from `download.opensuse.org` to `cdn.opensuse.org`. The `.repo` config files are located in the directory `/etc/zypp/repos.d`. Run `zypperoni refresh` after the update.

2. Use bash aliases to run zypperoni commands less verbosely. For example, add the following aliases to your `~/.bashrc` file and run `source ~/.bashrc` to apply it:
```
# Zypperoni refresh
alias zref='sudo zypperoni refresh'
# Zypperoni download
alias zdown='sudo zypperoni dup-download'
```

Now you can use `zref` and `zdown` to refresh repos and download packages respectively.

## Known issues

Generally, zypperoni should work out of the box with the default zypp and zypper configs.
Custom or experimental configs may result in bugs.

1. Using the experimental option `techpreview.ZYPP_SINGLE_RPMTRANS=1` in `zypp.conf` would result in `zypperoni dup-download` appearing to hang indefinitely, but in reality zypper is doing its sequential download in the background even though `zypperoni` specifies the `--dry-run` option when invoking it. If you would like to use this feature, do not add the option in `zypp.conf` but pass it as an environment variable: `sudo env ZYPP_SINGLE_RPMTRANS=1 zypper dup`.
