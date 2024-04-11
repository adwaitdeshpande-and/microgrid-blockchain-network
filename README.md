# Microgrid
Hyperledger network for microgrids


## Basic system setup

Assumes Ubuntu

### Check docker version
```bash
docker --version
```
Docker compose version greater than 1.14.0 is required
```bash
docker-compose --version
```

### Check GoLang
Check
```bash
go version
```

### Add go location to ```GOPATH``` and ```GOPATH``` to ```PATH```
Add path to ```.profile```
```bash
echo 'export GOPATH=/usr/local/go' >> ~/.profile
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile
source ~/.profile
```


### Download and install fabric binaries version 1.2 into current directory
```bash
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh

./install-fabric.sh --fabric-version 1.2.0 binary
```
### Clone git repo
```bash
git clone https://github.com/adwaitdeshpande-and/microgrid-blockchain-network.git
 
```
## Set up the network

### Quick
Configure
```bash
bash configure.sh
```
Start
```bash
bash start.sh
```

Stop
```bash
bash stop.sh
```

### Step-by-step

```bash
$ ../bin/cryptogen generate --config crypto-config.yaml --output=crypto-config
```

These lines should be printed:
```bash
house01.microgrid.org
house02.microgrid.org
house03.microgrid.org
house04.microgrid.org
house05.microgrid.org
house06.microgrid.org
```

and a directory called ```crypto-config``` should created.

#### Create the orderer genesis block, channel configuration transaction and the anchor peer transactions, one for each peer organisation

The ```configtxgen``` command doesn't automatically create the ```channel-artifacts``` directory.
Do it manually.
```bash
$ mkdir channel-artifacts
```

##### Genesis block:
```bash
$ ../bin/configtxgen -profile OrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

The last line printed should look something like:
```bash
2018-08-21 17:26:31.191 EDT [common/tools/configtxgen] doOutputBlock -> INFO 013 Writing genesis block
```

##### Channel configuration transaction:
```bash
$ ../bin/configtxgen -profile hachannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID hachannel
```
```channel.tx``` should be created in ```channel-artifacts``` and the last line printed should look something like this:
```
2024-03-02 14:38:49.079 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 010 Writing new channel tx
```

##### Anchor peer transactions:
```bash
$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House01Anchor.tx -channelID hachannel -asOrg House01MSP

$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House02Anchor.tx -channelID hachannel -asOrg House02MSP

$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House03Anchor.tx -channelID hachannel -asOrg House03MSP

$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House04Anchor.tx -channelID hachannel -asOrg House04MSP

$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House05Anchor.tx -channelID hachannel -asOrg House05MSP

$ ../bin/configtxgen -profile hachannel -outputAnchorPeersUpdate ./channel-artifacts/House06Anchor.tx -channelID hachannel -asOrg House06MSP
```

```House01Anchor.tx```, ```House02Anchor.tx``` ... should be located in ```channel-artifacts```
and the last lines printed should look something like this:
```
2024-03-02 14:46:37.491 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2024-03-02 14:46:37.499 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2024-03-02 14:46:37.499 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```
#### Peer-base and docker-compose

```bash
$ docker-compose -f docker-compose-cli.yaml up -d
```

The last few lines printed should look something like this:
```bash
dde25187799d: Pull complete
Digest: sha256:24cca44a2f2ab6325c6ccc1c91a10bd3e0e71764037a85a473f7e9621b3a0f91
Status: Downloaded newer image for hyperledger/fabric-tools:latest
Creating peer1.house03.microgrid.org ... done
Creating peer0.house01.microgrid.org ... done
Creating peer1.house02.microgrid.org ... done
Creating peer0.house02.microgrid.org ... done
Creating peer1.house06.microgrid.org ... done
Creating peer1.house01.microgrid.org ... done
Creating peer1.house04.microgrid.org ... done
Creating peer0.house04.microgrid.org ... done
Creating peer0.house05.microgrid.org ... done
Creating peer0.house06.microgrid.org ... done
Creating orderer.microgrid.org       ... done
Creating peer0.house03.microgrid.org ... done
Creating peer1.house05.microgrid.org ... done
Creating cli                         ... done
Creating chaincode                   ... done
```
Start docker cli:
```bash
docker start cli
```
Enter the ```cli``` docker container:
```bash
docker exec -it cli bash
```
Should look something like this:

```
root@b411f1d6d95e:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

