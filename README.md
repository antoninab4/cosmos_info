# Шпаргалка для работы с нодами на __космосе__ 
## Все команды я буду показывать на примере последнего проекта Ollo, то есть:
`````
Бинарный файл будет называться ollod
ID сети ollo-testnet-0 (при установке через наши скрипты мы записываем это значение в .bash_profile как $CHAIN_ID)
Деном (токен условно) utollo
От проекта к проекту эти ☝️ данные будут меняться (соответственно и значения в командах). Погнали 🚀
`````
# Переменные и .bash_profile
``````
Для удобства работа с командами ноды я рекомендую пользоваться переменными окружения.
Пользовательские переменные обычно хранятся в файле .bash_profile, который расположен
в домашней директории пользователя (обычно в .root).

Переменные .bash_profile вступают в силу каждый раз когда пользователь подключается удаленно по SSH.

Главное преимущество переменных в нашей работе с нодами это то, что нам нет необходимости вспоминать,
к примеру, название своего Moniker (имя ноды) или какой сейчас ID у сети - все эти данные будут 
храниться в .bash_profile.
``````
````
Я обычно в .bash_profile добавляю следующие параметры:

Моникер (название ноды)
ID сети (CHAIN_ID)
Имя кошелька
Адрес кошелька
VALOPER-адрес
````

### Вот как выглядит, к примеру, добавление имени кошелька, название ноды и ID сети:
`````
source .bash_profile # обращаемся к .bash_profile

wallet="wallet" # присваиваем переменной wallet значение wallet

node="Moniker" # присваиваем переменной node значение Moniker

chain_id="ollo-testnet-0" # присваиваем переменной chain_id значение ollo-testnet-0

echo "export WALLET=$wallet" >> $HOME/.bash_profile # записываем в .bash_profile значение имени кошелька

echo "export NODENAME=$node" >> $HOME/.bash_profile # записываем в .bash_profile значение имени ноды

echo "export CHAIN_ID=$chain_id" >> $HOME/.bash_profile # записываем в .bash_profile значение ID сети
`````

### После этого можете открыть файл через: 
```
nano .bash_profile
```

#### и увидите там сохраненные значения. Использовать их можно так: $WALLET. Или $NODENAME. Или $CHAIN_ID.  Идея думаю понятна. 

#### По аналогии делается и для других параметров (адрес кошелька, VALOPER, но об этом чуть ниже).

## Работа с кошельком

#### Создать новый кошелек:
```
ollod keys add $WALLET --keyring-backend os
```

#### Восстановить существующий кошелек:
```
ollod keys add $WALLET --recover
```

#### Записать адрес кошелька и VALOPER в .bash_profile:
`````
source .bash_profile
ADDRESS=$(ollod keys show $WALLET -a --keyring-backend os) # здесь попросит пароль от кошелька
VALOPER=$(ollod keys show $WALLET --bech val -a --keyring-backend os) # здесь попросит пароль от кошелька
echo 'export ADDRESS='${ADDRESS} >> $HOME/.bash_profile
echo 'export VALOPER='${VALOPER} >> $HOME/.bash_profile
`````
#### Просмотреть список кошельков:
```
ollod keys list
```

#### Удалить кошелек:
```
ollod keys delete $WALLET
```

#### Экспортировать кошелек (сохранить в wallet.backup):
```
ollod keys export $WALLET
```

#### Импортировать кошелек:
```
ollod keys import $WALLET wallet.backup
```

#### Проверить баланс кошелька:
```
ollod q bank balances $ADDRESS
```

### P.S. В некоторых случаях (когда не работают команды, связанные с кошельком)
###  необходимо использовать флаг  по аналогии как в создании кошелька.
````
--keyring-backend os
````

## Работа с валидатором

