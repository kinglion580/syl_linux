# help
- `F1`
- `:ver` display version and parameter
- `h shiftwidth` open the help file named shiftwidth
- `$ vimtutor`

# frequent
- `:set nu` set nuber
- `:set ts` :set tabstop(tab indent)
- `:set sw` set shiftwidth(the indent of `>>` command)
- `:set ai` set autoindent
- `:set aw` autowrite
- `:set bk` auto backup
> `~/.vimrc`

## get current setting
- `:se` display all modified configurations
- `:set all` display all set values
- `:set option?` display the set value of `option`
- `:set nooption` cancel the current set value

## 粘贴乱格式解决

先复制粘贴到粘贴板，点击保存。然后在 vim 中输入命令 `:set paste` ，再按 i 进入到 插入（粘贴）模式，然后用右键粘贴就不会乱了。
