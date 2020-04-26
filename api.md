# api基本使用

## 1. 水龙头

`http://<ip>:3000/api/v1/accounts`

```bash
curl -H "Content-Type:application/json" -X POST --data '{"account":{"name":"fuckuser","owner_key":"RUI8ahfEujigjL8CcU26wJeqnDqGjumLneE31CShLimBAfWbcHFtS","active_key":"RUI5mJSXTdWs7qFuYTZCUJPrhiasgQatXx8LKxD6jHQXw6NL8h6rc","memo_key":"RUI5mJSXTdWs7qFuYTZCUJPrhiasgQatXx8LKxD6jHQXw6NL8h6rc","refcode":null,"referrer":null}}' http://13.124.230.82:3000/api/v1/accounts
```

```json
{
  "account":{
    "name":"fuckuser",
    "owner_key":"RUI5mJSXTdWs7qFuYTZCUJPrhiasgQatXx8LKxD6jHQXw6NL8h6rc",
    "active_key":"RUI5mJSXTdWs7qFuYTZCUJPrhiasgQatXx8LKxD6jHQXw6NL8h6rc",
    "memo_key":"RUI5x8pAYQvPtNU5TkanuZyiFcBDCCvsWydhYg99W2YZotyuX3mxu",
    "refcode":null,
    "referrer":null
  }
}
```

## 2. 链api

### 2.1 http + rpc2.0

`curl`

```bash
curl --data '{"jsonrpc": "2.0", "method": "get_accounts", "params": [["1.2.0", "1.2.1"]]}' http://13.124.230.82:38090/rpc

curl --data '{"id":1,"method":"call","params":[0,"get_account_by_name",["bishengtop-ex"]]}' http://13.124.230.82:38090/rpc
curl --data '{"id":1,"method":"call","params":[0,"get_account_by_name",["bisheng-exchange"]]}' http://13.124.230.82:38090/rpc

curl --data '{"id":1,"method":"call","params":[0,"get_account_by_name",["bishengtop-ex"]]}' http://13.124.230.82:38090/rpc
curl --data '{"id":1,"method":"call","params":[0,"get_account_by_name",["bisheng-exchange"]]}' http://13.124.230.82:38090/rpc

curl --data '{"id":1,"method":"call","params":[1,"",[]]}' http://13.124.230.82:38090/rpc

curl -H "Content-Type:application/json" -X POST --data '{"id":1,"method":"call","params":[0,"get_full_accounts",[["rui-eddie"],true]]}' http://13.124.230.82:38090/rpc


curl -H "Content-Type:application/json" -X POST --data '{"id":1,"method":"call","params":[0,"get_full_accounts",[["rui-eddie"],true]]}' http://13.124.230.82:38090/rpc


curl -H "Content-Type:application/json" -X POST --data '' http://13.124.230.82:38090/rpc
curl -H "Content-Type:application/json" -X POST --data '{"id":1,"method":"call","params":[1,"history",[]]}' http://13.124.230.82:38090/rpc


curl -H "Content-Type:application/json" -X POST --data '{"id":1,"method":"call","params":[3,"get_account_history",["1.2.26","1.11.0",10,"1.11.0"]]}' http://13.124.230.82:38090/rpc

curl -H "Content-Type:application/json" -X POST --data '{"id":1,"method":"call","params":[3,"get_assets",["1.3.0","1.3.1"]]}' http://13.124.230.82:38090/rpc
```

### 2.2 websocket

`wscat -c ws://<ip>:38090`
`wscat -c wss://<ip>:38090`

```json
{"id":1,"method":"call","params":[0,"get_objects",[["2.0.0"]]]}
{"id":1,"method":"call","params":[0,"get_accounts",[["1.2.0"]]]}

{"id":1,"method":"call","params":[0,"get_block_header",[100]]}
{"id":1,"method":"call","params":[0,"get_block",[1]]}


{"id":1,"method":"call","params":[0,"get_full_accounts",[["rui-eddie"],true]]}

{"id":1,"method":"call","params":[0,"",[]]}
{"id":1,"method":"call","params":[1,"",[]]}
{"id":1,"method":"call","params":[2,"",[]]}
{"id":1,"method":"call","params":[3,"",[]]}
{"id":1,"method":"call","params":[0,"get_chain_id",[]]}

{"id":1,"method":"call","params":[3,"get_account_history",["1.2.17","1.11.0",10000,"1.11.0"]]}
{"id":1,"method":"call","params":[3,"get_account_history",["1.2.65","1.11.0",10,"1.11.0"]]}
```

获取账户列表，返回对应的id

