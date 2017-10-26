---
title: BetterGitCommits for Visual Studio Code
tags:
  - typescript
  - vscode
  - git
date: 2017-09-28 23:00
---

I sometimes find myself using Visual Studio Code, thanks to its excellent support for .NET Core, JavaScript, and TypeScript, as well as other niceties - I actually write most of my blog posts in it thanks to its _excellent_ Markdown support, including syntax highlighting fenced code blocks. While I generally like it, I find the built-in git support a bit lackluster. It _works_, but there are some things I don't like about it.

I spent a year or so using emacs heavily for JavasScript development, and in the process, became a _huge_ fan of magit -- I think it's by far the best interface to git that anyone's made. As a side effect, I got used to certain ways that magit works. One of the things I particularly like about it is commit message editing - when you're entering a commit message into magit, it gives you a full size buffer to use for that. Additionally, it'll highlight if your commit title (line one) goes beyond about 50 characters and automatically wraps the rest of the commit message at 72 character lines.

Needless to say, Visual Studio Code's little textbox for editing commit messages isn't really the same. Maybe this should go without saying, but I do want to note that this shouldn't be read as a slam against or criticism of the VS Code team - they've done an amazing job making an Electron app feel fast and the release cadence they work on is _quite_ impressive. I just happen to disagree with the choice they made here, and that's cool. That's why the editor is extensible!

On that note... I've created [Better Git Commits](https://marketplace.visualstudio.com/items?itemName=davidedmiston.bettergitcommits) for Visual Studio Code. Once you have it installed, stage some files and hit `Cmd-Shift-P` and run the `Better Commits: Show Git Commit Editor` command. Enter a commit message, hit `Cmd-Shift-P` again, and run `Better Commits: Run Git Commit` to commit the file. I didn't set up any shortcuts for these since I don't want to stomp on anyone's existing ones, but you can bind them to whatever you like.

It's a _very_ simple extension right now, and a bit barebones. I have some improvements I plan to make as time allows:

* Move the temporary COMMIT_EDITMSG file into the workspace .git directory rather than using node's `os.tempDir` functionality.
* Display a brief summary of which files are staged as a comment in the commit message.
* Rewrap the commit message when it hits 72 characters. Right now I'm using the [Rewrap](https://marketplace.visualstudio.com/items?itemName=stkb.rewrap) extension to do this.

If you're reading this and a VS Code user, I hope you find this as useful as I do - I'd love if you gave it a try and [tweet at me](http://twitter.com/edmistond) if you have any other suggestions for improvements. You can also [find the code on GitHub](https://github.com/edmistond/vscode-better-git-commits) if you'd like to take a look.

This was my first foray into TypeScript and I found it pretty pleasant to work with overall.