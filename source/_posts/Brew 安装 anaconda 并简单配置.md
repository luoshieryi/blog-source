---
title: Brew 安装 anaconda
date: 2023-07-05
tags: [mac, zsh, homebrew, python]
---

# Brew 安装 anaconda

```shell
# 直接安装 anaconda
brew install anaconda
# zsh 配置环境变量
echo 'export PATH="/opt/homebrew/anaconda3/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
# conda 初始化
conda init zsh
```

使用 bash 则将相应的文件/命令换为 bash(.bash_profile)

# anaconda 配置 python 环境

```shell
# 查看 conda 版本
conda --version
# 创建环境
conda create -n $env_name python=$python_version $dependencies
# 列出当前环境
conda env list
# 切换环境
conda activate $env_name
# 删除环境
conda remove -n $env_name --all
# 列出安装的包
conda list
# 更新所有包
conda update --all
#卸载包
conda uninstall $package_name