```json
{"id":1,"method":"call","params":[0,"lookup_accounts",["mytest-user",1]],"id":503}
{"id":1,"method":"call","params":[0,"lookup_account_names",[["mytest-user"]]]}
{"id":1,"method":"call","params":[0,"lookup_account_names",[["benfu82"]]]}
```

```json
{"id":1,"method":"call","params":[0,"get_objects",[["2.0.0"]]]}
{"id":1,"method":"call","params":[0,"get_objects",[["2.1.0"]]]}
```

获取转账所需的手续费

```json
{"id":1,"method":"call","params":[0,"get_required_fees",[[[0,{"fee":{"amount":"0","asset_id":"1.3.0"},"from":"1.2.17","to":"1.2.18","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"1.3.0"]],"id":503}

{"id":1,"method":"call","params":[0,"get_required_fees",[[[0,{"fee":{"amount":"0","asset_id":"1.3.0"},"from":"1.2.17","to":"1.2.18","amount":{"amount":"30000000","asset_id":"1.3.0"},"extensions":[]}]],"1.3.0"]],"id":503}


{"id":503,"jsonrpc":"2.0","result":[{"amount":2000000,"asset_id":"1.3.0"}]}


应该是校验交易的合法性，成功返回交易对应的私钥的对应公钥最小集合。即获取发送者和接受者的公钥，判断双方是否在公链上。
{"id":1,"method":"call","params":[0,"get_potential_signatures",[{"ref_block_num":0,"ref_block_prefix":0,"expiration":"1970-01-01T00:00:00","operations":[[0,{"fee":{"amount":"2000000","asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"extensions":[],"signatures":[]}]],"id":506}

{"id":506,"jsonrpc":"2.0","result":["RUI6t1yAxtwuSxQmFrA1Ky51DS4zWgXn6fHYs1Lr4w56ugt5xdTqv","RUI6Zisy7pvETTeab2fRPZX7W1jR8E5wh2cgo2mSWQjzHMGEzxvMX"]}



不知道做了啥，没有给出解释。
{"id":1,"method":"call","params":[0,"get_potential_address_signatures",[{"ref_block_num":0,"ref_block_prefix":0,"expiration":"1970-01-01T00:00:00","operations":[[0,{"fee":{"amount":"2000000","asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"extensions":[],"signatures":[]}]],"id":507}

{"id":507,"jsonrpc":"2.0","result":[]}



发起交易请求，成功返回转出者公钥
{"id":1,"method":"call","params":[0,"get_required_signatures",[{"ref_block_num":0,"ref_block_prefix":0,"expiration":"1970-01-01T00:00:00","operations":[[0,{"fee":{"amount":"2000000","asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"extensions":[],"signatures":[]},["RUI6t1yAxtwuSxQmFrA1Ky51DS4zWgXn6fHYs1Lr4w56ugt5xdTqv","RUI6Zisy7pvETTeab2fRPZX7W1jR8E5wh2cgo2mSWQjzHMGEzxvMX"]]],"id":508}

{"id":508,"jsonrpc":"2.0","result":["RUI6t1yAxtwuSxQmFrA1Ky51DS4zWgXn6fHYs1Lr4w56ugt5xdTqv"]}


广播签名交易
{"id":1,"method":"call","params":[0,"broadcast_transaction_with_callback",[510,{"ref_block_num":21270,"ref_block_prefix":3517951317,"expiration":"2018-06-29T14:35:05","operations":[[0,{"fee":{"amount":"2000000","asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"extensions":[],"signatures":["1f69d6e806d'.'f92d3154be41c092962226565e4bb274ffe893407165857710936869c6c2a8232f07692e59672431d51106182ad81d5e580b5d2fa079cc8b4d1867"]}]],"id":510}

{"id":510,"jsonrpc":"2.0","result":null}



{"id":1,"method":"call","params":[3,"get_account_history",["1.2.26","1.11.0",100,"1.11.0"]],"id":511}
{"id":511,"jsonrpc":"2.0","result":[{"id":"1.11.15","op":[0,{"fee":{"amount":2000000,"asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":10000000,"asset_id":"1.3.0"},"extensions":[]}],"result":[0,{}],"block_num":86807,"trx_in_block":0,"op_in_trx":0,"virtual_op":25}]}


{"id":1, "method":"call", "params":[2,"get_accounts",[["1.2.0"]]]}


{"id":1,"method":"call","params":[0,"get_block_header",[560547]]}
{"id":1,"method":"call","params":[0,"get_block_header",[1]]}
```

## 3. 钱包api

### 3.2 http + rpc2.0

