Dependencies Installation

**Install dependencies for building from source**
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
Node Installation
```

**Clone project repository**
```
cd && rm -rf bcna
git clone https://github.com/BitCannaGlobal/bcna
cd bcna
git checkout v3.1.0
```

**Build binary**
```
make install
```
**Set node CLI configuration**

```
bcnad config chain-id bitcanna-dev-1
bcnad config keyring-backend test
bcnad config node tcp://localhost:13057
```

**Initialize the node**
```
bcnad init "Your Node Name" --chain-id bitcanna-dev-1
```

**Download genesis and addrbook files**
```
curl -L https://snapshots-testnet.nodejumper.io/bitcanna-testnet/genesis.json > $HOME/.bcna/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/bitcanna-testnet/addrbook.json > $HOME/.bcna/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "471341f9befeab582e845d5e9987b7a4889c202f@144.91.89.66:26656,496ac0ba20188f70f41e0a814dfd4d9a617338f8@bcnadev-seed.ibs.team:16656,80ee9ed689bfb329cf21b94aa12978e073226db4@212.227.151.143:26656,20ca909b49106aacbf516ba28fa8a2409f825a82@212.227.151.106:26656"|' $HOME/.bcna/config/config.toml
```

**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ubcna"|' $HOME/.bcna/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.bcna/config/app.toml
```

**Change ports**
```
sed -i -e "s%:1317%:13017%; s%:8080%:13080%; s%:9090%:13090%; s%:9091%:13091%; s%:8545%:13045%; s%:8546%:13046%; s%:6065%:13065%" $HOME/.bcna/config/app.toml
sed -i -e "s%:26658%:13058%; s%:26657%:13057%; s%:6060%:13060%; s%:26656%:13056%; s%:26660%:13061%" $HOME/.bcna/config/config.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots-testnet.nodejumper.io/bitcanna-testnet/bitcanna-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.bcna"
```

**Create a service**
```
sudo tee /etc/systemd/system/bcnad.service > /dev/null << EOF
[Unit]
Description=Bitcanna node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which bcnad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable bcnad.service
```

**Start the service and check the logs**
```
sudo systemctl start bcnad.service
sudo journalctl -u bcnad.service -f --no-hostname -o cat
Secure Server Setup (Optional)
```

**generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE**
```
ssh-keygen -t rsa
```

**save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY**
```
cat ~/.ssh/id_rsa.pub
```

**upgrade system packages**
```
sudo apt update
sudo apt upgrade -y
```

**add new admin user**
```
sudo adduser admin --disabled-password -q
```

**upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above**
```
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys

echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

**disable root login, disable password authentication, use ssh keys only**
```
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config

sudo systemctl restart sshd
```

**install fail2ban**
```
sudo apt install -y fail2ban
```

**install and configure firewall**
```
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656
```

**make sure you expose ALL necessary ports, only after that enable firewall**
```
sudo ufw enable
```

**make terminal colorful**
```
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)
```

**update servername, if needed, replace YOUR_SERVERNAME with wanted server name**
```
sudo hostnamectl set-hostname YOUR_SERVERNAME
```

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
