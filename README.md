# How to merge core extension changes to your app overrides

The repository has the following structure

```
.
+-- README.md
+-- core/
    +-- lib/
        +-- myfile.txt
+-- app/
    +-- lib/
        +-- myfile.txt
```

The app and core libraries are identical, as if the app override was just created.

Once changes have been made to the code, we do the following:

## Create a temporary branch consisting of only the core library

```shell
git checkout -b tmp-branch
git filter-branch --subdirectory-filter core/lib
git checkout master
```
## Merge the temporary branch into the app directory and delete temporary branch

```shell
git merge -X subtree=app/lib tmp-branch --squash --allow-unrelated-histories
git branch -D tmp-branch
```

**NOTES**

 * The `--squash` command is used to avoid duplicate commits in the tree (one from core and one from the merge into app)
 * The `--allow-unrelated-histories` is necessary for this to work
 * This keeps the main tree really clean!  All you will see is one commit for the core -> app merge.
 * A rather large cost of going manual, even though you get a clean tree, is the need to do address a lot more conflicts (for some reason).  The subtree strategy below addresses this - far fewer conflicts on merge, with a slightly more complicated tree.  **I'm inclined to say it's worth it, and will most likely use the subtree strategy below.**

# Alternate strategy using subtrees

The steps above are manual, and while they have the benefit if leaving a clean tree, it makes it more difficult by requiring more manual handling of conflicts.  This is due to the "unrelated histories" between the trees.  Using subtrees can help this, with of course the tradeoff of a much messier tree.  There are two main commands to (1) create the branch of the core library, and (2) merge the branch into the app library.  You can also keep the branch around permanently (this makes the tree a lot cleaner -- good topic for a future post).  Also note that it is probably a good idea, if you choose to keep the branches around, to push them to GitHub with `git checkout core/lib; git push -u origin core/lib`.  **DO NOT PUSH THE CORE/LIB BRANCH AFTER CHANGES, THOUGH!**  This really muddies up the graph, creating extra branches.  In truth, the branch seen is a temporary branch that git creates, and isn't *really* the `core/lib` branch, but as long as you don't push the `core/lib` branch after changes are made to it, it will be labeled as such.  Plus, the `core/lib` branch is really only there for reference - it's contents don't really matter, as it simply mirrors the `core/lib` directory contents.  In summary: *only push the core/lib branch after creation so the "temporary" branch is labeled on GitHub*.  Okay, onward...

This command will create the app library from the core library (assume for now the app directory doesn't exist - see below):

```shell
git subtree split -P core/lib -b core/lib
git subtree add --prefix=app/lib core/lib
```

*Note*: I have to do a `git status` in between these two commands, otherwise I get the statement `Working tree has modifications.  Cannot add.`  <shrug>

These two commands are all you need when you want to merge core to app:
```shell
git subtree split -P core/lib -b core/lib
git subtree merge --prefix=app/lib --squash core/lib
```

The `git subtree split` command will update the branch, and the `git subtree merge` will perform the merge.

## What if my app directory already exists, like in this repo?

There's a nifty way to set everything up without having to redo the commits.  

```shell
git subtree split -P app/lib -b app/lib
git rm -r app/lib
git subtree add --prefix=app/lib core/lib
git subtree merge --prefix=app/lib --squash app/lib 
git branch -D app/lib
```

What this does is (1) create a temporary app/lib branch with the app/lib code, (2) remove the directory, (3) create the subtree for the app/lib directory, (4) merge in the app/lib changes on top of the core/lib code, and (5) delete the temporary app/lib branch.

You can now proceed as though you knew what you were doing from the start!
