一、配置 Git

设置用户名和邮箱（提交记录用）：

git config --global user.name "你的名字"
git config --global user.email "你的邮箱"


查看配置：

git config --list

二、初始化仓库

初始化本地仓库：

git init


克隆远程仓库：

git clone <仓库地址>
# 示例：
git clone https://github.com/user/repo.git

三、工作区操作

查看文件状态：

git status


添加文件到暂存区：

git add <文件名>
git add .   # 添加当前目录所有修改


提交到本地仓库：

git commit -m "提交说明"


查看提交历史：

git log
git log --oneline --graph --all

四、分支管理

查看分支：

git branch


创建分支：

git branch <分支名>


切换分支：

git checkout <分支名>


创建并切换分支：

git checkout -b <分支名>


合并分支：

git merge <分支名>


删除分支：

git branch -d <分支名>

五、远程操作

查看远程仓库：

git remote -v


添加远程仓库：

git remote add origin <仓库地址>


拉取远程分支：

git pull origin <分支名>


推送本地分支到远程：

git push origin <分支名>


克隆指定分支：

git clone -b <分支名> <仓库地址>

六、撤销操作

撤销工作区修改（未 add 的文件）：

git checkout -- <文件名>


撤销暂存区修改：

git reset <文件名>


修改最近一次提交信息：

git commit --amend

七、查看差异

查看工作区修改：

git diff


查看暂存区和上次提交差异：

git diff --cached

八、标签（版本标记）

创建标签：

git tag <标签名>


查看标签：

git tag


推送标签：

git push origin <标签名>

九、常用快捷命令

快速 add + commit：

git commit -am "说明"


重置到指定提交：

git reset --hard <commit_id>


删除远程分支：

git push origin --delete <分支名>
