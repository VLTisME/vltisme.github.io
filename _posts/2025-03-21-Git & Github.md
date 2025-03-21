---
title: How to use Git & Github intuitively
date: 2025-03-21 09:55:00 +0700
categories: [tools]
tags: [productivity, management]
authors: [tuan]

description: A brief explanation about how to use Git and Github, visually.
comments: true

---  

Best tutorial video: [tutorial](https://www.youtube.com/watch?v=apGV9Kg7ics&t=404s&ab_channel=KunalKushwaha);

(just image, oi)
will write when i have time

## Must-do when using Git/ Github
First, imagine starting a new world, nothing is there! We will start our adventure for now!

### Create a repository on Github
Then... all of a sudden, a massive tank appeared!

### cd
Welp firstly, there must be somewhere so that I can put my code there... So let's find it!

### Create a repository on your device
OMG, another tank appeared!

### mkdir, ls, touch, echo, vi, cat
- mkdir: create a new folder.
  e.g: mkdir Lel $$\rightarrow$$ create a folder named Lel
- ls: list all stuff inside a folder.
- touch: create a new file.
  e.g: touch "haha.txt" $$\rightarrow$$ create a file called lel.txt
- echo: create (if doesn't exist) and append content in there.
  e.g: echo "this is the content" > "haha.txt" $$\rightarrow$ if "haha.txt" doesn't exist, create one and append "this the the content" in the content of the file.
- vi: allows you to write contents for a specific file
  e.g: vi "haha.txt" $$\rightarrow$ then it will show you the current content of "haha.txt" and do what ever you want there. If you want to exist, press "Esc" then type ":wq" or ":x".
- cat: allows you to view the content of a file.
  
### add, status, commit, log, reset
- add:
- status:
- commit:
- log:
- reset:

### rm, restore
- rm --cached:
- rm:
- restore --staged
- restore

### stash
- stash: ehh stash is like a collection... sometimes you need sth there and you will find (pop) it, sometimes you don't need it anymore, just clear it!
- stash pop:
- stash clear:

### remote add origin, push origin main

### branch, checkout
- branch:
- checkout:
- checkout -b:

### fork, clone, upstream

### git rebase
- git rebase -i: merge all commits from the... to before the one with the harsh code provided into 1 commit (or you can simply git reset then these commits go to staging area then commit it :D). But git rebase allow you to "squash" (which you have to type "s") or "pick" (remain "pick"), it will merge all commits from this "pick" to before the next "pick", then you ":x" then you type "commits merged" (it is actually a commit message :v, so type watever u want) then you ":x" again (x = exit), git log and you will see the change.

### conflict
- conflict occurs when you have two branches, two both changed the same line of a same file, then both commit, push, then PR

### why everytime forked, cloned, must create a new branch and work on it?
- Imagine working in a big project, 10 people contribute to 10 different features, but Github allows only one PR for one branch, so if 10 people work on the same branch "main", how would they be able to control, to read, to fix when bugs or discussions happen... all the freaking 10 people from 10 features work in one discussion place is totally a mess.
- So yeah, each feature must create a new branch to work on it, and when working on another branch, everytime a commit is executed, it will go to that branch, not the main one! And we can PR for a branch, and discuss only about things related to that feature.

### Notes
- Imagine HEAD is a pointer points to the current branch.
- When commit, it shows u a sth sth that tells that you want to merge the branch u just created into the upstream, so before open PR, make sure it passes tests? or just create a PR? then run tests later??? and discuss with the author!...
- When want to delete a commit in your PR, use "git reset" to the commit before it, then stash all the changes you don't need, then FORCE push to the branch because the remote repo has commits that your local doesn't have so you must force it to interlink.
- Pay attention before merging PR (i mean when you the one you forked also changes as you made to the upstream repo), take a look if is there any commit behind, if yes then fetch upstream. Or let's say, if there is another person that MADE the changes to the main branch of upstream, and you want to look at the code, how?: 
  + 1: You can use "fetch upstream" like above.
  + 2: Or manually, use Git, checkout to main branch in the forked, then git log, see how many commits were made? is it equal to the number of commits made to the main branch of the upstream? If not equal, then your main forked branch is not updated with the main from the upstream, then you have to git --fetch --all --prune (all = all branches, prune = fetch even the deleted commits :V), then you now can see all the commits ahead you (i mean commits in the main of the upstream, which your forked main doesn't have) and use "git reset --hard upstream/main" to have the lastest commit from the main branch of upstream. git log again and you will see all the commits from the main of the upstream are now visible. Then if you want to update the forked main branch, given now you have all the commits of the main upstream haha, you just basically push all the commits to the forked main branch! = git push origin main (origin is your work you forked from another repo, that another repo is the upstream, now you want to update the main... cool af).
  + OR, like the above method, but from 'git --fetch --all --prune', you can replace it with 'git pull upstream main' !!!!!!!! very easy (meaning: pull all the commits in the main branch of the upstream), and then just git log and you will see that you don't even need to 'git reset --hard' but ALL THE COMMITS FROM MAIN UPSTREAM ARE NOW IN FORKED MAIN BRANCH! so yeah just then 'git push origin main'. ALL in one single command! (or 'fetch upstream' button then merge on github, very fast lmao').
- Make sure whenever you create a new branch from the main origin branch, the main must be updated (the same as the upstream branch).
- Remember when 'reset', all the commits resetted go to the staging area (maybe:D, i just know its not dissappeared)

- a person who is new to Github when creating a repo always choose "add a readme.md", to resolve the error when pushing commits to that repo, just remember to update your main origin branch with the main origin but remote first by 'git pull origin main' and add a flag "--allow-unrelated-histories".
