# Allora Worker Setup


__________________
## Amount Raised 
Allora raised a total of $33.75M from Delphi, Polychain Capital among other Investors 
![image](https://github.com/papaperez1/Allora/assets/118633093/1b0aa7b8-16f0-4862-a9d8-26c0b2689c7c)

### Min System Requirement
CPU: Minimum of 1/2 core.
Memory: 2 to 4 GB.
Storage: SSD or NVMe with at least 5GB of space.

### Install Perequisite 
```
sudo apt update
sudo apt upgrade -y
```
### Install Go
```
curl -OL https://go.dev/dl/go1.22.4.linux-amd64.tar.gz
```
```
tar -C /usr/local -xvf go1.22.4.linux-amd64.tar.gz
```
```
export PATH=$PATH:/usr/local/go/bin
```
```
go version
```
### Install Python3
```
apt install python3-pip
```
### Install Docker
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Install Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/2.28.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```
```
docker compose version
```

### Install allocmd CLI
```
pip install allocmd --upgrade
```
```
allocmd --version
```
### Generate worker scaffold files
```
allocmd generate worker --name papaperez --topic 1 --env dev
```
![image](https://github.com/papaperez1/Allora/assets/118633093/43015389-07ba-47be-838b-ed4f573e19d4)
```
cd papaperez/worker/data
```
```
mkdir head
```
```
mkdir worker
```
```
chmod -R 777 head
```
```
chmod -R 777 worker
```
![image](https://github.com/papaperez1/Allora/assets/118633093/e0c02c71-fc1f-429e-a4c9-9d51ed93f971)
### Wallet Setup
```
curl -sSL https://raw.githubusercontent.com/allora-network/allora-chain/main/install.sh | bash -s -- v0.0.10
```
```
git clone -b v0.0.10 https://github.com/allora-network/allora-chain.git
```
```
cd allora-chain && make all
```
```
allorad version
```

Recover your wallet with seed-phrase (for keyring passphrase enter wallet password)
```
allorad keys add testkey --recover
```
#OR

# Create a new wallet
```
allorad keys add testkey
```
Get faucet from https://faucet.testnet.allora.network/ and make sure you imported the wallet in keplr.
