# How to _really_ understand git
git is a great tool. You can store your changes locally before deciding to push them to the central repository, you can create branches at will to keep several different pieces of work going at once, you can even stash work you don't want to commit just yet.
_However_, if you don't truly understand how the system works, it's equally easy to put your repository into a state that seems totally unrecoverable. Not only that, but you're missing out on a large part of the _true_ power of git.

The primary focus of this document will be how **branches** really work, because once you are confident fiddling with branches, you can do everything from quickly recovering from a wayward `git reset --hard` to splicing a single feature branch into multiple independent changesets.

# Under the hood
To really hammer things home, we're going to go through creating a new, empty repository and adding a few simple files, but instead of looking at what _commands_ we need, we're going to be focussing on what changes those commands produce in the mysterious hidden `.git` folder that you'll find at the root of every git repository. For this guide I'll be using the terminal commands, but I'll note the equivalent Visual Studio Team Explorer operation in brackets.

## _BEGIN!_

The first step of any git repo is the `git init` command (the VS equivalent is creating a solution and ticking 'Add to Source Control' or selecting the same option from the File menu for an existing solution). Our repo is just going to be in a 'Test' folder so we run `git init Test` and get back:

```
$ git init Test
Initialized empty Git repository in //Mac/Home/Desktop/Test/.git/
```

