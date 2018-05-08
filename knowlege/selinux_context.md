# SELinux 策略与规则管理

## 1 实验介绍

### 1.1 实验内容

通过上节实验我们对 SELinux 有一个初步的了解，在本节实验中我们开始 SELinux 中核心内容：上下文、策略，对于上下文与策略的管理将直接影响相关应用的使用，希望大家能够灵活的使用相关的工具，能够结合上节的日志工具在出现问题时能够及时的排查与修复。

### 1.2 实验知识点

- SELinux 的上下文
  + SELinux 上下文的概念
     + SELinux 的身份标识
     + SELinux 的角色
     + SELinux 的类型
     + SELinux 的级别
  + SELinux 上下文的操作
     + SELinux 上下文的查看
     + SELinux 上下文的临时修改
     + SELinux 上下文的永久修改
     + SELinux 上下文的恢复
     + SELinux 上下文的保持
- SELinux 的策略与管理
  - SELinux 策略查看
  - SELinux 策略管理
     + SELinux 策略配置
     + SELinux 策略布尔值

### 1.3 推荐阅读

+ [SELinux 官方文档](http://selinuxproject.org/page/AdminDocs)
+ [策略存储迁移](https://github.com/SELinuxProject/selinux/wiki/Policy-Store-Migration)
+ [SELinux 政策管理](https://opensource.com/business/13/11/selinux-policy-guide#.UoWMDG-jYlE.sinaweibo)

## 2 SELinux 的上下文

### 2.1 SELinux 上下文的概念

#### 2.1.1 语法结构

安全性上下文主要存在于主体程序中与目标文件资源中，放置的位置是在文件的 inode 内。可以通过 `ls -Z` 命令来查看当前目录下的安全性上下文的具体结构内容。

```
ls -Z
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986023655.png/wm)

从打印出来的信息可以看到，中间由冒号分隔开的字段就是安全性上下文的主要内容。

它的语法结构是用冒号分为几个字段：

> user:role:type:level 

+ **user**

用户身份是被授权用于一组特定角色以及特定的多级安全（MLS）和多分类安全（MCS）使用级别的策略的已知身份。每个 Linux 用户都通过 SELinux 策略映射到 SELinux 用户。SELinux 用户和 Linux 用户之间的一个显著区别是 SELinux 用户在用户会话期间不会改变，而 Linux 用户可能通过 su 或 sudo 进行更改。

常见的用户身份有：

unconfined_u：不受限的用户
system_u：系统用户


+ **role**

SELinux 的一部分是基于角色的访问控制（RBAC）安全模型。role 是 RBAC 的一个属性。SELinux 用户被授权于 role，role 被授权于领域(domians)。可以通过输入的 role 决定哪些域可以输入，最后控制着哪些对象类型可以被访问。

一般的角色包括：

object_r：代表文件或目录等文件资源
system_r：代表程序

+ **type**

在 targeted 政策中，type 的地位相当重要，决定着主体程序是否能够读取文件资源，以及定义的 type 决定如何访问和访问的域等规则。

type：在文件资源 （Object） 上面称为类型 （Type）
domain：在主体程序 （Subject） 则称为领域 （domain） 

注：domain 需要和 type 结合使用才能顺利读取文件资源。

+ **level**

level 是 MLS 范围， 这个字段是可选的，会在后面说明。

### 2.2 SELinux 上下文的操作

#### 2.2.1 SELinux 上下文的查看

在 SELinux 系统上运行的所有进程和文件都会被相关安全信息进行标记，这就是 SELinux 上下文（也叫标签）。

+ 通过 `ls -Z` 命令来查看文件、目录的安全上下文。

```
$ ls -Z 
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986109523.png/wm)

+ 通过 `ps -Z` 命令查看进程的安全上下文。

```
$ ps -Z 
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986179405.png/wm)

+ 通过 `id -Z` 命令查看 当前用户 的安全上下文。

```
$ id -Z 
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986271778.png/wm)

> 补充：默认情况下，新建的文件或目录会继承其父目录的 SELinux 类型。

#### 2.2.2 SELinux 上下文的临时修改 : `chcon`

`chcon` 命令是修改文件的安全性上下文，比如：用户、角色、类型、安全级别。使用 `--reference` 选项时，把指定文件的安全环境设置为与参考文件相同。`chcon` 命令位于 `/usr/bin/chcon`。

语法格式

```
chcon [选项][-u 用户] [-r 角色] [-l 范围] [-t 类型] 文件
```

|选项|说明|
|-|-|
|`-u` |--user 用户：设置指定用户的目标安全环境|
|`-r` |--role 角色：设置指定角色的目标安全环境|
|`-t` |--type 类型：设置指定类型的目标安全环境|
|`-l` |--range 范围：设置指定范围的目标安全环境|
|`-R` |--recursive：递归处理所有的文件及子目录|

eg 1：更改文件或目录的类型

1.首先创建一个测试文件 `context1`。通过 `ls -Z context1` 命令查看 SELinux 安全上下文。

```
$ touch context1
$ ls -Z context1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986611970.png/wm)

输出的信息可以看到 SELinux 的安全上下文中包含了 `system_u` 用户，`object_r` 角色，`user_home_t` 类型和 `s0` 级别。

2.通过 `chcon -t httpd_sys_content_t context1` 命令将类型更改为 `httpd_sys_content_t`。然后查看更改情况 `ls -Z `
（类型段 `httpd_sys_content_t` 代表是网站服务系统文件）

```
$ chcon -t httpd_sys_content_t context1
$ ls -Z context1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512986969824.png/wm)

