# zypperoni 🍕
Speed up [openSUSE's](https://en.wikipedia.org/wiki/OpenSUSE) package manager [zypper](https://en.wikipedia.org/wiki/ZYpp).

## Synopsis
Zypperoni (a portmanteau of zypper and pepperoni 🍕) is a simple single file program without any external dependencies that can be used to massively speed up zypper's most used and time consuming commands.

Zypperoni uses various techniques to safely group together zypper operations where possible in an async manner and does not by itself make any changes to your configs or system, making it suitable for production use.

## Installation

```
curl -s https://raw.githubusercontent.com/pavinjosdev/zypperoni/main/zypperoni | sudo tee /usr/bin/zypperoni > /dev/null
sudo chmod 755 /usr/bin/zypperoni
```

## Usage
Type in `zypperoni --help` for usage help.

```
Usage: zypperoni [options] command
       zypperoni [options] in pkg1 [pkg2 ...]
       zypperoni [options] in-download pkg1 [pkg2 ...]

zypperoni provides parallel operations
for zypper's oft-used time consuming commands.

Commands:
  ref           - Refresh all enabled repos
  force-ref     - Force refresh all enabled repos
  in            - Install packages
  in-download   - Download packages for later installation
  dup           - Perform distribution upgrade
  dup-download  - Download packages required for distribution upgrade
  inr           - Install new packages recommended by already installed ones
  inr-download  - Download new packages recommended by already installed ones

Options:
  --debug       - Enable debug output
  --help        - Print this help and exit
  --version     - Print version number and exit
  --no-confirm  - Automatic yes to prompts, run non-interactively
  --max-jobs    - Maximum number of parallel operations [default: 10 / max: 20]
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
Zypperoni keeps it's working directory in `/tmp/zypperoni_*`, so a reboot would always cleanup.
Should zypperoni somehow mess up, it's very simple to clear whatever it has done wrong by doing:
```
sudo rm -rI /var/cache/zypp
sudo zypper refresh --force
```

## Optional changes

1. For faster connections to the official openSUSE repos without hardcoding it to a local mirror, change the repo URLs from `download.opensuse.org` to `cdn.opensuse.org`. The `.repo` config files are located in the directory `/etc/zypp/repos.d`. Run `zypperoni ref` after the update.

2. Use bash aliases to run zypperoni commands less verbosely. For example, add the following aliases to your `~/.bashrc` file and run `source ~/.bashrc` to apply it:
```
# Zypperoni alias
alias z='sudo zypperoni'
```

Now you can use `z ref` and `z dup` to refresh repos and perform distribution upgrade respectively.

## Known issues

Generally, zypperoni should work out of the box with the default zypp and zypper configs.
Custom or experimental configs may result in bugs.

- Using the experimental option `techpreview.ZYPP_SINGLE_RPMTRANS=1` in `zypp.conf` would result in `zypperoni dup-download` appearing to hang indefinitely, but in reality zypper is doing it's sequential download in the background due to RPM single transaction requirements. This config is not necessary when using zypperoni as it passes `ZYPP_SINGLE_RPMTRANS=1` as an environment variable when calling zypper.
