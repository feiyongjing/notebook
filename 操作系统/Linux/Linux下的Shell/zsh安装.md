~~~shell
# 安装zsh
sudo apt install zsh



# 安装ohmyzsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


# plugin 插件安装

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 修改zsh plugins= (git)
vim ~/.zshrc
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)

# 重启zsh 应用生效
exec zsh
~~~