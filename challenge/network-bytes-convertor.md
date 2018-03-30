# Bash 实现网络单位换算器

## 介绍

在 Linux 中系统将很多数据实时更新在 `/proc` 下的一些文件文件中，这些数据并不是很直观，看起来不够人性化，需要实现一个单位转换器，能够帮助我们在满 1024 的时候便转换成较为直观的单位。

我们希望实现的单位转换器 `Conversion.sh` 里面主要是一个单位转换的函数，只要满 1024 就转换单位，直到 GB 单位，若还是满 1024 则可以不用转。

例如转换 1024 时：

```bash
bash /home/shiyanlou/Conversion.sh 1024
1 KB
```

例如转换 1099511627776 时：

```bash
bash /home/shiyanlou/Conversion.sh 1099511627776
1024 GB
```

脚本名称为 `Conversion.sh`，可以使用 source 之后调用其中的 Convert 函数，使用方式及预期输出如下：

```
$ source /home/shiyanlou/Conversion.sh 
$ Convert 1024
1 KB
```

## 目标

1. 脚本路径必须放在 `/home/shiyanlou/Conversion.sh`
3. 该函数能够将单位转换成合理的单位，例如 1024 Bytes 转换成 1 KB，3221225472 Bytes 转换成 3 GB。
2. 换算出来的结果中只需要包含 `B KB MB GB` 中的一种，另外输出为整数即可，比如 31.54 GB，那么只需要显示 `31 GB` 即可。
5. 能够在子进程中调用脚本中的核心函数 Convert

## 提示语

1. export 导出函数

## 知识点

* export
* Bash 函数


> **让数字一直循环除以 1024，直到小于 1024 就退出，这样直接判断除的次数即可知道 单位** 💡

```bash
#!/bin/bash 
Convert(){
local value=$1
local unit=0
while [[ value -ge 1024 && unit -le 2 ]]
    do
let "value=value/1024"
let unit=unit+1
    done

case $unit in
0)
echo "$value B"
;;
1)
echo "$value KB"
;;
2)
echo "$value MB"
;;
3)
echo "$value GB"
;;
esac
}

export -f Convert
```

> 常规思路

```bash
#!/bin/bash

function checkNumber() {
	re='^[0-9]+([.][0-9]+)?$'
	if ! [[ $1 =~ $re ]]; then
        return 1
	fi
    return 0
}

function Convert() {

    if ! checkNumber $1; then
        echo "expect number but receive $1"
        return 1
    fi

	local gb=$((1024 * 1024 * 1024))
	local mb=$((1024 * 1024))
	local kb=1024

	if (($1 > $gb)); then
		echo "$(($1 / $gb)) GB" 
	elif (($1 >= $mb)); then
		echo "$(($1 / $mb)) MB"
	else
		echo "$(($1 / $kb)) KB"
	fi
}

export -f Convert
```




