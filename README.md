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
    3. git log
    4. git revert
    5. git cherry-pick
    6. git blame
    7. git bisect
    8. gitk
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
After you've cloned, make sure to run a `git fetch` because we'll use several different branches for these exercises.
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
You can also put diffs into a patch which you can email out and others can apply to their code.
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

Github's UI provides a link most any time you see a commit hash displayed and if you click the link you get a pretty page with basically a `git show` of that commit.
E.G. https://github.com/ssj71/Git_From_A_Git_Exercises/commit/18a11877e68c595ff34a9549a6285dec71875934#diff-5b8a8b56dada6ce7567442b4935298df7de2badd7becdcf4915a59487338ca4b
The command `git show` has a lot of options but I don't use them (yet).
So we're going to move on to the next command.


## git log

This simply squishes down the history to a linear list of commits and shows the hash, date, author and commit message for each.
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

Another great use for log is to look at the history of a specific file.
```
git log features
```

This example isn't great because it's the only file in the project, but in a real world project it's helpful to filter out all other commits and just look at when and why a particular file was changed.
If you are looking at a file on Github you can click the history button to get the same results as git log on that file.
E.G. https://github.com/ssj71/Git_From_A_Git_Exercises/commits/master/features
Sometimes that's more convenient.
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
```git revert <commit-ish>```

inserting the hash of your previous commit and it will make a new commit.
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

Pretend that our next feature assignment is really dependent on feature 13.
Another developer implemented that on the branch `teen_features`.
So let's give it a look.
```
git log teen_features
```

Ugh. There's a bunch of stuff there.
I don't even know what all that does.
We just need 13.
We could merge that branch into ours, but it has all those other commits adding things that maybe aren't yet working or complete.

So we'll cherry-pick only the commit we do want.
```
git cherry-pick 730dd45
```

You see we only need to provide the commit-ish, not the whole hash, and if two commits match git will tell us and we can give a few more characters.
Now you can see in `git log` we have feature 13 added, and in the features file too.
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

It's useful to have your terminal open large for this one, though Github's blame interface is sometimes cleaner and sometimes more convenient.
E.g. https://github.com/ssj71/Git_From_A_Git_Exercises/blame/master/features
But blame tells you the commit, author and date for each line of code.
With useful commit messages, it's like having very thoroughly commented code that you only see the comments when you want to.
Nifty, oh?


Even without good commit messages, `blame` can be really helpful as an overview for the history of a file.
I often find a situation where I'll ask, why is this line of code here?
`git blame`, oh it was added when we had to add that fix for the hardware bug (or whatever).
You can also run `git log` for the file as mentioned earlier, but I find seeing the lines of code associated to each commit gives me more granular insight on how the file changed.
The log is still useful though because blame doesn't show code that was removed, whereas the log does, such as our revert commit.
It's just another tool that serves a bit different purpose.

The catch with blame, is it only shows the most recent commit on a line.
If you have great commit messages and small contained commits, but then your last commit is "run astyle" and 50% of lines changed, blame won't be very useful.


## git bisect

Bisect is a pretty cool command.
It automates a binary search through history.
It is awesome for chasing down a regression that you can't tell when it was introduced.
You pretty much enter the last known good commit that didn't have the bug, and it checks out the middle commit between the current code (which has the bug) and the good commit you enter.
You build and test and tell git whether it worked or not.
Git then checks out another commit, and you test it again.
Pretty quickly you can figure out what commit broke it.
We won't do an example, but it's handy to know.

The rub is your commits have to compile and be testable.
When you try this and every single commit that gets checked out requires a bunch of touching up to run the test and check for the bug, it's nowhere near as convenient.
Similary a very non-linear history can make this command less useful.


## gitk

This isn't a git command, but it's a very useful tool.
It just visualizes the history as the various branches and helps you browse the commits there.
Our current branch doesn't have a very interesting history so let's look at a different branch with gitk:
```
gitk messy_master
```

Now you should see the history of this branch.
You can see here an example of a highly non-linear history; branches merging back and forth and all eventually feeding into this branch.
We'll talk about this more later.

