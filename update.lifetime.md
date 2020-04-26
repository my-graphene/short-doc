# 升级终身账户

```bash
import_key rui 5HwoBsALtcA9HM2cxZ3D5vzEFT7NJ5FyVURwfbxPzSSfhFoUmAe
import_balance rui ["5HwoBsALtcA9HM2cxZ3D5vzEFT7NJ5FyVURwfbxPzSSfhFoUmAe"] true
transfer nathan rui 500000 RUI "" true
upgrade_account rui true

create_witness rui "http://www.mygraphene.com" true

unlocked >>> get_witness rui
{
  "id": "1.6.13",
  "witness_account": "1.2.20",
  "last_aslot": 0,
  "signing_key": "RUI8B3Utm6Vqbjayt1GMwdQKtBmBYRaPtwbZLyM4eGW4o5zs6N7tV",
  "vote_id": "1:23",
  "total_votes": 0,
  "url": "http://www.mygraphene.com",
  "total_missed": 0,
  "last_confirmed_block_num": 0
}
unlocked >>> get_private_key "RUI8B3Utm6Vqbjayt1GMwdQKtBmBYRaPtwbZLyM4eGW4o5zs6N7tV"
get_private_key "RUI8B3Utm6Vqbjayt1GMwdQKtBmBYRaPtwbZLyM4eGW4o5zs6N7tV"
"5JPCiyqk2fQY9hzZCvdPgap2StdbK32eii8MsHhViGcxnQHakzK"

vote_for_witness rui rui true true





cli_wallet -w wallet.json  -s ws://0.0.0.0:38090 --chain-id 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
```
