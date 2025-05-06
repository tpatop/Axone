## [English version](README.md)

---
## Мой валидатор: [Moon Core](https://explore.axone.xyz/Axone%20testnet/staking/axonevaloper18ey3gh27akq07mdzykemz3knhyq3mkn5l57uvq)

---

# Axone

Axone Protocol — децентрализованный оркестрационный протокол, созданный для эффективного обмена цифровыми ресурсами в AI-экосистеме. Протокол обеспечивает справедливое вознаграждение всех участников — от поставщиков данных до разработчиков моделей, предотвращая концентрацию ценности в руках крупных корпораций.

## Основные возможности

- Совместная оркестрация AI-рабочих процессов
- Совместимость с любыми данными, моделями и инфраструктурой
- Гибкая настройка прав доступа и процессов обмена
- Создание пользовательских сред с настраиваемыми правилами и токенами
- Поддержка как on-chain, так и off-chain обмена ресурсами

## Сеть

| Тип     | Сеть              |
|---------|-------------------|
| Testnet | axone-dentrite-1  |

## Полезные ссылки

- [Website](https://www.axone.xyz/)
- [Documentation](https://docs.axone.xyz/)
- [Explorer](https://explore.axone.xyz/Axone%20testnet)
- [Discord](https://discord.gg/x5CsyFmx)
- [Twitter](https://x.com/axonexyz)

---

## Руководство по установке узла

### 0. Создание отдельного пользователя для узла (рекомендуется)

Для повышения безопасности рекомендуется запускать узел от отдельного пользователя, например `axone`.

#### Создание пользователя

```bash
sudo useradd -m -s /bin/bash axone
```

#### Добавление прав суперпользователя (опционально)

```bash
sudo usermod -aG sudo axone
```

#### Переключение на пользователя `axone`

```bash
sudo su - axone
```

#### Проверка прав пользователя

```bash
whoami
```

Теперь все последующие шаги выполняются от имени пользователя `axone`.

---

### 1. Установка Go и Cosmovisor

> 💡 **Примечание:** Если у вас уже установлены Go и Cosmovisor, этот шаг можно пропустить!

#### Установка Go (v1.23.4)

```bash
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.23.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
rm go1.23.4.linux-amd64.tar.gz
```

#### Настройка переменных окружения Go

Добавьте в файл `~/.profile`:

```bash
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

Примените изменения:

```bash
source ~/.profile
```

#### Установка Cosmovisor (v1.0.0)

```bash
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

---

### 2. Установка узла

#### Скачивание и установка бинарного файла узла

```bash
git clone https://github.com/axone-protocol/axoned axone
cd axone
git checkout v10.0.0
make install
```

---

### 3. Настройка узла

#### Инициализация узла

Замените `YOUR_MONIKER` на ваш уникальный монникер:

```bash
axoned init YOUR_MONIKER --chain-id axone-dentrite-1
```

#### Загрузка genesis-файла

```bash
wget -O genesis.json https://raw.githubusercontent.com/axone-protocol/networks/911b2d34631ac242e9ef3be577163653ed644726/chains/dentrite-1/genesis.json --inet4-only
mv genesis.json ~/.axoned/config
```

#### Настройка сидов

```bash
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:17656"/' ~/.axoned/config/config.toml
```

В файле ~/.axoned/config/config.toml можешь изменить порты, если будет выдавать ошибку


---

### 4. Запуск узла

#### Настройка папок Cosmovisor

```bash
mkdir -p ~/.axoned/cosmovisor/genesis/bin
mkdir -p ~/.axoned/cosmovisor/upgrades

cp ~/go/bin/axoned ~/.axoned/cosmovisor/genesis/bin
```

#### Создание systemd-сервиса

Создайте файл `/etc/systemd/system/axone.service` со следующим содержимым:

```bash
sudo nano /etc/systemd/system/axone.service
```

```ini
[Unit]
Description=Axone Node
After=network-online.target

[Service]
User=axone
ExecStart=/home/axone/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=axoned"
Environment="DAEMON_HOME=/home/axone/.axoned"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

---

### 5. Загрузка снапшота

Используйте снапшот от [p10node](https://docs.p10node.com/services/axone-testnet/install.html):

```bash
# Сделать резервную копию priv_validator_state.json
cp $HOME/.axoned/data/priv_validator_state.json $HOME/.axoned/priv_validator_state.json.backup

# Очистить базу данных
axoned tendermint unsafe-reset-all --home $HOME/.axoned --keep-addr-book

# Скачать и распаковать последний снапшот
curl -L https://files.p10node.com/axone-testnet/latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.axoned

# Восстановить priv_validator_state.json из бэкапа
mv $HOME/.axoned/priv_validator_state.json.backup $HOME/.axoned/data/priv_validator_state.json
```

---

### 6. Запуск сервиса узла

```bash
# Включить сервис
sudo systemctl enable axone.service

# Запустить сервис
sudo systemctl start axone.service

# Проверить логи
sudo journalctl -fu axone
```

---

### 7. Проверка статуса
```bash
curl http://localhost:26657/status | jq
```


---

## Создание валидатора

Для следующих действий необходимо дождаться полной синхронизации ноды.

### Шаг 1. Создаём аккаунт оператора

```bash
axoned keys add operator
```

> ⚠️ **Важно:** Обязательно сохраните мнемоническую фразу и приватный ключ в надёжном месте!

### Шаг 2. Получаем адрес аккаунта

```bash
axoned keys show operator -a
```

### Шаг 3. Проверяем баланс

```bash
axoned query bank balances $(axoned keys show operator -a)
```

Убедитесь, что на аккаунте достаточно токенов для создания валидатора (минимум **1 AXON** = 1_000_000 uaxon). Кран проекта по [ссылке](https://faucet.axone.xyz/)

---

### Шаг 4. Получаем публичный ключ валидатора

```bash
axoned tendermint show-validator
```

Скопируйте вывод команды — это будет значение поля `key` в `validator.json`.

---

### Шаг 5. Создаём файл `validator.json`

Создайте файл `validator.json` следующего содержания:

```
nano ~/.axoned/config/validator.json
```

> сохранить изменения - CTRL+S и CTRL+X

```json
{
  "pubkey": {
    "@type": "/cosmos.crypto.ed25519.PubKey",
    "key": "<ВСТАВЬТЕ_СЮДА_ПУБЛИЧНЫЙ_КЛЮЧ из шага 4>"
  },
  "amount": "1000000uaxon",
  "moniker": "<YOUR_MONIKER>",
  "identity": "<ВАШ_IDENTITY (опционально, например, keybase ID)>",
  "website": "<ВАШ_САЙТ (опционально)>",
  "security": "<ВАШ_EMAIL (опционально)>",
  "details": "<Описание вашего валидатора (опционально)>",
  "commission-rate": "0.10",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

> ✅ Замените все поля в `<>` на свои данные!

---

### Шаг 6. Создаём валидатора (отправляем транзакцию)

```bash
axoned tx staking create-validator validator.json --from operator --chain-id axone-dentrite-1 --gas-adjustment 1.5 --gas auto -y
```

---

## Дополнительно: Делегирование на своего валидатора

Чтобы увеличить стейк на валидаторе, используйте команду делегирования:

```bash
axoned tx staking delegate $(axoned keys show operator --bech val -a) 1000000uaxon --chain-id axone-dentrite-1 --from operator --gas auto --gas-adjustment 1.5 -y
```

### Проверка валидатора

Проверить своего валидатор можно в [explorer](https://explore.axone.xyz/Axone%20testnet/staking)