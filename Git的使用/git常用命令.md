- ## 新建代码库
~~~shell
#下载一个项目和它的整个代码历史  
git clone [url]                   

#新建一个目录，将其初始化为Git代码库
git init [project-name]

#取回远程仓库的变化，并与本地分支合并
git pull [remote] [branch]
# git pull --rebase 保持线性提交记录。该操作会暂存本地提交，拉取远程更新后重新应用提交，避免生成多余合并节点。若遇到冲突，可通过`git rebase --abort`回退然后git pull 手动解决冲突。
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
# 规范的message格式写法：<type>[scope]: <subject>
# 案例：fix(user): 修复用户修改密码后session未刷新问题
# type：提交类型（必填，定义修改的性质）通过 type 快速区分提交的目的，常用类型如下（覆盖 90%+ 场景）
type	    中文含义	 适用场景
feat	    新功能	       新增业务功能（如 “用户登录接口”“订单列表分页”）
fix	        修复 bug	   修复代码逻辑错误、功能异常（如 “修复登录时密码加密失败”“解决分页数据重复”）
docs	    文档修改	   仅更新 README、注释、API 文档等（不涉及代码逻辑）
style	    代码格式调整   不影响代码运行的格式修改（如缩进、空格、变量命名规范、删除空行）
refactor	代码重构	   既不新增功能也不修复 bug，仅优化代码结构（如 “提取公共工具函数”“简化条件判断”）
perf	    性能优化	   提升代码运行效率（如 “减少数据库查询次数”“优化循环逻辑”）
test	    测试相关	   新增 / 修改测试用例（如 “添加登录接口单元测试”“修复测试断言错误”）
build	    构建相关	   影响项目构建流程的修改（如 “更新依赖版本”“调整 webpack 配置”）
ci	        CI 配置修改	   持续集成 / 部署流程的调整（如 “修改 Jenkinsfile”“更新 GitHub Actions 脚本”）
chore	    日常琐事	   其他不涉及代码逻辑的修改（如 “删除无用文件”“统一文件编码”）

# scope：修改范围（可选，定义影响的模块）scope 用于明确提交影响的 “模块 / 功能域”，让读者快速定位修改位置，格式为小写名词，示例：
业务模块：user（用户模块）、order（订单模块）、payment（支付模块）
技术模块：api（接口层）、db（数据库层）、utils（工具类）、router（路由）
全局模块：global（全局配置）、common（公共组件）

# subject 是对提交内容的简短描述，建议不超过 50 个字符，结尾不加句号

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

#选择一个commit，合并进当前分支
git cherry-pick [commit]​

#删除分支                                                                      
git branch -d [branch-name]

#删除远程分支                
git push origin --delete [branch-name]                    
git branch -dr [remote/branch]
~~~


- ## 分支合并
方式一：合并指定分支到当前分支，会产生一个新的节点记录合并   
~~~shell                                         
git merge [branch]
# 假设当前在 main 分支，要合并 feature 分支的修改：
# 1、确保当前分支是目标分支（如 main）
git checkout main
# 2、合并 feature 分支到当前main分支
git merge feature
# 3、如果合并冲突就打开冲突文件，Git 会用以下标记标出冲突部分，手动编辑文件，选择保留的内容，并删除这些标记
<<<<<< HEAD
当前分支的更改
======
目标分支的更改
>>>>>> <目标分支>
# 4、保存修改后，暂存已解决的文件
git add [冲突文件]
# 5、提交合并，无需参数，Git 会自动生成合并提交信息
git commit  
# 6、若合并中遇到冲突，想放弃合并回到操作前的状态
git merge --abort
~~~

方式二：合并指定分支到当前分支，会产生一个新的节点记录压缩提交合并 
~~~shell
git merge --squash [branch]
# 合并前：feature 分支有 3 个提交
main:    A → B
feature: B → C1 → C2 → C3

# 执行 git merge --squash feature（在 main 分支）后：
main:    A → B → [暂存了 C1+C2+C3 的修改]
# 手动提交后：
main:    A → B → S（S 包含 C1、C2、C3 的所有修改）
出现冲突和方式一解决冲突的方式是一样的
~~~


方式三：以目标分支的最新提交为新起点重新应用一遍，仿佛当前分支是基于目标分支的最新状态创建的
注意：不要在公共分支（如 main、develop）上使用 rebase，公共分支被多人共享，rebase 会改变提交历史，导致团队成员的本地仓库与远程冲突。
~~~shell
git rebase [branch]
# 变基前：feature 基于 main 的旧提交 B 创建，main 已新增提交 C
main:    A → B → C
feature:     B → D → E

# 执行 git rebase main（在 feature 分支上）后：
main:    A → B → C
feature:          C → D' → E'  # D、E 被重新应用到 C 之后（提交哈希值改变与原D、F不同，记为 D'、E'）
# feature分支的历史改变了，feature分支原始的B节点消失了，取而代之的是C节点，造成在main分支删除的情况下feature分支无法回滚到B节点

# git rebase 合并出现冲突，按照以下步骤
# 1、查看冲突文件，输出中会列出所有标记为 Unmerged 的文件，这些文件需要手动解决
git status
# 2、打开冲突文件，Git 会用以下标记标出冲突部分，手动编辑文件，选择保留的内容，并删除这些标记
<<<<<< HEAD
当前分支的更改
======
目标分支的更改
>>>>>> <目标分支>
# 3、保存修改后，暂存已解决的文件
git add [冲突文件]
# 4、继续 Rebase，如果还有其他提交需要处理，Git 会重复上述步骤，直到完成Rebase变基
git rebase --continue
# 5、如果无法解决冲突或想放弃Rebase变基，可以中止操作，回滚到Rebase变基之前的状态
git rebase --abort

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

 