For now, just note that gitk is handy when the log seems confusing because commits appear in a weird order or something.
Honestly I used to use `gitk` heavily, but now I tend to just use `log` and `show` for most of my needs.
For those who like GUI's it probably seems a bit more comfortable.
Github also has a decent UI to show history this way, except it visualizes every branch on the remote.
They call it the "network" and it can be accessed through the "insights" tab.
E.g. https://github.com/ssj71/Git_From_A_Git_Exercises/network
I look at this most any time I'm looking at a new repository and want to know what branch to use.
You can see easily what branches are being worked on and which are stale.

A nice linear history is always easier to digest than a highly nonlinear one.

Go ahead and close gitk and we'll move on with the second part of this tutorial.

I hope you see a few new ways now that you can use your history more to your advantage.
Even if you don't like the command line, a lot of these tools are available in a prettier fashion through Github's UI.

Ultimately the point of history is to save time and help reduce mistakes.
But many of these tools and tricks I've described had caveats about how they're only useful if you've done this or that with your history, and that is the hard part.
That is why the second half of this tutorial is focused on methods that will help keep your history clean.



# How to Curate Your History


*"History is always written by the winners"* --some other dead person

Ok, so we want to keep our history clean.
Linear.
With focused, single-purpose commits.

But the problem is that, man, I'm just so good, every time I open a file I find those bugs and I fix 'em dang it.
Plus I always leave excellent comments to make the code better while I'm working on it.
And sometimes thoese things don't fit into the feature that this branch was made to implement.
So what to do?


## git add -p

I use this command all the time.
To the degree that the only case where I `git add` without the `-p` flag if for a brand new file.
So what's the `-p` do?
The "-p" is short for `--patch` or in other words, one little diff at a time
(Note that I use diff, patch, and hunk fairly interchangeably, there are subtle differences that I gloss over).
This begins an interactive experience that goes through each file and shows different chunks of changes and you choose to either add it to the commit (called staging), or leave it out.

This is immensely powerful to control what goes into your commits.
You can fix ten things, add the first one, make the commit, then add the second one, make another commit, etc.
You could even add some commits to one branch, submit a pull request on Github, then create a new branch and add the rest of the commits there.

This gives you careful control of your commit size and contents.
This is huge!
I try to do this to make sure each commit holds a complete (compileable) thought.
One fix per commit, though often that fix spans several files.

I don't ever actually make a fix in a single contained effort, I just commit it that way.
Usually I'm making comments, fixing spelling errors, moving lines around a little, other fixes that aren't related, and a healthy amount of debugging hacks and tests.
Once the fix actually works, then I start committing.
I try to isolate each feature and add the necessary parts for it and commit them.
Usually the last one is something like "comments and cleanup."
Then by the end of these several commits I have a bunch of stuff I don't want to save.
We'll talk about what to do with that in a bit.

Truthfully though I'm usually doing `git add -Ap` which goes through all changed files with the interactive dialog.
With `git add -p` you have to tell which file to add, and usually that's too much work for me.
I almost always run `git diff` first and remind myself of the overview of everything that I changed.
This does take a bit more time and effort than just adding all your changes to a single commit, but it makes using Git much more powerful.
Sharpen your saw a little, right?

So let's give it a go. 
In your editor add a feature 0 and  add an empty line then a few features at the end after 13, whatever features you like.
I'm doing 22, 23, and 24, but you substitute in your features, they're probably better anyway.
Open the README and add a line in there too like "just for practice."
Save those and then run
```
git add -Ap
```

The first hunk to show up is the message in the README.
before we go any further, enter in "?" just to see what the options are.
There are several and most all are useful but 95% of the time, I'm just using "y" and "n."

So looking at our diff it's asking us about, I'd say that it was only put there for practice.
We don't want to add that, it should stay a local change and we don't want to sully the README on the remote.
So enter "n" for no.

Next diff, it's showing all the changes we made to the features file.
Since they're so close to each other it thinks they're related, but in this case they're totally different features.
So we want to split the hunk into smaller hunks.
Enter "s" to split it.

Ah, now we can just add feature 0 and put that into a single commit.
Enter "y" to add the hunk, and then "q" to say we're done.
We could also have just entered "n" for the rest but if you reviewed the diff before you started, you can be fairly confident that you have added everything you need for the commit.