#### При работе с валидатором необходимо учитывать несколько основных параметров,
тоже зависящих от проекта к проекту (хотя некоторые можно и не менять, ниже поймете):
`````
Моникер (Moniker - имя ноды / валидатора) по-умолчанию пусть будет Moniker (при запуске наших установочных 
         скриптов вы сами вводите это значение, поэтому не забывайте его менять  в командах ниже на $NODENAME)

ID сети ollo-testnet-0 (или $CHAIN_ID если ставили через наши скрипты)

(ОПЦИОНАЛЬНО) Идентификатор (Identity), нужен для отображения вашей аватарки, генерится на http://keybase.io

(ОПЦИОНАЛЬНО) Детали (Details) для отображения детальной информации  о валидаторе

(ОПЦИОНАЛЬНО) Веб-сайт (Website) для отображения сайта валидатора

Commission-rate / Commission max rate / Commission max change rate -  параметры комиссий валидатора. 
                       По-умолчанию будут 0.05 / 0.20 / 0.1.
`````

## Создать валидатора:
```
ollod tx staking create-validator \
--amount=1000000utollo \
--from=$WALLET \
--keyring-backend os \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--pubkey=$(ollod tendermint show-validator) \
--moniker="Moniker" \
--chain-id=ollo-testnet-0 \
--fees 200utollo \
--website="https://my-website.com" \
--identity=FFB0AA51A2DF5954 \
--details="I'm the best validator in Cosmos" \
-y 
```

## Редактировать валидатора:
````
ollod tx staking edit-validator \
--new-moniker="Moniker" \
--chain-id=ollo-testnet-0 \
--from=$WALLET \
--keyring-backend os \
--fees 200utollo
````

## Проверить статус валидатора:
```
ollod query staking validator $(ollod keys show wallet --bech val -a)
```

## Выйти из тюрьмы:
```
ollod tx slashing unjail --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Просмотреть причину попадания в тюрьму:
````
ollod query slashing signing-info $(ollod tendermint show-validator)
````

## Просмотреть список активных валидаторов:
``````
ollod q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl 
``````

## Просмотреть список неактивных валидаторов:
````
ollod q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl 
````

# Работа с токенами

## Здесь важны будут такие параметры как (их необходимо будет заменить):

``````
VALOPER-адрес валидатора, в который собираемся делегировать  (OTHER_VALOPER)
Кошелек получателя (TO_WALLET)
``````

## Заделегировать токены в своего валидатора (1000000 = 1 токен):
```
ollod tx staking delegate $(ollod keys show wallet --bech val -a) 1000000utollo --from wallet --keyring-backend os --chain-id=ollo-testnet-0 --fees 200utollo -y
```

## Заделегировать токены в другого валидатора:
```
ollod tx staking delegate MY_VALOPER 1000000utollo --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Ределегировать токены в другого валидатора:
```
ollod tx staking redelegate $(ollod keys show wallet --bech val -a) OTHER_VALOPER 1000000utollo --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Снять реварды со всех валидаторов:
```
ollod tx distribution withdraw-all-rewards --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Вывод комиссионных и вознаграждений из своего валидатора:
```
ollod tx distribution withdraw-rewards $(ollod keys show wallet --bech val -a) --commission --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Сделать UNBOND токенов (снять токены со своего валидатора на кошелек):
```
ollod tx staking unbond $(ollod keys show wallet --bech val -a) 1000000utollo --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Отправить токены на другой кошелек:
```
ollod tx bank send wallet TO_WALLET 1000000utollo --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

# Работа с голосованием (Governance)
``
Здесь обращаем внимание только на 1 параметр, это ID голосования (Proposal id), по-умолчанию он будет равен 1.
``

## Просмотреть список голосований:
```
ollod query gov proposals
```

## Просмотреть голосование по его ID:
```
ollod query gov proposal 1
```

## Проголосовать ЗА:
```
ollod tx gov vote 1 yes --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Проголосовать ПРОТИВ:
```
ollod tx gov vote 1 no --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Проголосовать ВОЗДЕРЖАТЬСЯ:
```
ollod tx gov vote 1 abstain --from wallet --chain-id ollo-testnet-0 --fees 200utollo -y
```

## Внести депозит в предложение (1 токен):
```
ollod tx gov deposit 1 1000000utollo --from wallet --keyring-backend os --fees 200utollo -y
```

# Работа с запросами
```
Здесь будет только 1 команда, где потребуется tx (hash).
```

## Запрос по транзакции:
```
ollod query tx <YOUR_TX_ID>
```

# Другие полезные команды

## В данном разделе собраны команды, не вошедшие ни в один из существующих разделов. По-большому счету это команды по работе самой ноды.
##### Замена дефолтных портов космос-ноды. По-умолчанию используются следующие порты:
```
gRPC - 9090
gRPC(web) - 9091
proxy_app - 26658
laddr(rpc) - 26657
pprof_laddr - 26656
laddr(p2p) - 6060
prometheus - 26660
api - 1317
```

## К примеру, заменяем на следующие порты:

```
gRPC - 9190
gRPC(web) - 9191
proxy_app - 27658
laddr(rpc) - 27657
pprof_laddr - 27656
laddr(p2p) - 6160
prometheus - 27660
api - 1417
```

## В этом случае команда смены портов будет иметь вид:
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:27658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:27657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:6160\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:27656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":27660\"%" $HOME/.ollo/config/config.toml && sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:9190\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:9191\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1417\"%" $HOME/.ollo/config/app.toml && sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:27657\"%" $HOME/.ollo/config/client.toml 
```

