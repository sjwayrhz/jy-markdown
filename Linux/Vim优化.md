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

