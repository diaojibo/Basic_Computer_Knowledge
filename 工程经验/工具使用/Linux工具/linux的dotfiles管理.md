## linux dotfiles管理
dotfiles 这东西，翻译过来就是「点文件」，就是以.开头的文件，常见的如.bash_profile,.vimrc,.zshrc等等。在linux,osx等系统中，这些dotfiles默认都是隐藏的，在shell下要看到这些文件得用ls -a命令。像程序猿们，常用到vim, sublime text2, iterm ,zsh 等等相关的工具和开发环境，为了用得顺手，也会根据个人习惯和喜好改改不同工具、环境的配置。

在系统中使用一个文件夹，通过ln命令，将不同的app，不同的系统设置文件都指引到这个文件夹，这样就可以在这个文件夹管理所有的系统app setting了