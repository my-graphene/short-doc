
# 节点搭建

## 1. 启动节点

```bash
mkdir -p /data/node && cd /data/node
witness_node --create-genesis-json=my-genesis.json
vim my-genesis.json
```

```json
"maintenance_interval": 600,
"cashback_vesting_period_seconds": 60,
```

```bash
witness_node --genesis-json my-genesis.json --seed-nodes "[]"

vim witness_node_data_dir/config.ini
cp config.ini witness_node_data_dir/
```

```ini
p2p-endpoint = 0.0.0.0:31010
seed-nodes = []
rpc-endpoint = 0.0.0.0:38090
genesis-json = my-genesis.json
enable-stale-production = true

# ID of witness controlled by this node (e.g. "1.6.5", quotes are required, may specify multiple times)
witness-id = "1.6.1"
witness-id = "1.6.2"
witness-id = "1.6.3"
witness-id = "1.6.4"
witness-id = "1.6.5"
witness-id = "1.6.6"
witness-id = "1.6.7"
witness-id = "1.6.8"
witness-id = "1.6.9"
witness-id = "1.6.10"
witness-id = "1.6.11"
```

```bash
witness_node
nohup witness_node &
tail -f nohup.out
```

## 2. 创建wallet服务

```bash
mkdir -p /data/wallet && cd /data/wallet

cli_wallet -w wallet.json -s ws://0.0.0.0:38090 -r 127.0.0.1:38091 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d

set_password supersecret
unlock supersecret
import_key nathan 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3

import_balance nathan ["5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"] true
upgrade_account nathan true
```

## 3. 重启cli_wallet

```bash
cli_wallet -w wallet.json -s ws://0.0.0.0:38090 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d

unlock supersecret

get_account nathan

> membership_expiration_date的属性值应该是1969-12-31T23:59:59

list_account_balances nathan
```

```ini
genesis-json = my-genesis.json
p2p-endpoint = 0.0.0.0:31010
seed-nodes = ["39.105.115.75:31010"]
rpc-endpoint = 0.0.0.0:38090
```

```bash
nohup cli_wallet -w wallet.json -s ws://0.0.0.0:38090 -r 127.0.0.1:38091 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d -d &

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "unlock", "params": ["supersecret"], "id": 1}' http://127.0.0.1:38091

tail -f nohup.out

pkill node ; pkill ruby ; pkill witness_node ; pkill cli_wallet
rm witness_node_data_dir/ nohup.out -rf
sleep 1
witness_node --create-genesis-json=my-genesis.json
witness_node --genesis-json my-genesis.json --seed-nodes "[]"

cp config.ini witness_node_data_dir/
nohup witness_node &
tail -f nohup.out

cli_wallet -w wallet.json -s ws://0.0.0.0:38090 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
```
