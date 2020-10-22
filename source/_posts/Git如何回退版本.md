title: Git如何回退版本
toc: true
categories: git
tags:
  - git
thumbnail: /logo/github.jpg
---
1. 首先使用git log查看最近几次提交的版本号，如版本号"0250cd";

2. 在命令行输入 git reset --hard 0250cd，成功后会提示"head is now at 0250cd"；

3. git push -f -u origin you_branch,you_branch需要回退的分支名

