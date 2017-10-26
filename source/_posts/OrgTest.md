---
title: Org Mode Rendering Test
date: 2017-10-03 20:00
tags: 
  - hexo
  - org-mode
---

I generally enjoy using Emacs, but have been mainly using Visual Studio Code for blogging since it supports syntax highlighting in fenced markdown code blocks without making me jump through annoying hoops. That said, I find certain editor tasks easier when I'm using Emacs, particularly with my evil-mode keybindings and extensions, not to mention being able to use magit for managing the repository. Accordingly, trying to figure out a way to get this working in Emacs has been something I'd been poking at off and on.

There's [mmm-mode](https://github.com/purcell/mmm-mode) but I found it more than a bit flaky and difficult to use.

After doing some digging, I found there's an org-mode renderer for Hexo, the [blog engine I use](https://www.davidedmiston.com/2017/03/01/Hello-Hexo/) to generate this site. There's also a [really helpful post](https://coldnew.github.io/hexo-org-example/2017/03/05/getting-started-with-hexo-and-org-mode/) on getting everything set up, and that's what I used for the model. Works great, correctly renders and highlights syntax blocks in org-mode, and lets me use the keybindings and git client I prefer... so I guess I'll try punching out a few posts and see how I like working with it. The only downside I've found is I can't seem to specify a filename for source code blocks, but it's a limitation I can live with for now.

**Update**: I ended up ripping this out, for various reasons. Mostly that it's easier to get things formatted the way I want in Markdown using Visual Studio Code, and the Markdown fenced code blocks functionality in Hexo allows you to specify filenames for a code block as well. Sometimes it's the little things. :)