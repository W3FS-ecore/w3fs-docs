# W3FS私网单节点部署(without-heimdall)

本节将指导您在二进制文件上启动和运行单节点(不含桥接服务)。

测试网建议最低硬件配置：

- 内存：16-32GB

- CPU：2-8-core

- 存储：1TB SSD（确保它是可扩展的）

  

## 先决条件

- 需要一台机器(ubuntu)。

  示例：

  ```
  ubuntu 20.04
  ```

- 机器上安装StorageLib相关依赖。

- 机器上安装Go 1.17。

- 准备一个账号和对应私钥和公钥地址。

  注：为了确保部署正常，请按照准备好的示例来部署(heimdall创世文件需要关联，如果换账号，对应的heimdall的创世文件需要对应做修改)。

  示例：

  ```
  账号：0x9fB29AAc15b9A4B7F17c3385939b007540f4d791
  私钥：9b28f36fbd67381120752d6172ecdcf10e06ab2d9a1367aac00cdcd6ac7855d3
  ```


## 概述

> 请务必按照以下顺序执行，如果这些步骤不按顺序执行，您将遇到配置问题。

- 准备好一台机器(ubuntu)。
- 安装构建要素。
- 安装二进制文件。
- 证明文件下载。
- 获取创世文件。
- 设置节点文件。
- 设置服务。
- 设置所有者和签名者密钥。
- 启动服务。
- 部署存储等相关合约。
- 注册矿工地址信息。
- 部署DAPP相关合约。

## 安装构建要素

```
#安装基础依赖
sudo apt update -y
sudo apt upgrade -y

sudo apt-get -y install docker.io jq curl git build-essential mesa-opencl-icd hwloc libhwloc-dev ocl-icd-opencl-dev pkg-config

sudo apt upgrade -y
#检查libhwloc是否安装成功
ll /usr/lib/x86_64-linux-gnu/libhwloc.so

#sudo ln -s /usr/lib/x86_64-linux-gnu/libhwloc.so /usr/lib/x86_64-linux-gnu/libhwloc.so.5

#rustup安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain stable -y

export PATH=$PATH:$HOME/.cargo/bin
source "$HOME/.cargo/env"
rustup -V

#其他
sudo apt install -y python3-pip nodejs npm
sudo npm -g install n
sudo n 12
hash -r
node -v

sudo pip3 install solc-select
sudo solc-select install 0.6.6
sudo solc-select use 0.6.6
solc --version

sudo npm install -g truffle@5.1.48 --unsafe-perm=true --allow-root
#检查truffle是否正常安装
truffle version
```

注意：如果是在中国有时候安装一些依赖包，网速慢，直接超时，可以指定中国对应的源镜像

```
#如果是在中国可配置对应的源
mv /etc/apt/sources.list /etc/apt/sources.list.bak && 
cat >> /etc/apt/sources.list <<EOF
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

EOF
```

### 安装 GO

```
wget https://gist.githubusercontent.com/ssandeep/a6c7197811c83c71e5fead841bab396c/raw/go-install.sh
bash go-install.sh
sudo ln -nfs ~/.go/bin/go /usr/bin/go
go version
```

> 注意：建议使用 Go 版本 1.17

## 安装二进制文件

安装最新版本的w3fs和heimdall相关二进制文件

```
cd ~/
git clone https://github.com/W3FS-ecore/w3fs-binaries.git
cd w3fs-binaries
git lfs pull
chmod +x ~/w3fs-binaries/bin/*
sudo ln -nfs ~/w3fs-binaries/bin/w3fs /usr/bin/w3fs
sudo ln -nfs ~/w3fs-binaries/bin/bootnode /usr/bin/bootnode
sudo ln -nfs ~/w3fs-binaries/bin/heimdallcli /usr/bin/heimdallcli
sudo ln -nfs ~/w3fs-binaries/bin/w3fs-fetch /usr/bin/w3fs-fetch
sudo ln -nfs ~/w3fs-binaries/bin/w3fs-worker /usr/bin/w3fs-worker
sudo cp ~/w3fs-binaries/sdk/libTSKLinux.so /lib/x86_64-linux-gnu/libTSKLinux.so
```

验证w3fs二进制文件是否正常：

```
w3fs version
```

验证heimdallcli二进制文件是否正常：

```
heimdallcli --help
```

验证相关二进制文件是否正常：

```
w3fs-fetch --help
w3fs-worker --help
w3fs-bench --help
w3fs --help
```

注意：如果是在中国有时候安装一些依赖包，网速慢，直接超时，可以指定中国对应的源镜像