Alright then, let's have a look at what's in that .../Test/.git/ folder:
![.git folder contents](https://i.imgur.com/QqBhqAU.png)

Note that because git was originally developed for managing Linux development, it doesn't use file extensions for its files. Windows doesn't know what to do with them so for some reason thinks Excel will be the best tool for the job. Go figure. ¯\\\_(ツ)_/¯

Let's go through these items one-by-one.

#### hooks folder

A lot of the time you don't need to worry about the hooks folder at all, it's more useful on servers than development stations. Inside you typically find a bunch of files called `pre-commit.sample` and `post-update.sample` which, as the filenames suggest, are samples for scripts that are run when various git events happen.

![hooks folder contents](https://i.imgur.com/J2UfFqf.png)

Say you want to push all changes to a backup repository when your main git server receives an update from a developer (i.e. when the developer pushes, or when a pull request is completed), you can add a `post-receive` file that includes a [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) script with the line `git push <backup repo name>` and that script will be run after every update.
Have a look at the [git-scm page for hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) for more details on what each hook does and what you can do with them.

#### info folder

This folder is a bit boring and normally you can just ignore it completely. The only file it contains at the moment is the `exclude` file, which works like `.gitignore`, allowing you to tell git to ignore certain file/directory patterns, but because it's inside the `.git` folder instead of in the repo itself, it **isn't** version-controlled and **won't** be transferred to others if you push your repository. This might seem useless but it can be handy if, for example, you use a different IDE to the rest of your team and want to exclude the working files your IDE adds to the repository.

#### objects folder

This is where a lot of the magic happens. Everything that is committed to a git repo is stored in here. It's a very important folder, but we'll look at it in more detail later.

#### refs folder

And this is where most of the rest of the magic happens! All of your branches and stashes are listed in this folder, but again, we'll look at this more closely later.

#### config file

The config file is another _super_ important item, but isn't nearly as interesting to us as `objects` or `refs`. This stores the repository config options, like whether git should follow symbolic links that are in the repository, whether filenames should be case-sensitive, and whether this is a 'bare' repository (a bare repo is one that _only_ has the .git folder, it's used on servers where you don't care about checking out a branch, you just want all the data there to serve to others).

#### description file

This file is really rather boring, all it does is give a description of the repo. If you're using some repository browsers like gitweb, it'll show the content of the description file alongside the repository name:

![example description usage](https://i.imgur.com/Uey6du5.png)

#### HEAD file

This magical little file tells git what you're currently looking at in the repo. If you've ever wondered why your current branch is always labelled 'HEAD', this is why. Currently, the contents of the HEAD file is `ref: refs/heads/master`, meaning we're pointed at the `master` branch.

### Quick Recap

Just to make sure everything's sunk in so far, the important bits are:

* There's a hidden `.git` folder at the root level of _every git repository_.
* The `objects` folder stores all the actual data (like source code, folder structures, commits, etc.) that git tracks.
* The `refs` folder stores all our branches.
* The `HEAD` file tells git what branch or commit we're current looking at.

## Getting some work done

Now that we've had a look at what's in an empty repository, let's start doing some work. To keep things simple, I'm just going to add a simple text file to the root of the repository. It'll be called `Test File.txt` and will contain the text `This is a test file.`:

![test file 1](https://i.imgur.com/Gf5M2qL.png)

Currently, git doesn't care about this file, if we run `git status`, it'll tell us so:

```
$ git status
On branch master
No commits yet
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Test File.txt
nothing added to commit but untracked files present (use "git add" to track)
```

'Untracked' files are files that git hasn't been told to include, but also hasn't been told to _exclude_. It won't do anything with them, but it'll still let you know they're there, just in case they're something you _do_ want and you'd just forgotten.

So far, **nothing has changed in the .git folder**, so let's add it to the repository proper using `git add "Test File.txt"`! If we re-run `git status` now, we'll see that git is tracking it:

```
$ git status
On branch master
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   Test File.txt
```

_More importantly_, if we check in the `.git/objects` folder, we'll see a new `af` folder, and inside that there's a file called `27ff4986a7bdb5c5150d972123bd50febb5267`! If we open that file up, we'll find our lovely new-

![test file 1 obj](https://i.imgur.com/3IQgQ5I.png)

**Hmm, that's not right.**

Of course, the problem is that git doesn't just store all the files in plaintext. Git uses SHA1 hashes and zlib compression to help store its objects. If we run the contents of the file through a zlib decompressor, we get:

```
blob 20This is a test file.
```

The structure of this mess is as follows: The kind of object (in this case a 'blob' of data), then a space, then the size of the data in bytes ('This is a test file.' is 20 bytes long), then a 'null' character, then the data itself.
If we run _that_ through a SHA1 hash generator, we get:

```
af27ff4986a7bdb5c5150d972123bd50febb5267
```

Which looks mighty familiar! For ease of organisation and storage, git strips off the first two characters from the hash and uses that as the subfolder name inside the `.git/objects` folder, and then uses the rest of the hash as the object's filename!

> Side note: Instead of doing all the decompressing and hashing manually, git has a couple of utility scripts that can be used to pack and unpack objects. `git hash-object <filename>` will return the hash of a file and `git cat-file -p <full/partial hash>` will print the de-mangled contents of a matching file in the objects subfolder. I'll be using the outputs of these commands from now on instead of using the manual method.

### Quick Recap

In this section, we:

* Initialised a new repository. git created the .git folder for us.
* Created a simple text file in the respository. git ignored it until we asked it to track the file
* Told git to track the new file. git compressed the file's new contents, copied the result to the .git/objects folder and gave it a name by SHA1 hashing the contents together with some metadata.

## Committing fully to the cause

So, we have a file, but it's still not _really_ version controlled yet. For that, we need to commit our changes and lock in that version of the file. We've already added our test file to git's index using `git add`, so let's commit it using `git commit`:

```
$ git commit -m 'Test commit'
[master (root-commit) b8d6ddc] Test commit
 1 file changed, 1 insertion(+)
 create mode 100644 Test File.txt
```

Excellent! If we now check our objects folder we have some new subfolders:

![objects folder post-first-commit](https://i.imgur.com/7HkAk2D.png)

We have two new objects, one called `ef12cc04f8efb7d5f085af423e22487e6c6750` in the /74 folder and the other called `d6ddc8efdd9ff97822ef2b3360ec5fac77c0d4` in the b8/ folder. Very snappy.

### Commit files

If we take a closer look at the commit message and the second file, we'll notice something interesting; the commit message includes a bit of hexadecimal (`b8d6ddc`) which matches the start of the second file's hash (remember the 2-character folder name is actually the start of the hash). If we take a look at this file using the `git cat-file -p` command mentioned in the side note above, we get:

```
tree 74ef12cc04f8efb7d5f085af423e22487e6c6750
author Ashley Clewes <ashleyclewes@deltek.com> 1523458524 +0100
committer Ashley Clewes <ashleyclewes@deltek.com> 1523458524 +0100

Test commit
```

Turns out `b8/d6ddc8efdd9ff97822ef2b3360ec5fac77c0d4` is a commit file. Let's break the contents down line-by-line:

* `tree` - The hash of the commit's 'root tree', we'll come back to that line shortly, but notice that the tree hash matches the full hash of the _other_ new file!
* `author` - The author of the code in this commit, their email address, the UNIX timestamp of the commit and the author's time zone relative to UTC.
* `committer` - *Generally* this is identical to author. There are some circumstances where they can be different but they're rare in modern git usage.
* `Test commit` - Look familiar? This is our commit message!

Commit files also normally contain one or more `parent` lines referencing the previous commit(s). This particular commit has none because it's the 'initial commit' and there's nothing to be its parent. *Merge commits* are another special case that have multiple parents since they're merging multiple sets of changes together. I'll point out these cases when we get to them.

Everything we need to know about the commit is in this file except the list of files included in the commit, which is the job of the tree file. Speaking of...

### Tree files

Running our cat-file command on `74ef12cc04f8efb7d5f085af423e22487e6c6750` spits out:

```
100644 blob af27ff4986a7bdb5c5150d972123bd50febb5267    Test File.txt
```

This tree only has one item in it (our test file), but some tree files will have many, many items. Let's break this down like we did with the commit file:

* `100644` - The 'mode' of the item. Don't worry about this too much, it's related to UNIX file permissions and tells git whether the item is a normal file, an executable file, a symbolic link, etc.
* `blob` - The type of the item. Since af27ff... is a file (our test file), its type is a blob. You can also get _other trees_ being referenced here for subfolders (we'll cover trees within trees in the next section).
* `af27ff4986a7bdb5c5150d972123bd50febb5267` - The hash of the object the item describes (you should recognise this as the hash of our test file).
* `Test File.txt` - The name of the object, in this case the filename (the af27ff.. file only includes the contents, not the filename, remember).

### Quick Recap

In this section we committed the test file we made to our repository and had a look at the objects that operation created.

The important bits are:

* Commit files store:
  * The root tree hash
  * Zero (initial commit), one (normal commit) or more (merge commit) parent hashes
  * The authors' and committers' names, email addresses, timestamps and time zone offsets.
  * The commit message
* Tree files store the mode, type, hash and name of each item in the tree.

## ~~Turtles~~ _Trees_ all the way down

## <Some joke juxtaposing Grandmaster Flash's 'White Lines' with the word 'Rebase'>