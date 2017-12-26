
```bash
$ sudo tar -c -v -f shiyanlou1.tar -g metadata /home/shiyanlou/Desktop
```
> `-g` 指定快照文件，
> `-c` 指示建立归档，
> `-f` 指定归档文件名称，
> `-v` 显示详情

在快照文件创建时，为 0 级备份。在上图中的提示信息，tar: 从成员名中删除开头的“/”，代表的意思是对于一个完整的路径 /home/shiyanlou/... 而言，在保存时，将会删除掉开头的 / ，即为 home/shiyanlou/...。这是为了在恢复文件时不会与根目录冲突。

解压 `tar -xvf` 
