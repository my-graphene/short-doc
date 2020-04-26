# mygraphene链交易所对接指南

此文目的是协助第三方交易所上线 RUI 交易。

## 1. 基本概念

### 1.1 共识

RUI 使用 DPOS 共识机制，由持有 RUI 的人投票产生区块见证人，标准区块间隔时间是 3 秒。

### 1.2 账户

1) RUI里，资金是存在账户里的，不像比特币是存在地址里。对交易所来说，需要公开一个账号供用户充值。
   可以使用网页钱包或者轻钱包注册新账号。
   不是所有账号都可以免费注册，一般带横杠的或者数字的账号可以免费注册，比如 my-exchange ，或者myexchange2017 。
   在轻钱包账户页面里，账号下面会显示一个数字，这个数字是该账号在 RUI 系统里的内置ID，下面会用到。
   为了资金安全，交易所可以用一个账号负责充值，另用一个账号负责提现。

2) 用户充值，就是从其他账号转账到交易所的公开的账号。
   账号名就是收款地址。
   每笔转账可以带一个备注，交易所通过这个备注来区分是哪个用户的充值。
   具体备注与交易所用户关联关系，请交易所自行设定。
   备注是加密的，只有拥有发送者或者接收者的备注密钥才可以解密。

3) 用户提现，就是从交易所账号转账到用户账号，目的账号名由用户提供。
   由于有用户需要把资金直接从一个交易所转到另一个交易所，而另一个交易所是根据备注入账，所以提现功能最好可以带备注。

4) 使用网页钱包注册的账号是基本账号。可付费升级为终身会员账号，升级后，后续交易手续费节省 80% 。
   终身会员可以创建新账号。
   当前手续费费率标准可以在钱包内查看，从界面依次点击 浏览-费率表 进入。

5) 每个账号默认有3对密钥，可以在账户-高级设定-权限页面里查看，分别为：活跃权限、账户权限、备注密钥。
   其中，活跃权限秘钥用于转账等日常操作；账户权限密钥用于修改密钥；备注密钥用来加密和解密转账备注。
   默认情况下，活跃权限密钥与备注密钥相同，但可以修改为不同。

### 1.3 资产

1) RUI 系统里有多种资产，其中，核心资产是 RUI 。交易所上线 RUI 系统内其他资产的方法，与上线核心资产（RUI）的方式类似。

2) 每个账户可以同时拥有多种资产。

## 2. 基础软硬件需求

配置 | 规格
:--- | :---
独立服务器或者VPS |
内存 | 16G及以上
硬盘 | 500G及以上
操作系统 | 64 位 Ubuntu 16.04 LTS (其它暂不支持)

## 3. 程序准备

要对接 RUI 系统，推荐方案需要运行这几个程序：

1. 普通节点 witness_node
2. 延迟节点 delayed_node
3. 命令行钱包 cli_wallet

### 3.1 架构说明

- witness_node
  witness_node 通过 P2P 方式连接到 RUI 网络，从网络接收最新区块，向网络广播本地签署的交易包；witness_node 通过 websocket + http rpc 的方式提供 API 供其他程序调用（以下称为节点 API）。

- delayed_node
  delayed_node 通过 websocket 方式连接到 witness_node ，只包含不可回退的区块；
  delayed_node 通常情况下最新区块比 witness_node 落后一分钟，异常时可能会落后很多，但可保证不可回退。
  delayed_node 通过 websocket + http rpc 的方式提供 API 供其他程序调用，API清单与 witness_node 相同，但无法使用交易广播功能。

- cli_wallet
  cli_wallet 通过 websocket 方式连接到 witness_node 和 delayed_node 其中之一。
  可以同时运行两个 cli_wallet 进程，分别连到 witness_node 和 delayed_node 。
  cli_wallet 管理钱包文件，钱包文件里包含经过加密的用户私钥，一个钱包文件可以包含多个私钥。
  cli_wallet 提供交易签名功能，签名后通过 witness_node 向外广播。
  cli_wallet 通过 http rpc 的方式提供 API 供其他程序调用（以下称为钱包 API）。

- 推荐
  推荐交易所使用一个连接到 delayed_node 的 cli_wallet 来监测用户充值，使用另一个连接到 witness_node 的 cli_wallet 来处理用户提现请求。

