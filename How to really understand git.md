# How to _really_ understand git
git is a great tool. You can store your changes locally before deciding to push them to the central repository, you can create branches at will to keep several different pieces of work going at once, you can even stash work you don't want to commit just yet.
_However_, if you don't truly understand how the system works, it's equally easy to put your repository into a state that seems totally unrecoverable. Not only that, but you're missing out on a large part of the _true_ power of git.

The primary focus of this document will be how **branches** really work, because once you are confident fiddling with branches, you can do everything from quickly recovering from a wayward `git reset --hard` to splicing a single feature branch into multiple independent changesets.

# Under the hood
To really hammer things home, we're going to go through creating a new, empty repository and adding a few simple files, but instead of looking at what _commands_ we need, we're going to be focussing on what changes those commands produce in the mysterious hidden `.git` folder that you'll find at the root of every git repository.
## _BEGIN!_
The first step of any git repo is the `git init` command. Our repo is just going to be in a 'Test' folder so we run `git init Test` and get back:
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
This file is _super_ boring, all it does is give a description of the repo. If you're using some repository browsers like gitweb, it'll show the content of the description file alongside the repository name:
![example description usage](https://i.imgur.com/Uey6du5.png)
#### HEAD file
This magical little file tells git what you're currently looking at in the repo. If you've ever wondered why your current branch is always labelled 'HEAD', this is why. Currently, the contents of the HEAD file is `ref: refs/heads/master`, meaning we're pointed at the `master` branch.

## Getting some work done
Now that we've had a look at what's in an empty repository, let's start doing some work. To keep things simple, I'm just going to add a simple text file to the root of the repository. I'll be called `Test File.txt` and will contain the text `This is a test file.`:
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