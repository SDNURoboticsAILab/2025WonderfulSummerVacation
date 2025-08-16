## 环境配置

首先安装好环境包

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install build-essential cmake git vim python3 python3-pip gdb -y
sudo apt-get install -y build-essential git python3 automake autoconf libtool flex bison libboost-all-dev
sudo apt-get install -y build-essential qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools libfontconfig1-dev libaudio-dev libxt-dev libglib2.0-dev libxi-dev libxrender-dev
```



https://github.com/rcsoccersim/rcssserver

https://github.com/rcsoccersim/rcssmonitor

https://github.com/helios-base/librcsc

https://github.com/helios-base/soccerwindow2

https://github.com/helios-base/fedit2
```bash
可以clone以上项目选合适版本进行安装，分别输入以下指令进行编译配置环境（具体指令根据项目readme决定）
mkdir build
cd build
cmake ..
make

或者
./configure
make
sudo make install
```



同样可以通过clone下方项目进行环境一键配置，同时如果需要特定版本，把对应版本的文件放到assets中，进行重新配置

```bash
git clone https://gitee.com/juzaizai/robocup2d-environment.git
cd robocup2d-environment
pip3 install -r requirements.txt
python3 install.py
```





### Cyrus2DBase

```bash
git clone https://github.com/Cyrus2D/Cyrus2DBase.git
mkdir build
cd build
cmake ..
make
```

之后运行`./start.sh`球员上场，之后ctrl+k开始比赛。

也许编译过程中有部分报错，基本交给ai改一下代码即可