```
#更改cargo对应的中国源镜像
cat >/root/.cargo/config << EOF 
[source.crates-io]
replace-with = 'tuna'

[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
EOF

#go代理加速
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

## 证明文件下载

```
w3fs-fetch fetch-params 512MiB
```

注意：如果出现下载错误，有可能是因为网络问题，重复执行以上命令即可。



注意：如果是在中国，网速慢，可设置以下代理进行加速

```
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
```

## 获取创世文件

```
git clone https://github.com/W3FS-ecore/w3fs-genesis-contracts.git
cd w3fs-genesis-contracts
```

修改validators.js对应的信息，参考如下：

```
const validators = [
        { address: "0x9fB29AAc15b9A4B7F17c3385939b007540f4d791", stake: "10000", balance: "1000000", }
]
exports = module.exports = validators
```

其中：

address表示桥接矿工地址

stake表示桥接对应的矿工质押金额

balance表示桥接矿工的余额



修改miner_validators.js对应的存储矿工信息，参考如下：

```
const storageMiners = [
    {
        signer: "0x9fB29AAc15b9A4B7F17c3385939b007540f4d791",
        stakeMount: 20000000000000000000000,
        storageSize: 1000000000,
        balance: 1000000
    }
];
exports = module.exports = storageMiners
```

其中：

signer表示矿工地址。

stakeMount表示存储矿工初始存储算力。

storageSize表示矿工初始质押算力，存储矿工承诺存储空间， 单位 KB。

balance表示矿工的余额



生成创世文件

>bash generate.sh [chaind-id] [heimdall-id]

```
bash generate.sh 15678 heimdall-15678
```

注意：如果是在中国有时候安装一些依赖包，网速慢，直接超时，可以指定中国对应的代理进行加速

```
git config --global url."https://github.com/".insteadOf git@github.com:
git config --global url."https://".insteadOf git://
```

## 设置节点文件

### 获取启动存储库

```
cd ~
git clone https://github.com/W3FS-ecore/w3fs-launch.git
```

### 设置启动目录

```
#创建node目录
mkdir -p ~/node
cp -rf w3fs-launch/localnet/without-sentry/* ~/node/
cp -f w3fs-launch/localnet/service.sh ~/node
cp w3fs-launch/scripts/ganache-start.sh ~/node

#将生成的w3fs创世文件拷贝到对应的w3fs目录下
cp -f ~/genesis-contracts/genesis.json ~/node/w3fs/
```

#### 设置W3FS

```
cd ~/node/w3fs
bash setup.sh
```

## 设置服务

```
cd ~/node
bash service-withoutheimdall.sh
sudo cp *.service /etc/systemd/system/
```

## 设置所有者和签名者密钥

### 生成一个W3FS keystore文件

#### 生成 W3FS 密钥库文件

> heimdallcli generate-keystore ETHEREUM_PRIVATE_KEY

- ETHEREUM_PRIVATE_KEY — your Ethereum private key.

```
heimdallcli generate-keystore 9b28f36fbd67381120752d6172ecdcf10e06ab2d9a1367aac00cdcd6ac7855d3

#会提示输入2次密码，例如：password

#将生成的keystore文件移到w3fs配置目录下
#mv ./UTC-<time>-<address> ~/.w3fs/keystore/
mv ./UTC-* ~/.w3fs/keystore/
```

#### 添加password.txt文件

```
#echo "同上输入的密码" >~/.w3fs/password.txt
echo "password" >~/.w3fs/password.txt
```

#### 添加以太坊地址

编辑vi /etc/W3FS/metadata，在metadata中，添加该节点对应的以太坊地址。

```
sudo mkdir -p /etc/w3fs
sudo chmod -R 777 /etc/w3fs/
echo "VALIDATOR_ADDRESS=0x9fB29AAc15b9A4B7F17c3385939b007540f4d791" >/etc/w3fs/metadata
```

示例：

```
VALIDATOR_ADDRESS=0x9fB29AAc15b9A4B7F17c3385939b007540f4d791
```

## 启动服务

#### 启动W3FS服务

```
sudo service w3fs start
```

查看W3FS日志：

```
journalctl -u w3fs.service -f
```

检查服务进程是否正常：

```
ps -ef|grep -v grep |grep w3fs
```

- 示例：

  ```
  root@ubuntu:~# ps -ef|grep -v grep |grep w3fs
  root      123328       1  0 09:19 ?        00:00:00 /bin/bash /root/node/w3fs/start.sh 0x9fB29AAc15b9A4B7F17c3385939b007540f4d791
  root      123358  123328  3 09:19 ?        00:13:17 w3fs --datadir /root/.w3fs/data --port 30303 --ws --ws.addr 0.0.0.0 --ws.port 8546 --nat=extip:192.168.54.19 --http --http.addr 0.0.0.0 --http.vhosts * --http.corsdomain * --http.port 8545 --ipcpath /root/.w3fs/data/w3fs.ipc --http.api eth,net,web3,txpool,bor,admin,mine,personal,w3fs,storage --syncmode full --networkid 15678 --miner.gaslimit 20000000 --miner.gastarget 20000000 --txpool.accountslots 16 --txpool.globalslots 131072 --txpool.accountqueue 64 --txpool.globalqueue 131072 --txpool.lifetime 1h30m0s --keystore /root/.w3fs/keystore --unlock 0x9fB29AAc15b9A4B7F17c3385939b007540f4d791 --password /root/.w3fs/password.txt --allow-insecure-unlock --maxpeers 200 --metrics --pprof --pprof.port 7071 --pprof.addr 0.0.0.0 --mine
  ```

检查出块是否正常：

```
w3fs attach http://localhost:8545 --exec "eth.blockNumber"
```

## 部署存储和质押相关合约

```
cd ~/genesis-contracts
```

部署合约

```
npm run migrate
```

## 注册矿工地址信息

参数1：UTC文件对应的密码

参数2：验证者对应的网络地址，需要用公网IP，端口默认30308，如无调整，请勿修改为其他值。

参数3：验证者对应的哨兵(proxy)对应的网络地址，要用公网IP，端口固定为8545，请确保哨兵的实际端口也为8545，否则不能被动态DNS所识别并使用。

如果没有哨兵的话，IP地址默认本机的公网地址。

```
w3fs attach http://localhost:8545 --exec "personal.setMinerInfo('password','/ip4/192.168.10.204/tcp/30308','/ip4/192.168.10.204/tcp/8545')"
```

注意：如果服务器IP或端口有变化或修改，需要重新注册矿工地址信息。

## 设置remoteworker进程

修改W3FS对应~/.w3fs/data/.w3fsminer/config.toml配置文件

vi ~/.w3fs/data/.w3fsminer/config.toml

```
[Storage]
  # env var: LOTUS_STORAGE_PARALLELFETCHLIMIT
  #ParallelFetchLimit = 10

  # env var: LOTUS_STORAGE_ALLOWADDPIECE
  #AllowAddPiece = true
  AllowAddPiece = true

  # env var: LOTUS_STORAGE_ALLOWPRECOMMIT1
  #AllowPreCommit1 = true
  AllowPreCommit1 = false
  	
  # env var: LOTUS_STORAGE_ALLOWPRECOMMIT2
  #AllowPreCommit2 = true
  AllowPreCommit2 = false

  # env var: LOTUS_STORAGE_ALLOWCOMMIT
  #AllowCommit = true
  AllowCommit = false

  # env var: LOTUS_STORAGE_ALLOWUNSEAL
  #AllowUnseal = true
  AllowUnseal = false

  # env var: LOTUS_STORAGE_RESOURCEFILTERING
  #ResourceFiltering = "hardware"
```

重启W3FS服务并获取令牌Token

> sudo service w3fs restart

待服务重启完成后，获取令牌Token

> W3FS attach http://localhost:8545 --exec "storage.authNew({perm:'admin'})" 

- 示例：

  ```
  ubuntu@ip-172-31-73-76:~/node-bash$ W3FS attach http://localhost:8545 --exec "storage.authNew({perm:'admin'})" 
  "MINER_API_INFO=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBbGxvdyI6WyJyZWFkIiwid3JpdGUiLCJzaWduIiwiYWRtaW4iXX0.6NXVdm-odIql5HmgXisxGFaKsTuBFiqHaj9f96szYg8:/ip4/127.0.0.1/tcp/2345/http"
  ```

  

修改W3FS-worker启动脚本，将MINER_API_INFO修改为如上新获取的令牌Token

vi ~/node/w3fs/worker.sh

```
#!/usr/bin/env sh
export MINER_API_INFO=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBbGxvdyI6WyJyZWFkIiwid3JpdGUiLCJzaWduIiwiYWRtaW4iXX0.6NXVdm-odIql5HmgXisxGFaKsTuBFiqHaj9f96szYg8:/ip4/127.0.0.1/tcp/2345/http
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
w3fs-worker --worker-repo="$HOME/.w3fs/data/.w3fsworker" run --addpiece=false --precommit1=true --precommit2=true --commit=true --unseal=true
```

启动remotewoker进程

> sudo service w3fs-worker start
