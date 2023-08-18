```shell
#设置用户名
git config --global user.name  "feiyongjing"  
#设置密码              
git config --global user.password "123456"    
#查看当前的配置信息              
git config --list                                           
```
 
---
1.安装好git  
2.桌面右键 Git Bash Here 打开git命令行  
3.ssh-keygen -t rsa -C "nideyouxiang@xxx.com"   （全部按enter）  
4.cd ~/.ssh   （如果没有执行第三步，则不会有这个文件夹）  
5.cat id_rsa.pub     在命令行打开这个文件，会直接输出密钥  
6.复制，打开github，点自己头像 >> settings >> SSH and GPG keys >>New SSH key  
7.titile 随便写, key里粘贴第六步的内容；完成。
***


## 生成多对密钥，注意生成的密钥在当前目录下，除非输入绝对路径

https://blog.csdn.net/jiangyu1013/article/details/103559435

 
***  
## 使用多对密钥，注意生成的密钥在当前目录下，除非输入绝对路径

https://help.aliyun.com/document_detail/322237.html

 

注意如果使用的是github，-t参数大写
~~~shell
ssh -T git@fyj
~~~

停止指定文件的版本控制，但该文件会保留在工作区      
~~~shell
git rm --cached [targetDir] 
~~~
 

 