#### Stop network and remove docker images
```bash
docker-compose -f docker-compose-cli.yaml down --volumes
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
```


## Create the channel

First enter the cli:
```
docker exec -it cli bash # Enter the cli
```
and then create the channel:
```bash
export CHANNEL_NAME=hachannel
peer channel create -o orderer.microgrid.org:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/microgrid.org/orderers/orderer.microgrid.org/msp/tlscacerts/tlsca.microgrid.org-cert.pem
```
Alternatively, without entering the cli
```bash
docker exec -ti cli sh -c "peer channel create -o orderer.microgrid.org:7050 -c hachannel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/microgrid.org/orderers/orderer.microgrid.org/msp/tlscacerts/tlsca.microgrid.org-cert.pem"
```

## Join peer to the channel

Join peer0.house01.microgrid.org to the channel

From the cli:
```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/users/Admin@house01.microgrid.org/msp CORE_PEER_ADDRESS=peer0.house01.microgrid.org:7051 CORE_PEER_LOCALMSPID="House01MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/peers/peer0.house01.microgrid.org/tls/ca.crt peer channel join -b hachannel.block

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/users/Admin@house01.microgrid.org/msp CORE_PEER_ADDRESS=peer1.house01.microgrid.org:7051 CORE_PEER_LOCALMSPID="House01MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/peers/peer1.house01.microgrid.org/tls/ca.crt peer channel join -b hachannel.block
```

Without entering the cli:
```bash
docker exec -ti cli sh -c 'CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/users/Admin@house01.microgrid.org/msp CORE_PEER_ADDRESS=peer0.house01.microgrid.org:7051 CORE_PEER_LOCALMSPID="House01MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/peers/peer0.house01.microgrid.org/tls/ca.crt peer channel join -b hachannel.block'
```

## Chaincode

### Quick
```
bash setup_chaincode.sh
```

### Step-by-step

Assumes you are in ```microgrid```
```bash
$ cd chaincode/go/src
```
Test compile the chaincode outside of a container
```
$ go get -u github.com/hyperledger/fabric-chaincode-go/shim
$ go build
```

Install the chaincode
```bash
CC_SRC_PATH=chaincode/go/src
docker exec -e "CORE_PEER_LOCALMSPID=House01MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/house01.microgrid.org/users/Admin@house01.microgrid.org/msp" cli peer chaincode install -n carecords -v 1.0 -p "$CC_SRC_PATH" -l golang
```

The last lines printed should be something like this:
```bash
2024-03-05 20:48:05.131 UTC [container] WriteFileToPackage -> DEBU 04e Writing file to tarball: src/chaincode/go/src/carecords.go
2024-03-05 20:48:05.135 UTC [msp/identity] Sign -> DEBU 04f Sign: plaintext: 0ACA070A5B08031A0B088582C1DC0510...F7C67F020000FFFF6A084374001A0000
2024-03-05 20:48:05.135 UTC [msp/identity] Sign -> DEBU 050 Sign: digest: 88E3BE1783D75457E36882431DCA4EDC77A7996F0C0BD3524E8DB208387A9B0B
2024-03-05 20:48:05.137 UTC [chaincodeCmd] install -> INFO 051 Installed remotely response:<status:200 payload:"OK" >
```


Instantiate the chaincode in the channel (hachannel)
```bash
docker exec -ti cli sh -c "peer chaincode instantiate -o orderer.microgrid.org:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/microgrid.org/orderers/orderer.microgrid.org/msp/tlscacerts/tlsca.microgrid.org-cert.pem -C hachannel -n carecords -v 1.0 -c '{\"Args\":[\"init\",\"a\", \"100\", \"b\",\"200\"]}' -P \"OR ('House01MSP.member','House02MSP.member')\""
```
## Test Chaincode
Test Carecords
```bash
bash test_carecords.sh
```
Test MSP
```
bash test_msp.sh
```

## Application
