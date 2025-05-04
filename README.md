# Aztec-Sequencer
Aztec Sequencer Node Kurulum Rehberi

8 cores CPU, 16GB RAM, 100GB+ SSD öneriliyor

HETZNER 10€ Hediyeli Link: https://hetzner.cloud/?ref=pRFkXFvi9sON

# Sistemi Güncelleyelim ve Paketleri Yükleyelim

```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```

# Docker'ı Yükleyelim

```
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```
echo \
 "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
 "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
 sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# Docker'ı Test Edelim

```
docker --version
```

Çıktı verirse kurulum başarılı, devam.

```
sudo systemctl enable docker
sudo systemctl restart docker
```

# Aztec'i Kuralım

```
bash -i <(curl -s https://install.aztec.network)
```

Şimdi Terminali kapatıp tekrar açmamız gerekiyor.

Terminali açtıktan sonra aşağırdaki kodu girip kurulumun başarılı olduğunu kontrol ediyoruz.

```
aztec
```

Hata vermediyse devam.

```
aztec-up alpha-testnet
```

# Şimdi Alchemy'den RPC URL almamız gerekiyor. Bunun için resimlerdeki adımları izliyoruz.

![1](https://github.com/user-attachments/assets/3d293822-9ce0-448f-9e6d-ce7e32aead76)
![2](https://github.com/user-attachments/assets/0ee50435-78c9-483d-a4bf-605347cf5eb3)
![3](https://github.com/user-attachments/assets/660ee8c8-9b9a-4e91-8a28-8fda7bf9128f)
![4](https://github.com/user-attachments/assets/8e39eeb4-f185-4d5f-b656-9db183aa62c8)

Bir de Consensus URL lazım. Onu da public link olan https://rpc.drpc.org/eth/sepolia/beacon linkini kullanacağız. İsteyen araştırıp ücretli olarak priv beacon kullanabilir.

EVM Cüzdan Private Key ve EVM Cüzdan Adresi de hazır olsun

# Portları Açalım

```
ufw allow 22
ufw allow ssh
ufw enable
ufw allow 40400
ufw allow 8080
```

# Screen Açıyoruz

```
screen -S aztec
```

Şimdi aşağıdaki kod bloğunu kendimize göre düzenleyip yapıştıracağız.

```
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls ALCHEMY_URL  \
  --l1-consensus-host-urls BEACON_URL \
  --sequencer.validatorPrivateKey PRIV_KEY \
  --sequencer.coinbase CUZDAN_ADRESI \
  --p2p.p2pIp SUNUCU_IP
```

ALCHEMY_URL: Alchemy'den aldığımız endpoint URL'si
BEACON_URL: https://rpc.drpc.org/eth/sepolia/beacon
PRIV_KEY: Cüzdan özel anahtarı (Metamask>3 nokta>hesap bilgileri>özel anahtarı göster)
CUZDAN_ADRESI: Özel anahtarını girdiğiniz cüzdanın adresi
SUNUCU_IP: Sunucunuzun IP'si

![Ekran görüntüsü 2025-05-02 094206](https://github.com/user-attachments/assets/acc90d7d-b848-4f89-a10a-4e2caeeffea0)

Şimdi node senkronize olacak. Peerlar eşleşiyor. 100 olunca nodeumuz sync halde olacak.

![Ekran görüntüsü 2025-05-02 095052](https://github.com/user-attachments/assets/b87c9ac3-9456-4a84-970e-00c84efa9ee1)

Nodeumuz senkronize oldu.

# Discord'a Girelim

![image](https://github.com/user-attachments/assets/9512236e-1080-4ae4-a0c2-f82d3ed82e73)

İşaretli kanala ```/operator start``` yazalım

![image](https://github.com/user-attachments/assets/55cc0787-e8f5-420e-8bb1-490bf0f3184d)

Address: Cüzdan adresi
block-number: Aşağıdaki kodu yazınca çıkan 5 haneli numara

```
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

proof: Aşağıdaki kodda 2 yerde BLOCK_NUMBER yerine 5 haneli numarayı yazınca çıkan hash

```
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```

Bu şekilde Apprentice Dc'de rolünü alırsınız.

# Validator Kaydı

```
aztec add-l1-validator \
  --l1-rpc-urls RPC_URL \
  --private-key PRIV_KEY \
  --attester CUZDAN_ADRESI \
  --proposer-eoa CUZDAN_ADRESI \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111

RPC_URL, PRIV_KEY ve CUZDAN_ADRESI önceki gibi düzenleyelim. Diğer yerler aynı kalıyor.
```

# BOL KAZANÇLAR
