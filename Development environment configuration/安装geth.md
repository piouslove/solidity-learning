智能合约需要以太坊虚拟机以及编译环境才能运行起来，安装Geth可以提供以太坊虚拟机，安装solc可以提供solidity编译环境。
# 安装Geth
Geth是以太坊的完整客户端，它可以搭建节点加入主链，也可以实现定制化私有链，当然它提供一切钱包服务。
## 在Mac系统上安装
Mac系统上可以用Homebrew安装Geth，如果没有Homebrew，请先[安装Homebrew](https://brew.sh/)。  
安装正常版本：

    brew tap ethereum/ethereum
    brew install ethereum
使用下面这个命令可以安装开发者版本： 

    brew install ethereum --devel
不过不推荐安装开发者版本，后边我们可以依靠正常版本自己定制化私有链。
## 在Windows系统上安装
所有版本的Geth可以在 [https://geth.ethereum.org/downloads/](https://geth.ethereum.org/downloads/) 下载，请尽量安装最新版本。
## 在Linux/Unix系统上安装
### 在Ubuntu上安装
    sudo apt-get install software-properties-common
    sudo add-apt-repository -y ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install ethereum
### 在Arch上安装
    pacman -S geth
现在，Geth已经安装好了，你可以点开exe文件或者在命令行输入geth来运行它。但是我不建议你这样做，因为它会默认连接到以太坊主链并更新区块，当然，如果你不在乎自己的网络带宽或者硬盘容量，你也可以试一下。  
后面会继续介绍如何搭建如你所愿的定制化私有链。
