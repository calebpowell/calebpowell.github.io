I began using distributed version control about 5 years ago, using a tool called Mercurial (aka hg). Mercurial began life at approximately the same time as Git, the development of both tools being the result of licensing changes to the BitKeeper application. From a distance, Git and Mercurial are essentially the same tool and work the same way. However, the exploding popularity of GitHub has really put git out in front. Git is the VHS of DVCS. 

One thing I miss from Mercurial were the incoming and outgoing commands, which list any commits that will be transferred via the next respective pull or push. However, thanks to Git's alias feature, I can create these commands for myself! 

To create the incoming alias, type the following at a command prompt:

```
>git config --global alias.incoming '!git remote update -p; git log ..@{u}' 
```

And to create the outgoing command:

```
>git config --global alias.outgoing 'log @{u}..'
```

Now you can run `git incoming` and `git outgoing` to see which commits will be transferred between your local and remote repositories.

Also notice that we are using the `git config`command. The git config is essentially a properties list consulted by git when you run commands. For example, when you commit code to your repository, the git command will look up your user name and email address from the config property list. There are three levels of scope in config:

local: The properties apply to a specific repository and override any global or system properties with the same name.
global: These properties belong to the logged in user. They are stored in your home directory's .gitconfig file. They override and system properties with the same name.
system: These properties are system wide and are the defaults for all users (if not overridden by the above the local or global property sets). 

