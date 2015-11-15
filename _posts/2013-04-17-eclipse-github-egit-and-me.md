---
title: Eclipse, GitHub, EGit, and Me
date: 2013-04-17 10:20
summary: Any method that actually connected the remote GitHub repo to my local repo also, by the nature of Eclipse, also created a new child folder with the project there, rather than in the root directory. But then I found a solution.
---

It seemed someone in the four is not allowed to be happy 98% of the time (usually me) when trying to connect them on my own or following various instructions. It seems to be a common issue. (A sampling: [1](http://stackoverflow.com/questions/6760115/importing-a-github-project-into-eclipse), [2](http://wiki.eclipse.org/EGit/User_Guide#GitHub_Tutorial), [3](http://forums.bukkit.org/threads/github-and-eclipse.40351/), [4](http://stackoverflow.com/a/14714898/1072141)) Any method that actually connected the remote GitHub repo to my local repo also, by the nature of Eclipse, created a new child folder with the project there, rather than in the root directory.

However, in the last StackOverflow link I noticed the “Repositories from GitHub” option in the answer. Searching for “GitHub” in the “Install New Software” dialog in Eclipse I found a plugin. In keeping with the difficulty the EGit process _must_ have, it can’t be installed alongside another specific git plugin. So, after removing all of my git plugins (“Help” &gt; “Install New Software” &gt; “What is already installed?”), these are the steps I took to (finally) connect Eclipse and GitHub:

* Install the GitHub software (restart)
* Install EGit (restart dos)
* “Import” &gt; “Git” : “Project from Git” &gt; “GitHub” &gt; etc<br>
![Eclipse dialog for selecting a GitHub repo](/images/2.png)
* Right-Click the project &gt; “Configure” &gt; “Convert to Faceted Form” or, in my case “Convert to Javascript Project”

And there was my README.md, making the fourth happy too.

If further details are needed, there’s a [great post](http://www.eclipse.org/forums/index.php/mv/msg/226301/706734/#msg_706734) in the Eclipse forums.