#### 2.2.3 SELinux 上下文的永久修改

`semanage ` 命令是用来查询与修改 SELinux 默认目录或文件的安全上下文。

在实验环境中，我们需要安装该工具，使用如下命令：

```bash
$ sudo yum install policycoreutils-python
```

语法格式

```
semanage fcontext [-a|d|m] [-frst] file
```

|参数|说明|
|-|-|
|`-l`|查询|
|`fcontext`|主要用在安全上下文方面| 
|`-a`|增加一些目录或文件的安全上下文类型设置|
|`-m`|修改|
|`-d`|删除|

eg 1：向网站数据目录中新增加一条 SELinux 安全上下文，让这个目录以及里面的所有文件能够被 `httpd` 服务程序所获取到。

1.首先创建一个新文件，并查看它的安全上下文信息

```
sudo touch /etc/context2
sudo ls -Z /etc/context2
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512987204963.png/wm)

默认情况下，/etc/ 目录中新建文件的类型为 `etc` 类型。 

2.运行`semanage fcontext -a -t httpd_sys_content_t /etc/context2`命令 context2 增加一条 `httpd_sys_content_t` 上下文。

```
sudo semanage fcontext -a -t httpd_sys_content_t /etc/context2 
```

该命令将新增的条目添加到 `/etc/selinux/targeted/contexts/files/file_contexts.local`

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512988295634.png/wm)

可以用 `-m` 参数对安全性上下文进行修改。

#### 2.2.4 SELinux 上下文的恢复

`restorecon` 命令来恢复文件的安全上下文。

语法

```
restorecon [-iFnrRv][filename]

-v：将过程显示在屏幕上
-i：忽略不存在文件
-R/-r：递归处理目录
```

通过 `restorecon -v context1` 命令恢复之前修改的 `context1` 文件的 SELinux 安全上下文。

```
$ restorecon -v context1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512988402175.png/wm)

