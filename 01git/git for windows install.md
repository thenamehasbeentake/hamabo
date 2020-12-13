# git for windows install

## 下载git

- [下载 ](https://gitforwindows.org/) 并安装

- git config --global core.autocrlf false

- git config --global user.name "name"

- git config --global user.email "email"

- ssh-keygen, 回车

  Your public key has been saved in /c/Users/wen/.ssh/id_rsa.pub

  复制id_rsa.pub到git web上，Settings->SSH keys->new SSH key

- git web上创建仓库，取个独特的名字。回到shell上git clone .....git

- 写点东西， git add --all, git commit -m "first commit", git push origin master

## gitbash方便界面设置

- 邮件windos上Git Bash图标，选择属性->目标（T)，去掉--cd-to-home,在起始位置（s）加入存放git仓库的目录。下次运行git时起始位置就会跳转到此地

- 修改git安装目录下 Git\etc\profile.d\git-prompt.sh，注释掉

  PS1="$PS1"'$MSYSTEM '          # show MSYSTEM

  该行， 打开git bash 就不会出现难看的MINGW64了