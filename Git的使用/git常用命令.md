- ## 新建代码库
~~~shell
#下载一个项目和它的整个代码历史  
git clone [url]                   

#新建一个目录，将其初始化为Git代码库
git init [project-name]

#取回远程仓库的变化，并与本地分支合并
git pull [remote] [branch]
~~~
 

- ## 增加/删除文件
~~~shell
#添加指定文件到暂存区
git add [targetDir]

#删除工作区文件，并且将这次删除放入暂存区
git rm [-r] [targetDir]

#停止指定文件的版本控制，但该文件会保留在工作区
git rm --cached [file]

#改名文件，并且将这个改名放入暂存区
git mv [sourceDir] [targetDir]
~~~
 

- ## 代码提交到本地
~~~shell
#将选择的文件提交到本地
git commit [-m "massage"]

#使用一次新的commit，替代上一次提交
git commit --amend
~~~


- ## 分支操作
~~~shell
#列出所有本地分支
git branch

#列出所有远程分支
git branch -r  ​

#列出所有本地和远程分支
git branch -a

#新建一个分支，但依然停留在当前分支
git branch [branch-name]

#新建一个分支，并切换到该分支
git checkout -b [branch]

#新建一个分支，指向指定commit状态
git branch [branch] [commit]

#新建一个分支，与指定的远程分支建立追踪关系     
git branch --track [branch] [remote-branch]​

#切换到指定分支，并更新工作区                                
git checkout [branch-name]​

#切换到上一个分支
git checkout -

#建立追踪关系，在现有分支与指定的远程分支之间  
git branch --set-upstream [branch] [remote-branch]

#合并指定分支到当前分支                                            
git merge [branch]

#选择一个commit，合并进当前分支
git cherry-pick [commit]​

#删除分支                                                                      
git branch -d [branch-name]

#删除远程分支                
git push origin --delete [branch-name]                    
git branch -dr [remote/branch]
~~~
 

- ## 远程同步
~~~shell
#下载远程仓库的所有变动
git fetch [remote]

#显示所有远程仓库
git remote -v

#显示某个远程仓库的信息
git remote show [remote]

#增加一个新的远程仓库，并命名
git remote add [remoteName] [url]

#取回远程仓库的变化，并与本地分支合并
git pull [remote] [branch]

#上传本地指定分支到远程仓库
git push [remote] [branch]

#强行推送当前分支到远程仓库，即使有冲突        
git push [remote] -f

#推送所有分支到远程仓库                                       
git push [remote] --all
~~~

 



