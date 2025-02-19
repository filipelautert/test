
# In this example we have 2 repos: test and simple. Simple will be a subtree inside test, so we'll have the following structure:
# test
#   \___ simple


### Initial setup/merge
# Checkout test project, then:
git remote add -f simple git@github.com:filipelautert/simple.git
git merge -s ours --no-commit --allow-unrelated-histories simple/main 
git read-tree --prefix=simple -u simple/main
git commit -m "Subtree merged in simple"
git push


### Syncing/Merging remote changes to simple

# 1. Commit to simple
# 2. Now in test:
git fetch simple
git diff main:simple/ simple/main # we can see the differences!
git subtree pull --prefix simple simple main
# git merge -s subtree --allow-unrelated-histories simple/main -- in case of issues this command may help

# check diff, push
git diff main:simple/ simple/main # no changes now
git push


### New file added inside simple in test respo
1. create file in test
echo -n "Test" > simple/new-file-in-test.txt
git add simple/new-file-in-test.txt 
git commit -m "New file added inside simple in test respo"
git push

git diff main:simple/ simple/main # shows that we have new file
# push back to simple
git push simple $(git subtree split --prefix simple main):main --force	


### Syncing/Merging remote changes to simple with a conflict
# 1. modify simple/new-file-in-test.txt 
git add simple/new-file-in-test.txt 
git commit -m "New file added inside simple in test respo"
git push

# 2. In simple, modify new-file-in-test.txt too. Commit, push
3. Now back to test:
git diff main:simple/ simple/main # we can see the differences, and a conflict
git subtree pull --prefix simple simple main # now we got a conflict to solve
vim simple/new-file-in-test.txt # fix conflict manually
git add simple/new-file-in-test.txt 
git commit -m "Fixing conflict"

git diff main:simple/ simple/main # all fine again
git push