> 如果不使用 delayed_node ，则需要自行判断交易是否可以回退（以下会说明推荐的判断方法）。

### 3.3 编译安装

#### 3.3.1 安装 clang 4.0

```bash
sudo apt update && apt -y install software-properties-common
sudo apt-add-repository -y "deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-4.0 main" && apt update
sudo apt -y install clang-4.0 lldb-4.0 libclang-4.0-dev wget make python-dev libbz2-dev libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool git ntp doxygen libc++-dev cmake  g++ libcurl4-openssl-dev unixodbc-dev libjsoncpp-dev uuid-dev build-essential vim ntp
sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test && apt update
sudo apt -y install libstdc++-7-dev
```

#### 3.3.2 安装cmake

```bash
mkdir -p /opt/cmake
mkdir -p ~/software && cd ~/software
wget https://cmake.org/files/v3.11/cmake-3.11.0-Linux-x86_64.sh
chmod +x cmake-3.11.0-Linux-x86_64.sh
bash cmake-3.11.0-Linux-x86_64.sh --prefix=/opt/cmake --skip-license
sudo ln -sfT /opt/cmake/bin/cmake /usr/local/bin/cmake
```

#### 3.3.3 安装 boost 1.67

```bash
mkdir -p ~/software && cd ~/software
wget https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.gz -O  boost_1_67_0.tar.gz
tar -zxvf boost_1_67_0.tar.gz && cd boost_1_67_0 && chmod +x bootstrap.sh
./bootstrap.sh --prefix=/usr
sudo ./b2 --buildtype=complete install
```

#### 3.3.4 安装 llvm with wasm

```bash
mkdir -p ~/software && cd ~/software
mkdir  wasm-compiler && cd wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ~/software/wasm-compiler/llvm
mkdir -p build && cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/opt/wasm -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ..
sudo make -j4 install
```

#### 3.3.5 编译

```bash
mkdir -p ~/software && cd ~/software
git clone https://github.com/my-graphene/core
cd core
git submodule update --init --recursive


echo "" >> ~/.bashrc
echo "export WASM_ROOT=/opt/wasm" >> ~/.bashrc
echo "export C_COMPILER=clang-4.0" >> ~/.bashrc
echo "export CXX_COMPILER=clang++-4.0" >> ~/.bashrc
echo "" >> ~/.bashrc

source ~/.bashrc

mkdir -p build &&  cd build
cmake -DWASM_ROOT=${WASM_ROOT} -DCMAKE_CXX_COMPILER="${CXX_COMPILER}" -DCMAKE_C_COMPILER="${C_COMPILER}" -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_BUILD_TYPE=Debug ..
sudo make -j4 install
```

## 5. 同步数据

由于需要同时运行多个程序， Ubuntu 下推荐在 Screen 或者 tmux 里启动程序。

以下描述主要针对 Ubuntu ，所以命令前都带 ./ 。对于 Windows ，在命令行界面 cd 到程序目录之后，执行时不需要 ./ 。

### 5.1 witness_node

可使用 ./witness_node --help 来查看命令参数。

#### 5.1.1 初次执行

./witness_node -d witness_node_data_dir

然后按 Ctrl+C 结束它。

这样会在当前目录生成一个数据目录 witness_node_data_dir ，里面包含 blockchain 目录是数据存储，以及一个 config.ini 配置文件。

对于交易所，推荐对 config.ini 配置文件作一些修改。

1. 可关闭 p2p 日志，以减小硬盘存储压力，方法是找到 filename=logs/p2p/p2p.log 行，在行首添加 # 号
   或者将 [logger.p2p] 下面的 level=info 修改为 level=error
2. 可考虑将控制台日志同时保存到文件，方法是将下述章节

[logger.default]
level=info
appenders=stderr

修改为

[log.file_appender.default]
filename=logs/default/default.log

[logger.default]
level=info
appenders=stderr,default

这样之后， witness_node_data_dir/logs/default/ 目录下会同步保留最近24小时的控制台日志。

3. 以下两个参数会大量减少运行需要的内存，原理是不保存与交易所账户无关的历史数据索引。

track-account = "1.2.12345"
partial-operations = true

