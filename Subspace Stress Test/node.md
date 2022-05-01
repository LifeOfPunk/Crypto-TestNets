устанавливаем docker
``` bash
sudo -i
apt update && apt install docker.io docker-compose -y
```
задаем переменные окружения с адресом и именем ноды. Адрес можно получить тут
https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ffarm-rpc.subspace.network#/accounts

в кавычках указываем свой адрес и произвольное имя ноды
``` bash
SUBSPACE_NODE_NAME="имя вашей ноды"
SUBSPACE_NODE_NAME="имя вашей ноды"
```

здесь ничего не меняем
``` bash
echo 'export SUBSPACE_WALLET_ADDRESS='$SUBSPACE_WALLET_ADDRESS >> $HOME/.bash_profile
echo 'export SUBSPACE_NODE_NAME="'${SUBSPACE_NODE_NAME}'"' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

теперь нужно создать docker-compose.yml
``` bash
mkdir $HOME/subspace
cd $HOME/subspace
```

весь блок вставляется в консоль целиком:
``` bash
tee $HOME/subspace/docker-compose.yml > /dev/null <<EOF
version: "3.7"
services:
  node:
    # Replace 'snapshot-DATE' with latest release (like 'snapshot-2022-mar-09')
    image: ghcr.io/subspace/node:snapshot-2022-mar-09
    volumes:
# Instead of specifying volume (which will store data in '/var/lib/docker'), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
      - node-data:/var/subspace:rw
#      - /path/to/subspace-node:/var/subspace:rw
    ports:
# If port 30333 is already occupied by another Substrate-based node, replace all
# occurrences of '30333' in this file with another value
      - "0.0.0.0:30333:30333"
# Un-comment following line to unlock node's WebSocket RPC
#      - "127.0.0.1:9944:9944"
    restart: unless-stopped
    command: [
      "--chain", "testnet",
      "--base-path", "/var/subspace",
      "--wasm-execution", "compiled",
      "--execution", "wasm",
      "--bootnodes", "/dns/farm-rpc.subspace.network/tcp/30333/p2p/12D3KooWPjMZuSYj35ehced2MTJFf95upwpHKgKUrFRfHwohzJXr",
      "--port", "30333",
      "--telemetry-url", "wss://telemetry.polkadot.io/submit/ 1",
      "--telemetry-url", "wss://telemetry.subspace.network/submit/ 1",
      "--rpc-cors", "all",
      "--rpc-methods", "unsafe",
      "--ws-external",
      "--validator",
# Replace 'INSERT_YOUR_ID' with your node ID (will be shown in telemetry)
      "--name", "$SUBSPACE_NODE_NAME"
    ]

  farmer:
# Replace 'snapshot-DATE' with latest release (like 'snapshot-2022-mar-09')
    image: ghcr.io/subspace/farmer:snapshot-2022-mar-09
# Un-comment following 2 lines to unlock farmer's RPC
#    ports:
#      - "127.0.0.1:9955:9955"
# Instead of specifying volume (which will store data in '/var/lib/docker'), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
    volumes:
      - farmer-data:/var/subspace:rw
#      - /path/to/subspace-farmer:/var/subspace:rw
    restart: unless-stopped
    command: [
      "farm",
      "--node-rpc-url", "ws://node:9944",
      "--ws-server-listen-addr", "0.0.0.0:9955",
# Replace 'WALLET_ADDRESS' with your Polkadot.js wallet address
      "--reward-address", "$SUBSPACE_WALLET_ADDRESS"
    ]
volumes:
  node-data:
  farmer-data:
EOF
```

запускаем
``` bash
docker-compose up -d
```

важно запускать команды из того же каталога где был создан файл docker-compose.yml.
Поэтмоу прописывыес каждый раз
``` bash
cd $HOME/subspace
```

просмотр Логов
``` bash
docker-compose logs --tail=1000 -f
```

посмотреть подписанные блоки
``` bash
docker-compse logs | grep "signed"
```

телеметрию нод можно смотреит тут 
https://telemetry.subspace.network/#/0x332ef6e751e25426e38996c51299dfc53bcd56f40b53dce2b2fc8442ae9c4a74
Для этого просто прописываем название своей ноды

токены будут капать на polkadot кошелек по указанному адресу
https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ffarm-rpc.subspace.network#/accounts

удаление ноды (docker)
``` bash
cd $HOME/subspace
docker-compose down && docker volume rm $(docker volume ls -q | grep subspace)
cd $HOME && rm -rf $HOME/subspace/ 
```
