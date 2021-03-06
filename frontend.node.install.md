![](https://s3.leryn.top/website/image/nodejs.svg#crop=0&crop=0&crop=1&crop=1&height=150&id=bDu7i&originHeight=2500&originWidth=2270&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=136)![](https://nodejs.org/static/images/logo.svg#crop=0&crop=0&crop=1&crop=1&height=150&id=obydF&originHeight=361&originWidth=590&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=245)
<a name="rUR6U"></a>
# Node.js 安装手册

参考文档:

- [Node.js - 官网](https://nodejs.org/en/)
- [如何在 Ubuntu 20.04 上安装 Node.js - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)

<a name="QOtgg"></a>
## 包管理器安装（推荐）
<a name="qx7nu"></a>
### 安装步骤

如果你使用默认存储库安装带有 Apt 的 Node.js, 下载的版本会比较古老. 所以请使用 NodeSource PPA 安装带有 Apt 的 Node.js. PPA 拥有比官方 ubuntu 存储库更多的 Node.js 版本.

```bash
curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt update
sudo apt install -y nodejs
```

```bash
# nodejs包包含node二进制和npm
node -v
```

<a name="RHkt8"></a>
## 二进制安装（不推荐）
<a name="KOQKX"></a>
### 前置准备

部署准备, 安装前需要准备如下材料.

- Node.js 安装包: `node-v*-linux-x64.tar.xz`:

```bash
wget https://nodejs.org/dist/v16.13.0/node-v16.13.0-linux-x64.tar.xz
```

<a name="UgZSE"></a>
### 安装步骤

配置 Node.js 环境变量, 并使其生效:

```bash
touch /etc/profile.d/env-nodejs.sh
```

```bash
#!/usr/bin/env bash
# Set Node.js environment.
export NODE_HOME=/usr/local/lib/nodejs/node
export PATH=${NODE_HOME}/bin:${PATH}
```

解压安装包到指定路径, 注意权限问题, 权限过低可能导致`npm install -g`失败:

```bash
VERSION=v16.13.0
DISTRO=linux-x64
sudo mkdir -p /usr/local/lib/nodejs
sudo tar -xf node-$VERSION-$DISTRO.tar.xz -C /usr/local/lib/nodejs
cd /usr/local/lib/nodejs
sudo ln -s ./node-$VERSION-$DISTRO node
```

验证 Node.js 和 npm 是否安装成功, 使用命令查看版本, 如果可以正确显示版本信息则安装成功:

```bash
node -v
npm -v
```

```
# node -v  ==>显示Node.js的版本信息
v16.13.0
# npm -v   ==>显示npm的版本信息
8.1.0
```

<a name="iQQBT"></a>
### 启动与验证

新建 JavaScript 脚本`hello_world.js`用于验证.

```bash
vim hello_world.js
```

```javascript
// 打印Hello World
console.log('Hello World!!');
```

使用`node`命令运行该 JavaScript 脚本文件, 终端会打印出一行 Hello World!!

```bash
node hello_world.js
```

<a name="SzdOM"></a>
## 安装全局插件

这里比较建议使用`pnpm`作为包管理器, pnpm 安装依赖时不会重复安装依赖, 该改用硬链接的方式指向了统一的依赖. 因此它的下载速度更快, 磁盘占用更小.<br />先用 npm 手动安装 pnpm, 之后在用 pnpm 安装所有的全局依赖.

```bash
# 包管理 && 构建工具
sudo npm install -g pnpm --registry=https://registry.npm.taobao.org
sudo pnpm install -g \
  typescript \
  vue @vue/cli vite \
  angular @angular/cli
```

如果使用了`npm`或`cnpm`作为包管理器, 查看全局组件:

```bash
npm list -g
```

```
/usr/local/lib/nodejs/node-v16.13.0-linux-x64/lib
├── @angular/cli@13.0.4
├── @vue/cli@4.5.15
├── angular@1.8.2
├── cnpm@7.1.0
├── corepack@0.10.0
├── npm@8.2.0
├── pnpm@6.23.6
├── typescript@4.5.2
├── vite@2.7.1
└── vue@2.6.14
```

如果使用了`pnpm`作为包管理器, 查看全局组件:

```bash
pnpm list -g
```

```
Legend: production dependency, optional only, dev only
/usr/pnpm-global/5
dependencies:
@angular/cli 13.0.4
@vue/cli 4.5.15
angular 1.8.2
typescript 4.5.2
vite 2.7.1
vue 2.6.14
```

<a name="rXLny"></a>
## 附件 nodesource_setup.sh

```bash
#!/bin/bash

# Discussion, issues and change requests at:
#   https://github.com/nodesource/distributions
#
# Script to install the NodeSource Node.js 16.x repo onto a
# Debian or Ubuntu system.
#
# Run as root or insert `sudo -E` before `bash`:
#
# curl -sL https://deb.nodesource.com/setup_16.x | bash -
#   or
# wget -qO- https://deb.nodesource.com/setup_16.x | bash -
#
# CONTRIBUTIONS TO THIS SCRIPT
#
# This script is built from a template in
# https://github.com/nodesource/distributions/tree/master/deb/src
# please don't submit pull requests against the built scripts.
#


export DEBIAN_FRONTEND=noninteractive
SCRSUFFIX="_16.x"
NODENAME="Node.js 16.x"
NODEREPO="node_16.x"
NODEPKG="nodejs"

print_status() {
    echo
    echo "## $1"
    echo
}

if test -t 1; then # if terminal
    ncolors=$(which tput > /dev/null && tput colors) # supports color
    if test -n "$ncolors" && test $ncolors -ge 8; then
        termcols=$(tput cols)
        bold="$(tput bold)"
        underline="$(tput smul)"
        standout="$(tput smso)"
        normal="$(tput sgr0)"
        black="$(tput setaf 0)"
        red="$(tput setaf 1)"
        green="$(tput setaf 2)"
        yellow="$(tput setaf 3)"
        blue="$(tput setaf 4)"
        magenta="$(tput setaf 5)"
        cyan="$(tput setaf 6)"
        white="$(tput setaf 7)"
    fi
fi

print_bold() {
    title="$1"
    text="$2"

    echo
    echo "${red}================================================================================${normal}"
    echo "${red}================================================================================${normal}"
    echo
    echo -e "  ${bold}${yellow}${title}${normal}"
    echo
    echo -en "  ${text}"
    echo
    echo "${red}================================================================================${normal}"
    echo "${red}================================================================================${normal}"
}

bail() {
    echo 'Error executing command, exiting'
    exit 1
}

exec_cmd_nobail() {
    echo "+ $1"
    bash -c "$1"
}

exec_cmd() {
    exec_cmd_nobail "$1" || bail
}

node_deprecation_warning() {
    if [[ "X${NODENAME}" == "Xio.js 1.x" ||
          "X${NODENAME}" == "Xio.js 2.x" ||
          "X${NODENAME}" == "Xio.js 3.x" ||
          "X${NODENAME}" == "XNode.js 0.10" ||
          "X${NODENAME}" == "XNode.js 0.12" ||
          "X${NODENAME}" == "XNode.js 4.x LTS Argon" ||
          "X${NODENAME}" == "XNode.js 5.x" ||
          "X${NODENAME}" == "XNode.js 6.x LTS Boron" ||
          "X${NODENAME}" == "XNode.js 7.x" ||
          "X${NODENAME}" == "XNode.js 8.x LTS Carbon" ||
          "X${NODENAME}" == "XNode.js 9.x" ||
          "X${NODENAME}" == "XNode.js 10.x" ||
          "X${NODENAME}" == "XNode.js 11.x" ||
          "X${NODENAME}" == "XNode.js 13.x" ||
          "X${NODENAME}" == "XNode.js 15.x" ]]; then

        print_bold \
"                            DEPRECATION WARNING                            " "\
${bold}${NODENAME} is no longer actively supported!${normal}

  ${bold}You will not receive security or critical stability updates${normal} for this version.

  You should migrate to a supported version of Node.js as soon as possible.
  Use the installation script that corresponds to the version of Node.js you
  wish to install. e.g.

   * ${green}https://deb.nodesource.com/setup_12.x — Node.js 12 LTS \"Erbium\"${normal}
   * ${green}https://deb.nodesource.com/setup_14.x — Node.js 14 LTS \"Fermium\"${normal} (recommended)
   * ${green}https://deb.nodesource.com/setup_16.x — Node.js 16 \"Gallium\"${normal}
   * ${green}https://deb.nodesource.com/setup_17.x — Node.js 17 \"Seventeen\"${normal} (current)

  Please see ${bold}https://github.com/nodejs/Release${normal} for details about which
  version may be appropriate for you.

  The ${bold}NodeSource${normal} Node.js distributions repository contains
  information both about supported versions of Node.js and supported Linux
  distributions. To learn more about usage, see the repository:
    ${bold}https://github.com/nodesource/distributions${normal}
"
        echo
        echo "Continuing in 20 seconds ..."
        echo
        sleep 20
    fi
}

script_deprecation_warning() {
    if [ "X${SCRSUFFIX}" == "X" ]; then
        print_bold \
"                         SCRIPT DEPRECATION WARNING                         " "\
This script, located at ${bold}https://deb.nodesource.com/setup${normal}, used to
  install Node.js 0.10, is deprecated and will eventually be made inactive.

  You should use the script that corresponds to the version of Node.js you
  wish to install. e.g.

   * ${green}https://deb.nodesource.com/setup_12.x — Node.js 12 LTS \"Erbium\"${normal}
   * ${green}https://deb.nodesource.com/setup_14.x — Node.js 14 LTS \"Fermium\"${normal} (recommended)
   * ${green}https://deb.nodesource.com/setup_16.x — Node.js 16 \"Gallium\"${normal}
   * ${green}https://deb.nodesource.com/setup_17.x — Node.js 17 \"Seventeen\"${normal} (current)

  Please see ${bold}https://github.com/nodejs/Release${normal} for details about which
  version may be appropriate for you.

  The ${bold}NodeSource${normal} Node.js Linux distributions GitHub repository contains
  information about which versions of Node.js and which Linux distributions
  are supported and how to use the install scripts.
    ${bold}https://github.com/nodesource/distributions${normal}
"

        echo
        echo "Continuing in 20 seconds (press Ctrl-C to abort) ..."
        echo
        sleep 20
    fi
}

setup() {

script_deprecation_warning
node_deprecation_warning

print_status "Installing the NodeSource ${NODENAME} repo..."

if $(uname -m | grep -Eq ^armv6); then
    print_status "You appear to be running on ARMv6 hardware. Unfortunately this is not currently supported by the NodeSource Linux distributions. Please use the 'linux-armv6l' binary tarballs available directly from nodejs.org for Node.js 4 and later."
    exit 1
fi

PRE_INSTALL_PKGS=""

# Check that HTTPS transport is available to APT
# (Check snaked from: https://get.docker.io/ubuntu/)

if [ ! -e /usr/lib/apt/methods/https ]; then
    PRE_INSTALL_PKGS="${PRE_INSTALL_PKGS} apt-transport-https"
fi

if [ ! -x /usr/bin/lsb_release ]; then
    PRE_INSTALL_PKGS="${PRE_INSTALL_PKGS} lsb-release"
fi

if [ ! -x /usr/bin/curl ] && [ ! -x /usr/bin/wget ]; then
    PRE_INSTALL_PKGS="${PRE_INSTALL_PKGS} curl"
fi

# Used by apt-key to add new keys

if [ ! -x /usr/bin/gpg ]; then
    PRE_INSTALL_PKGS="${PRE_INSTALL_PKGS} gnupg"
fi

# Populating Cache
print_status "Populating apt-get cache..."
exec_cmd 'apt-get update'

if [ "X${PRE_INSTALL_PKGS}" != "X" ]; then
    print_status "Installing packages required for setup:${PRE_INSTALL_PKGS}..."
    # This next command needs to be redirected to /dev/null or the script will bork
    # in some environments
    exec_cmd "apt-get install -y${PRE_INSTALL_PKGS} > /dev/null 2>&1"
fi

IS_PRERELEASE=$(lsb_release -d | grep 'Ubuntu .*development' >& /dev/null; echo $?)
if [[ $IS_PRERELEASE -eq 0 ]]; then
    print_status "Your distribution, identified as \"$(lsb_release -d -s)\", is a pre-release version of Ubuntu. NodeSource does not maintain official support for Ubuntu versions until they are formally released. You can try using the manual installation instructions available at https://github.com/nodesource/distributions and use the latest supported Ubuntu version name as the distribution identifier, although this is not guaranteed to work."
    exit 1
fi

DISTRO=$(lsb_release -c -s)

check_alt() {
    if [ "X${DISTRO}" == "X${2}" ]; then
        echo
        echo "## You seem to be using ${1} version ${DISTRO}."
        echo "## This maps to ${3} \"${4}\"... Adjusting for you..."
        DISTRO="${4}"
    fi
}

check_alt "SolydXK"       "solydxk-9" "Debian" "stretch"
check_alt "Kali"          "sana"     "Debian" "jessie"
check_alt "Kali"          "kali-rolling" "Debian" "bullseye"
check_alt "Sparky Linux"  "Tyche"    "Debian" "stretch"
check_alt "Sparky Linux"  "Nibiru"   "Debian" "buster"
check_alt "MX Linux 17"   "Horizon"  "Debian" "stretch"
check_alt "MX Linux 18"   "Continuum" "Debian" "stretch"
check_alt "MX Linux 19"   "patito feo" "Debian" "buster"
check_alt "Linux Mint"    "maya"     "Ubuntu" "precise"
check_alt "Linux Mint"    "qiana"    "Ubuntu" "trusty"
check_alt "Linux Mint"    "rafaela"  "Ubuntu" "trusty"
check_alt "Linux Mint"    "rebecca"  "Ubuntu" "trusty"
check_alt "Linux Mint"    "rosa"     "Ubuntu" "trusty"
check_alt "Linux Mint"    "sarah"    "Ubuntu" "xenial"
check_alt "Linux Mint"    "serena"   "Ubuntu" "xenial"
check_alt "Linux Mint"    "sonya"    "Ubuntu" "xenial"
check_alt "Linux Mint"    "sylvia"   "Ubuntu" "xenial"
check_alt "Linux Mint"    "tara"     "Ubuntu" "bionic"
check_alt "Linux Mint"    "tessa"    "Ubuntu" "bionic"
check_alt "Linux Mint"    "tina"     "Ubuntu" "bionic"
check_alt "Linux Mint"    "tricia"   "Ubuntu" "bionic"
check_alt "Linux Mint"    "ulyana"   "Ubuntu" "focal"
check_alt "Linux Mint"    "ulyssa"   "Ubuntu" "focal"
check_alt "Linux Mint"    "uma"      "Ubuntu" "focal"
check_alt "LMDE"          "betsy"    "Debian" "jessie"
check_alt "LMDE"          "cindy"    "Debian" "stretch"
check_alt "LMDE"          "debbie"   "Debian" "buster"
check_alt "elementaryOS"  "luna"     "Ubuntu" "precise"
check_alt "elementaryOS"  "freya"    "Ubuntu" "trusty"
check_alt "elementaryOS"  "loki"     "Ubuntu" "xenial"
check_alt "elementaryOS"  "juno"     "Ubuntu" "bionic"
check_alt "elementaryOS"  "hera"     "Ubuntu" "bionic"
check_alt "elementaryOS"  "odin"     "Ubuntu" "focal"
check_alt "Trisquel"      "toutatis" "Ubuntu" "precise"
check_alt "Trisquel"      "belenos"  "Ubuntu" "trusty"
check_alt "Trisquel"      "flidas"   "Ubuntu" "xenial"
check_alt "Trisquel"      "etiona"   "Ubuntu" "bionic"
check_alt "Uruk GNU/Linux" "lugalbanda" "Ubuntu" "xenial"
check_alt "BOSS"          "anokha"   "Debian" "wheezy"
check_alt "BOSS"          "anoop"    "Debian" "jessie"
check_alt "BOSS"          "drishti"  "Debian" "stretch"
check_alt "BOSS"          "unnati"   "Debian" "buster"
check_alt "bunsenlabs"    "bunsen-hydrogen" "Debian" "jessie"
check_alt "bunsenlabs"    "helium"   "Debian" "stretch"
check_alt "bunsenlabs"    "lithium"  "Debian" "buster"
check_alt "Tanglu"        "chromodoris" "Debian" "jessie"
check_alt "PureOS"        "green"    "Debian" "sid"
check_alt "PureOS"        "amber"    "Debian" "buster"
check_alt "Devuan"        "jessie"   "Debian" "jessie"
check_alt "Devuan"        "ascii"    "Debian" "stretch"
check_alt "Devuan"        "beowulf"  "Debian" "buster"
check_alt "Devuan"        "chimaera"  "Debian" "bullseye"
check_alt "Devuan"        "ceres"    "Debian" "sid"
check_alt "Deepin"        "panda"    "Debian" "sid"
check_alt "Deepin"        "unstable" "Debian" "sid"
check_alt "Deepin"        "stable"   "Debian" "buster"
check_alt "Pardus"        "onyedi"   "Debian" "stretch"
check_alt "Liquid Lemur"  "lemur-3"  "Debian" "stretch"
check_alt "Astra Linux"   "orel"     "Debian" "stretch"
check_alt "Ubilinux"      "dolcetto" "Debian" "stretch"

if [ "X${DISTRO}" == "Xdebian" ]; then
  print_status "Unknown Debian-based distribution, checking /etc/debian_version..."
  NEWDISTRO=$([ -e /etc/debian_version ] && cut -d/ -f1 < /etc/debian_version)
  if [ "X${NEWDISTRO}" == "X" ]; then
    print_status "Could not determine distribution from /etc/debian_version..."
  else
    DISTRO=$NEWDISTRO
    print_status "Found \"${DISTRO}\" in /etc/debian_version..."
  fi
fi

print_status "Confirming \"${DISTRO}\" is supported..."

if [ -x /usr/bin/curl ]; then
    exec_cmd_nobail "curl -sLf -o /dev/null 'https://deb.nodesource.com/${NODEREPO}/dists/${DISTRO}/Release'"
    RC=$?
else
    exec_cmd_nobail "wget -qO /dev/null -o /dev/null 'https://deb.nodesource.com/${NODEREPO}/dists/${DISTRO}/Release'"
    RC=$?
fi

if [[ $RC != 0 ]]; then
    print_status "Your distribution, identified as \"${DISTRO}\", is not currently supported, please contact NodeSource at https://github.com/nodesource/distributions/issues if you think this is incorrect or would like your distribution to be considered for support"
    exit 1
fi

if [ -f "/etc/apt/sources.list.d/chris-lea-node_js-$DISTRO.list" ]; then
    print_status 'Removing Launchpad PPA Repository for NodeJS...'

    exec_cmd_nobail 'add-apt-repository -y -r ppa:chris-lea/node.js'
    exec_cmd "rm -f /etc/apt/sources.list.d/chris-lea-node_js-${DISTRO}.list"
fi

print_status 'Adding the NodeSource signing key to your keyring...'
keyring='/usr/share/keyrings'
node_key_url="https://deb.nodesource.com/gpgkey/nodesource.gpg.key"
local_node_key="$keyring/nodesource.gpg"

if [ -x /usr/bin/curl ]; then
    exec_cmd "curl -s $node_key_url | gpg --dearmor | tee $local_node_key >/dev/null"
else
    exec_cmd "wget -q -O - $node_key_url | gpg --dearmor | tee $local_node_key >/dev/null"
fi

print_status "Creating apt sources list file for the NodeSource ${NODENAME} repo..."

exec_cmd "echo 'deb [signed-by=$local_node_key] https://deb.nodesource.com/${NODEREPO} ${DISTRO} main' > /etc/apt/sources.list.d/nodesource.list"
exec_cmd "echo 'deb-src [signed-by=$local_node_key] https://deb.nodesource.com/${NODEREPO} ${DISTRO} main' >> /etc/apt/sources.list.d/nodesource.list"

print_status 'Running `apt-get update` for you...'

exec_cmd 'apt-get update'

yarn_site='https://dl.yarnpkg.com/debian'
yarn_key_url="$yarn_site/pubkey.gpg"
local_yarn_key="$keyring/yarnkey.gpg"

print_status """Run \`${bold}sudo apt-get install -y ${NODEPKG}${normal}\` to install ${NODENAME} and npm
## You may also need development tools to build native addons:
     sudo apt-get install gcc g++ make
## To install the Yarn package manager, run:
     curl -sL $yarn_key_url | gpg --dearmor | sudo tee $local_yarn_key >/dev/null
     echo \"deb [signed-by=$local_yarn_key] $yarn_site stable main\" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt-get update && sudo apt-get install yarn
"""

}

## Defer setup until we have the complete script
setup

```
