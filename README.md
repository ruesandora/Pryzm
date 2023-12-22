<h1 align="center">Pryzm</h1>

> Pryzm bir Cosmos Hub'ı, donanım yaptığım ayarlardan ötürü oldukça düşük şimdilik, ayrı sunucu temin etmedim sizde öyle yapın.

> Bana gelen kuşlar teşvikli dedi - siz kendi araştırmanızı yapınız net bilgi yok.

> Fena olmayan bir proje ama sadetinde bir hub - haliyle ezbere bildiğim için 2 dakikada kurdum node'u, arkada çalışsın.

> TOPLULUK KANALLARI: [Sohbet Kanalımız](https://t.me/RuesChat) - [Duyurular ve Gelişmeler](https://t.me/RuesAnnouncement) - [Whatsapp](https://whatsapp.com/channel/0029VaBcj7V1dAw1H2KhMk34) - [Kenshi Telegram](https://t.me/KenshiTech)

#

<h1 align="center">Donanım</h1>

> Benim kurulumuma göre donanım: - ayrıca [Hetzner](https://hetzner.cloud/?ref=gIFAhUnYYjD3) kullandım - her VPS aynı değil.

```
2 CPU - 4 RAM - 40/80 SSD yeterli
```

#

<h1 align="center">Kurulum</h1>

```console
# sunucumuzu güncelleyelim
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

# go yükleyelim (tek komut olarak yapıştırın)
! [ -x "$(command -v go)" ] && {
VER="1.20.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin

# variablelar ve binary ayarları
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export PRYZM_CHAIN_ID="indigo-1"" >> $HOME/.bash_profile
echo "export PRYZM_PORT="41"" >> $HOME/.bash_profile
source $HOME/.bash_profile

# tek tek girelim burayı arkadaşlar
wget https://storage.googleapis.com/pryzm-resources/pryzmd-0.9.0-linux-amd64.tar.gz
tar -xzvf pryzmd-0.9.0-linux-amd64.tar.gz
mkdir -p $HOME/go/bin
mv pryzmd $HOME/go/bin

# intialize işlemleri
pryzmd config node tcp://localhost:${PRYZM_PORT}657
pryzmd config keyring-backend os
pryzmd config chain-id indigo-1
pryzmd init "test" --chain-id indigo-1

# sırasıyla genesis ve addrbook indirelim
wget -O $HOME/.pryzm/config/genesis.json https://testnet-files.itrocket.net/pryzm/genesis.json
wget -O $HOME/.pryzm/config/addrbook.json https://testnet-files.itrocket.net/pryzm/addrbook.json
```
<h1 align="center">Gerekli ayarlamalar</h1>

```console
# seedler - peerler - portlar - pruning ve indexer
# not: indexer veya pruning opsiyoneldir - yeni başladıysanız umursamadan kullanın.
SEEDS="fbfd48af73cd1f6de7f9102a0086ac63f46fb911@pryzm-testnet-seed.itrocket.net:41656"
PEERS="713307ce72306d9e86b436fc69a03a0ab96b678f@pryzm-testnet-peer.itrocket.net:41656,4af5be7666e9ee756a9a588c181f9631064b9cf8@37.27.55.69:26656,5d9bcb33eef94e045fe51105c89f5d77709b3183@144.76.101.167:5000,9515a13bbdeb233eb59efd6e8db892ac46e5bac5@142.132.153.6:56656,f9ade689abb3c59d3e3d8edf26c65bde3db58676@116.202.85.52:35656,7397a1bcbf413b76bd710fcf363f8259acdc4d29@144.91.84.168:23256,db0e0cff276b3292804474eb8beb83538acf77f5@195.14.6.192:26656,794b538577a59f789ce942fd393730da3e8c0ffe@34.65.224.175:26656,565e54f6b12672fba48861fc72654c39dc0f2d97@195.3.223.138:36656,2c7bb6ad931b0b2b24a0d8e6b7b5e0636b8bafb0@38.242.230.118:48656,b3a96da3b8738a47c1c0fabd2abd827a49b4b2a4@65.21.32.216:56656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.pryzm/config/config.toml

sed -i.bak -e "s%:1317%:${PRYZM_PORT}317%g;
s%:8080%:${PRYZM_PORT}080%g;
s%:9090%:${PRYZM_PORT}090%g;
s%:9091%:${PRYZM_PORT}091%g;
s%:8545%:${PRYZM_PORT}545%g;
s%:8546%:${PRYZM_PORT}546%g;
s%:6065%:${PRYZM_PORT}065%g" $HOME/.pryzm/config/app.toml

sed -i.bak -e "s%:26658%:${PRYZM_PORT}658%g;
s%:26657%:${PRYZM_PORT}657%g;
s%:6060%:${PRYZM_PORT}060%g;
s%:26656%:${PRYZM_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${PRYZM_PORT}656\"%;
s%:26660%:${PRYZM_PORT}660%g" $HOME/.pryzm/config/config.toml

sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.pryzm/config/app.toml

sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.015upryzm"|g' $HOME/.pryzm/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.pryzm/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.pryzm/config/config.toml
```

<h1 align="center">Node'u başlatma</h1>

```console
# servis dosyasını tek komut olarak girelim
sudo tee /etc/systemd/system/pryzmd.service > /dev/null <<EOF
[Unit]
Description=Pryzm node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.pryzm
ExecStart=$(which pryzmd) start --home $HOME/.pryzm
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# tek komut olarak snapshot girelim (opsiyonel)
pryzmd tendermint unsafe-reset-all --home $HOME/.pryzm
if curl -s --head curl https://testnet-files.itrocket.net/pryzm/snap_pryzm.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/pryzm/snap_pryzm.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.pryzm
    else
  echo no have snap
fi

# tek tek girelim aksın loglar.
sudo systemctl daemon-reload
sudo systemctl enable pryzmd
sudo systemctl restart pryzmd && sudo journalctl -u pryzmd -f
````

<h1 align="center">Validator kurulumu</h1>

```console
# cüzdan oluşturalım - rues kısmını cüzdan adınızla değiştirin
pryzmd keys add rues
# cüzdan bilgileriniz kaydedin lütfen

# çıktıda en altta false yazıyorsa devam edin yazmıyorsa bir kaç dakika bekleyin.
pryzmd status 2>&1 | jq .SyncInfo
```
> Çıktıda çıkan cüzdan adresine [buradan](https://testnet.pryzm.zone/faucet) test tokeni alalım.

> Not: faucet'i suistimal etmek rewards almanızı engelleyebilir.

```console
# şimdi validatorumuzu kuruyoruz
# lütfen notlarımı okuyup bu komutu kullanın komutun altına notları bırakıyorum
pryzmd tx staking create-validator \
--amount 1000000upryzm \
--pubkey $(pryzmd tendermint show-validator) \
--moniker "Rues" \
--identity "128462B2F5F8552F" \
--details "Rues Community" \
--website "https://github.com/ruesandora" \
--chain-id indigo-1 \
--commission-rate 0.1 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.03 \
--min-self-delegation 1 \
--from rues \
--gas-adjustment 1.5 \
--gas auto \
--gas-prices 0.015upryzm \
-y
# Moniker = Validatör İsminiz olsun
# identity eğer varsa keybase idiniz olsun yoksa o satırı silin
# website yoksa "tırnakların" içini silin veya kendi github koyun
# from kısmında yukarıda verdiğiniz cüzdan adresini girin
```

> Komut başarılı tamamlanınca tx hash verecek [burada](https://explorer.stavr.tech/Pryzm-Testnet) aratın ve success çıktısını aldınız hayırlı olsun.

> Ayrıca projenin node dışında app testneti ve farklı rol alma süreçleri var isterseniz onlarada katılırsınız.


<h1 align="center">Bazı faydalı komutlar</h1>

```console
# Node silme
sudo systemctl stop pryzmd
sudo systemctl disable pryzmd
sudo rm -rf /etc/systemd/system/pryzmd.service
sudo rm $(which pryzmd)
sudo rm -rf $HOME/.pryzm
sed -i "/PRYZM_/d" $HOME/.bash_profile

# validatore token stake etme
pryzmd tx staking delegate <OperatorAdresi> 1000000upryzm --from <Cüzdan> --chain-id indigo-1 --gas-adjustment 1.5 --gas auto --gas-prices 0.035upryzm -y

# unjail etme
pryzmd tx slashing unjail --from <Cüzdan> --chain-id indigo-1 --gas-adjustment 1.5 --gas auto --gas-prices 0.015upryzm -y

# Cüzdanları listele
pryzmd keys list

# cüzdan silme
pryzmd keys delete <Cüzdanİsmi>
```

> Daha eklerim aklıma geldikçe
