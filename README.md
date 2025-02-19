# Git subtree POC

In this example we have 2 repos: test and simple. Simple will be a subtree inside test, so we'll have the following structure:
```
test
 \___ simple
```

# Test Scenarios
1. Initial setup/merge (only once, but some steps may be required in automation)
2. Syncing/Merging remote changes from simple to test (aka syncing OSS repo to PRO)
3. Adding a new file in test/simple directory and syncing it back to simple (aka syncing a file from PRO back to OSS)
4. Syncing/Merging remote changes to test with a conflict (aka syncing OSS repo to PRO with conflicts)
5. Syncing/Merging remote changes to test with a conflict Using a branch and PR (aka syncing OSS repo to PRO with conflicts)
6. Cloning and syncing in a new location

### 1. Initial setup/merge
*  Checkout test project, then:
```
git clone git@github.com:filipelautert/test.git
git remote add -f simple git@github.com:filipelautert/simple.git
git merge -s ours --no-commit --allow-unrelated-histories simple/main 
git read-tree --prefix=simple -u simple/main
git commit -m "Subtree merged in simple"
git push
```


### 2. Syncing/Merging remote changes from simple
1. Commit to simple
2. Now in test:
```
git fetch simple
git diff main:simple/ simple/main # we can see the differences!
git subtree pull --prefix simple simple main

# check diff, push
git diff main:simple/ simple/main # no changes now
git push
```
The following command may be useful in emergencies to resync:
```
git merge -s subtree --allow-unrelated-histories simple/main
```

### 3. Adding a new file in test/simple directory and syncing it back to simple
1. create file in test
```
echo -n "Test" > simple/new-file-in-test.txt
git add simple/new-file-in-test.txt 
git commit -m "New file added inside simple in test respo"
git push
git diff main:simple/ simple/main # shows that we have new file
# push back to simple
git push simple $(git subtree split --prefix simple main):main --force
```

Some docs says that a simple `git push simple main simple` should work, but looks like as branch gets different we need the `subtree split` command .


### 4. Syncing/Merging remote changes to test with a conflict
1. modify simple/new-file-in-test.txt
```
git add simple/new-file-in-test.txt 
git commit -m "New file added inside simple in test respo"
git push
```
2. In simple, modify new-file-in-test.txt too. Commit, push
3. Now back to test:
```
git diff main:simple/ simple/main # we can see the differences, and a conflict
git subtree pull --prefix simple simple main # now we got a conflict to solve
vim simple/new-file-in-test.txt # fix conflict manually
git add simple/new-file-in-test.txt 
git commit -m "Fixing conflict"
git diff main:simple/ simple/main # all fine again
git push
```

### 5. Syncing/Merging remote changes to test with a conflict Using a branch and PR (aka syncing OSS repo to PRO with conflicts)
1. modify simple/new-file-in-test.txt
```
git add simple/new-file-in-test.txt 
git commit -m "New file added inside simple in test respo"
git push

```
2. In simple, modify new-file-in-test.txt too. Commit, push
3. Still in simple, create a new branch called `conflict-fix`
4. Now back to test:
```
git diff main:simple/ simple/main # we can see the differences, and a conflict
git subtree pull --prefix simple simple main # now we got a conflict to solve
vim simple/new-file-in-test.txt # fix conflict manually
git add simple/new-file-in-test.txt 
git commit -m "Fixing conflict"
git diff main:simple/ simple/main # all fine again
git push
```


### 6. Cloning and syncing in a new location
```t push simple main simple
git clone git@github.com:filipelautert/test.git
git remote add -f simple git@github.com:filipelautert/simple.git
git subtree pull --prefix simple simple main
```

### 7. Automatic Sync with GitHub Actions
This repository includes an automated workflow that syncs changes from the Simple repository daily. The workflow:
- Runs automatically at midnight UTC
- Can be triggered manually via GitHub Actions
- Creates a pull request when changes are detected
- Automatically enables auto-merge on created PRs

To manually trigger the sync:
1. Go to the "Actions" tab in GitHub
2. Select "Sync Simple Subtree" workflow
3. Click "Run workflow"
4. Select the branch (usually `main`) and run

The workflow will:
- Pull latest changes from the Simple repository
- Create a PR if there are any updates
- Enable auto-merge on the PR
- Use the branch name `sync-simple-updates` for the changes
- Fail if there are merge conflicts (requiring manual intervention)

To modify the sync schedule, edit the cron expression in `.github/workflows/sync.yml`:
```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Runs every day at midnight UTC
```

#### Handling Sync Failures
When the automatic sync encounters merge conflicts:
1. The workflow will fail with an error message
2. No PR will be created
3. Manual intervention will be required to resolve conflicts

To resolve conflicts manually:
```bash
git fetch simple
git subtree pull --prefix simple simple main
# Fix conflicts in the affected files
git add .
git commit -m "fix: resolve sync conflicts with Simple repository"
git push
```
