<h1 align="center">STRIDE RELAYER</h1>
Merhabalar bugün Stride testneti için görevlerimizden biri olan Relayer kurulumunu anlatacağım. Öncelikle Stride kurduğumuz sunucumuza Relayer kurabiliriz.

#Diskimizde yer açalım

```sh
rm -rf /root/.stride/data/tx_index.db
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
sed -i.bak -e "s/indexer *=.*/indexer = \"null\"/g" $HOME/.stride/config/config.toml
systemctl restart strided
```

#Relayer ismi olarak discord ID girmemiz gerekli

```sh
RELAYER_NAME='DiscordID'
```

#Daha sonra aşağıdaki sırasıyla tek tek aşağıdaki kodları girelim

```sh
CHAIN_ID_A='STRIDE-TESTNET-2'
RPC_ADDR_A='stride.stake-take.com:26657'
GRPC_ADDR_A='stride.stake-take.com:9090'
ACCOUNT_PREFIX_A='stride'
TRUSTING_PERIOD_A='8hours'
DENOM_A='ustrd'
```

#Burada daha önce oluşturduğumuz stride cüzdanımınız mnemonic kodlarını gireceğiz.

```sh
MNEMONIC_A='$MNEMONIC'
CHAIN_ID_B='GAIA'
RPC_ADDR_B='stride.stake-take.com:46657'
GRPC_ADDR_B='stride.stake-take.com:9490'
ACCOUNT_PREFIX_B='cosmos'
TRUSTING_PERIOD_B='8hours'
DENOM_B='uatom'
```

#Tekrardan Mnemonicleri giriyoruz.

```sh
MNEMONIC_B='$MNEMONIC'
```

#Şimdi HERMES Kurulumuna geliyoruz

```sh
cd $HOME
wget https://github.com/informalsystems/ibc-rs/releases/download/v1.0.0-rc.1/hermes-v1.0.0-rc.1-x86_64-unknown-linux-gnu.tar.gz
mkdir -p $HOME/.hermes/bin
tar -C $HOME/.hermes/bin/ -vxzf hermes-v1.0.0-rc.1-x86_64-unknown-linux-gnu.tar.gz
echo 'export PATH="$HOME/.hermes/bin:$PATH"' >> $HOME/.bash_profile
source $HOME/.bash_profile
hermes version
```

Hermes Version yazdıktan sonra "hermes 1.0.0-rc.1+54ae389" bu yazı çıkıyorsa kuruluma devam edebiliriz.

#Burayı tek seferde giriyoruz.

```sh
sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF
[global]
log_level = 'info'

[mode]

[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true

[rest]
enabled = true
host = '127.0.0.1'
port = 3000

[telemetry]
enabled = true
host = '127.0.0.1'
port = 3001

[[chains]]
### CHAIN_A ###
id = '${CHAIN_ID_A}'
rpc_addr = 'http://${RPC_ADDR_A}'
grpc_addr = 'http://${GRPC_ADDR_A}'
websocket_addr = 'ws://${RPC_ADDR_A}/websocket'
rpc_timeout = '10s'
account_prefix = '${ACCOUNT_PREFIX_A}'
key_name = 'wallet'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 100000
max_gas = 2500000
gas_price = { price = 0.0025, denom = '${DENOM_A}' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '${TRUSTING_PERIOD_A}'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = '${RELAYER_NAME}'
[chains.packet_filter]
policy = 'allow'
list = [
  ['ica*', '*'], # allow relaying on all channels whose port starts with ica
  ['transfer', 'channel-0'],
]

[[chains]]
### CHAIN_B ###
id = '${CHAIN_ID_B}'
rpc_addr = 'http://${RPC_ADDR_B}'
grpc_addr = 'http://${GRPC_ADDR_B}'
websocket_addr = 'ws://${RPC_ADDR_B}/websocket'
rpc_timeout = '10s'
account_prefix = '${ACCOUNT_PREFIX_B}'
key_name = 'wallet'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 100000
max_gas = 2500000
gas_price = { price = 0.0025, denom = '${DENOM_B}' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '${TRUSTING_PERIOD_B}'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = '${RELAYER_NAME}'
[chains.packet_filter]
policy = 'allow'
list = [
  ['ica*', '*'], # allow relaying on all channels whose port starts with ica
  ['transfer', 'channel-0'],
]
EOF
```
#Hermes'in sağlam çalıştığını kontrol ediyoruz.

```sh
hermes health-check
```

#eğer bu şekilde bir çıktı alıyorsak devam edebiliriz
2022-08-06T13:41:30.827435Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
2022-08-06T13:41:30.827909Z  INFO ThreadId(01) [STRIDE-TESTNET-2] performing health check...
2022-08-06T13:41:32.249441Z  INFO ThreadId(01) chain is healthy chain=STRIDE-TESTNET-2
2022-08-06T13:41:32.249482Z  INFO ThreadId(01) [GAIA] performing health check...
2022-08-06T13:41:33.633862Z  INFO ThreadId(01) chain is healthy chain=GAIA
SUCCESS performed health check for all chains in the config

#Burayı da tek seferde giriyoruz

```sh
sudo tee $HOME/.hermes/${CHAIN_ID_A}.mnemonic > /dev/null <<EOF
${MNEMONIC_A}
EOF
```

#Tek seferde burasını da girip enter yapıyoruz 
```sh
sudo tee $HOME/.hermes/${CHAIN_ID_B}.mnemonic > /dev/null <<EOF
${MNEMONIC_B}
EOF
```

#Hermes için cüzdanlarımızı bağlıyoruz. önce Stride cüzdanımız daha sonra Cosmos Cüzdanımız için yapıyoruz bu işlemi.

```sh
hermes keys add --chain ${CHAIN_ID_A} --mnemonic-file $HOME/.hermes/${CHAIN_ID_A}.mnemonic
```

```sh
hermes keys add --chain ${CHAIN_ID_B} --mnemonic-file $HOME/.hermes/${CHAIN_ID_B}.mnemonic
```

#Bu kısımda Discord'dan faucet kullanıyoruz her iki cüzdanımız için. 
-[Discord](https://discord.com/channels/988945059783278602/992572020535599244).

$faucet-stride:$STRIDEADRESI
$faucet-atom:$COSMOSADRESI

faucet kanalında 15 dakika limiti vardır. iki fauceti ayrı ayrı kullanalım.

#Hermes için systemd oluşturuyoruz

```sh
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=hermes
After=network-online.target

[Service]
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```sh
sudo systemctl daemon-reload
```

```sh
sudo systemctl enable hermesd
```

```sh
sudo systemctl restart hermesd
```

#Log Kontrolü yapıyoruz
Bu konud girdikten sonra biraz beklememiz gerekiyor.

```sh
journalctl -u hermesd -f -o cat
```
