* git add
    * For new files(Untracked files), modified files or deleted files, you need to use git add to move the files from working directory to staging area.
    * For deleted files, you can use "git rm" instead of "rm" + "git add"

    * git add {filename}
    * git add *.{fileExtension}
    * git add -all
    * git add {filename} -p

* git commit -m
    * Moving the files from staging area to repository.
    * git commit -a -m: git add(the files already existed in repository) + git commit
    * git commit --amend -m: rename the last commit and move the files from staging area to repository with the last commit.
     
* git log 
    * git log --oneline --author="Sherly\|Eddie"
    * git log --oneline --grep="WTF": search for message contents of commits.
    * git log {filename}: check logs of specific file. 

* git mv
    * Move or/and rename files
    
* git clean
    * git clean -fX: clean the files should be ignored.

* git blame     
    * git blame {filename}: check the last commit author of each line of code.

* git checkout
    * git checkout {filename}: Use the content of the file in staging area to override the content of the file in working directory.
    * git checkout .:Use the content of all files in staging area to override the content of all files in working directory.
    * git checkout HEAD~2 .: Use the content of all files from the second last version to override the content of the file in working directory. And also update the staging area. 
    * git checkout {branch name}
    
* git reset
    * git reset {commit id}^
    * git reset {commit id}~n 
    * git reset {branch name}^
    * git reset HEAD^
    
* git reflog

* git rebase -i {commit id}: chose which commits should be picked
 