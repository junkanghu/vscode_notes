[TOC]
# Git Command
## 1. create local repo
1. create a directory
   ``` ubuntu
   mkdir nerf-pytorch
   cd nerf-pytorch
   ```
2. make the directory a manageable repo (which will automatically generate ".git" file in this directory)
```
   git init -b main 
```
3. configure the user identification
   ```
   git config --global user.name "hujunkang"
   git config -global user.email "hujunkang123@qq.com"
   ```

4. add **existing** files to the repo (before using ```git commit```, ```git add``` can be called for many times)
   ```
   git add xxx.md
   git add xxx.md xx.py
   ```
5. confirm the submission (confirm the submissions by ```git add```, cuz ```git add```)
   ```
   git commit -m "description" # -m is for description which'll be recorded in the 'log' file
   ```


## 2. version rollback
1. modify the existing files and call ```git add``` and ```git commit``` to resubmit the files
2. check the log to choose which version to rollback
   ```
   git log
   ```
   examples:
   ```
   commit xxxxxxx # the string follows the 'commit' is the commit id which can be used to version rollbcak
   Author: xxxxx
   Date: xxxx
   ```
3. version rollback
   1. version representations
    **HEAD**: the current version
    **HEAD^**: the last version
    **HEAD^^**: the last version of last version
    **HEAD~n**: the n-last version
   2. rollback
      1. using **HEAD** etc.
        ```
        git reset --hard HEAD^
        ```
      2. using commit id 
        ```
        git reset --hard xxx # when typing the commit id, only the first several characters needed, git will automatically find the whole id
        ```
      3. restore the version disappeared for rollback
        
        calling
        ```
        git reflog
        ```
        to check the history command where you'll find the commit id 

## 3. check the status of current repo
```
git status
``` 
which will show the status of files in current directory.

## 4. working directory and storage cache
1. working directory: the directory of the git repo
2. ```git add``` sumbit the files to the storage cache, ```git commit``` submit the files in storage to the master branch (automatically generated when init git)
![schematic](./images/directory.jpeg)

## 5. manage the modification
1. ```git commit``` only submits the files in storage.
2. if you modify the same file twice, the two files all need to be sumbitted to the storage, then the master branch. (git will take the latest one)

## 6. revoke the modification
1. discard the modification in working directory (not yet submitted to storage)
   
   ```
    git checkout -- xxx.md
   ```
2. discard the modification already submitted to the storage (not yet submitted to the master branch)
   
    ```
    git reset HEAD xxx.md # discard the modification in storage
    git checkout -- xxx.md # discard the modification in source files 
    ```
3. discard the modification in master branch
   using version rollback

## 7. delete files and restore files
1. delete files in workding directory (not yet submitted to the master branch)

```
rm xxx.md
```
2. delete files both in workding directory and master branch

```
git rm xxx.md # delete files in master branch
rm xxx.md # delete files in working directory
git commit -m "remove xxx.md"
```
3. restore files deleted by mistake in working directory (exists in master branch)

```
git checkout -- xxx.md
```
actually, it's replacing the version in working directory with the version in the master branch
## 8. add local repo to remote github
1. create ssh keys
   1. create keys in root path
    in root path type command
    ```
    ssh-keygen -t rsa -C "hujunkang123@qq.com"
    ```
    and keep clicking Enter to use default configuratin. Then '.ssh' directory will be created in root path.

   2. in '.ssh' directory, "id_rsa" is the personal key, "id_rsa.pub" is the public keys which you can transfer to others.

   3. Add keys to Github
       1. go to ```https://www.github.com/settings/ssh``` , type the username and password
       2. click "Add Key" and randomly type a title
       3. paste the content in the "id_rsa.pub"
2. add files to remote Github
    1. creating repo in Github
    2. link local repo with remote repo
        ```
        git remote add origin git@github.com:junkanghu/xxx.git # origin is the name of remote repo, we can use other names
        ```
        xxx is the name of remote repo.
    3. push local files to remote linked repo for the first time
        ```
        git push -u origin master # -u only used in the first time
        ```
    4. modify local files in repo and repush
        ```
        git push origin master
        ``` 
3. cut the link between two repos
    1. check the information about remote repo (check which remote repo is linking with local repo)
        ```
        git remote -v
        ```
    2. remove the link (the files still exist in remote repo, once the link is cut, local repo can link with new remote repo)
        ```
        git remote rm origin
        ```

## 9. clone repo from github
If remote repo is in your own account, link will be automatically created, and you can modify and push.

## Error solution
1. git fatal: 拒绝合并无关的历史
```shell
git pull origin master --allow-unrelated-histories
```
2. 