`curl http://127.0.0.1:38091`

```bash
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "help", "params": []}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "unlock", "params": ["supersecret"]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["nathan","redpacket","1","RUI","", true]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["dbxroot","carlospaton88","10000","RUI","", true]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["redpacket","ailsa1","1","RUICANDY","", true]}' http://127.0.0.1:38081

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["dbxcandy","s1","10000000","RUICANDY","", true]}' http://127.0.0.1:38091



curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "suggest_brain_key", "params": []}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "register_account", "params": ["user", "pubkey", "pubkey", "dbxroot", "dbxroot", 0, true]}' http://127.0.0.1:38091

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "import_key", "params": ["user", "privkey"]}' http://127.0.0.1:38091

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "import_balance", "params": ["user", ["privkey"], true]}' http://127.0.0.1:38091

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "get_account", "params": ["dbxroot"]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "get_account", "params": ["dbxroot"]}' http://127.0.0.1:380
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["dbxcandy","eddie","70000","RUICANDY","", true]}' http://127.0.0.1:38061

nohup cli_wallet --wallet-file wallet.json --server-rpc-endpoint ws://0.0.0.0:11112 --rpc-endpoint 127.0.0.1:11113 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d -u nathan -p supersecret -d &

curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "unlock", "params": ["supersecret"]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "set_chaind_url", "params": ["13.124.230.82",5000]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["nathan","eddie","1","RUI","", true]}' http://127.0.0.1:38091
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["ruyi","eddie","1","RUYI","", true]}' http://127.0.0.1:38091
```

### 3.2 websocket

`wscat -c ws://<ip>:38090`
`wscat -c wss://<ip>:38090`

#### 3.2.1 0 : database-api

```s
"cancel_all_subscriptions"
"get_24_volume"
"get_account_balances"
"get_account_by_name"
"get_account_count"
"get_account_references"
"get_accounts"
"get_all_workers"
"get_assets"
"get_balance_objects"
"get_blinded_balances"
"get_block"
"get_block_header"
"get_block_header_batch"
"get_call_orders"
"get_chain_id"
"get_chain_properties"
"get_collateral_bids"
"get_committee_count"
"get_committee_member_by_account"
"get_committee_members"
"get_config"
"get_dynamic_global_properties"
"get_full_accounts"
"get_global_properties",
"get_key_references",
"get_limit_orders",
"get_margin_positions",
"get_named_account_balances",
"get_objects",
"get_order_book",
"get_potential_address_signatures",
"get_potential_signatures",
"get_proposed_transactions",
"get_recent_transaction_by_id",
"get_required_fees",
"get_required_signatures",
"get_settle_orders",
"get_ticker",
"get_top_markets",
"get_trade_history",
"get_trade_history_by_sequence",
"get_transaction",
"get_transaction_hex",
"get_vested_balances",
"get_vesting_balances",
"get_withdraw_permissions_by_giver",
"get_withdraw_permissions_by_recipient",
"get_witness_by_account",
"get_witness_count",
"get_witnesses",
"get_worker_count",
"get_workers_by_account",
"is_public_key_registered",
"list_assets"
"lookup_account_names"
"lookup_accounts"
"lookup_asset_symbols"
"lookup_committee_member_accounts"
"lookup_vote_ids"
"lookup_witness_accounts"
"set_block_applied_callback"
"set_pending_transaction_callback"
"set_subscribe_callback"
"subscribe_to_market"
"unsubscribe_from_market"
"validate_transaction"
"verify_account_authority"
"verify_authority"
```

