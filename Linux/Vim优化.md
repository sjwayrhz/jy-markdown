## Vim编辑器不变色

执行如下脚本，可以使得root用户的vim变色

```
#!/bin/bash/
path=/etc/nginx/

mkdir /root/.vim/syntax -pv  
cd /root/.vim/syntax/
wget http://www.vim.org/scripts/download_script.php?src_id=14376 -O nginx.vim
`echo "au BufRead,BufNewFile $path* set ft=nginx" > /root/.vim/filetype.vim`
```

## Vim编辑器不支持中文

（1）、vim ~/.vimrc (~/.vimrc为vim配置文件)

（2）、输入：

```
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```

