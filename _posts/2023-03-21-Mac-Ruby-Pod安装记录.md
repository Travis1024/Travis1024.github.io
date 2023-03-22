---
title: Mac Ruby Pod安装记录
author: Travis <Hongxu Wei>
date: 2023-03-21 10:34:17 +0800
categories: [IOS Space]
tags: [ios, ruby, pod]
math: false
---

## 一、下载rvm

- RVM 是一个便捷的多版本 Ruby 环境的管理和切换工具

执行命令：

```bash
curl -sSL https://get.rvm.io | bash -s stable
```

出现报错信息==Failed to connect to raw.githubusercontent.com:443==

完整报错信息

```bash
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

解决方法：

设置DNS为114.114.114.114或者8.8.8.8就好了

```bash
weihongxudeMacBook-Pro:~ travis$ curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.29.12.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc
curl: (92) HTTP/2 stream 1 was not closed cleanly before end of the underlying stream

Could not download 'https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc'.
  curl returned status '92'.

Installing RVM to /Users/travis/.rvm/
    Adding rvm PATH line to /Users/travis/.profile /Users/travis/.mkshrc /Users/travis/.bashrc /Users/travis/.zshrc.
    Adding rvm loading line to /Users/travis/.profile /Users/travis/.bash_profile /Users/travis/.zlogin.
Installation of RVM in /Users/travis/.rvm/ is almost complete:

  * To start using RVM you need to run `source /Users/travis/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
Thanks for installing RVM 🙏
Please consider donating to our open collective to help us maintain RVM.

👉  Donate: https://opencollective.com/rvm/donate
```

- 载入RVM环境

