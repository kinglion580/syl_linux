```bash
ls -l 
```

## 修改
```bash
chmod [u|g|o]+[r|w|x|s] <text_name>
```

## 查看文件权限数字：`stat` 💡

```bash
stat -c %a <text_name>
``` 
## 查看文件所有者，所有组
```bash
stat -c %U <text_name>
stat -c %G <text_name>
```
*参考* :https://www.linuxidc.com/Linux/2015-11/125326.htm
