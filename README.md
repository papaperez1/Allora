# Allora Incentivized testnet v2 Worker Setup with 1 topic
![image](https://github.com/user-attachments/assets/68b1cf9f-15ac-41f5-9f38-503eceada52c)

```
If you are already running a worker just update these new changes for v2 testnet:
1. Get testnet tokens from new faucet url: https://faucet.testnet-1.testnet.allora.network/
2. new rpc url: https://explorer.testnet-1.testnet.allora.network
3. Update in docker compose file with this new explorer url
4. stop docker compose and rebuild the compose file
5. Update worker status curl command
```
__________________
### Login to Dashboard
Link: https://shorturl.at/PTix1
Connect your keplr wallet and note down your alloxx address
![image](https://github.com/papaperez1/Allora/assets/118633093/48048769-5906-4769-8ba2-c5b4c35f5f87)

Points will be added retrospectively later.

### Amount Raised 
Allora raised a total of $33.75M from Delphi, Polychain Capital among other Investors 
![image](https://github.com/papaperez1/Allora/assets/118633093/1b0aa7b8-16f0-4862-a9d8-26c0b2689c7c)

### Min System Requirement
CPU: Minimum of 1/2 core.
Memory: 2 to 4 GB.
Storage: SSD or NVMe with at least 5GB of space.
OS :UBUNTU 22.04 LTS

### Install Prerequisite
```
sudo su -
```
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

### Wallet Setup
```
curl -sSL https://raw.githubusercontent.com/allora-network/allora-chain/main/install.sh | bash -s -- v0.0.10
```
```
export PATH="$PATH:/root/.local/bin"
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
OR

Create a new wallet
```
allorad keys add testkey
```
Get faucet from https://faucet.testnet-1.testnet.allora.network/

![image](https://github.com/user-attachments/assets/89ff07de-b3de-4afa-9653-f9c960230656)


Check balance from https://explorer.testnet-1.testnet.allora.network. (connect wallet -> Click More as below)
![image](https://github.com/user-attachments/assets/0c059eff-b00c-4f73-a0cc-3bfc84661ed6)



### Setup coin prediction worker ( reference  @0xmoei)
```
cd $HOME && git clone https://github.com/allora-network/basic-coin-prediction-node && cd basic-coin-prediction-node
```
```
mkdir worker-data
mkdir head-data
```
```
sudo chmod -R 777 worker-data
sudo chmod -R 777 head-data
```
Create head keys
```
sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```
Create worker keys
```
sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```
Copy the head-id and keep it in notepad (copy before rootxxx as heighlighted)
```
cat head-data/keys/identity
```
![image](https://github.com/papaperez1/Allora/assets/118633093/0fe235f7-18d1-458f-9b46-c334583be6ad)
### Run the worker node
```
rm -rf docker-compose.yml && nano docker-compose.yml
```
Copy & Paste the following code in it
Replace `head-id` & `WALLET_SEED_PHRASE`
```
version: '3'

services:
  inference:
    container_name: inference-basic-eth-pred
    build:
      context: .
    command: python -u /app/app.py
    ports:
      - "8000:8000"
    networks:
      eth-model-local:
        aliases:
          - inference
        ipv4_address: 172.22.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/inference/ETH"]
      interval: 10s
      timeout: 5s
      retries: 12
    volumes:
      - ./inference-data:/app/data

  updater:
    container_name: updater-basic-eth-pred
    build: .
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
    command: >
      sh -c "
      while true; do
        python -u /app/update_app.py;
        sleep 24h;
      done
      "
    depends_on:
      inference:
        condition: service_healthy
    networks:
      eth-model-local:
        aliases:
          - updater
        ipv4_address: 172.22.0.5

  worker:
    container_name: worker-basic-eth-pred
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
      - HOME=/data
    build:
      context: .
      dockerfile: Dockerfile_b7s
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        # Change boot-nodes below to the key advertised by your head
        allora-node --role=worker --peer-db=/data/peerdb --function-db=/data/function-db \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9011 \
          --boot-nodes=/ip4/172.22.0.100/tcp/9010/p2p/head-id \
          --topic=allora-topic-1-worker \
          --allora-chain-key-name=testkey \
          --allora-chain-restore-mnemonic='WALLET_SEED_PHRASE' \
          --allora-node-rpc-address=https://allora-rpc.testnet-1.testnet.allora.network/ \
          --allora-chain-topic-id=1
    volumes:
      - ./worker-data:/data
    working_dir: /data
    depends_on:
      - inference
      - head
    networks:
      eth-model-local:
        aliases:
          - worker
        ipv4_address: 172.22.0.10

  head:
    container_name: head-basic-eth-pred
    image: alloranetwork/allora-inference-base-head:latest
    environment:
      - HOME=/data
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        allora-node --role=head --peer-db=/data/peerdb --function-db=/data/function-db  \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9010 --rest-api=:6000
    ports:
      - "6000:6000"
    volumes:
      - ./head-data:/data
    working_dir: /data
    networks:
      eth-model-local:
        aliases:
          - head
        ipv4_address: 172.22.0.100


networks:
  eth-model-local:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/24

volumes:
  inference-data:
  worker-data:
  head-data:
```
Build and run the image
```
docker compose build
docker compose up -d
```
```
docker ps
```
copy the container ID of the worker and run the below command
![image](https://github.com/papaperez1/Allora/assets/118633093/c165c1f2-a357-4572-97ba-a545acc08f6e)

```
docker logs -f CONTAINER_ID
```
![image](https://github.com/papaperez1/Allora/assets/118633093/72dd4f6e-a10a-43c8-afd7-ef862d281dd2)

### Check node status
```
apt install jq
```
```
network_height=$(curl -s -X 'GET' 'https://allora-rpc.testnet-1.testnet.allora.network/abci_info?' -H 'accept: application/json' | jq -r .result.response.last_block_height) && \
curl --location 'http://localhost:6000/api/v1/functions/execute' --header 'Content-Type: application/json' --data '{
    "function_id": "bafybeigpiwl3o73zvvl6dxdqu7zqcub5mhg65jiky2xqb4rdhfmikswzqm",
    "method": "allora-inference-function.wasm",
    "parameters": null,
    "topic": "1",
    "config": {
        "env_vars": [
            {
                "name": "BLS_REQUEST_PATH",
                "value": "/api"
            },
            {
                "name": "ALLORA_ARG_PARAMS",
                "value": "ETH"
            },
            {
                "name": "ALLORA_BLOCK_HEIGHT_CURRENT",
                "value": "'"${network_height}"'"
            }
        ],
        "number_of_nodes": -1,
        "timeout": 10
    }
}' | jq
```
Response:
![image](https://github.com/user-attachments/assets/e48a687c-28ec-499d-bcb2-f6fe3bee3681)


### Step to Restart docker containers for Troubleshooting

```
docker compose down
```
```
docker compose up -d
```

References: 
https://docs.allora.network/
https://github.com/0xmoei/allora-testnet/edit/main/README.md
