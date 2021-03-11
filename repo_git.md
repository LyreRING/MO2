## 学习时间：
* 2021/3/11
## 学习内容
* Git
## 学习计划：
* 复习git的基本操作，理解git
## 学习总结：

#### 创建版本库：
* mkdir一个新目录 将其作为repository cd后进行操作
* git init将当前目录变为git可管理仓库（会生成.git版本库）
* git add将repository中的文件添加到stage（暂存区）
* git commit（-m "修改信息"）将stage中的文件提交到当前分支
* git status查看当前文件状态
#### 版本修改：
* HEAD指针指向当前版本，HEAD^(^^^/~100)表示前面的版本
* git log可查看版本提交的历史，git reflog可以查看命令历史（帮助查找需跳转的版本）
* git reset -hard (版本号/指针）回退到某一版本
* git checkout -- (文件名）用版本库里的版本替换工作区的版本
* git rm删除文件
#### 远程仓库：
* ssh-keygen -t rsa -C "用户名/邮箱" 创建SSH key后将id_rsa.pub添加至GitHub（为了确保提交来源）
* 在GitHub创建远程仓库
* git remote add “仓库名” git@github.com:仓库地址 关联远程仓库
* git push (-u) 仓库名 推送分支 可将本地库内容全部推送
* git clone git@github.com:仓库地址 可以直接克隆一个本地库
* git tag (name) (id) 查看（打）标签
#### 分支管理；
* git branch (name) 查看（创建）分支
* git switch -c (name) 创建+切换分支
* git switch name 切换分支
* git merge (name)合并某分支
* git branch -d (name)删除分支
* git分支可以使团队使用不同分支工作，完成后再推送到主分支
* 上面的条件是git merge --no-ff -m "信息" （name）才可以看出分支信息
* git stash可以将目前的工作储存 git stash pop可恢复
* git pull可解决因远程分支比本地更新的冲突
* 若提示no tracking informaition则需用git branch --set-upstream-to <branchname> origin/<branchname>创建关系
* git rebase可将提交历史整体成直线
#### 其他：
* Gitee也提供免费仓库，Sourcetree让git操作可视化
* Git可以在.git中创建.gitignore文件忽略某些文件使其不被提交
* git config (-- global) ...可以配置别名等自定义git
* linux可以自行搭建git服务器



