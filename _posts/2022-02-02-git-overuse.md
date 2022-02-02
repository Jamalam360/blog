## Do we overuse Git features?

Hey. Today, in my first real post on this site, I feel like talking about my view of Git.

Git is a wonderful thing, it's perfect for what I use it for, but do I use it 'for the sake of using it'? I'm specifically talking about the use of branches, and for an example I'm going to use [Pinguino](https://github.com/JamCoreDiscord/Pinguino). Pinguino is a Discord bot I maintain that uses Git to manage its code (obviously).

Now, when I'm updating the README, for example, I make a new Git branch for that. I open up my terminal and do this:

```shell
git switch -c fix/update-readme
// Edit the README
git add README.md
git commit -m "fix: typo in README"
git switch develop
git merge fix/update-readme
git branch -d fix/update-readme
git push origin develop
// And deploy it to the release branch
git switch release
git merge develop
git push origin release
```

Now this is an _absolutely_ ridiculous workflow. I'm just updating a typo - the actual fix takes me seconds. I have got into this habit, though, and I'm not sure if I'll ever get rid of it. It slows me down significantly. The thing is, it sometimes saves me. Sometimes I'll be adding a new command and it won't work out so I'll just scrap it - very simple if I'm working in a branch (I can just delete the branch and be back where I started). But fixing a typo in a README has never personally caused my projects unrecoverable errors, the kind I would need to delete the branch for.

So is this a good habit? On one hand, it's probably good to always make sure you can _swiftly and immediately destroy your mistakes with one command_. It's also good to know that you can always just delete the branch and be back where you started. On the other hand, it's costing me precious time (which I don't have a lot of at the moment (why am I writing a blog?)) and I'm not sure if it's worth it.

That's about the end of that rant! Goodbye for now!