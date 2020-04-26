# 合约测试

```bash
mycc -n helloworld

mycc -o helloworld.wast helloworld.cpp
mycc -g helloworld.abi helloworld.cpp
deploy_contract helloworld nathan 0 0 /root/dbxchain/contracts/examples/helloworld RUI true
call_contract nathan helloworld null hi "{\"user\":\"abcdefg\"}" RUI true


mycc -n hello
mycc -o hello.wast hello.cpp
mycc -g hello.abi hello.cpp
deploy_contract hello nathan 0 0 /root/dbxchain/contracts/examples/hello RUI true
call_contract nathan hello null hi "{\"user\":\"test\"}" RUI true



deploy_contract transfer nathan 0 0 /root/dbxchain/contracts/examples/transfer RUI true
call_contract nathan transfer null recept "{\"account_to\":\"dbxroot\", \"asset\":\"RUI\", \"amount\":10}" RUI true
call_contract nathan transfer null show "{\"account_to\":\"dbxroot\", \"asset\":\"RUI\", \"amount\":10}" RUI true





deploy_contract contracta nathan 0 0 /root/dbxchain/contracts/examples/contracta RUI true
deploy_contract contractb nathan 0 0 /root/dbxchain/contracts/examples/contractb RUI true
call_contract nathan contractb null hi "{}" RUI true
call_contract nathan contracta null hicontract "{\"act_id\":20}" RUI true




set_password supersecret
unlock supersecret
import_key nathan 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
import_balance nathan ["5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"] true
upgrade_account nathan true

mycc -o voting.wast voting.cpp
mycc -g voting.abi voting.cpp
deploy_contract voting nathan 0 0 /root/dbxchain/contracts/examples/voting RUI true
call_contract nathan voting null list "{}" RUI true
call_contract nathan voting null vote "{\"id\":1, \"name\":\"user1\"}" RUI true
call_contract nathan voting null count "{\"id\":1, \"name\":\"user1\"}" RUI true

deploy_contract votec nathan 0 0 /root/dbxchain/contracts/examples/voting RUI true

call_contract nathan votec null list "{}" RUI true
call_contract nathan votec null vote "{\"id\":1, \"name\":\"user1\"}" RUI true
call_contract nathan votec null count "{\"id\":1, \"name\":\"user1\"}" RUI true
call_contract nathan votec null vote "{\"id\":2, \"name\":\"user2\"}" RUI true
call_contract nathan votec null count "{\"id\":2, \"name\":\"user1\"}" RUI true



mycc -o store.wast store.cpp
mycc -g store.abi store.cpp
deploy_contract store nathan 0 0 /root/dbxchain/contracts/examples/store RUI true
call_contract nathan store null set "{}" RUI true
call_contract nathan store null show "{}" RUI true



mycc -o contract_storage_demo.wast contract_storage_demo.cpp
mycc -g contract_storage_demo.abi contract_storage_demo.cpp


deploy_contract mystore nathan 0 0 /root/dbxchain/contracts/examples/test/contract_storage_demo RUI true
call_contract nathan mystore null store "{\"id\":1, \"manufactor\":\"manufactor1\", \"name\":\"user1\", \"frequency\":100}" RUI true
call_contract nathan mystore null store "{\"id\":1, \"manufactor\":\"manufactor1\", \"name\":\"user1\", \"frequency\":100}" RUI true

call_contract nathan mystore null find "{\"id\":1}" RUI true
call_contract nathan mystore null find "{\"id\":2}" RUI true
call_contract nathan mystore null find "{\"id\":0}" RUI true

deploy_contract mystoreaaa nathan 0 0 /root/dbxchain/contracts/examples/test/contract_storage_demo RUI true
call_contract nathan mystoreaaa null store "{\"id\":1, \"manufactor\":\"manufactor1\", \"name\":\"user1\", \"frequency\":100}" RUI true
```
