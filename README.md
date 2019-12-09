# Hyperledger-meetup demo (18-09-2019)

*Tested on 18.04 Ubuntu*

### Table of contents
* [Demo 1](#demo-1)
* [Demo 2](#demo-2)

## Demo-1

### Setup - General
1. Install docker-ce community edition > 17.06. [Fabric pre-reqs](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html),
[Docker-ce](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1)
```
sudo apt install -y docker.io
sudo usermod -a -G docker $USER
newgrp docker
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
sudo apt install golang-go -y
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
source ~/.bashrc
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

## Demo-2

### Create a successful/unsuccessful transaction
0. Set some environment variable
```
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
1. Query the value of 'a' and 'b'
```
peer chaincode query -C mychannel -n mycc -c '{
"Args":["query","a"]}'
peer chaincode query -C mychannel -n mycc -c '{
"Args":["query","b"]}'
```
2. Create a transaction with both the org1 and org2 peers
```
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA -C mychannel -n mycc \
--peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
--peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
-c '{"Args":["invoke","a","b","10"]}'
```

3. Repeat step 1 to query the value of 'a' and 'b'

**Did you notice the change in values?**

4. Create a transaction with ONLY the org1 peer
```
peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}' --tls --cafile $ORDERER_CA
```

5. Repeat step 1 to query the value of 'a' and 'b'

**Was there a change in values?**

6. Open the explorer to witness what had happened.

Go to the `transactions` tab in the top