SELinux 上下文操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/10-1.mp4
@`

## 3 SELinux 的策略与管理

在 SELinux 运行流程中我们知道一个事件流（或叫主体程序）能否读取到目标文件的重点在于 SELinux 的安全策略及策略内的规则，最后再根据规则去处理目标文件的安全上下文。

### 3.1 SELinux 策略查看

那么在 SELinux 中策略具体提供了什么规则，我们可以通过 `seinfo` 命令来进行查看。

+ seinfo

首先需要安装：`sudo yum install setools-console`

语法格式

```
seinfo [-Atrub]
```

|参数|说明|
|-|-|
|`--all` |列出 SELinux 的状态、规则布尔值、身份识别、角色、类别等所有信息|
|`-u` |列出 SELinux 的所有身份识别 （user） 种类|
|`-r` |列出 SELinux 的所有角色 （role） 种类|
|`-t` |列出 SELinux 的所有类别 （type） 种类|
|`-b` |列出所有规则的种类 （布尔值）|

**实例**

eg 1：列出与 `httpd` 有关的规则

```
sudo seinfo -b | grep httpd
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512989353717.png/wm)

+ sesearch

`sesearch` 命令可以查询相关类型或布尔值的详细规则。

语法格式

```
sesearch [-A] [-s 主体类别] [-t 目标类别] [-b 布尔值]
```
|参数|说明|
|-|-|
|`-A` |列出后面数据，允许“读取或放行”的相关数据|
|`-t` |类别|
|`-b` | SELinux 的规则|


**实例**

eg 1：找出目标文件资源类型为 `httpd_sys_content_t` 的有关信息： 

```
sesearch -A -t httpd_sys_content_t
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512989726421.png/wm)


### 3.2 SELinux 策略管理

SElinux 是一套强大的标签系统。每个进程都拥有自己的标签（上下文），操作系统中的每个文件或目录对象也拥有自己的标签。甚至包括网络端口、设备以及潜在主机名也会被分配到标签。通过编写规则来控制对某个进程标签以及对象标签的访问，这就是政策规则管理。SELinux 内核会强制执行这些规则，我们将这种方式称为强制访问控制（MAC）。

#### 3.2.1 强制机制类别

SELinux 通过内核对每一个单独进程进行访问控制。其中最主要的功能机制就是类型强制（TE），其中规则的作用是根据进程与对象双方的标签类型来判断某个进程是否有权读取相应的对象。此外另有两种控制机制，其中 MCS 负责对同类进程进行彼此区分，而 MLS 则允许一种进程拥有指向另一种进程的支配权限。

#### 类型强制(TE)
 
在 SELinux 中，所有访问都必须明确授权，SELinux 默认不允许任何访问。与标准 Linux 中的 root 不一样，通过指定主体类型（即域）和客体类型使用 allow 规则授予访问权限，allow 规则由四部分组成：

+ 源类型（Source type ） ：尝试访问的进程的域类型
+ 目标类型（Target type ） ：被进程访问的客体的类型
+ 客体类别（Object class）： 指定允许访问的客体的类型
+ 许可（Permission） ：象征目标类型允许源类型访问客体类型的访问种类

TE allow 规则语法：

```
allow user_t bin_t : file {read execute getattr};  
``` 

> 在前面操作中，通过 `sesearch -A -t httpd_sys_content_t` 命令就可以看到 `httpd_sys_content_t` 类型的相关规则。

> 规则包含了两个类型标识符：源类型(或主体类型或域) user_t，目标类型(或客体类型) bin_t。标识符 file 定义在策略中的客体类别名称(这里表示一个普通的文件)，大括号中包括的许可是文件客体类别有效许可的一个子集。

#### MCS 强制

多类安全 (`MCS: Multi-Category Security`)是一个增强的 SELinux，允许用户使用类别来标记文件。如果类型强制（TE）规则没问题，并且随机 MCS 标签匹配准确，那么访问就会被通过；如果二者中有一项不符合要求，访问就会被拒绝。

#### MLS 强制

`MLS(Multi Level Security)` 称为多级别安全是另一种强制访问控制方法，适合于政府机密数据的访问控制，早期对计算机安全的研究大多数都是以在操作系统内实现 MLS 访问控制为驱动的。MLS 与 MCS 非常相似，但它为强制机制添加了"支配"这一概念。MCS 标签必须在完全匹配的情况下才能通过，但一条 MLS 标签可以支配另一条 MLS 标签从而获得访问许可。在 SELinux 中 MLS 是一种可选访问控制方式。

在大多数 SELinux 策略中，是支持敏感度(s0，s1，...)和范畴(c0，c1，...)，将它留给用户空间程序和程序库，以指定有意义的用户名。

为了支持 MLS，安全上下文被扩展，包括了安全级别，如：

```
user:role:type:sensitivity[:category,...] [-sensitivity[:category,...]]  
```

可以通过下面的命令进行查看

```
ps -aZ
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512989802298.png/wm)

