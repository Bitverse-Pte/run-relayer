## 本地启两条teleport链，同时创建relayer

### 配置启动项
完成项目clone之后，直接按步骤执行

```shell
cd chainwork

# 删除旧的keys
rm -rf ~/.hermes/keys/teleport_90*

# 用hermes添加keys
hermes keys add teleport_9000-1 -n local0  -p "m/44'/60'/0'/0/0" -f local0.json
hermes keys add teleport_9001-1 -n local1  -p "m/44'/60'/0'/0/0" -f local1.json
# 查看keys
hermes keys list teleport_9000-1
hermes keys list teleport_9001-1

# 导入config
cp ../config.toml ~/.hermes/config.toml
```

### 启动

```shell
### 新开进程
cd chainwork
teleport start --home mytestnet/node0/teleport
### 新开进程
cd chainwork
teleport start --home mytestnet1/node0/teleport

### 启动ibc relayer
#### 在之前的进程中执行
#### 设置环境变量
export LOCAL0=teleport1cxaj4tdxalddrng26nxrcad2wyypkxk2z8kly2
export LOCAL1=teleport130udupx40z39nfx0pfgl3tllk4a4dyylg4vjnc

#### 发起交易
echo '1234567890\n'| teleport tx bank send node0 $LOCAL0  10tele  --chain-id teleport_9000-1  -b block -y --gas auto --keyring-backend file --keyring-dir mytestnet/node0/teleport
echo '1234567890\n'| teleport tx bank send node0 $LOCAL1  10tele  --chain-id teleport_9001-1  -b block -y --gas auto --keyring-backend file --keyring-dir mytestnet1/node0/teleport --node tcp://localhost:36657

#### 启动
hermes -c ~/.hermes/config.toml config validate
hermes create channel teleport_9000-1 teleport_9001-1 --port-a transfer --port-b transfer -o unordered
hermes start
```
新开终端即可进行正常交易


### 初始化
```shell
# 清空原先的配置内容
rm -rf chainwork/*

cd chainwork

# 初始化配置
# 密码默认输入1234567890
teleport testnet init-files --output-dir ./mytestnet --chain-id teleport_9000-1 --keyring-backend file
teleport testnet init-files --output-dir ./mytestnet1 --chain-id teleport_9001-1 --keyring-backend file

# 确认初始化之后的文件结构
(base) ➜  chainwork git:(master) ✗ tree -L 5
.
├── mytestnet
│   ├── gentxs
│   │   └── node0.json
│   └── node0
│       └── teleport
│           ├── config
│           │   ├── app.toml
│           │   ├── config.toml
│           │   ├── genesis.json
│           │   ├── node_key.json
│           │   └── priv_validator_key.json
│           ├── data
│           │   └── priv_validator_state.json
│           ├── key_seed.json
│           └── keyring-file
│               ├── 2c8f0c9a2d32ed7be30b17cbbf51280e1eef91c4.address
│               ├── keyhash
│               └── node0.info
└── mytestnet1
    ├── gentxs
    │   └── node0.json
    └── node0
        └── teleport
            ├── config
            │   ├── app.toml
            │   ├── config.toml
            │   ├── genesis.json
            │   ├── node_key.json
            │   └── priv_validator_key.json
            ├── data
            │   └── priv_validator_state.json
            ├── key_seed.json
            └── keyring-file
                ├── 281d24cc8035e5ab1e97f29eb7a885caf447dde7.address
                ├── keyhash
                └── node0.info

14 directories, 22 files

# 备份，指定行 修改 voting_period
sed -i '.bak' '229s/"voting_period": "172800s"/"voting_period": "30s"/' mytestnet/node0/teleport/config/genesis.json
sed -i '.bak' '229s/"voting_period": "172800s"/"voting_period": "30s"/' mytestnet1/node0/teleport/config/genesis.json

# 继续修改配置
sed -i '.bak' '15s/proxy_app = "tcp:\/\/127.0.0.1:26658"/proxy_app = "tcp:\/\/127.0.0.1:36658"/' mytestnet1/node0/teleport/config/config.toml
sed -i '' '91s/laddr = "tcp:\/\/0.0.0.0:26657"/laddr = "tcp:\/\/0.0.0.0:36657"/' mytestnet1/node0/teleport/config/config.toml
sed -i '' '115s/address = "tcp:\/\/0.0.0.0:1317"/address = "tcp:\/\/0.0.0.0:2317"/' mytestnet1/node0/teleport/config/app.toml
sed -i '' '202s/laddr = "tcp:\/\/0.0.0.0:26656"/laddr = "tcp:\/\/0.0.0.0:36656"/' mytestnet1/node0/teleport/config/config.toml

sed -i '.bak' '166s/address = "0.0.0.0:9090"/address = "0.0.0.0:9190"/' mytestnet1/node0/teleport/config/app.toml
sed -i '' '179s/address = "0.0.0.0:9091"/address = "0.0.0.0:9191"/' mytestnet1/node0/teleport/config/app.toml
sed -i '' '223s/address = "0.0.0.0:8545"/address = "0.0.0.0:8645"/' mytestnet1/node0/teleport/config/app.toml
sed -i '' '226s/ws-address = "0.0.0.0:8546"/ws-address = "0.0.0.0:8646"/' mytestnet1/node0/teleport/config/app.toml
```

#### keys的处理
```shell
# 查看已有的 key
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet/node0/teleport
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet1/node0/teleport

# 查看现有的助记词
cat mytestnet/node0/teleport/key_seed.json
cat mytestnet1/node0/teleport/key_seed.json

# 用现有的助记词添加 keys
# echo '1234567890\n'| teleport keys add local0 -i --output json --keyring-backend file --keyring-dir mytestnet/node0/teleport > local0.json
teleport keys add local0 -i --output json --keyring-backend file --keyring-dir mytestnet/node0/teleport > local0.json
# 现有助记词
# happy damp dignity drum radio gain win decade mesh lady foam shift wrist dust loyal outside pet fantasy square attend emotion section kidney split

# echo '1234567890\n'| teleport keys add local1 -i --output json --keyring-backend file --keyring-dir mytestnet1/node0/teleport > local1.json
teleport keys add local1 -i --output json --keyring-backend file --keyring-dir mytestnet1/node0/teleport > local1.json
# 现有助记词
# good provide auction recipe document inquiry elite indoor ethics other used brush adapt source hold pipe mango cat original load window major shy cable

# 查看添加后的 keys
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet/node0/teleport
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet1/node0/teleport
```

#### 删除旧的key
```shell
teleport keys delete local0 --keyring-backend file --keyring-dir mytestnet/node0/teleport -y
teleport keys delete local1 --keyring-backend file --keyring-dir mytestnet1/node0/teleport -y
```

#### 添加新的keys
如果不想使用现有的助记词，可以使用下述命令添加新的keys
```shell
# 新增key
echo '1234567890\n'| teleport keys add local0 --output json --keyring-backend file --keyring-dir mytestnet/node0/teleport > local0.json
echo '1234567890\n'| teleport keys add local1 --output json --keyring-backend file --keyring-dir mytestnet1/node0/teleport > local1.json

# 查看添加后的 keys
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet/node0/teleport
echo '1234567890\n'| teleport keys list --keyring-backend file --keyring-dir mytestnet1/node0/teleport
```

至此已完成配置，请移步”配置启动项“，执行启动！