# 配置同时使用 Gitlab、Github、Gitee(码云) 共存的开发环境

首先确认已安装Git，可以通过 `git –version` 命令可以查看当前安装的版本。 

Mac OSX 中都已经安装了Git。但是，Git的版本未必是最新的。

可以通过命令 `git clone https://github.com/git/git` 进行更新

Git共有三个级别的config文件，分别是`system、global和local`。

在当前环境中，分别对应

%GitPath%\mingw64\etc\gitconfig文件
$home.gitconfig文件
%RepoPath%.git\config文件

其中`%GitPath%`为Git的安装路径，`%RepoPath%`为某仓库的本地路径。

所以 system 配置整个系统只有一个，global 配置每个账户只有一个，而 local 配置和git仓库的数目相同，并且只有在仓库目录才能看到该配置。

大致`思路`，**建立两个密钥，不同账号配置不同的密钥，不同仓库配置不同密钥。**

## 1. 清除 git 的全局设置（针对已安装 git）

新安装 git 跳过。

若之前对 git 设置过全局的 `user.name` 和 `user.email`。
类似 (用 `git config --global --list` 进行查看你是否设置)

```batch
$ git config --global user.name "你的名字"
$ git config --global user.email  "你的邮箱"
```

必须删除该设置

```
$ git config --global --unset user.name "你的名字"
$ git config --global --unset user.email "你的邮箱"
```

## 2. 生成新的 SSH keys

### 1）GitHub 的钥匙

指定文件路径，方便后面操作：`~/.ssh/id_rsa.gitlab`

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "lx@qq.com"
```

直接回车3下，什么也不要输入，就是默认没有密码。

注意 github 和 gitlab 的文件名是不同的。

### 2）GitLab 的钥匙

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitlab -C "lx@vip.qq.com"
```

### 2）Gitee 的钥匙

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitee -C "lx@vip.qq.com"
```


### 3)完成后会在~/.ssh / 目录下生成以下文件

- id_rsa.github
- id_rsa.github.pub
- id_rsa.gitlab
- id_rsa.gitlab.pub

---

## 3.添加识别 SSH keys 新的私钥

`亲测Mac下，新增一个 id_rsa.gitee，没加进去 也识别到了。
所以此步骤可忽略，如有问题删除所有密钥 重新按步骤操作一遍。`

默认只读取 id_rsa，为了让 SSH 识别新的私钥，需要将新的私钥加入到 SSH agent 中

```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa.github
$ ssh-add ~/.ssh/id_rsa.gitlab
$ ssh-add ~/.ssh/id_rsa.gitee
```

## 4. 多账号必须配置 config 文件(重点)

若无 config 文件，则需创建 config 文件

### 创建config文件
```
$ touch ~/.ssh/config    
```

### config 里需要填的内容

亲测可以不缩进，为了方便看，建议缩进。

### 最简配置

```
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa.github
```

### 完整配置
```
#Default gitHub user Self
Host github.com
    HostName github.com
    User git #默认就是git，可以不写
    IdentityFile ~/.ssh/id_rsa.github

#Add gitLab user 
    Host git@gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_rsa.gitlab

# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_rsa.gitee


# 其他自己搭建的
Host git@git.startdt.net
    Port 22
    HostName http://git.startdt.net
    User git
    IdentityFile ~/.ssh/lab_rsa.startdt

```

下面对上述配置文件中使用到的配置字段信息进行简单解释：

- Host
它涵盖了下面一个段的配置，我们可以通过他来替代将要连接的服务器地址。 
这里可以使用任意字段或通配符。 
当ssh的时候如果服务器地址能匹配上这里Host指定的值，则Host下面指定的HostName将被作为最终的服务器地址使用，并且将使用该Host字段下面配置的所有自定义配置来覆盖默认的/etc/ssh/ssh_config配置信息。 

- Port
自定义的端口。默认为22，可不配置 

- User
自定义的用户名，默认为git，可不配置 

- HostName
真正连接的服务器地址 

- PreferredAuthentications
指定优先使用哪种方式验证，支持密码和秘钥验证方式 

- IdentityFile
指定本次连接使用的密钥文件


## 5.在 github 和 gitlab 网站添加 ssh

可以用cat命令把 公钥 的内容打出来复制

```
$ cat id_rsa.gitee.pub

