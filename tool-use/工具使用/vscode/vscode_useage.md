[toc]

# vim-plugin

> vim folding issue

ref: https://coderedirect.com/questions/576762/code-folds-are-automatically-opened-when-cursor-moves-over-them-in-vs-code-vim

```text

It looks like this is an issue many people have had for a while, and the solution is to do the following (original source):

Open up your user settings. On windows the shortcut is CTRL + ,
Search for vim.foldfix and check the checkbox so the setting is set to true.
Alternatively, open your settings.json file by opening the command palette (CTRL + SHIFT + P), select Preferences: Open Settings (JSON), then add the following line: "vim.foldfix": true

Now the folds should no longer automatically expand when you scroll past them with j or k.

Be aware that this is a hack because of various problems with VS Code itself that make fixing this difficult.

```





# shortcut

```shell
# format code
alt+shift+F


```





# VS Code不能跳转到定义

友们推荐了一个vue-helper的插件。