Now we can commit it.
```
git commit -m "added feature 0"
```

MAKE SURE YOU DON'T ADD THE `-A` FLAG!
In fact, I want you to never run `git commit -A` again.
Well, ok, almost never.
If you've already made a bunch of commits and the only changes left are 100% diffs you want in the commit you can do it, but it shouldn't be a habit.
It definitely was for me and took some time to break.
But I'm very glad I did.

Let's keep going and add the other features.
Starting again
```
git add -Ap
```

Skip that change in the README again, and then we see the 3 features we added at the end.
Now we want to add each feature individually, so let's try "s".
Oh no! It didn't work! Our history is doomed!
Just kidding.
Here we will manually edit the diff to add exactly what we want and leave the rest for later commits.
Start with entering "e" for edit mode.
This should open up your default editor with the diff and some notes near the bottom explaining what to do.
We have 3 additions (each preceded with the "+") and we only want to add the first.
Therefore (according to the instructions at the bottom) delete the last 2 "+" lines, leaving only the first addition.
Now save and close that file, and finish the -p dialog (since this was the last diff it closes as soon as we're done).
Save that commit
```
git commit -m "adding feature 22"
```

Remembering to substitute your feature number if you are more creative than me.

Now if we look at our current diff we see just the last 2 features, plus our practice change in the readme.

You can repeat this process until all the changes you want to keep have been committed.

If you ever get interrupted while doing this and forget where you are (happens to me all the time), you can use `git status` to see which files you've added diffs from, or you can run `git diff --cached` and it shows the full diff of what's been added to the current commit already.

So what about that diff in the README that's lurking there threatening to accidentally get added to some otherwise pristine commit?
There are a few different things that make sense under different conditions.
If you want to save it to use later, maybe it's a useful debugging hack that you don't want to clean up yet or something unique to your setup, the easiest thing is to just make a new branch and commit it all and make sure you don't push that branch to the remote.

If you don't care about the changes, you can call `git stash` and it will take all changes from all files, leaving you with a clean working directory.
Stashing is nice because the history is saved on a stack and can be accessed again if you messed it up.
Alternatively you can just do a `git checkout <file>` on the file and it will remove all diffs, putting it back in the state it was on the last commit affecting it.
Hopefully you realize that this should be approached with some care as you can lose your work if you are doing that willy-nilly and haven't commited the changes someplace.

Let's go ahead and just dump that change that was for practice.
```
git checkout README
```

So now that we are able to fine tune our commits, let's talk a little bit more about where to put them.


## git branch

This command is one you probably use a fair bit, and we've already played with several branches in this exercise.
I just want to mention how I use branches in better ways than I did when I first started using Git.
Originally my projects would always have a main branch and a dev branch.
That was it.
Everything was on the dev branch.
That worked great because I was the only developer on them.
No matter what I was working on, it only effected me.
My commits also typically had messages such as "stuff," "everything I'm working on," and, my favorite, "oops."

Anyway, fast-forward to today when I'm always working on projects with many other developers and it has become much more important that I take care of my use of branches.
Why?
Because as useful as clean commits are, clean branches are easier.

Making separate branches for the 3 different bugs I've been working on makes it so that I can submit 2 pull requests for the 2 I've fixed already.
They are useful fixes, and merging them early allows others to benefit from them.
The 2 pull requests are easy for others to review and merge (because they are small), and now I can take the 3rd branch and continue my work on it without hauling along a ton of baggage commits to create a monster code review in a few months.

Truthfully I never make new branches with `git branch` I always just do `git checkout -b <branchname>` to make a new branch based on the current history.
Then I add and commit the code appropriate for that branch.

If you make several branches as you go through and your changes remember that the full history keeps building, so your second branch still has the history of your first, and the third branch has the history of the previous two, etc.
Sometimes this is fine, you just need to make sure they get merged in order, before the next one gets reviewed.
Sometimes though you'll want to have separate histories so that different devs can work on reviewing the code affecting their module at the same time.

Let's try that.
You've just made several commits for feature 0, 22, 23 and 24 on your branch called "work."
Now we need to go back to the main branch and create our new branches from there so they can be merged in independently.
```
git checkout master
git checkout -b feature0
```

We could have started with any branch, but we'll start with zero.
Now we cherry-pick the commit. On my "work" branch the feature 0 commit was 17bb02f.
```
git cherry-pick 17bb02f
```

Now we would push that and make a pull request, but here were just moving on to the next feature.
Let's do feature 13.
Really that other dev should've done this so that we didn't have to cherry-pick the fix from their "teen_features" branch, but they're on vacation now.
What can you do?
Make the PR for them.

```
git checkout master
git checkout -b feature13
git cherry-pick 730dd45
```

And then we'd push and make another PR.
Repeat ad nauseum.


## git reset

What do we do if we're going through carefully adding diffs to a commit, manually editing where needed to get the perfect clean commit and we accidentally type a "y" in the wrong place and add the wrong thing?
That's what `git reset` is for.
You can run it to "unstage" all the diffs that you have added.
Now you'll see them reappear in `git diff` and no longer appear in `git diff --cached`.
Often if you've really been working hard to get a clean commit, you'll want to run `git reset -p` to go through the same patch dialog but in reverse.
You can also just `git reset <file>` for unstaging everything from a single file.

Reset can also help "undo" commits.
Quite often when I want to keep a bunch of changes, but they're mingled in with other things or maybe I just need to quickly change to a new branch I will very often just run a quick `git commit -A -m "WIP"`.
This saves everything into one big messy commit, but when I come back to the branch, I habitually run a `git log` to check where I was and whenever I see "WIP" in the message, it reminds me that the commit wasn't meant to be that way.
If you run `git reset HEAD^1` then it basically removes the commit but keeps the changes in the files, so now you can `git add -p` to make more careful commits.
This is a form of rewriting history though, since you remove that WIP commit.
That means it should be done with care, and probably should only be done on local branches.
This will be discussed more later.


## git rebase

So now let's say that our counterparts reviewed and merged feature 10 to main and we want our "feature13" branch merged so we don't have to cherry-pick that one again.
If we just create a pull request now we will have... merge conflicts (play horrified scream sound here).

Merge conflicts happen when git tries to combine branches or commits and finds that the same lines changed in a different commit already in the history.
If your branch has merge conlicts with the main branch it won't be merged in.

So there are two ways to approach it.
Most developers will just merge the main branch into their working branch and figure it out, but that is what creates histories like the spaghetti we saw when we ran `gitk messy_master`.
There is another way.
A better way.
Well, except for when it's not better, but we'll get into that later.

First let's go to that branch
```
git checkout feature13
```

Now we do the rebase
```
git rebase master_ten
```

And it tells us there is a conflict, which we expected.
Our simple examples here are actually rife with merge conflicts waiting to happen because all the branches are working on the same file, usually adding lines to the same place at the end of the list.
In real-world projects, you can often have no conflicts if you are working in different parts of the code.
But eventually you're going to run into one, and if many people are working simultaneously on the same file(s) then you'll run into lots of them.
I've been hiding them with these carefully contrived examples, now we're going to stare them straight in the face.
And we're going to win.
They aren't that scary.
I promise.

So we have to resolve the conflict.
Maybe just to help us feel less afraid let's load the features file into our editor and take a peek.
You see that it shows the conflicting section in the file two times.
Once for each branch involved with the merge.
You can actually resolve the conflict manually with your editor by removing the meta lines and leaving the code as you think it should be, but that method is pretty easy to make mistakes.

There are much better tools for doing this.
We typically call them 3 way merge tools because it shows the two branches current state (or branch and the new commit being rebased) plus the last common commit in the two histories.
My goto is kdiff3, though there are many others that are just great.
The details vary but inevitably they help you go through the conflicts and you chose which source of changes to use.
Typically they label them A, B and C, and you can usually chose any combination of the 3.

Let's do it.
```
git mergetool
```

(Note if you don't have a mergetool set up, you'd better google it and come back). Here in our example we want both feature 10 and feature 13, and it's nice to keep them in order so in kdiff3 I click the B and then the C.
Once you've resolved all the conflicts you save the file and go back to the terminal.

Now you can run
```
git rebase --continue
```

Which then checks that every conflict was resolved and provides you the opportunity to adjust the commit message (which may be needed) and then makes the commit that adds feature 13.
If we had a longer branch then it would then apply the next commit check for conflicts and so on.

If you've done this sort of thing before, that seemed pretty much the same as merging.
So what's the difference?
Merging takes the two histories and puts them together at the end, so you end up with a fork in your history.
Rebasing takes the commits and pretends they were all added AFTER the history.
It's like you procrastinated until everyone else finished their stuff, but without the stressful part at the end.

While it can be a little extra work rebasing and going through the commits again, I've found that as long as I'm the author of at least one of the branches (typically I am) then it's really not very hard to make sure the code ends up the way you intended it.
And then you don't have conflicts in your PR, and your history is squeaky clean.
Also remember that you can run `git rebase --abort` if things get too hairy.
So don't be afraid to try it out without totally breaking your setup.
And you can always make a temporary backup branch before you begin rebasing

To help illustrate rebasing versus merging, imagine a situation like the following:

    Dev A writes the first 3 features in main, then gets others to help with the project.
    Dev B works on feature 10 in the branch "ten"
    Dev C works on the features 13-19 in the branch "teen_features"
    Dev A continues on features 4-6 in a new branch "next_3"
    Dev B finishes, makes a PR and the group merges branch "ten" to main.
    Dev A makes a PR for "next_3" and sees it has a conflict.
    Dev A tells Dev C about the conflicts so C merges main into her WIP on "teen_features".
    Dev A merges main into her "next_3" branch and makes some changes to fix things after the merge.
    Dev A gets the PR for features 3-6 and "next_3" gets merged
    Dev B begins working on features 7-9, 11 and 12 in a new "missing features" branch
    Dev C merges master again into her "teen_features" branch.
    Dev C finishes and merges "teen_features" into the main branch
    Dev B finishes and merges master in her "missing_features" branch to avoid conflicts in the PR
    Dev B submits the PR and "missing_features" merges into the main branch

This is exactly the process I followed to create the messy_master branch we looked at in gitk.
If you look in gitk again you can follow along.
If you check the network of some busy projects, their history looks like that; it's pretty easy to let that happen.
The alternative is rebasing onto main whenever you have merge conflicts in your PR, rather than merging main into your branch.
This makes the history look like what you see when you run
```
gitk clean_master
```

I hope it looks nicer to you too.
But it's not just about looks, it's also about being able to use things like `git bisect` and for being able to review and understand the history when problems arise.

Now it is very important to notice a few caveats about rebasing.
There are times when rebasing is absolutely the wrong thing.
If you paid attention, you noticed the hash changed in the feature 13 adding commit.
That's because it is a new commit.
If you look at `git show` the context line shown before the diff is now the "feature 10" line whereas before it was the "feature 3" line.
So rebasing is re-writing your history, removing old commits and making new ones that do the same (or similar) changes.

If you have a branch that someone else is also using and you push it after rebasing, they will try to pull and Git won't be able to recognize the history, because it was all changed!
So the rule is that you never rebase a public or shared branch.
Any time you rebase it means basically anybody who is working with that branch in their history needs to abandon it and move their work to your new branch, which may or may not be easy (can be lot's of cherry-picking).
So it's good to communicate about it.
Sometimes it works fine, maybe they are done with their changes, other times, you just can't really rebase cleanly.
If you are the only one who has ever used the branch, then you are fine.

It's usually safest to create a new branch, rebase and push it, then delete the old branch.
In fact git won't allow a regular `git push` to work anymore because of the mismatching history.


If you want more practice rebasing, try now checking out the teen_features branch (which is where we cherry-picked the feature 13 from) and rebasing it on feature13.
Remember both these histories have a commit that adds feature 13, but they are different commits.
When it tries to rebase that commit it will realize nothing changed and wonder what to do.
You can tell it to skip that commit with `git rebase --skip` when it brings it up.


## git checkout -p

So what can you do if you've already messed up and made a huge branch with tons of messy commits?
Well, you can panic.

## Pre-Commit Hook
