---
description: Centralized VS distributed
---

# SVN / GIT

### [https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)\`

### CMS: content management system

### VCS: version control system

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2F2ERTmVwDS4pK1DgBlAqA%2Ffile.png?alt=media)

### SVN： version control + backup server

{% hint style="info" %}
SVN更适合项目管理，Git更适合代码管理
{% endhint %}





## Github

### search keyword:

xxx in:name, description, readme

xxx star:>=5000

xxx fork:>=3000

**awesome xxx    -->一般是收集学习资料，工具等类型的内容**

 highlight: url+L#num - L#num



### IntelliJ Idea:

1. install git first, and set direct in IDE
2. login your github account

### Operations:

1. add.
2. ignore exclude, add file and folder on the exclude list (hand added-->better)
3. commit -> to local
4. check history: copy version number
5. git->reset HEAD --> hard reset --> put version number and execute --> change to specific version
6. **branches-->create new branch --> change to new branch**
7. **after finish the branch, --> checkout to master, --> merger--> choose "dev" branch --> merge**

#### 如果本地不是最新的，push会被rejected

> github， setting里面add collaborated worker， 别人也就可以push了。

![](<../.gitbook/assets/image (167).png>)

![feat / fix / refactor / docs / test / chroe / style](<../.gitbook/assets/image (240).png>)

```
Git stash         //hide the changes
git stash pop    //pop to the last one, and remove the history from stash list
git stash apply //apply to the last one, keep the stash list history
git stash drop //drop the stash history

git rebase //merge many commits into one commit
git rebase -i [start point] 
git rebase -i [start point] [end point]

git log  //show all the commits




```





