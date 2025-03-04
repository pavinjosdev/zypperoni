# zypperoni 🍕
Speed up [openSUSE's](https://en.wikipedia.org/wiki/OpenSUSE) package manager [zypper](https://en.wikipedia.org/wiki/ZYpp). 🐌 ➡️ 🐱

## Synopsis
Zypperoni (a portmanteau of zypper and pepperoni 🍕) is a simple single file program without any external dependencies that can be used to massively speed up zypper's most used and time consuming commands. 🚀

Zypperoni uses various techniques to safely group together zypper operations where possible in an async manner and does not by itself make any changes to your configs or system, making it suitable for production use. 💫

## Requirements
There are no external dependencies other than common shell tools and the Python interpreter which should already be installed on your system.
Minimum python version required: `3.9`. If you're on Leap and have an older Python version, use [pyenv](https://github.com/pyenv/pyenv) to get a newer version.

## Installation

```
curl -s https://raw.githubusercontent.com/pavinjosdev/zypperoni/main/zypperoni | sudo tee /usr/bin/zypperoni > /dev/null
sudo chmod 755 /usr/bin/zypperoni
```

> [!CAUTION]
> Breaking changes when moving from version 0.x.x to 1.x.x

## Usage
Type in `zypperoni --help` for usage help.

```
usage: zypperoni [-h] [-v] [-y] [-j {5,10,15,20}] [--debug] [--no-color] {refresh,ref,dist-upgrade,dup,install,in,install-new-recommends,inr} ...

zypperoni provides parallel operations for zypper's oft-used time consuming commands.

options:
  -h, --help            show this help message and exit
  -v, -V, --version     print version number and exit (default: False)
  -y, --no-confirm      automatic yes to prompts, run non-interactively (default: False)
  -j {5,10,15,20}, --jobs {5,10,15,20}
                        number of parallel operations (default: 10)
  --debug               enable debug output (default: False)
  --no-color            disable color output (default: False)

commands:
  type 'zypperoni <command> --help' to get command-specific help

  {refresh,ref,dist-upgrade,dup,install,in,install-new-recommends,inr}
    refresh (ref)       refresh all enabled repos
    dist-upgrade (dup)  perform distribution upgrade
    install (in)        install one or more packages
    install-new-recommends (inr)
                        install new packages recommended by already installed ones
```

## Examples
1. Refresh all repos
```
sudo zypperoni ref
# or verbosely
sudo zypperoni refresh
```

2. Force refresh all repos
```
sudo zypperoni ref -f
```

3. Perform distribution upgrade
```
sudo zypperoni dup
```

4. Download packages to perform distribution upgrade
```
sudo zypperoni dup -d
```

5. Install packages
```
sudo zypperoni in htop btop
```

6. Get help for specific command
```
zypperoni inr -h
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
Zypperoni keeps its working directory in `/tmp/zypperoni_*`, so a reboot would always cleanup.
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

- Using the experimental option `techpreview.ZYPP_SINGLE_RPMTRANS=1` in `zypp.conf` would result in `zypperoni dup` appearing to hang indefinitely, but in reality zypper is doing its sequential download in the background due to RPM single transaction requirements. This config is not necessary when using zypperoni as it passes `ZYPP_SINGLE_RPMTRANS=1` as an environment variable when calling zypper.
