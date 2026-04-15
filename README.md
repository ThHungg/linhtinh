curl -LO https://github.com/hyperledger/fabric-ca/releases/download/v1.5.10/hyperledger-fabric-ca-linux-amd64-1.5.10.tar.gz

tar -xzf hyperledger-fabric-ca-linux-amd64-1.5.10.tar.gz -C .


echo 'export PATH=$HOME/fabric-samples/bin:$PATH' >> ~/.bashrc
echo 'export FABRIC_CFG_PATH=$HOME/fabric-samples/config' >> ~/.bashrc
source ~/.bashrc



docker pull hyperledger/fabric-peer:3.1
docker pull hyperledger/fabric-orderer:3.1
docker pull hyperledger/fabric-ccenv:3.1
docker pull hyperledger/fabric-ca:1.5.10

# (tùy môi trường, Docker có thể tự kéo thêm baseos khi cần)


docker images "hyperledger/*" --format "{{.Repository}}:{{.Tag}}" | sort


git clone https://github.com/PacktPublishing/Blockchain-Development-for-Finance-Projects.git

cd Blockchain-Development-for-Finance-Projects/Chapter4/bankchain


docker compose -f docker-compose-bankchain.yaml config --services

docker compose -f docker-compose-bankchain.yaml up -d 

docker compose -f docker-compose-bankchain.yaml up -d orderer.bankchain.com

docker compose -f docker-compose-bankchain.yaml up -d cli

docker compose -f docker-compose-bankchain.yaml ps

# In your bankchain app folder (where connection-banka.json is)

# Cài deps một lần
npm i fabric-ca-client fabric-network

# Enroll admin và tạo user
node enrollAdmin-BankA.js
node registerUser-BankA.js

node enrollAdmin-BankB.js
node registerUser-BankB.js 

# verify files appear
ls -l wallet-BankA


docker compose -f docker-compose-bankchain.yaml up -d cli
docker exec -it cli bash


cd ~/fabric-samples/bankchain

# Lấy tên network của các container (ví dụ: bankchain_default)
docker inspect peer0.banka.bankchain.com \
  --format '{{range $k,$v := .NetworkSettings.Networks}}{{println $k}}{{end}}'


export NET=bankchain_bankchain-net

docker run --rm -it --network $NET \
  -v "$PWD:/work" -w /work \
  -e FABRIC_CFG_PATH=/work/config \
  hyperledger/fabric-tools:3.1 bash
