# 构建石墨烯docker系统

```bash
docker run -itd --name faucet -p 3000:3000 -p 127.0.0.1:3306:3306 -v /etc/localtime:/etc/localtime:ro ubuntu:16.04 /bin/bash


docker run -itd --name node -p 31010:31010 -p 38090:38090 -p 127.0.0.1:38091:38091 -p 127.0.0.1:38092:38092 -v /etc/localtime:/etc/localtime:ro  ubuntu:16.04 /bin/bash

docker run -itd --name test -p 1010:127.0.0.1:1010 -v /etc/localtime:/etc/localtime:ro  ubuntu:16.04 /bin/bash


docker run -itd --name node -p 31010:31010 -p 38090:38090 -p 172.17.0.1:38091:38091 -p 172.17.0.1:38092:38092 -v /etc/localtime:/etc/localtime:ro  ubuntu:16.04 /bin/bash

docker run -itd --name faucet -p 3000:3000 -v /etc/localtime:/etc/localtime:ro ubuntu:16.04 /bin/bash

docker run -itd --name node -v /etc/localtime:/etc/localtime:ro ubuntu:16.04 /bin/bash
```
