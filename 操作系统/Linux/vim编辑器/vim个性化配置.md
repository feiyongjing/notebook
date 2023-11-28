~~~shell
#设置某个用户vim永久生效的配置或快捷键编辑 ~/.vimrc
vim ~/.vimrc   # 编辑vim配置文件，添加一下一些例子并保存

# 设置系统下全部用户vim的配置或快捷键 /etc/vim/vimrc 建议不要修改它，可能影响其他的用户
vim /etc/vim/vimrc

# 设置tab键缩进4个空格，默认tab键是缩进8个空格
set tabstop=4
set shiftwidth=4
set expandtab
 

set nu # vim永久显示行号
set ic # vim搜索时永久不区分大小写
set shiftwidth=4 # 设置在可视模式（按v进入可视模式）下，使用=格式化选中代码块时缩进的空格数
ab mymail feiyongjing@cloudgrow.cn  # vim编辑时永久输入mymail回车后将mymail直接替换为feiyongjing@cloudgrow.cn
map ^_ I#<ESC> # vim永久设置Ctrl+/ 在光标所在行前添加一个#（起到注释作用），但是注意设置^_是由Ctrl+v打出的^和Ctrl+/打出的_
map ^P 0x<ESC> # vim永久设置Ctrl+p 删除光标所在行开头第一个字符（起到去除注释作用），但是注意设置^P是由Ctrl+v打出的^和Ctrl+p打出的P
~~~