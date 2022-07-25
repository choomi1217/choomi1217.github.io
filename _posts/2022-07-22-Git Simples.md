# Git

[참고 자료](https://learngitbranching.js.org/?locale=ko)

전 문제를 

----------------------------------

git branch -f main C6
git checkout HEAD~1
git branch -f bugFix HEAD~1

----------------------------------

git branch -f main C6
git checkout HEAD^
git branch -f bugFix HEAD^

----------------------------------

git reset HEAD~1
git checkout pushed
git revert HEAD

----------------------------------

git rebase main bugFix
git rebase bugFix side
git rebase side another
git rebase another main


----------------------------------

git branch -f three c2
git rebase -i three
git rebase -i three
git branch -f main c5
git branch -f one c2'
git rebase -i three
git rebase -i three
git branch -f main c5
git branch -f three c2
git branch -f two c2'

-solution
git checkout one
git cherry-pick C4 C3 C2
git checkout two
git cherry-pick C5 C4 C3 C2
git branch -f three C2

----------------------------------