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
Ah, cleaning up after submodules—classic Git hygiene. Here’s how you can completely remove a submodule like `G4app` from your repo, step by step:

1. **Unstage and remove the submodule directory**  
   ```bash
   git rm --cached G4app
   rm -rf G4app
   ```

2. **Delete the submodule entry in `.gitmodules`**  
   Open `.gitmodules` and remove the entire section related to `G4app`.

3. **Commit the changes**  
   ```bash
   git commit -m "Removed G4app submodule"
   ```

4. **Optionally clean up `.git/config`** (if the submodule left a config trace)  
   ```bash
   git config -f .git/config --remove-section submodule.G4app
   ```

5. **Remove residual data in `.git/modules`**  
   ```bash
   rm -rf .git/modules/G4app
   ```

This cleans things up nicely, both in your repo and behind the scenes. If you're doing this in a Terraform or ECS-related repo, it's a good idea to double-check that no build scripts or CI references are expecting that submodule to exist.

Need a quick script to automate this cleanup or want help doing it across multiple branches? I can help with that too.