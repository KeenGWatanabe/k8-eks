# steps
```
cd /path/to/your/main/repo
git submodule add <repository-url> G4app
git commit -m "Added G4app as submodule"
git submodule update --remote
```
# since you can only update your own branch commit there first
```
git add .
git commit -m "add submodule"
git push origin -u branch
```