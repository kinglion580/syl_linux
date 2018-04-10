👉 **`/opt/data/lab-docker-images/shiyanlou/sub_images/seed`**

杀掉 sshd
```
pkill sshd
```

- 启动 vnc 不应写在 dockerfile 中，而是写在初始化脚本中，再在 CMD 中执行脚本

- CMD 中写 /usr/sbin/sshd ，run 容器的时候退出的原因：
写在 CMD 中是在 fork 出的进程中执行的，主进程没有，所以自动退出了。需要修改为 `/usr/sbin/sshd -D` 。

- 设置 vnc 密码
```
su - shiyanlou -c 'mkdir ~/.vnc' ; \
su - shiyanlou -c 'echo 123456 | /usr/bin/tigervncpasswd -f > ~/.vnc/passwd' ;\su - shiyanlou -c 'chmod 600 ~/.vnc/passwd' ; \
su - shiyanlou -c 'touch ~/.vnc/xstartup ; chmod 755 ~/.vnc/xstartup' ; \
echo '#!/bin/sh \n export XMODIFIERS="@im=SCIM" \n export GTK_IM_MODULE="scim"\n scim -d & \n startxfce4 & \n' > /home/shiyanlou/.vnc/xstartup ; \
chmod 600 /home/shiyanlou/.vnc/passwd; \
chmod 755 /home/shiyanlou/.vnc/xstartup; \ 
```

- `init.sh` 文件的权限要有执行权限

- vnc 时，删除 tmp 缓存要用 `rm -rf /tmp/.X*` 而不是 `rm -f` 。不要少了 r 。

- `run` 的时候，末尾不要跟命令，如 bash 等，会覆盖 dockerfile 中 CMD 的命令，导致 init.sh 脚本并未运行。

- 可以使用 `docker create container ...` 创建容器

👉 **`/opt/data/lab-docker-images/shiyanlou/sub-images/centos7-desktop`**

使用的阿里云的源。

- 添加中文 centos

```dockerfile
# install for chinese
RUN \
yum install -y kde-l10n-Chinese.noarch cjkuni-ukai-fonts ;\
true

COPY ./conf /etc/shiyanlou/conf

RUN \
# move the config file
mv /etc/shiyanlou/conf/locale.conf /etc/locale.conf;\
true

ENV LC_ALL "zh_CN.UTF-8"
```

💡 在 init.sh 中添加了 export 各种语言相关变量的值为中文还是不行。后来发现 vnc 的启动脚本 `~/.vnc/xstartup` 中有一系列 export 语言相关变量的值为英文。这里肯定有问题，需要更改为中文。

另外需要配置 xfce 的语言：`/etc/xdg/xfce4/xinitrc`
```bash
# locale
echo 'export LANG="zh_CN.UTF-8"' >> /etc/xdg/xfce4/xinitrc; \
echo 'export LANGUAGE="zh_CN"' >> /etc/xdg/xfce4/xinitrc; \
```

- 时区
```bash
# timezone
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
sed -i 's/UTC=yes/UTC=no/' /etc/default/rcS; \
```

- 添加中文输入法

可以使用 ibus 或者 fcitx+sogou ，这里使用 ibus 。因为 centos 自带。

**问题：** ibus 没法使用

**解决：** reinstall 安装发现提示 ibus 不可用。说明 ibus 安装有问题。remove 后重新 install 。

```bash
yum search ibus*
yum remove -y ibus
yum install -y ibus
yum list|grep ibus
```

其中有关于中文主要的东西是 `ibus-libpinyin` 以及 `ibus-table-chinese-*.noarch` 。

**问题：** 安装了 ibus ，xfce pannel 也就是下面的状态栏没有键盘图标。

**解决：** 使用 `imsettings-switch ibus` 切换或者使用 `ibus-daemon -d -x` 启动。

*参考：* 
1. https://zhidao.baidu.com/question/1771912838782126380.html

2. https://wiki.archlinux.org/index.php/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

**问题：** vnc 启动时没有自动启动 ibus ，要设置自动启动。

**解决：** 设置环境变量 `~/.bashrc` 或者 `~/.profile` 文件。

```bash
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -d -x
```

**问题：** ibus 首选项中输入法没有汉语的选项，也就是没有中文输入法。

**解决：** 重装中文输入法：

```bash
yum install -y ibus-table-chinese*
yum install -y ibus-libpinyin
```

- x 安装 `im-chooser` 并且执行 `im-chooser` 选择中文。还是不行

```bash
yum install -y im-chooser
```
*参考：* https://github.com/ibus/ibus/wiki/CentOS

- `ibus list-engine` 可以看到有哪些语言。
- 验证是否有中文字体。查看本机安装了哪些中文字体 `fc-list :lang=zh` 
*参考：* https://blog.csdn.net/u012724167/article/details/53510589

- 清除历史命令纪录

```bash
history -c //清空历史执行命令
echo > ./.bash_history //或清空用户目录下的这个文件即可
```

*参考：* https://blog.csdn.net/leadway123/article/details/49155421
