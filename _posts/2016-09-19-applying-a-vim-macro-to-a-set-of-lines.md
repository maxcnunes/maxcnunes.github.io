---
layout: post
title: Applying a VIM macro to a set of lines
tags:
- vim
---

<script type="text/javascript" src="https://asciinema.org/a/86341.js" id="asciicast-86341" data-size="small" async></script>

1. Start a macro typing `q` + macro name (in that case was `q` as well)
1. Record the commands. I was using `ds'` which uses the surround plugin to delete the quotes.
1. Finish the recording with `q` again.
1. Select the lines with `V`
1. Apply the macro to the select lines with `:norm! @q`

**Reference:**

* http://stackoverflow.com/questions/390174/in-vim-how-do-i-apply-a-macro-to-a-set-of-lines
* https://github.com/tpope/vim-surround
