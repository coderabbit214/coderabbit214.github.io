# Git


# git命令

## 在本地新建git项目
在项目根目录  右键 - git bash - git init

## 在远程建立git项目
new-建立项目- 生成  URL或者SSH

## 本地项目-远程项目关联

git remote add origin URL或者SSH

## 第一次发布项目 （本地-远程）

git add .      //文件-暂存区
git commit -m "注释内容"  //暂存区-本地分支（默认master）
git push -u origin master

## 第一次下载项目（远程-本地）

git clone git@github.com:yanqun/mygitremote.git

## 提交(本地-远程)

(在当前工作目录 右键-git bash)
git add.
git commit -m "提交到分支"
git push  origin master

## 更新(远程-本地)

git pull

## 用于提供自己的名字和email地址

git config --global user.name [name]
git config --global user.email []
## 保存登陆的用户密码，避免每次都要重新输入
git config --gloable credential.helper store
## 查看配置信息列表
git config --list
## 克隆项目
git clone URL
## 拉取项目变动
git pull res_name
## 上传更改
git commit -m "上传说明"
git push
## 撤销上传（版本回退）
git reset --hard HEAD~N (到退N次提交前)
git commit
git push -f （强制传送到远程respostiries）
注意：-f 是一个强制推送的操作，是危险操作！设置倒退N个版本之前，那么在N个版本之前到最新版本的内容会全部消失，若是同时有人想该respostories上传了文件，也会一并消失

## 直接由本地，上传文件到指定respostories

git init
git remote add res_name URL
git add .
git commit -m "上传说明"
git pull res_name master
git push res_name master

## 添加文件的追踪
git add file_name
git status (可以看见文件追踪的变动情况)
git commit -m "提交说明" (这一步特别注意，不知道为什么，使用 git commit 是无法生效的，必须按照左边的格式提交)
git push res_name branch_name

## 本地项目-远程项目关联

git remote add origin git@github.com:yanqun/mygitremote.git


