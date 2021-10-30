# Git_From_A_Git
Training for Advanced Git Maneuvers


Overall I'd like to convince you that using Git is valuable and that it is worth spending some extra effort to keep a clean project history.
In the process I hope to help you use it a little more powerfully.
This is mostly targeted at people who have been using git for a while but aren't actually taking advantage the history.
I'll try to explain basic concepts as we come across them but this is not meant to be a total beginners tutorial. 

Most of this exercise is done through the command line, because that is the interface I use the most. I started out using the Git GUI and it helped me get my feet wet, but it also simplified things in a way that soon became limiting. When I always had to turn to the terminal to fix the issues I'd created through the GUI I soon just dropped using the GUI and have just used the command line ever since. You may be using some other interface, a GUI or context menu one. These are fine and hopefully you can figure out how to do all of these commands, but I can't really help much.

## Table of Contents
1. The Power of History
    1. git diff
    2. git show
    3. git revert
    4. git cherry-pick
    5. git blame
    6. git bisect
    7. gitk
2. How to Curate Your History
    1. git add -p
    4. git branch
    7. git reset
    2. git rebase
    3. git mergetool
    5. git cherry-pick
    6. git checkout -p
    7. Pre-Commit Hook

# The Power of History

*"He who doesn't know history is destined to repeat it"* --Some dead dude.

Many of us use git routinely or as a requirement from our workplace but fail to appreciate the full power git provides.
Git is extremely powerful and versitile, to the point that you will find many different tribal opinions about best practices and how to use git "correctly."
I can't really claim any authority on the methods I'm describing in this document, other than I've been using git for over a decade of coding professionally and in personal projects and have found them useful.
Like any good tool with some amount of complexity, the best way to use it will usually be situational.

In this section I'm going to describe several git commands and how I use them to help me get work done faster and better.
To follow along, clone the repository at https://github.com/ssj71/Git_From_A_Git_Exercises and open a terminal in that directory.
As prerequisites, you'll need git of course, but also gitk and a merge tool like kdiff3.
If you are not able to do that, then you probably aren't quite ready for this tutorial, and should focus more on basic git useage first.
Or you could just google it and press forward.
I'd probably do that but that doesn't mean it's a good idea.

## git diff

The first command to cover is the lowly `git diff`. Maybe it's not lowly, but it's fairly basic.
It shows you all the current changes in files that git is keeping track of that haven't yet been committed or added to the current commit. 
Ugh, that sounds pretty academic.
Git diff really shows the essense of how git works.
I wouldn't say I have full comprehension of how git does stuff, but a basic understanding of the diff is pretty powerful, so let's step back for a moment to go over the basic principles here.

At its core a diff is just a list of changes. It could be changes to a single file, a group of files, directory structures or whatever part of the project you are tracking, but it is ONLY what changed.
However (at least from a project perspective) every file in a project starts from nothing
The inital commit on that file, where it goes from nothing to something is just a single big diff.
Sometimes it's a small diff I guess, but it's just one is the point.
It may end there, but not typically.
Typically your file needs to change as it is developed, bugs are found, or requirements change.
When you change it, that's making a new diff.
So theoretically you can describe a file at any point in history completely as a sequence of diffs.

So git is basically keeping a "list" of diffs in a sequence and that is the history of the file.

So what size is a diff?
Every keystroke?
Every change between when the file is written to disk?
Every change between releases?
Every change between winter olympic games?

Technically any of these can be true.
Git provides the power to choose the size of your diffs by only saving the ones added to *commits*.
A commit is simply a diff (a.k.a. a set of changes) that is saved and has a bit of meta data saved with it.
We'll get to this a bit more later.

Ok, lets focus on diff before I get too further into some rabbit hole of theory.
For a diff to exist there has to be some change.
In your editor of preference, open the file *features* from the exercises repository you cloned.
You will see this project is amazing.
All you have to do to add features is write a line with the feature number in it.
Go ahead and add a feature, make it a cool one, like, "feature 42"...
Oh yeah.
Nice work.
Now save the file and go back to our terminal and run the command
```
git diff
```

