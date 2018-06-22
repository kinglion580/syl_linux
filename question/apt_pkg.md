### 2.2 安装包

本实验用到了 pillow 这个模块，在安装它前更新源

```sh
$ sudo apt-get update
```

首先我们需要处理一个问题：当前实验楼的环境中 python3 命令使用的 python 版本为 3.5，但源中却没有 python3.5-dev，这会导致安装 Pillow 出错。所以我们必须将 python3 命令使用的 python 版本切换为 3.4，然后再安装 python3-dev 和 python3-setuptools。

```sh
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.4 70 --slave /usr/bin/python3m python3m /usr/bin/python3.4m
$ sudo apt-get install python3-dev python3-setuptools
```

然后安装 Pillow 依赖包

```sh
$ sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk
```

最后安装 Pillow：

```sh
$ sudo pip3 install Pillow
```

**还有一个参考链接：https://stackoverflow.com/questions/13708180/python-dev-installation-error-importerror-no-module-named-apt-pkg **
```
sudo ln -s apt_pkg.cpython-{35m,34m}-x86_64-linux-gnu.so apt_pkg.so
```

## 运行 add-apt-repository 报错的问题
```
$ sudo python3.4 /usr/bin/add-apt-repository ppa:ansible/ansible
```
由于 python 升级之后使用 `add-apt-repository` 会报错（ImportError：No module named 'apt_pkg'），这是因为在 `/usr/lib/python3/dist-packages` 中 `apt_pkg` 的连接只有 `python3.4`，所以为了避免因为版本的问题这里我们就特别的指定 python 的版本信息。
