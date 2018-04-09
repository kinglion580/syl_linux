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

在 init.sh 中添加了 export 各种语言相关变量的值为中文还是不行。后来发现 vnc 的启动脚本 `~/.vnc/xstartup` 中有一系列 export 语言相关变量的值为英文。这里肯定有问题，需要更改为中文。

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
