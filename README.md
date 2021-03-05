# dive
[![Go Report Card](https://goreportcard.com/badge/github.com/wagoodman/dive)](https://goreportcard.com/report/github.com/wagoodman/dive)
[![Pipeline Status](https://circleci.com/gh/wagoodman/dive.svg?style=svg)](https://circleci.com/gh/wagoodman/dive)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg?style=flat)](https://www.paypal.me/wagoodman)

**一个用于查看docker镜像，层内容，并发现缩小你的Docker/OCI镜像方式的工具。**

![Image](.data/demo.gif)

要分析一个Docker镜像只需运行dive和提供镜像的tag/id/摘要：
```bash
dive <你的镜像tag>
```

或者说你想构建你的镜像紧接着就进行分析：
```bash
dive build -t <某些tag> .
```

在Macbook构建 (只支持Docker容器引擎)

```bash
docker run --rm -it \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v  "$(pwd)":"$(pwd)" \
      -w "$(pwd)" \
      -v "$HOME/.dive.yaml":"$HOME/.dive.yaml" \
      wagoodman/dive:latest build -t <some-tag> .
```

Additionally you can run this in your CI pipeline to ensure you're keeping wasted space to a minimum (this skips the UI):
```
CI=true dive <your-image>
```

![Image](.data/demo-ci.png)

**This is beta quality!** *Feel free to submit an issue if you want a new feature or find a bug :)*

## Basic Features

**Show Docker image contents broken down by layer**

As you select a layer on the left, you are shown the contents of that layer combined with all previous layers on the right. Also, you can fully explore the file tree with the arrow keys.

**Indicate what's changed in each layer**

Files that have changed, been modified, added, or removed are indicated in the file tree. This can be adjusted to show changes for a specific layer, or aggregated changes up to this layer.

**Estimate "image efficiency"**

The lower left pane shows basic layer info and an experimental metric that will guess how much wasted space your image contains. This might be from duplicating files across layers, moving files across layers, or not fully removing files. Both a percentage "score" and total wasted file space is provided.

**Quick build/analysis cycles**

You can build a Docker image and do an immediate analysis with one command:
`dive build -t some-tag .`

You only need to replace your `docker build` command with the same `dive build`
command.

**CI Integration**

Analyze an image and get a pass/fail result based on the image efficiency and wasted space. Simply set `CI=true` in the environment when invoking any valid dive command.

**Multiple Image Sources and Container Engines Supported**

With the `--source` option, you can select where to fetch the container image from:
```bash
dive <your-image> --source <source>
```
or
```bash
dive <source>://<your-image>
```

With valid `source` options as such:
- `docker`: Docker engine (the default option)
- `docker-archive`: A Docker Tar Archive from disk
- `podman`: Podman engine (linux only)

## Installation

**Ubuntu/Debian**
```bash
wget https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_linux_amd64.deb
sudo apt install ./dive_0.9.2_linux_amd64.deb
```

**RHEL/Centos**
```bash
curl -OL https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_linux_amd64.rpm
rpm -i dive_0.9.2_linux_amd64.rpm
```

**Arch Linux**

Available as [dive](https://aur.archlinux.org/packages/dive/) in the Arch User Repository (AUR).

```bash
yay -S dive
```

The above example assumes [`yay`](https://aur.archlinux.org/packages/yay/) as the tool for installing AUR packages.

**Mac**

```bash
brew install dive
```

or download the latest Darwin build from the [releases page](https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_darwin_amd64.tar.gz).

**Windows**

Download the [latest release](https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_windows_amd64.zip).

**Go tools**
Requires Go version 1.10 or higher.

```bash
go get github.com/wagoodman/dive
```
*Note*: installing in this way you will not see a proper version when running `dive -v`.

**Docker**
```bash
docker pull wagoodman/dive
```

or

```bash
docker pull quay.io/wagoodman/dive
```

When running you'll need to include the docker socket file:
```bash
docker run --rm -it \
    -v /var/run/docker.sock:/var/run/docker.sock \
    wagoodman/dive:latest <dive arguments...>
```

Docker for Windows (showing PowerShell compatible line breaks; collapse to a single line for Command Prompt compatibility)
```bash
docker run --rm -it `
    -v /var/run/docker.sock:/var/run/docker.sock `
    wagoodman/dive:latest <dive arguments...>
```

**Note:** depending on the version of docker you are running locally you may need to specify the docker API version as an environment variable:
```bash
   DOCKER_API_VERSION=1.37 dive ...
```
or if you are running with a docker image:
```bash
docker run --rm -it \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e DOCKER_API_VERSION=1.37 \
    wagoodman/dive:latest <dive arguments...>
```

## CI Integration

When running dive with the environment variable `CI=true` then the dive UI will be bypassed and will instead analyze your docker image, giving it a pass/fail indication via return code. Currently there are three metrics supported via a `.dive-ci` file that you can put at the root of your repo:
```
rules:
  # If the efficiency is measured below X%, mark as failed.
  # Expressed as a ratio between 0-1.
  lowestEfficiency: 0.95

  # If the amount of wasted space is at least X or larger than X, mark as failed.
  # Expressed in B, KB, MB, and GB.
  highestWastedBytes: 20MB

  # If the amount of wasted space makes up for X% or more of the image, mark as failed.
  # Note: the base image layer is NOT included in the total image size.
  # Expressed as a ratio between 0-1; fails if the threshold is met or crossed.
  highestUserWastedPercent: 0.20
```
You can override the CI config path with the `--ci-config` option.

## KeyBindings

Key Binding                                | Description
-------------------------------------------|---------------------------------------------------------
<kbd>Ctrl + C</kbd>                        | Exit
<kbd>Tab</kbd>                             | Switch between the layer and filetree views
<kbd>Ctrl + F</kbd>                        | Filter files
<kbd>PageUp</kbd>                          | Scroll up a page
<kbd>PageDown</kbd>                        | Scroll down a page
<kbd>Ctrl + A</kbd>                        | Layer view: see aggregated image modifications
<kbd>Ctrl + L</kbd>                        | Layer view: see current layer modifications
<kbd>Space</kbd>                           | Filetree view: collapse/uncollapse a directory
<kbd>Ctrl + Space</kbd>                    | Filetree view: collapse/uncollapse all directories
<kbd>Ctrl + A</kbd>                        | Filetree view: show/hide added files
<kbd>Ctrl + R</kbd>                        | Filetree view: show/hide removed files
<kbd>Ctrl + M</kbd>                        | Filetree view: show/hide modified files
<kbd>Ctrl + U</kbd>                        | Filetree view: show/hide unmodified files
<kbd>Ctrl + B</kbd>                        | Filetree view: show/hide file attributes
<kbd>PageUp</kbd>                          | Filetree view: scroll up a page
<kbd>PageDown</kbd>                        | Filetree view: scroll down a page

## UI Configuration

No configuration is necessary, however, you can create a config file and override values:
```yaml
# supported options are "docker" and "podman"
container-engine: docker
# continue with analysis even if there are errors parsing the image archive
ignore-errors: false
log:
  enabled: true
  path: ./dive.log
  level: info

# Note: you can specify multiple bindings by separating values with a comma.
# Note: UI hinting is derived from the first binding
keybinding:
  # Global bindings
  quit: ctrl+c
  toggle-view: tab
  filter-files: ctrl+f, ctrl+slash

  # Layer view specific bindings
  compare-all: ctrl+a
  compare-layer: ctrl+l

  # File view specific bindings
  toggle-collapse-dir: space
  toggle-collapse-all-dir: ctrl+space
  toggle-added-files: ctrl+a
  toggle-removed-files: ctrl+r
  toggle-modified-files: ctrl+m
  toggle-unmodified-files: ctrl+u
  toggle-filetree-attributes: ctrl+b
  page-up: pgup
  page-down: pgdn

diff:
  # You can change the default files shown in the filetree (right pane). All diff types are shown by default.
  hide:
    - added
    - removed
    - modified
    - unmodified

filetree:
  # The default directory-collapse state
  collapse-dir: false

  # The percentage of screen width the filetree should take on the screen (must be >0 and <1)
  pane-width: 0.5

  # Show the file attributes next to the filetree
  show-attributes: true

layer:
  # Enable showing all changes from this layer and every previous layer
  show-aggregated-changes: false

```

dive will search for configs in the following locations:
- `$XDG_CONFIG_HOME/dive/*.yaml`
- `$XDG_CONFIG_DIRS/dive/*.yaml`
- `~/.config/dive/*.yaml`
- `~/.dive.yaml`
