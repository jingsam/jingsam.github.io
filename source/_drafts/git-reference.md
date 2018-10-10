---
title: Git内部原理之Git引用
date: 2018-06-19 17:41:21
tags:
---


# HEAD引用

# 标签引用

# 远程引用

# 总结


```
$ git log --pretty=oneline 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 third commit
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

$ echo "3ac728ac62f0a7b5ac201fd3ed1f69165df8be31" > .git/refs/heads/master
$ git update-ref refs/heads/master 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31

$ git log master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 (HEAD -> master) third commit
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

$ git update-ref refs/heads/test d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c
$ git log --pretty=oneline test
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c (test) second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

$ cat .git/HEAD
ref: refs/heads/master
$ git checkout test
$ cat .git/HEAD
ref: refs/heads/test
$ git symbolic-ref HEAD refs/heads/master
$ git symbolic-ref HEAD refs/heads/test

$ git update-ref refs/tags/v1.0 d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c
$ cat .git/refs/tags/v1.0

$ git tag -a v1.1 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 -m "test tag"
$ cat .git/refs/tags/v1.1
8be4d8e4e8e80711dd7bae304ccfa63b35a6eb8c
$ git cat-file -p 8be4d8e4e8e80711dd7bae304ccfa63b35a6eb8c
object 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
type commit
tag v1.1
tagger jingsam <jing-sam@qq.com> 1529481368 +0800

test tag


$ git remote add origin git@github.com:jingsam/git-test.git
$ git push origin master
Counting objects: 9, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (9/9), 720 bytes | 360.00 KiB/s, done.
Total 9 (delta 0), reused 0 (delta 0)
To github.com:jingsam/git-test.git
 * [new branch]      master -> master
$ cat .git/refs/remotes/origin/master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
$ cat .git/refs/heads/master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
```