## Обновить значение indexer на KV (дефолт):
```
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.ollo/config/config.toml
```

## Обновить значение indexer на null:
```
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.ollo/config/config.toml
```

## Получить информацию о валидаторе:
```
ollod status 2>&1 | jq .ValidatorInfo
```

## Проверить значение catсhing_up:
```
ollod status 2>&1 | jq .SyncInfo.catching_up
```

## Проверить последний синхронизированный блок:
```
ollod status 2>&1 | jq ."SyncInfo"."latest_block_height"
```

## Получить значение PEER:
```
echo $(ollod tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.ollo/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

## Сбросить состояние ноды (не забыть заменить .ollo в пути на другой путь):
```
ollod tendermint unsafe-reset-all --home $HOME/.ollo --keep-addr-book
```

## Удалить ноду (ollod и ollo заменить на данные своей ноды):
```
sudo systemctl stop ollod && sudo systemctl disable ollod && sudo rm /etc/systemd/system/ollod.service && sudo systemctl daemon-reload && rm -rf $HOME/.ollo  && rm $(which ollod)
```

# Работа со службами

# Это общий раздел по работе со службами Ubuntu.

## Обновить конфигурацию всех служб:
```
sudo systemctl daemon-reload
```

## Добавить службу в автозагрузку:
```
sudo systemctl enable ollod
```

## Удалить службу из автозагрузки:
```
sudo systemctl disable ollod
```

## Запустить службу Ollo:
```
sudo systemctl start ollod
```

## Остановить службу Ollo:
```
sudo systemctl stop ollod
```

## Перезагрузить службу Ollo:
```
sudo systemctl restart ollod
```

## Проверить статус службы Ollo:
```
sudo systemctl status ollod
```

## Проверить статус службы Ollo:
```
sudo systemctl status ollod
```

## Проверить логи службы Ollo:
```
sudo journalctl -u ollod -f -o cat
```

# как добавить цвет в логи? Легко.

```
sudo apt install ccze
```
## Проверка логов в цвете
```
Пример

journalctl -n 100 -f -u ollod | ccze -A
```
# P.S. Стоит принимать во внимание, что иногда команды немного изменяются - добавляются какие-то
 флаги или параметры, это уже необходимо смотреть по каждой конкретной космос-ноде. Но азы - в этой статье.

# Саморазвивайтесь и пользуйтесь этой шпаргалкой. 

# Не забываем нажать на звездочку.