```bash
source ~/.bashrc
rvm -v

weihongxudeMacBook-Pro:~ travis$ rvm -v
rvm 1.29.12 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

- 修改 RVM 下载 Ruby 的源 到 Ruby China 的镜像

```bash
weihongxudeMacBook-Pro:~ travis$ source ~/.rvm/scripts/rvm
weihongxudeMacBook-Pro:~ travis$ echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > ~/.rvm/user/db
weihongxudeMacBook-Pro:~ travis$ rvm -v
rvm 1.29.12 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```



---

==如果之前安装过 RVM 更新则 $ rvm get stable==

---



## 二、下载Ruby

- 查询Ruby版本信息

  ```bash
  rvm list known
  
  # MRI Rubies
  [ruby-]1.8.6[-p420]
  [ruby-]1.8.7[-head] # security released on head
  [ruby-]1.9.1[-p431]
  [ruby-]1.9.2[-p330]
  [ruby-]1.9.3[-p551]
  [ruby-]2.0.0[-p648]
  [ruby-]2.1[.10]
  [ruby-]2.2[.10]
  [ruby-]2.3[.8]
  [ruby-]2.4[.10]
  [ruby-]2.5[.8]
  [ruby-]2.6[.6]
  [ruby-]2.7[.2]
  [ruby-]3[.0.0]
  ruby-head
  
  # for forks use: rvm install ruby-head-<name> --url https://github.com/github/ruby.git --branch 2.2
  
  # JRuby
  jruby-1.6[.8]
  jruby-1.7[.27]
  jruby-9.1[.17.0]
  jruby[-9.2.14.0]
  jruby-head
  
  # Rubinius
  rbx-1[.4.3]
  rbx-2.3[.0]
  rbx-2.4[.1]
  rbx-2[.5.8]
  rbx-3[.107]
  rbx-4[.20]
  rbx-5[.0]
  rbx-head
  
  # TruffleRuby
  truffleruby[-20.3.0]
  
  # Opal
  opal
  
  # Minimalistic ruby implementation - ISO 30170:2012
  mruby-1.0.0
  mruby-1.1.0
  mruby-1.2.0
  mruby-1.3.0
  mruby-1[.4.1]
  mruby-2.0.1
  mruby-2[.1.1]
  mruby[-head]
  
  # Ruby Enterprise Edition
  ree-1.8.6
  ree[-1.8.7][-2012.02]
  
  # Topaz
  topaz
  
  # MagLev
  maglev-1.0.0
  maglev-1.1[RC1]
  maglev[-1.2Alpha4]
  maglev-head
  
  # Mac OS X Snow Leopard Or Newer
  macruby-0.10
  macruby-0.11
  macruby[-0.12]
  macruby-nightly
  macruby-head
  
  # IronRuby
  ironruby[-1.1.3]
  ironruby-head
  ```

- 开始下载

  ```bash
  rvm install 2.7 --default
  ```

  - 出现报错信息，查看信息日志，我这里是需要更新brew

  ```bash
  weihongxudeMacBook-Pro:~ travis$ rvm install 2.7 --default
  Searching for binary rubies, this might take some time.
  No binary rubies available for: osx/12.6/arm64/ruby-2.7.2.
  Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
  Checking requirements for osx.
  Installing requirements for osx.
  Updating system..........Failed to update Homebrew, follow instructions at
  
      https://docs.brew.sh/Common-Issues
  
  and make sure `brew update` works before continuing.
  ...
  Error running 'requirements_osx_brew_update_system ruby-2.7.2',
  please read /Users/travis/.rvm/log/1679363684_ruby-2.7.2/update_system.log
  Requirements installation failed with status: 1.
  ```

- 更新brew

  ```bash
  brew update
  ```

- 再次下载ruby 2.7, 下载成功！

  ```bash
  weihongxudeMacBook-Pro:~ travis$ rvm install 2.7 --default
  Searching for binary rubies, this might take some time.
  No binary rubies available for: osx/12.6/arm64/ruby-2.7.2.
  Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
  Checking requirements for osx.
  Installing requirements for osx.
  Updating system..........
  Installing required packages: autoconf, automake, libtool, pkg-config, coreutils, libyaml, libksba, zlib.............
  ==> Upgrading 1 outdated package:
  readline 8.1.2 -> 8.2.1
  ==> Fetching readline
  ==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bottles/readline-8.2.1.arm64_monterey.bo
  ######################################################################## 100.0%
  ==> Upgrading readline
    8.1.2 -> 8.2.1 
  
  ==> Pouring readline-8.2.1.arm64_monterey.bottle.tar.gz
  🍺  /opt/homebrew/Cellar/readline/8.2.1: 50 files, 1.7MB
  ==> Running `brew cleanup readline`...
  Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
  Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
  Removing: /opt/homebrew/Cellar/readline/8.1.2... (48 files, 1.7MB)
  ==> Upgrading 2 dependents of upgraded formula:
  Disable this behaviour by setting HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK.
  Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
  python@3.9 3.9.12 -> 3.9.16, sqlite 3.38.2 -> 3.41.1
  ==> Fetching sqlite
  ==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bottles/sqlite-3.41.1.arm64_monterey.bot
  ######################################################################## 100.0%
  ==> Fetching dependencies for python@3.9: xz
  ==> Fetching xz
  ==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bottles/xz-5.4.2.arm64_monterey.bottle.t
  ######################################################################## 100.0%
  ==> Fetching python@3.9
  ==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bottles/python%403.9-3.9.16.arm64_monter
  ######################################################################## 100.0%
  ==> Upgrading sqlite
    3.38.2 -> 3.41.1 
  
  ==> Pouring sqlite-3.41.1.arm64_monterey.bottle.tar.gz
  ==> Caveats
  sqlite is keg-only, which means it was not symlinked into /opt/homebrew,
  because macOS already provides this software and installing another version in
  parallel can cause all kinds of trouble.
  
  If you need to have sqlite first in your PATH, run:
    echo 'export PATH="/opt/homebrew/opt/sqlite/bin:$PATH"' >> ~/.zshrc
  
  For compilers to find sqlite you may need to set:
    export LDFLAGS="-L/opt/homebrew/opt/sqlite/lib"
    export CPPFLAGS="-I/opt/homebrew/opt/sqlite/include"
  
  For pkg-config to find sqlite you may need to set:
    export PKG_CONFIG_PATH="/opt/homebrew/opt/sqlite/lib/pkgconfig"
  ==> Summary
  🍺  /opt/homebrew/Cellar/sqlite/3.41.1: 11 files, 4.5MB
  ==> Running `brew cleanup sqlite`...
  Removing: /opt/homebrew/Cellar/sqlite/3.38.2... (11 files, 4.4MB)
  ==> Upgrading python@3.9
    3.9.12 -> 3.9.16 
  
  ==> Installing dependencies for python@3.9: xz
  ==> Installing python@3.9 dependency: xz
  ==> Pouring xz-5.4.2.arm64_monterey.bottle.tar.gz
  🍺  /opt/homebrew/Cellar/xz/5.4.2: 162 files, 2.5MB
  ==> Installing python@3.9
  ==> Pouring python@3.9-3.9.16.arm64_monterey.bottle.tar.gz
  ==> /opt/homebrew/Cellar/python@3.9/3.9.16/bin/python3.9 -m ensurepip
  ==> /opt/homebrew/Cellar/python@3.9/3.9.16/bin/python3.9 -m pip install -v --no-deps --no-index --upg
  ==> Caveats
  Python has been installed as
    /opt/homebrew/bin/python3.9
  
  Unversioned and major-versioned symlinks `python`, `python3`, `python-config`, `python3-config`, `pip`, `pip3`, etc. pointing to
  `python3.9`, `python3.9-config`, `pip3.9` etc., respectively, have been installed into
    /opt/homebrew/opt/python@3.9/libexec/bin
  
  You can install Python packages with
    pip3.9 install <package>
  They will install into the site-package directory
    /opt/homebrew/lib/python3.9/site-packages
  
  tkinter is no longer included with this formula, but it is available separately:
    brew install python-tk@3.9
  
  If you do not need a specific version of Python, and always want Homebrew's `python3` in your PATH:
    brew install python3
  
  See: https://docs.brew.sh/Homebrew-and-Python
  ==> Summary
  🍺  /opt/homebrew/Cellar/python@3.9/3.9.16: 3,068 files, 57.3MB
  ==> Running `brew cleanup python@3.9`...
  Removing: /opt/homebrew/Cellar/python@3.9/3.9.12... (3,111 files, 57.7MB)
  ==> Checking for dependents of upgraded formulae...
  ==> No broken dependents found!
  ==> Caveats
  ==> sqlite
  sqlite is keg-only, which means it was not symlinked into /opt/homebrew,
  because macOS already provides this software and installing another version in
  parallel can cause all kinds of trouble.
  
  If you need to have sqlite first in your PATH, run:
    echo 'export PATH="/opt/homebrew/opt/sqlite/bin:$PATH"' >> ~/.zshrc
  
  For compilers to find sqlite you may need to set:
    export LDFLAGS="-L/opt/homebrew/opt/sqlite/lib"
    export CPPFLAGS="-I/opt/homebrew/opt/sqlite/include"
  
  For pkg-config to find sqlite you may need to set:
    export PKG_CONFIG_PATH="/opt/homebrew/opt/sqlite/lib/pkgconfig"
  ==> python@3.9
  Python has been installed as
    /opt/homebrew/bin/python3.9
  
  Unversioned and major-versioned symlinks `python`, `python3`, `python-config`, `python3-config`, `pip`, `pip3`, etc. pointing to
  `python3.9`, `python3.9-config`, `pip3.9` etc., respectively, have been installed into
    /opt/homebrew/opt/python@3.9/libexec/bin
  
  You can install Python packages with
    pip3.9 install <package>
  They will install into the site-package directory
    /opt/homebrew/lib/python3.9/site-packages
  
  tkinter is no longer included with this formula, but it is available separately:
    brew install python-tk@3.9
  
  If you do not need a specific version of Python, and always want Homebrew's `python3` in your PATH:
    brew install python3
  
  See: https://docs.brew.sh/Homebrew-and-Python
  Updating certificates bundle '/opt/homebrew/etc/openssl@1.1/cert.pem'
  Requirements installation successful.
  Installing Ruby from source to: /Users/travis/.rvm/rubies/ruby-2.7.2, this may take a while depending on your cpu(s)...
  ruby-2.7.2 - #downloading ruby-2.7.2, this may take a while depending on your connection...
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
  100 14.0M  100 14.0M    0     0  9801k      0  0:00:01  0:00:01 --:--:-- 9818k
  ruby-2.7.2 - #extracting ruby-2.7.2 to /Users/travis/.rvm/src/ruby-2.7.2.....
  ruby-2.7.2 - #configuring.........................................................................
  ruby-2.7.2 - #post-configuration.
  ruby-2.7.2 - #compiling.............................................................................|
  ruby-2.7.2 - #installing............
  ruby-2.7.2 - #making binaries executable...
  Installed rubygems 3.1.4 is newer than 3.0.9 provided with installed ruby, skipping installation, use --force to force installation.
  ruby-2.7.2 - #gemset created /Users/travis/.rvm/gems/ruby-2.7.2@global
  ruby-2.7.2 - #importing gemset /Users/travis/.rvm/gemsets/global.gems...............................-
  ruby-2.7.2 - #generating global wrappers........
  ruby-2.7.2 - #gemset created /Users/travis/.rvm/gems/ruby-2.7.2
  ruby-2.7.2 - #importing gemsetfile /Users/travis/.rvm/gemsets/default.gems evaluated to empty gem list
  ruby-2.7.2 - #generating default wrappers........
  ruby-2.7.2 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
  Install of ruby-2.7.2 - #complete 
  Ruby was built without documentation, to build it run: rvm docs generate-ri
  ```

- 查看版本信息

  ```bash
  weihongxudeMacBook-Pro:~ travis$ ruby -v
  ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [arm64-darwin21]
  ```



## 三、下载pod

- 下载命令

  ```bash
  sudo gem install cocoapods
  
  # 下载成功
  34 gems installed
  ```

- 可以pod --help测试一下



### pod install or pod update

```bash
# 只安装新添加的库
pod install --verbose --no-repo-update

# 会在安装相关库时 更新其他库版本
pod update --verbose --no-repo-update

# 只更新指定的库，其它库忽略
pod update 库名 --verbose --no-repo-update
```




## 参考链接

- [Failed to connect to raw.githubusercontent.com:443 - 知乎](https://zhuanlan.zhihu.com/p/115450863) 见评论区
- [mac ruby下载](https://www.jianshu.com/p/c073e6fc01f5)