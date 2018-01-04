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

# Alternate strategy using subtrees

The steps above are manual, and while they have the benefit if leaving a clean tree, it makes it more difficult by requiring more manual handling of conflicts.  This is due to the "unrelated histories" between the trees.  Using subtrees can help this, with of course the tradeoff of a much messier tree.  There are two main commands to (1) create the branch of the core library, and (2) merge the branch into the app library.  You can also keep the branch around permanently (this makes the tree a lot cleaner -- good topic for a future post).  Also note that it is probably a good idea, if you choose to keep the branches around, to push them to GitHub with `git checkout core/lib; git push -u origin core/lib`.

This command will create the app library from the core library (assume for now the app directory doesn't exist - see below):

```shell
git subtree split -P core/lib -b core/lib
git subtree add --prefix=app/lib core/lib
```

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
