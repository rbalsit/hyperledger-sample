/home/ubuntu/orderer

copy bin directory from fabric samples 

create certs:
bin/cryptogen generate --config=crypto.yaml --output="organizations"
 
create genesis block: 
bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block

copy docker-compose file and change cert path

docker-compose up -d

bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx mychannel.tx -channelID mychannel

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=./organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=./organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_TLS_ENABLED=true


export PATH=/home/ubuntu/orderer/bin:${PWD}:$PATH
export FABRIC_CFG_PATH=/home/ubuntu/orderer

copy core.yaml from fabric-samples/config dir to orderer dir

peer channel create -o localhost:7050 -c mychannel --ordererTLSHostnameOverride orderer.example.com -f mychannel.tx --outputBlock mychannel.block --tls --cafile ./organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer channel join -b mychannel.block 

deployCC:
ref : https://hyperledger-fabric.readthedocs.io/en/latest/cc_service.html


export CC_NAME=basic
export CC_SRC_PATH=/home/ubuntu/fabric-samples/asset-transfer-basic/chaincode-go
export CC_RUNTIME_LANGUAGE=golang
export CC_VERSION=1.0
export CHANNEL_NAME=mychannel



set x
peer lifecycle chaincode package ${CC_NAME}.tar.gz --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label ${CC_NAME}_${CC_VERSION}
>/dev/null


peer lifecycle chaincode install ${CC_NAME}.tar.gz

peer lifecycle chaincode queryinstalled


export ORDERER_CA=./organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export PACKAGE_ID=$(peer lifecycle chaincode queryinstalled | sed -n "/${CC_NAME}_${VERSION}/{s/^Package ID: //; s/, Label:.*$//; p;}")
export CC_VERSION=1.0
export CC_SEQUENCE=1
export INIT_REQUIRED=false
export CC_END_POLICY=
export CC_COLL_CONFIG=
export PEER_CONN_PARMS=


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name ${CC_NAME} --version ${CC_VERSION} --package-id ${PACKAGE_ID} --sequence ${CC_SEQUENCE} 


peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name ${CC_NAME} --version ${CC_VERSION} --sequence ${CC_SEQUENCE} 

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name ${CC_NAME} $PEER_CONN_PARMS --version ${CC_VERSION} --sequence ${CC_SEQUENCE}  ${CC_END_POLICY} ${CC_COLL_CONFIG}

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=./organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=./organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name ${CC_NAME} --version ${CC_VERSION} --package-id ${PACKAGE_ID} --sequence ${CC_SEQUENCE} 

(or)

    peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com \
        --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA \
        --channelID $CHANNEL_NAME --name ${CC_NAME} \
        --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
        --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
        --peerAddresses localhost:11051 --tlsRootCertFiles $PEER0_ORG3_CA \
        --version ${VERSION} --sequence ${SEQUENCE} --init-required

peer lifecycle chaincode querycommitted --channelID $CHANNEL_NAME --name ${CC_NAME}



