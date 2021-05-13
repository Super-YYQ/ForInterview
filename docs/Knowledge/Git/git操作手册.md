[廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

[GIT官网下载地址](https://git-scm.com/downloads)

## 一、初始配置

### 1.1 用户

```sh
## 官网安装包--》安装完成后右键 打开bash页面--》安装成功
## Git 用户设置
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

### 1.2 SSH Key

```shell
## 用户主目录下，看看有没有 .ssh 目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key
## 一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码
## id_rsa和id_rsa.pub两个文件就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人
$ ssh-keygen -t rsa -C "youremail@example.com"
```

## 二、代理设置

> C盘用户目录下 .gitconfig 文件



## 三、仓库

> Windows系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文

### 3.1 创建本地仓库并与远程库关联

```shell
## 创建文件夹--learngit
$ mkdir learngit
$ cd learngit
## 查看当前目录
$ pwd
/Users/michael/learngit

## 创建版本库
$ git init
Initialized empty Git repository in /Users/michael/learngit/.git/

## 本地仓库与远程库关联
$ git remote add origin git@github.com:michaelliao/learngit.git
```

### 3.2 本地分支回退后强制推送到仓库

```shell
## 当代码需要回退时，本地回退后强制远程仓库和本地一致
git pull -f 
```



## 四、分支管理

### 4.1 cherry-pick

[廖雪峰cherry-pick教程](http://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)

```shell
# 例：将指定提交应用于master分支

# 切换到 master 分支
$ git checkout master

# Cherry pick 操作
$ git cherry-pick kj835j35h2j
```

![image-20210409152953268](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20210409152953.png)

