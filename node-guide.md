![STORYGUIDE](https://github.com/user-attachments/assets/fc64ceef-46ff-42ec-8bdf-55bb7459f4ac)

# Step-by-step guide to setting up and running a node on the Story blockchain

### Table of Contents
- [System Specs](#system-specs)
- [Default folder](#default-folder)
- [Install dependencies](#install-dependencies)
- [Install Go 23](#install-go-23)
- [Install piplabs](#install-piplabs)
- [Download Story-Geth binary](#download-story-geth-binary)
- [Download Story binary](#download-story-binary)
- [Init Iliad node](#init-iliad-node)
- [Create story-geth service file](#create-story-geth-service-file)
- [Create story service file](#create-story-service-file)
- [Reload and start story-geth](#reload-and-start-story-geth)
- [Reload and start story](#reload-and-start-story)
- [Check logs](#check-logs)
  - [story-geth](#story-geth)
  - [story](#story)
- [Check sync status](#check-sync-status)
- [Create Validator](#create-validator)

![LINEA](https://github.com/user-attachments/assets/6cbf6840-7d91-482b-9f97-bdbaf8187e9f)

## System Specs   

**Hardware	Requirement**  
CPU	4 Cores  
RAM	8 GB  
Disk	200 GB  
Bandwidth	10 MBit/s  

**Ubuntu min version**  
22.04  

**story-geth version**  
version: 0.9.2-stable  

**story version**  
v0.9.11-stable   

## Default folder
By default, we set up the following default data folders for the consensus and execution clients:   
o	Story root data:~/.story/story  
o	story-geth data root: ~/.story/geth  

### CHAIN_ID="iliad"

## Install dependencies  
```bash
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y
```

## Install Go 23  
```bash
cd $HOME
VERSION=1.23.0

# Remove any previous Go installation
sudo rm -rf /usr/local/go

# Download and install the specified version of Go
wget -O go.tar.gz https://go.dev/dl/go$VERSION.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go.tar.gz
rm go.tar.gz

# Update environment variables to use the new Go installation
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$GOROOT/bin:$GOPATH/bin:$PATH' >> $HOME/.bash_profile

# Reload the bash profile to apply the changes
source $HOME/.bash_profile

# Verify the installation
go version

```

## Install piplabs
```bash
# install Story
cd $HOME
rm -rf story
git clone https://github.com/piplabs/story
cd story
git checkout v0.11.0
go build -o story ./client 
mv $HOME/story/story $HOME/go/bin/
```

## Download Story-Geth binary
```bash
cd $HOME
wget -O geth https://github.com/piplabs/story-geth/releases/download/v0.9.4/geth-linux-amd64
chmod +x $HOME/geth
mv $HOME/geth ~/go/bin/
[ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
[ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"
story-geth version
```

## Download Story binary
```bash
wget -O story-linux-amd64-0.9.11-2a25df1.tar.gz https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar xvf story-linux-amd64-0.9.11-2a25df1.tar.gz
sudo chmod +x story-linux-amd64-0.9.11-2a25df1/story
sudo mv story-linux-amd64-0.9.11-2a25df1/story /usr/local/bin/
story version
```

## Init Iliad node  
```bash
story init --network iliad --moniker <your_moniker>
```
![image](https://github.com/user-attachments/assets/c9e49230-4c08-407f-a564-a2fe17d596b1)

## Create story-geth service file  
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF  
[Unit]
Description=Story execution daemon
After=network-online.target

[Service]
User=$USER
#WorkingDirectory=$HOME/.story/geth
ExecStart=/usr/local/bin/story-geth --iliad --syncmode full
Restart=always
RestartSec=3
LimitNOFILE=infinity
LimitNPROC=infinity

[Install]
WantedBy=multi-user.target
EOF
```

## Create story service file  
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF  
[Unit]
Description=Story consensus daemon
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.story/story
ExecStart=/usr/local/bin/story run
Restart=always
RestartSec=3
LimitNOFILE=infinity
LimitNPROC=infinity

[Install]
WantedBy=multi-user.target
EOF

sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```

## Reload and start story-geth   
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story-geth && \
sudo systemctl enable story-geth && \
sudo systemctl status story-geth
```

## Reload and start story    
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl enable story && \
sudo systemctl status story
```

## Check logs  
story-geth  
```bash
sudo journalctl -u story-geth -f -o cat
```
story   
```bash
sudo journalctl -u story -f -o cat
```

## Check sync status
```bash
curl localhost:26657/status | jq
```
![image](https://github.com/user-attachments/assets/0b6be018-522a-4aab-ac4d-57f0572505e3)

## Create Validator  
Export validator Public Key & Private key  
```bash
story validator export
```
Export the derived EVM private key
```bash
story validator export --export-evm-key
```
Faucet link:  [https://faucet.story.foundation/](https://faucet.story.foundation/)

Check key wallet  
```bash
story validator export | grep "EVM Public Key:" | awk '{print $NF}'
```

Create Validator  
```bash
story validator create --stake 1000000000000000000 --private-key <private-key>
```