请将 12345 替换成轻钱包中看到的账户数字 ID 。数字前的 "1.2." 表示类型是账户。

如果需要监控多个账户，则使用多个 track-account 配置，如：

track-account = "1.2.12345"
track-account = "1.2.12346"
partial-operations = true

注：
目前存在一个 BUG ，配置多个 track-account 会导致下面的日志修改不生效。
绕过这个问题的方法，是不动 config.ini ，而是启动 witness_node 的时候，在命令行后面添加 --track-account 参数，比如：

./witness_node --track-account "\"1.2.12345\"" --track-account "\"1.2.12346\""

> 1. 参数首尾双引号需要保留，所以使用 \ 进行转义。 Linux 下可以使用双引号外加一层单引号的方式，则不需要转义。
> 2. 如果需要增加、修改、删除追踪账号，修改后，需要重建索引才能生效。
  方法是按 Ctrl + C 结束程序，然后加 --replay-blockchain 参数重新启动，如：

  ./witness_node -d witness_node_data_dir --replay-blockchain

#### 5.1.2 重新执行

再次启动 witness_node ，开始同步数据。根据网络条件、服务器硬件条件不同，初次同步可能需要花几个小时到几天时间。

./witness_node -d witness_node_data_dir --rpc-endpoint 127.0.0.1:8090 --track-account "\"1.2.12345\"" --track-account "\"1.2.12346\"" --partial-operations true

上述命令中，使用 --rpc-endpoint 开启节点 API 服务，这样就可以使用 cli_wallet 和其他程序连接使用

## 5.2 连接到 witness_node 的 cli_wallet

./cli_wallet -w witness_wallet.json -s ws://127.0.0.1:8090 -H 127.0.0.1:8091

上述命令使用 -w 参数指定钱包文件， -s 参数连接到 witness_node ， -H 参数开启钱包 API 服务

执行成功会显示：

new >>>

首先需要为钱包设置一个密码

new >>> set_password my_password_1234

执行成功会显示：

locked >>>

然后解锁钱包

locked >>> unlock my_password_1234

解锁成功会显示：

unlocked >>>

使用 info 命令可以查看当前同步情况

unlocked >>> info
info
{
  "head_block_num": 17249870,
  "head_block_id": "0107364e2bf1c4ed1331ece4ad7824271e563fbb",
  "head_block_age": "23 seconds old",
  "next_maintenance_time": "31 minutes in the future",
  "chain_id": "4018d7844c78f6a6c41c6a552b898022310fc5dec06da467ee7905a8dad512c8",
  "participation": "96.87500000000000000",
  ...
}

### 5.3 delayed_node

初次执行：

./delayed_node -d delayed_node_data_dir --trusted-node ws://127.0.0.1:8090

然后按 Ctrl+C 结束它。

参考 witness_node 章节修改 delayed_node_data_dir 目录下的 config.ini 文件，然后重新启动

./delayed_node -d delayed_node_data_dir --trusted-node ws://127.0.0.1:8090 --rpc-endpoint 127.0.0.1:8092  --track-account "\"1.2.12345\"" --track-account "\"1.2.12346\"" --partial-operations true -s "0.0.0.0:0" --p2p-endpoint "0.0.0.0:0" --seed-nodes "[]"

上述命令中， 最后的3个参数 -s --p2p-endpoint --seed-nodes 的目的是防止节点直接与 p2p 网络建立连接，导致数据不可信。

注：
delayed_node 初次同步时，控制台输出会很多，可以采取重定向到文件然后查看文件的方式来检查同步情况

### 5.4 连接到 delayed_node 的 cli_wallet

./cli_wallet -w delayed_wallet.json -s ws://127.0.0.1:8092 -H 127.0.0.1:8093

请参考前面的章节设置密码及解锁。

## 6. 账户设置

考虑到安全性，可以使用两个账号分别处理充值和提现，这里假设 deposit-account 用于充值，withdrawal-account 用于提现。

### 6.1 修改充值账户的备注密钥

在上述任何一个 cli_wallet 中执行 suggest_brain_key ，会得到一对密钥，示例如下：

