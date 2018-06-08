## git push 不用每次输入密码

```bash
git config --global credential.helper store
```

## git 强制覆盖本地内容

```
git fetch --all  
git reset --hard origin/master 
git pull
```
