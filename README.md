# Hyperledger-meetup demo (18-09-2019)

## Demo 1 (Tested on 18.04)

### Setup - General
1. Install docker-ce community edition > 17.06. [Fabric pre-reqs](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html),
[Docker-ce](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1)
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo usermod -a -G docker ubuntu
```

2. Install `docker-compose`
 ```
wget -O- https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m) | sudo tee /usr/local/bin/docker-compose >/dev/null
sudo chmod a+x /usr/local/bin/docker-compose
 ```

3. Install golang
```
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go
```

4. Verify the installations
```
docker version
docker-compose -v
go version
```

### Setup - Hyperledger fabric
1. Install fabric 1.4.3
```
mkdir -p ~/hyperledger
cd ~/hyperledger
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.3 1.4.3 0.4.15
```

2. Setup path
```
cat >> ~/.bashrc << EOF
export PATH=$PATH:~/hyperledger/fabric-samples/bin
EOF
. ~/.bashrc
```

3. Launch fabric cluster
```
cd ~/hyperledger/fabric-samples/first-network
echo 'y' |./byfn.sh generate -s couchdb -o kafka
echo 'y' |./byfn.sh up -s couchdb -o kafka up
```

### Setup - Hyperledger explorer
1. Install hyperledger explorer
```
mkdir -p ~/hyperledger
cd ~/hyperledger
git clone https://github.com/cloudronics/blockchain-explorer.git
```

2. Modify the Admin user's keystore in the docker file accordingly
```
cd ~/hyperledger/blockchain-explorer
sed -i "s/TO_REPLACE/$(ls  ~/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/)/g" examples/net1/connection-profile/first-network.json
```
3. Launch explorer for the cluster
```
docker-compose up
```

### TearDown - Hyperledger fabric/explorer
1. TearDown hyperledger fabric
```
cd ~/hyperledger/fabric-samples/first-network
echo 'y' |./byfn.sh down
```

2. TearDown hyperledger explorer
```
cd ~/hyperledger/blockchain-explorer
docker-compose down
```
