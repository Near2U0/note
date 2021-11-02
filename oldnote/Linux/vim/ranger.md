直接修改系统默认文本编辑器，ranger就会跟着变了

```shell
vim
echo export EDITOR=/usr/bin/vim >> ~/.bashrc
echo export EDITOR=/usr/bin/vim >> ~/.zshrc


nvim
echo export EDITOR=/usr/bin/nvim >> ~/.bashrc
echo export EDITOR=/usr/bin/nvim >> ~/.zshrc

```
然后重启终端就能生效了



原文链接：https://blog.csdn.net/weixin_43372529/article/details/112242335