> MLS 安全上下文至少必须有一个安全级别（由单个敏感度和 0 个或多个范畴组成），但可以包括两个安全级别，这两个安全级别分别被叫做低（或进程趋势）和高（或进程间隙），如果高安全级别丢失，它会被认为与低安全级别的值是相同的（最常见的情况）。

**这里有一个官方的描述大家可以参考一下[链接的内容](https://opensource.com/business/13/11/selinux-policy-guide#.UoWMDG-jYlE.sinaweibo)**

#### 3.2.2 SELinux 策略配置

SELinux 的全局配置文件主要是在 `/etc/selinux/config` 下，包括了：

+ /etc/selinux/config
+ /etc/selinux/semanage.conf
+ /etc/selinux/restorecond.conf and restorecond-user.conf
+ /etc/selinux/newrole_pam.conf
+ /etc/sestatus.conf
+ /etc/security/sepermit.conf

SELinux 的内核配置文件位于 `/sys/fs/selinux` 目录下，反映了 SELinux 当前的活动策略配置。

SELinux 的策略配置文件主要是和策略的名称有关。

```
/etc/selinux/<policy_name>
```
大部分文件是通过参考策略，`semanage` 或 `semodule` 命令来进行安装配置的。也可以构建自定义的单一策略，比如：`context/files/file_contexts` 允许文件系统重新标记。

详细的策略配置文件，大家可以参考[SELinux 官方策略配置文件说明](http://selinuxproject.org/page/PolicyConfigurationFiles)

#### 3.2.3 SELinux 策略布尔值

布尔值主要是 SELinux 策略中启用和禁用规则的开关，也就是控制主体进程和目标文件是否有访问的权限。安全管理员可以通过使用布尔值来调整策略。

**1.查看布尔值**

通过 `semanage boolean -l` 命令并以 Linux root 身份运行。

```
sudo semanage boolean -l
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512989918614.png/wm)

还可以通过 `getsebool -a` 命令来列出所有的布尔值。

```
getsebool -a
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512989964787.png/wm)

运行 `getsebool boolean-name` 命令仅列出布尔型名称的状态

```
getsebool <boolean_name> #boolean_name 替换为策略名
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512990182119.png/wm)

**2.修改布尔值**

修改布尔值主要是通过 `setsebool` 命令来实现规则的启用或禁用。 

格式：

```
setsebool <boolean_name> on/off
```

**临时修改**

1.默认情况下，`httpd_can_network_connect_db` 布尔值为 `off`，是阻止 `Apache HTTP Server` 脚本和模块连接到数据库服务器

```
getsebool httpd_can_network_connect_db
```

2.修改布尔值，临时启用 Apache HTTP Server 脚本和模块以连接到数据库服务器

```
sudo setsebool httpd_can_network_connect_db on
```

3.再次验证 

```
getsebool httpd_can_network_connect_db
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4127timestamp1512990342253.png/wm)

这样就允许 Apache HTTP Server 脚本和模块连接到数据库服务器。

**永久修改**

要使更改后在重新启动时保持不变，需要用 Linux root 身份运行命令：`setsebool -P boolean-name on`

SELinux 策略与管理操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/10-2.mp4
@`

## 4 总结

本节实验学习了 SELinux 中的核心内容：上下文、策略，对于上下文与策略的管理将直接影响相关应用的使用。
在下一节将对 SELinux 进行一些实际的操作练习，来加深大家的理解。