```json
{"id":1,"method":"call","params":[0,"help",[]]}
{"id":1,"method":"call","params":[0,"unlock",[""]]}
{"id":1,"method":"call","params":[0,"transfer",["nathan","rui-eddie","1000","RUI","fuck", true]]}
{"id":1,"method":"call","params":[0,"get_key_references",[["RUI6jngeUrqxNwX8HASbDY4deNmq1yQ4wyzDUp3W5JNRMHzNsH2Ez"]]]}
{"id":1,"method":"call","params":[0,"get_key_references",[["RUI6oYdZec5KRi2UWdf7dMM2TLYQAWPNLBwm8Sd8jMfkqQm5mKv8W"]]]}
{"id":1,"method":"call","params":[0,"get_transaction",[1238661,1]]}
{"id":1,"method":"call","params":[0,"get_recent_transaction_by_id",["1.11.99237"]]}
{"id":1,"method":"call","params":[0,"get_recent_transaction_by_id",[571e45a2d48baa6d76f8d73ece3b12e4e5eb0fcf]]}



{"id":1,"method":"call","params":[0,"get_raw_transaction",[{"ref_block_num":0,"ref_block_prefix":0,"expiration":"1970-01-01T00:00:00","operations":[[0,{"fee":{"amount":"2000000","asset":"RUI"},"from":"user1","to":"user2","amount":{"amount":"10000000","asset":"RUI"},"extensions":[]}]]}]],"id":508}

{"ref_block_num": 59701,"ref_block_prefix": 155886624,"expiration": "2018-10-10T14:19:20", "operations": [[0,{"fee": {"amount": 100,"asset_id": "1.3.0"},"from": "1.2.2413","to": "1.2.6816","amount": {"amount": 100000,"asset_id": "1.3.0"},"extensions": []}]],"extensions": [],"signatures": ["1f0ccea9d48606961f0139df570dd6f5f63e2fcfb63bbec4a38efb2459c9b88500029ede1602d007b8e93a59db6f7540c8c20e0fcf2cd8b0e6a5923cfe531ae28b"]}


{"id":1,"method":"call","params":[0,"get_raw_transaction",[{"ref_block_num":0,"ref_block_prefix":0,"expiration":"1970-01-01T00:00:00","operations":[[0,{"fee":{"amount":"2000000","asset":"RUI"},"from":"user1","to":"user2","amount":{"amount":"10000000","asset":"RUI"}}]]}]],"id":508}

{"id":1,"method":"call","params":[0,"broadcast_transaction_with_callback",[510,{"ref_block_num":21270,"ref_block_prefix":3517951317,"expiration":"2018-06-29T14:35:05","operations":[[0,{"fee":{"amount":"2000000","asset_id":"1.3.0"},"from":"1.2.18","to":"1.2.26","amount":{"amount":"10000000","asset_id":"1.3.0"},"extensions":[]}]],"extensions":[],"signatures":["1f69d6e806d7f92d3154be41c092962226565e4bb274ffe893407165857710936869c6c2a8232f07692e59672431d51106182ad81d5e580b5d2fa079cc8b4d1867"]}]],"id":510}



{"id":1,"method":"call","params":[0,"get_account_balances",["1.2.17",["1.3.0"]]]}
{"id":1,"method":"call","params":[0,"get_account_balances",["nathan",["1.3.0"]]]}


{"id":1,"method":"call","params":[1,"login",["",""]]}
{"id":1,"method":"call","params":[1,"database",[]]}
{"id":1, "method":"call", "params":[2,"get_accounts",[["1.2.0"]]]}


{"id":1,"method":"call","params":[1,"login",["",""]]}
{"id":1,"method":"call","params":[1,"history",[]]}
{"id":1,"method":"call","params":[3,"get_account_history",["1.2.26","1.11.0",10,"1.11.0"]]}
{"id":1,"method":"call","params":[3,"get_account_history",["1.2.17","1.11.0",10,"1.11.0"]]}
{"id":1,"method":"call","params":[3,"get_account_history",["1.2.19","1.11.0",10,"1.11.0"]]}

{"id":1,"method": "call", "params": ["history", "get_account_history_by_operations", ["1.2.18", [], 1, 10]]}

{"id":1,"method": "call", "params": ["history", "get_market_history_buckets", []]}
{"id":1,"method": "call", "params": ["history", "get_account_history_size", ["1.2.17"]]}

{"id":1,"method": "call", "params": ["history", "get_account_history", ["1.2.17","1.11.0",10,"1.11.0"]]}

{"id":1,"method":"call","params":[2,"",[]]}

{"id":1,"method": "call", "params": ["history", "get_account_history_by_operations", ["1.2.40", [0], 1, 10]]}
{"id":1,"method": "call", "params": ["history", "get_account_history_by_operations", ["1.2.40", [0], 0, 10]]}
{"id":1,"method": "call", "params": ["history", "get_account_history_by_operations", ["1.2.17", [0], 10, 10]]}

{"id":1,"method":"call","params":[1,"login",["",""]]}
{"id":1,"method":"call","params":[1,"network_broadcast",[]]}
{"id":1,"method":"call","params":[3,"get_info",[]]}


{"id":1,"method":"call","params":[0,"",[]]}
{"id":1,"method":"call","params":[1,"",[]]}
```

```json
{"id":1,"method":"call","params":[0,"get_assets",[["1.3.0", "1.3.1"]]]}
{"id":1,"method":"call","params":[0,"get_assets",[["1.3.0", "1.3.1"]]]}
{"id":1,"method":"call","params":[0,"get_objects",[["1.3.0"]]]}
```