unlocked >>> suggest_brain_key
suggest_brain_key
{
  "brain_priv_key": ".....",
  "wif_priv_key": "5JxyJx2KyDmAx5kpkMthWEpqGjzpwtGtEJigSMz5XE1AtrQaZXu",
  "pub_key": "RUI69uKRvM8dAPn8En4SCi2nMTHKXt1rWrohFbwaPwv2rAbT3XFzf"
}

在轻钱包里，账户权限页面，将备注密钥修改为上述结果中的 pub_key

注：修改过后请注意备份轻钱包，否则轻钱包里可能无法解密修改前或者修改后的备注。

### 6.2 将充值账户的备注秘钥导入到连接到 delayed_node 的 cli_wallet

如果钱包已锁定，需要先用 unlock 命令解锁。

这里需用到上述 suggest_brain_key 结果中的 wif_priv_key ：

unlocked >>> import_key deposit-account 5JxyJx2KyDmAx5kpkMthWEpqGjzpwtGtEJigSMz5XE1AtrQaZXu

导入时 cli_wallet 会自动生成一个备份文件，可删除。

这时可按 Ctrl + D 退出钱包，对钱包文件 delayed_wallet.json 进行备份。

退出时会报个错，不用管它。

如果编译时没有引入 readline 库，则需要用 Ctrl + C 退出

### 6.3 从轻钱包中取得提现账户的活动密钥

参考 <http://RUIabc.org/article-761-1.html>

### 6.4 将提现账户的活动密钥导入到连接到 witness_node 的 cli_wallet

unlocked >>> import_key withdrawal-account 5xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

同样可以对钱包文件作个备份。

## 7. 钱包命令

- 使用 help 命令可以列出命令清单及参数
- 如果编译时有 doxygen ，使用 gethelp 命令可以获取具体命令的参数说明及示例，如

unlocked >>> gethelp get_account

## 8. 钱包 API

钱包开启了 http rpc 方式的 API 服务时，可以通过 http rpc 方式调用所有钱包命令，效果与在钱包里输入命令相同。

示例：

```bash
curl -d '{"jsonrpc": "2.0", "method": "get_block", "params": [1], "id": 1}' http://127.0.0.1:8093/rpc
```

即：method 传入命令名，params 数组传入参数清单。

返回：

{"id":1,"result":{"previous":"0000000000000000000000000000000000000000","timestamp":"2015-10-13T14:12:24","witness":"1.6.8","transaction_merkle_root":"0000000000000000000000000000000000000000","extensions":[],"witness_signature":"1f53542bb60f1f7a653bac70d6b1613e73b9adc952031e30e591e601dd60d493ba5c9a832e155ff0c40ea1dd53512e9f93bf65a8191497ea67d701bc2502f93af7","transactions":[],"block_id":"00000001b656820f72f6b28cda811778632d4998","signing_key":"RUI6ZQEFsPmG6jWspNDdZHkehNmPpG7gkSHkphmRZQWaJ2LrcaVSi","transaction_ids":[]}}

如果执行成功，结果会有 result ，否则会有 error 。

> 在钱包里输入命令，返回结果是经过美化的；使用 http rpc 请求时，返回的是 json 格式的原始数据。关于原始数据，需要注意的有：

- 金额是 {"amount":467116432,"asset_id":"1.3.0"} 格式，其中
  - asset_id 可以通过 get_asset 命令查到具体名称， RUI 的 asset_id 是 1.3.0 ，其他资产有其他 id
  - amount 是数量去掉小数点之后的值，比如 RUI 是 5 位小数，上面例子中实际是 4671.16432 RUI
- 账户是 1.2.xxxxx 的格式，可以通过 get_account 获取账户信息
- 操作类型（op）是数值格式，比如 0 表示转账操作

## 9. 处理充值

使用 get_relative_account_history 命令来查询充值账户历史，检测是否有新的充值。如：

```bash
unlocked >>> get_relative_account_history deposit-account 1 100 100

unlocked >>> get_relative_account_history deposit-account 101 100 200

curl -d '{"jsonrpc": "2.0", "method": "get_relative_account_history", "params": ["deposit-account",1,100,100], "id": 1}' http://127.0.0.1:8093/rpc
```

四个参数分别为：账户名，最小编号，最大返回数量，最大编号。编号从 1 开始。

> 目前输入最大数量超过 100 时有个bug，导致结果不准，使用时请避免 limit 超过 100 。

