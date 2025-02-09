```shell
Git 全局设置:

git config --global user.name "wenhaizhou"
git config --global user.email "14492812+wenhaizhou@user.noreply.gitee.com"
创建 git 仓库:

mkdir learn_info
cd learn_info
git init 
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com:wenhaizhou/learn_info.git
git push -u origin "master"
已有仓库?

cd existing_git_repo
git remote add origin git@gitee.com:wenhaizhou/learn_info.git
git push -u origin "master"
```