Look! There it is!
Your diff!

You can see that first it lists the file name, then some line numbers, some lines from the file for context, and finally the actual change.
The change can be lines removed or deletions (indicated by a "-" in front of the line) or lines added or insertions, which you see in the current diff with the "+" preceding the line.

That's how git does it: one line at a time.
Even if you change exactly one character in the line, the whole line is marked as the old line removed and the new line added in the diff.
Let's try it!
In your editor go change "feature 2" to be "feature 22" just for fun (don't worry this project won't break if features are't numerically ordered).
Save it and then run git diff again.
Now you see 1 line removed and 2 added, even though the one + and - are the same line.

Diffs can cover any number of lines and any number of files.
I'd argue that their usefulness is inversely proportional to their size, but as long as you can grok what it's showing you it works.
This command is useful to refresh your memory of what you've changed when you've made several changes and broken something, or need to make a commit.

The diff command can be used for more than just showing what is currently changed on the disk, you can use it to show an aggregated diff from multiple commits, different branches, even different copies of the repository.

So hopefully we understand diffs and some more fudamentals of how git works, lets move forward.
Before we come to the next command we want to "save" this state of the files in history as a commit.
We shouldn't work on the main branch of any project with multiple devs so we'll make a new branch called "work" and commit to it.
So let's just run
```
git checkout -b work
git commit -a -m "my first commit (in this project)!"
```

You'll see there that the commit command outputs the number of changes and some other data that we'll discuss more later.

Now that we've made a commit we're ready for the next command to discuss.


## git show

This command shows a commit. Run it now
```
git show
```

By default it shows the most recent commit.
It looks a lot like the diff we had before commiting. 
Wait a minute, it *IS* the diff we had before commiting!
Only now it has some meta data and is actually recorded as part of our history.

The meta data includes the commit hash, which is a long, unique hexadecimal number identifier for each commit.
This hash is used in a lot of the commands we'll be using later, so it's useful to make a note what to look for.
Usually you only need the first 6 or so numbers in the hash (sometimes referred to as a commit-ish).

The other data include the author, date, and a commit message which should describe what the diff is or does.
There are other data that aren't displayed by the `show` command though there are ways to see them.
One that we'll mention is the parent(s) of the commit.
When I said that git history is basically just a list of diffs, that was a bit disengenuous because the history can be nonlinear.
More on that later.

The command `git show` has a lot of options but I don't use them (yet).
So we're going to move on to the next command.

## git log

This simply squishes down the history to a linear list of commits and shows the hash, date author and commit message for each.
Try it out.
```
git log
```

There's our commit! Aww, cute.
I use the log very often to remind myself what was going on in a branch or where I left it. You can run it on different branches without needing to switch over too. Try it on the branch called "ten"
```
git log ten
```

There you see that I added feature 10 on that branch, but otherwise it's the same as the main.
Sometimes if you are looking for something deeper in the history it can be nice to use `git log --oneline` for a more compact list with less info.
But enough about `git log`.
It's pretty straightforward.

## git revert

Revert is to undo a commit.
It pretty much takes the diff, reverses the insertions and deletions and the applies that diff.
As long as you haven't changed lines effected by the diff after the commit you are trying to revert it works cleanly.
If you have changed lines then you'll have merge conflicts but we aren't ready to address those.

First, why revert? Why not just make a new commit that undoes the changes that need to be done?
Well, first, I'm lazy!
It's much easier to just revert one than to track down every change in the editor and make sure I do it right.
Next reason is because it has a different message in your history.
It shows that you tried something and it didn't work, but you want to remember that you tried it.
It helps indicate that you needed to undo it, rather than just hiding that fact in 

Let's try it now.
You worked hard on that feature 42 but management decided they don't want it.
So just call
```git revert```

and it will make a new commit.
You can use the message make a good comment about them that they'll (probably) never look at or just leave it at the boring default.
Now look in the log and you see that your commit is still there and now there's a new revert commit.

The `revert` command can be convenient but it only works when your commit only has exactly the diff you want to revert. If you added some comments, fixed a typo, ran your code beautifier on a couple files, AND did that bugfix that caused a regression all in the same commit, you either have to wholesale undo those changes, or you have to do it some other way than `git revert`.

That may not seem like a huge cost.
Reverting is pretty well in the category of "one way to do it."
It might not be your style.
That's ok, but I hope some of these other ideas will help you realize how important clean commits are.
So lets go to the next command.

## git cherry-pick

This command takes a single commit and applies it to the current code, rather than the parent it had when the commit was made.
This command is extremely useful when you have a bunch of commits and important changes in a branch that just isn't ready yet. Or someone else has a change you need on their branch and you don't want to wait for them to merge everything in.

This is another case for having nice clean commits. The more independent you can make your commits, with one idea per commit the more useful this will be.

Pretend that our next feature assignment is really dependent on feature 14.
Another developer implemented that on the branch `teen_features`.
So let's give it a look.
```
git log teen_features
```

Ugh. There's a bunch of stuff there.
I don't even know what all that does.
We just need 14.
We could merge that branch into ours, but it has all those other commits adding things that maybe aren't yet working or complete.

So we'll cherry-pick only the commit we do want.
```
git cherry-pick 80280a
```

You see we only need to provide the commit-ish, not the whole hash, and if two commits match git will tell us and we can give a few more characters.
Now you can see in the log we have feature 14 added, and in the features file too.
But won't that cause problems when you both merge to master?
Well, yes it could, but we'll cover that more later.

## git blame

Ah, blame.
What a great command.
It's not JUST to identify who caused a bug (though it's very good at that).
It also makes it easy to see when each line in a file was added.
Take a look:
```
git blame features
```