返回结果是一个数组。

- 如果没有新充值，则数组长度为 0 。
- 如果有新的记录，其中第N条数据为 result[N]，格式可能为：

{  
   "memo":"",
   "description":"Transfer 1 RUI from a to b -- Unlock wallet to see memo.   (Fee: 0.22941 RUI)",
   "op":{  
      "id":"1.11.1234567",
      "op":[  
         0,
         {  
            "fee":{  
               "amount":22941,
               "asset_id":"1.3.0"
            },
            "from":"1.2.12345",
            "to":"1.2.45678",
            "amount":{  
               "amount":100000,
               "asset_id":"1.3.0"
            },
            "memo":{  
               "from":"RUI7NLcZJzqq3mvKfcqoN52ainajDckyMp5SYRgzicfbHD6u587ib",
               "to":"RUI7SakKqZ8HamkTr7FdPn9qYxYmtSh2QzFNn49CiFAkdFAvQVMg6",
               "nonce":"5333758904325274680",
               "message":"0b809fa8169453422343434366514a153981ea"
            },
            "extensions":[  
            ]
         }
      ],
      "result":[  
         0,
         {  
         }
      ],
      "block_num":1234567,
      "trx_in_block":7,
      "op_in_trx":0,
      "virtual_op":1234
   }
}

其中 result[N]["op"]["op"] 是数组格式，取数组里第一个元素 result[N]["op"]["op"][0] ，如果是 0 ，则表示转账；
则可以取第二个元素中 "to" 字段，即 result[N]["op"]["op"][1]["to"] ，判断它是否与 deposit-account 的 ID 相同，来判断是否转入；
如果是，则取第二个字段中 "amount" 字段里 "asset_id" 字段 result[N]["op"]["op"][1]["amount"]["asset_id"] 判断是否正确资产类型，
  然后取 "amount" 里面的 "amount" ，即 result[N]["op"]["op"][1]["amount"]["amount"] ，加上小数点位数，得出充值金额；
  取最外层的 "memo" 字段，即 result[N]["memo"] ，得出用户在交易所的 ID ，进行入账。
result[N]["op"]["id"] 是这笔转账的唯一 ID ，可以记录备查。也可以使用 block_num + trx_in_block + op_in_trx 组合记录。

> 1. 钱包必须先解锁才能解密备注。
> 2. 如果检测到有充值的备注不正确，或者资产类型不正确，注意不要简单退回，因为可能是从其他交易所转来的，退回后对方处理起来也会很麻烦。
> 3. 如果不使用 delayed_node ，而只使用一个 witness_node ，需要自行判断交易是否会被回退。
   可参考下面“提现”章节的网络状态检查，再加上一个较大的确认数来判断，比如需要 50 个确认。

## 10. 处理提现

### 10.1 网络状态检查

为了安全起见，只有当 witness_node 网络正常时，才处理提现。

在连接到 witness_node 的 cli_wallet 中使用 info 命令检查网络状态。

unlocked >>> info
info
{
  "head_block_num": 17249870,
  "head_block_id": "0107364e2bf1c4ed1331ece4ad7824271e563fbb",
  "head_block_age": "23 seconds old",
  "next_maintenance_time": "31 minutes in the future",
  "chain_id": "4018d7844c78f6a6c41c6a552b898022310fc5dec06da467ee7905a8dad512c8",
  "participation": "96.87500000000000000",
  ...
}

需检查的字段有：

- head_block_age 最好是在 1 分钟以内
- participation 最好在 80 以上，表示 witness_node 所连网络有 80% 的出块节点正常工作

另外，网络正常时，在连接到 delayed_node 的 cli_wallet 中查看 info 时，
看到的 head_block_num 与连接到 witness_node 的 cli_wallet 中看到的不会相差太大（一般 30 以内）；
同时 head_block_age 也不会太久远（一般 1 分钟左右）。
这个可以作为参考。

### 10.2 提现账户余额检查

使用 list_account_balances 命令检查提现账户余额是否足够（注意资产类型、并且算上手续费）

unlocked >>> list_account_balances withdrawal-account

> 1. 注意资产类型
> 2. 注意加上手续费。因为备注是按长度收费，所以带备注时手续费会比不带备注时高一些。