ssh-rsa AWQeI5ELxwnHoJmtafQEfJ1OJflTchpdBGbzv lx@vip.qq.com
```


### Github

直达地址：https://github.com/settings/keys

过程如下： 
1. 登录 Github 
2. 点击右上方的头像，点击 `settings`
3. 选择 `SSH key` 
4. 点击 `Add SSH key` 

在出现的界面中填写 SSH key 的名称，填一个你自己喜欢的名称即可。
将上面拷贝的` ~/.ssh/id_rsa.xxx.pub `文件内容粘帖到 key 一栏，在点击 “add key” 按钮就可以了。 

添加过程 github 会提示你输入一次你的 github 密码 ，确认后即添加完毕。

### Gitlab

直达地址：https://gitlab.com/profile/keys

1. 登录 Gitlab 
2. 点击右上方的头像，点击 `settings`
3. 后续步骤如 Github

### Gitee / 码云

直达地址：https://gitee.com/profile/sshkeys

1. 登录 Gitee 
2. 点击右上方的头像，点击 `设置`
3. 后续步骤如 Github

添加过程 码云 会提示你输入一次你的 Gitee 密码 ，确认后即添加完毕。


## 6.测试是否连接成功

由于每个托管商的仓库都有唯一的后缀，比如 Github 的是 git@github.com:*。

所以可以这样测试： 
ssh -T git@github.com 

而 gitlab 的可以这样测试： 
ssh -T git@gitlab.corp.xyz.com 
如果能看到一些 Welcome 信息，说明就是 OK 的了


- ssh -T git@github.com
- ssh -T git@gitlab.com
- ssh -T git@gitee.com

```
$ ssh -T git@github.com

Warning: Permanently added the RSA host key for IP address '13.250.177.223' to the list of known hosts.
Hi dragon! You've successfully authenticated, but GitHub does not provide shell access.

$ ssh -T git@gitlab.com

The authenticity of host 'gitlab.com (35.231.145.151)' can't be established.
ECDSA key fingerprint is SHA256:HbW3g8zUjNSksFbqTiUWPWg2Bq1x8xdGUrliXFzSn.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'gitlab.com,35.231.145.151' (ECDSA) to the list of known hosts.
Welcome to GitLab, @dragon!

$ ssh -T git@gitee.com 

The authenticity of host 'gitee.com (116.211.167.14)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrp+KkGYoFgbVr17bmjeyc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'gitee.com,116.211.167.14' (ECDSA) to the list of known hosts.
Hi 我是x! You've successfully authenticated, but GITEE.COM does not provide shell access.

```
结果如果出现这个就代表成功：

- GitHub -> successfully
- GitLab -> Welcome to GitLab
- Gitee -> successfully

### 测试 clone 项目

```bash
$ git clone git@gitlab.com:d-d-u/java-xxx.git
Cloning into 'java-basic'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.
```

## 7.操作过程出现的问题/报错

### tilde_expand_filename: No such user

检查是否成功的时候，报错：`tilde_expand_filename: No such user .`

```batch
$ ssh -T git@github.com
tilde_expand_filename: No such user . 
```

解决方法：

此问题是因为`写错了文件路径` 或者 `大小写没写对`，删除重新配置，或者复制我的改好粘贴进去。

## 8. 参考链接

- https://www.cnblogs.com/kelsen/p/8342239.html
- [git 多账号 ssh-key 管理（github和gitlab共同使用）](https://blog.csdn.net/qq_30227429/article/details/80229167), by 长是人千离
- https://stackoverflow.com/questions/27708264/when-trying-to-clone-a-repository-i-get-the-following-message-tilde-expand-file
- https://blog.csdn.net/u014296452/article/details/79984867