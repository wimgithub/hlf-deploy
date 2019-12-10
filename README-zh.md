# Hyperledger Fabric 部署

本项目不建议零基础直接上手, 可以先根据官方的 
[fabric-samples/first-network](https://github.com/hyperledger/fabric-samples) 入门

## 已支持功能

- createChannel
- updateAnchorPeer
- joinChannel
- installChaincode
- instantiateChaincode
- upgradeChaincode
- invokeChaincode
- queryChaincode
- addOrgChannel (动态添加组织, 支持 system channel)
- deleteOrgChannel (动态删除组织, 支持 system channel)
- changeOrgCertificate (动态修改组织证书, 支持 system channel)

## 启动测试网络

进入 `test-network` 目录

### 启动网络

```bash
./hlfn.sh up

# output
Creating network "test-network_byfn" with the default driver
Creating volume "test-network_orderer.example.com" with default driver
Creating volume "test-network_peer0.org1.example.com" with default driver
Creating volume "test-network_peer0.org2.example.com" with default driver
Creating volume "test-network_peer1.org1.example.com" with default driver
Creating volume "test-network_peer1.org2.example.com" with default driver
Creating peer0.org2.example.com ... done
Creating peer1.org2.example.com ... done
Creating peer0.org1.example.com ... done
Creating peer1.org1.example.com ... done
Creating orderer.example.com    ... done
2019/12/10 17:55:18 create channel txID: ec92b11651b4457cb10cceb4f67670ddc90f938c0f329fbce6116be6b1a50602
2019/12/10 17:55:18 Org1 update anchor peer txID: abd5de7ea4cc72728a60fa9437da766c8df990ebfb47de49d95353a210dcc81f
2019/12/10 17:55:18 Org2 update anchor peer txID: 2e4680f3c0409387cd499f4d521555f638612714de37355b9cfa6d87b17741fc
2019/12/10 17:55:18 Org1 join channel successful
2019/12/10 17:55:18 Org2 join channel successful
2019/12/10 17:55:18 Org1 install chaincode successful
2019/12/10 17:55:18 Org2 install chaincode successful
2019/12/10 17:55:24 Org1 instantiate chaincode txID: 969805f856bf98ac6aa9fc1afa2b52ff2343f5711c51677031d73e090c634beb args: [a 100 b 200]
2019/12/10 17:55:34 Org1 query chaincode txID: 356450b292f3636d6cbe913c1e2ffe7286c17e30819160170d49a093916cb669 args: [query a] result: 100
2019/12/10 17:55:34 Org1 query chaincode txID: fef70c04942c28308d30df3e62dccac1f7eb2244491c34ecb7b6b2a3af8134f4 args: [query b] result: 200
2019/12/10 17:55:36 Org1 invoke chaincode txID: 3fce7db1f779d9d8187f26cd66431aa88807ed32ca7996dc1c94c37c140e4296 args: [invoke a b 50]
2019/12/10 17:55:36 Org1 query chaincode txID: 09323d03d505a09918c9585d25c5dce2456702bda167b1f127ea80d5930979bb args: [query a] result: 50
2019/12/10 17:55:37 Org1 query chaincode txID: 01e939dc3aef2fc32100dba1cfc6c446448fcb103fbe84f20f2cf645b405235c args: [query b] result: 250
2019/12/10 17:55:37 save Org3MSP to mychannel txID: b9146c47a22470d31d9456abdfb7cf106384f019a5c1f2b0ad8a6fe172d736c9
2019/12/10 17:55:37 save Org3MSP to mychannel txID: 3d31bbdb4ad19d42be364ac1ae2326ee0a7ebbee799ae9751b840813e587b977
2019/12/10 17:55:37 delete Org3MSP to mychannel txID: 4ea217bf2d2ded1304e9df2b9dc3ef6764d4d4e72aab033e99880184834889e1
```

### 停止网络

```bash
./hlfn.sh down

# output
Stopping orderer.example.com    ... done
Stopping peer1.org2.example.com ... done
Stopping peer1.org1.example.com ... done
Stopping peer0.org1.example.com ... done
Stopping peer0.org2.example.com ... done
Removing orderer.example.com    ... done
Removing peer1.org2.example.com ... done
Removing peer1.org1.example.com ... done
Removing peer0.org1.example.com ... done
Removing peer0.org2.example.com ... done
Removing network test-network_byfn
Deleted Volumes:
test-network_orderer.example.com
test-network_peer0.org1.example.com
test-network_peer1.org2.example.com
test-network_peer0.org2.example.com
test-network_peer1.org1.example.com

Total reclaimed space: 1.475MB

```

## 启动网络 (单节点为例子)

### 创建集群网络

```bash
docker network create --driver=overlay --attachable hlf
```

### 启动 Orderer

```bash
ORDERER_HOSTNAME=orderer \
    ORDERER_DOMAIN=example.com \
    ORDERER_GENERAL_LOCALMSPID=OrdererMSP \
    FABRIC_LOGGING_SPEC=debug \
    NODE_HOSTNAME=master \
    NETWORK=hlf \
    PORT=7050 \
    NFS_ADDR=127.0.0.1 \
    NFS_PATH=/nfsvolume \
    docker stack up -c orderer.yaml orderer
```

### 启动 peer

使用 LevelDB

```bash
PEER_HOSTNAME=peer0 \
    PEER_DOMAIN=org1.example.com \
    FABRIC_LOGGING_SPEC=debug \
    CORE_PEER_LOCALMSPID=Org1MSP \
    NODE_HOSTNAME=master \
    NETWORK=hlf \
    PORT=7051 \
    NFS_ADDR=127.0.0.1 \
    NFS_PATH=/nfsvolume \
    docker stack up -c peer-leveldb.yaml peer0org1
```

使用 CouchDB

```bash
PEER_HOSTNAME=peer0 \
    PEER_DOMAIN=org1.example.com \
    FABRIC_LOGGING_SPEC=debug \
    CORE_PEER_LOCALMSPID=Org1MSP \
    NODE_HOSTNAME=master \
    NETWORK=hlf \
    PORT=7051 \
    NFS_ADDR=127.0.0.1 \
    NFS_PATH=/nfsvolume \
    docker stack up -c peer-couchdb.yaml peer0org1
```

### 启动 CA Server

```bash
PEER_DOMAIN=org1.example.com \
    NODE_HOSTNAME=master \
    USERNAME=admin \
    PASSWORD=adminpwd \
    NETWORK=hlf \
    PORT=7054 \
    NFS_ADDR=127.0.0.1 \
    NFS_PATH=/nfsvolume \
    CA_PRIVEATE_KEY=$(cd ${NFS_PATH}/crypto-config/peerOrganizations/${PEER_DOMAIN}/ca && ls *_sk) \
    docker stack up -c ca.yaml peer0org1ca
```

## 部署

### 创建 Channel

```bash
hlf-deploy createChannel --configFile config.yaml \
    --channelTxFile channel.tx \
    --channelName mychannel \
    --ordererOrgName OrdererOrg \
    Org1 Org2
```

### 更新 Anchor Peer

```bash
hlf-deploy uptateAnchorPeer --configFile config.yaml \
    --anchorPeerTxFile anchor.tx \
    --channelName mychannel \
    --ordererOrgName OrdererOrg \
    Org1
```

### 加入 Channel

```bash
hlf-deploy joinChannel --configFile config.yaml \
    --channelName mychannel \
    Org1 Org2
```

### 安装 Chaincode

```bash
hlf-deploy installChaincode --configFile config.yaml \
    --goPath ./chaincode \
    --chaincodePath example_02 \
    --chaincodeName example \
    --chaincodeVersion v1.0 \
    Org1 Org2
```

### 更新 Chaincode

`chaincodePolicy`: 设置需要哪些组织签名 (目前只支持 Member)
`chaincodePolicyNOutOf`: 用来设置多少个组织签名检验成功后返回 true

```bash
hlf-deploy upgradeChaincode --configFile config.yaml \
    --channelName mychannel \
    --orgName Org1 \
    --chaincodePolicy Org1MSP,Org2MSP \
    --chaincodePolicyNOutOf 2 \
    --chaincodePath example_02 \
    --chaincodeName mycc \
    --chaincodeVersion v1.0 \
    a 200 b 100
```

### 实例化 Chaincode

```bash
hlf-deploy instantiateChaincode --configFile config.yaml \
    --channelName mychannel \
    --orgName Org1 \
    --chaincodePolicy Org1MSP,Org2MSP \
    --chaincodePolicyNOutOf 1 \
    --chaincodePath example02 \
    --chaincodeName example \
    --chaincodeVersion v0.0.0 \
    a 100 b 200
```

### 查询 Chaincode

```bash
hlf-deploy queryChaincode --configFile config.yaml \
    --channelName mychannel \
    --orgName Org1 \
    --chaincodeName example \
    query a
```

### 调用 Chaincode

```bash
hlf-deploy invokeChaincode --configFile config.yaml \
    --channelName mychannel \
    --orgName Org1 \
    --endorsementOrgsName Org1,Org2 \
    --chaincodeName example \
    invoke a b 50
```
