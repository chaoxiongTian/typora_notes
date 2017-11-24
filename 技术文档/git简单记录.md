---
typora-copy-images-to: ../pic
---

### git 简单记录

**需求**:

把本地的项目同步的一个新的git仓库上.

**步骤**:

1. `git init`
2. `git add .`
3. `git commit -m 'first commits' -a`
4. `git remote add origin https://github.com/chaoxiongtian/typora_notes.git`
5. `git push -u origin master` #这一步可能会出错 若是空项目则可以强行更新 `git push -u origin +master`

