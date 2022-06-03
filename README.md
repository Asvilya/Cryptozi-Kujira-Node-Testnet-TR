Ubuntu 20.04'ü yeni bir sunucuya kurun ve root olarak giriş yapın
Güvenlik duvarını kurun ufwve güvenlik duvarını yapılandırın

apt-get update
apt-get install ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow 22
ufw allow 26656
ufw enable

Yeni bir Kullanıcı oluştur
# add user
adduser node

# add user to sudoers
usermod -aG sudo node

# login as user
su - node

Ön Koşulları yükleyin
sudo apt update
sudo apt install pkg-config build-essential libssl-dev curl jq git libleveldb-dev -y
sudo apt-get install manpages-dev -y

# install go
curl https://dl.google.com/go/go1.18.2.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile

# check go version
go version

Kujira Testnet Düğümünü Yükleyin
git clone https://github.com/Team-Kujira/core
cd core
make clean
make install
cd

Validator düğümünüzü başlatın
#Choose a name for your validator and use it in place of “<moniker-name>” in the following command:
kujirad init <moniker-name> --chain-id harpoon-3

#Example
kujirad init My_Node --chain-id harpoon-3
Genesis.json, addrbook.json dosyalarını indirin
cd ~/.kujira/config
rm genesis.json
cd

wget https://raw.githubusercontent.com/Team-Kujira/networks/master/testnet/harpoon-3.json -O $HOME/.kujira/config/genesis.json
wget https://raw.githubusercontent.com/Team-Kujira/networks/master/testnet/addrbook.json -O $HOME/.kujira/config/addrbook.json

Düğümü başlatın ve Senkronize etmesine izin verin
kujirad start

# Press Ctrl+c to exit

Doğrulayıcıyı bir systemd birimi olarak çalıştırma
cd /etc/systemd/system
sudo nano kujirad.service

Aşağıdaki içeriği içine kopyalayın kujirad.serviceve kaydedin.

[Unit]
Description=Kujirad Daemon
#After=network.target
StartLimitInterval=350
StartLimitBurst=10

[Service]
Type=simple
User=node
ExecStart=/home/node/go/bin/kujirad start
Restart=on-abort
RestartSec=30

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=1048576
sudo systemctl daemon-reload
sudo systemctl enable kujirad

# Start the service
sudo systemctl start kujirad

# Stop the service
sudo systemctl stop kujirad

# Restart the service
sudo systemctl restart kujirad


# For Entire log
journalctl -t kujirad

# For Entire log reversed
journalctl -t kujirad -r

# Latest and continuous
journalctl -t kujirad -f

Doğrulayıcı Düğümünüz için bir Cüzdan oluşturun
24 kelimelik Anımsatıcı Cümleyi kopyaladığınızdan emin olun, bir dosyaya kaydedin ve güvenli bir yerde saklayın.

kujirad keys add <wallet-name>

#Example
kujirad keys add my_wallet

Bu cüzdandan biraz KUJI jetonu alın.

Doğrulayıcı Düğümünüzü Oluşturun ve Kaydedin

kujirad tx staking create-validator -y \
  --chain-id harpoon-3 \
  --moniker <moniker-name> \
  --pubkey "$(kujirad tendermint show-validator)" \
  --amount 5000000ukuji \
  --identity "<Keybase.io ID>" \
  --website "<website-address>" \
  --details "Some description" \
  --from <wallet-name> \
  --gas-prices 1ukuji \
  --commission-rate=0.0 \
  --commission-max-rate=0.05 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation 1
  
#Example

kujirad tx staking create-validator -y \
  --chain-id harpoon-3 \
  --moniker "My Node" \
  --pubkey "$(kujirad tendermint show-validator)" \
  --amount 5000000ukuji \
  --identity "D74433D32938F013" \
  --website "http://www.mywebsite.com" \
  --details "Some description" \
  --from my_wallet \
  --gas-prices 1ukuji \
  --commission-rate=0.0 \
  --commission-max-rate=0.05 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation 1
  
Doğrulayıcı Operatör Adresini Al (Valapor Adresi)
<wallet-name>Doğru değerlere değiştirdiğinizden emin olun .

kujira keys show <wallet-name> --bech val --output json | jq -r .address

#Example
kujirad keys show my_wallet --bech val --output json | jq -r .address

KUJIDüğümünüze Yetkilendirin
kujirad tx staking delegate <validator-address> 1000000ukuji --from <wallet-name> --fees 30000ukuji --chain-id harpoon-3 -y

#Example
kujirad tx staking delegate kujiravaloper1xesqr8vjvy34jhu027zd70ypl0nnev5esqejlw 1000000ukuji --from my_wallet --fees 30000ukuji --chain-id harpoon-3 -y

Yedek Doğrulayıcı düğüm dosyası
Doğrulayıcı düğümünüzü başarıyla oluşturup kaydettikten sonra aşağıdaki dosyaların yedeğini alın.

/home/node/.kujira/config/node_key.json
/home/node/.kujira/config/priv_validator_key.json
/home/node/.kujira/data/priv_validator_state.json

Ödülleri Geri Çek
Değerleri düzeltmek <validator-operator-address>için değiştirdiğinizden emin olun .<wallet-name>

kujirad tx distribution withdraw-rewards <validator-address> --from <wallet-name> --chain-id=harpoon-3 --gas auto --fees 1ukuji --gas-adjustment 1.4 -y

#Example
kujirad tx distribution withdraw-rewards kujiravaloper1xesqr8vjvy34jhu027zd70ypl0nnev5esqejlw --from synergy --chain-id=harpoon-3 --gas auto --fees 1ukuji --gas-adjustment 1.4 -y

Bir Adresin Bakiyesini Kontrol Edin
kujirad query bank balances <wallet-address>

#Example:
kujirad query bank balances kujira1xesqr8vjvy34jhu027zd70ypl0nnev5eh42prp