It's useful to have your terminal open large for this one, though github's blame interface is arguably better than the command line.
See TODO: URL as an example
But blame tells you the commit, author and date for each line of code.
With useful commit messages, it's like having very thoroughly commented code that you only see the comments when you want to.
Nifty, oh?


Even without good commit messages, `blame` can be really helpful as an overview for the history of a file.
You can also run `git log features` for a list of the commits that changed that file, but I find seeing the lines of code associated to each commit gives me better insights.
The log is still useful though because blame doesn't show code that was removed, but that revert is still there in the log.




    3. git revert
    4. git cherry-pick
    5. git blame
    6. git bisect
    7. gitk
2. How to Curate Your History
    1. git add -p
    4. git branch
    7. git reset
    2. git rebase
    3. git mergetool
    5. git cherry-pick
    6. git checkout -p
    7. Pre-Commit Hook

    notes:
    dev A writes 3 features, then got others to help with the project.
    Dev B works on ten 10
    Dev C works on the teen_features 13-19
    Dev A continues on next_3 4-6
    Dev B finishes, makes a PR and merges to main.
    Dev A makes a PR and sees it has a conflict.
    Dev A tells Dev C about the conflicts so C merges main into her WIP.
    Dev A merges main into her branch and makes some changes to fix things.
    Dev A gets the PR for features 3-6 merged
    Dev B begins working on features 7-9, 11 and 12
    Dev C merges master again into her branch.
    Dev C finishes and merges her features into master
    Dev B finishes and merges master in her branch, then merges into master branch

    notes 2:
    dev A writes 3 features, then got others to help with the project.
    Dev B works on ten 10
    Dev C works on the teen_features 13-19
    Dev A continues on next_3 4-6
    Dev B finishes, makes a PR and merges to main.
    Dev A makes a PR and sees it has a conflict.
    Dev A tells Dev C about the conflicts so C merges main into her WIP.
    Dev A merges main into her branch and makes some changes to fix things.
    Dev A gets the PR for features 3-6 merged
    Dev B begins working on features 7-9, 11 and 12
    Dev C merges master again into her branch.
    Dev C finishes and merges her features into master
    Dev B finishes and merges master in her branch, then merges into master branch
