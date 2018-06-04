设置环境变量。

## 不打开文件直接使用 linux 命令添加内容

### 追加
```bash
echo xxx >> filename

echo DEV_SERVER="https://dev.shiyanlou.com" | sudo tee -a /etc/profile

sudo sed -i '$ a\DEV_SERVER=https://dev.shiyanlou.com' /etc/profile

sudo ex -sc 'a|DEV_SERVER=https://dev.shiyanlou.com' -cx /etc/profile
#这是使用的 vim 的 ex 模式
```