### 10.3 发送提现

使用 transfer2 命令发送提现交易。如：

unlocked >>> transfer2 withdrawal-account to-account 100 RUI "some memo"

参数分别是：源账户名，目的账户名，金额，币种，备注

该命令会广播交易，然后返回一个数组，第一个元素是交易 id ，第二个元素是详细交易内容

> 1. 如果币种是 RUI ，金额小数位数最多 5 位。如果是其他资产，通过 get_asset 命令可以查看资产的小数位数，"precision"字段。
> 2. 也可以使用 transfer 命令，但是这样不会直接返回交易 ID ，而是需要调用其他 API 来计算出来，所以不推荐。

### 10.4 提现结果检查

在连接到 delayed_node 的 cli_wallet 中进行结果检查。

用 get_relative_account_history 命令获取 withdrawal-account 的提现历史，如果发现有新的记录，表示交易已经进块并且不可回退；

根据该条记录的 block_num 字段，使用 get_block 命令查询详情

unlocked >>> get_block 12345

返回结果中， transaction_ids 字段数据里应该包含前面的交易 id 。

### 10.5 关于失败重发

在有些情况下可能交易发送后，没有及时被打包进入区块。

与比特币不同， RUI 的交易里面有个超时时间字段（expiration），
使用 cli_wallet 签名广播交易时，该字段值默认是本机系统时间加 2 分钟。
本机交易特别多的时候，超时时间会加长。

如果在网络时间到达该超时时间之后，交易仍然没有被打包进块，则该交易会被所有网络节点丢弃，不再有可能被打包。

因此，如果出现交易广播了但没有被打包，先检查本机系统时间是否滞后。

此外，如果发现 delayed_node 里一直没有该笔提现，

- 如果 delayed_node 最新块时间已经过了交易的超时时间，那么重发是安全的。
- 如果还没超时，先检查 witness_node 里该笔交易是否已经被打包。
  - 如果 witness_node 里也没有，则继续耐心等超时。
  - 如果 witness_node 里有，那么请耐心等待该笔出现在 delayed_node ，或者从 witness_node 里消失。
    - 如果等待一段时间（比如几分钟）后交易仍然没有出现在 delayed_node ，也没从 witness_node 里消失，则很可能 witness_node 进入了一个较短分叉链，或者网络出现问题，导致交易无法完全确认。

## 11. 其他

- cli_wallet 有个参数 --daemon ，使用此参数启动后，会在后台运行

- 需要关闭 witness_node 时，按一次 Ctrl C，然后等待程序自己退出。
  - 正常退出后，重新启动时，不需重建索引，启动会比较快
  - 正常退出后，可以对数据目录 witness_node_data_dir 打包备份，需要时可直接恢复使用
  - 如果异常退出，则重新启动时，很可能需要重建索引，启动比较慢

- 如果 witness_node 出现异常，一般先尝试带 --replay-blockchain 参数重启，即手工触发重建索引
  - 如果没有解决，则使用备份恢复
  - 如果没有备份，则重新同步，可能耗时较长

- delayed_node 同上。

- 多重签名： RUI 原生支持账户级多重签名，并且有提案-批准的机制，可以在线发起多签请求，然后确认完成多签交易，具体参考相关教程

- 硬件钱包： 暂无支持

- 冷存储： 可以实现，步骤有些复杂，示例：
  - 在离线机器，启动 witness_node 及 cli_wallet ，使用 suggest_brain_key 命令离线生成密钥对；
  - 然后用轻钱包将账户密钥修改为上述密钥，则账户进入冷存状态
  - 需要动用冷存账户时，
    - 可以用暂时变热的方式，即将私钥导入轻钱包使用，用完后再换成新密钥
    - 纯冷模式也可实现，但当前 cli_wallet 支持不好，有需要的请单独联系

## 12. 相关资料

- 图文教程 <http://jc.RUIabc.org/>
  - 自建节点教程 <http://RUIabc.org/article-477-1.html>
  - 获取账户私钥 <http://RUIabc.org/article-761-1.html>
- 英文对接文档 <http://docs.bitshares.org/integration/exchanges/step-by-step.html>
- 英文 API 文档 <http://docs.bitshares.org/api/index.html>
