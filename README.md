Contents
TỔNG QUAN KIẾN TRÚC HỆ THỐNG	3
1. Mạng Blockchain (Hyperledger Fabric Network)	3
2. Cơ sở dữ liệu ngân hàng (PostgreSQL Databases)	3
3. Hệ thống lưu trữ phi tập trung (IPFS – InterPlanetary File System)	3
4. Hệ thống Backend (Bank Server Applications)	3
5. Giao diện người dùng (Frontend – React App)	4
BỘ CÔNG CỤ ĐỂ CHẠY LAB CHAPTER 4	4
LAB 4.0. Chuẩn bị môi trường Hyperledger Fabric	5
LAB 4.1. Setting up the hyperledger fabric bankchain network	10
Thư mục dự án Bankchain tham khảo:	10
Trong phần LAB 4.1 cần các các file quan trọng:	10
Launching the network - Thực hiện 4 bước cho Fabric 3.x:	10
LAB 4.2. Creating blockchain identities for the banks	11
Các file cần có trong phần này:	11
Run utilities:	11
LAB 4.3. Building the corporate remittance contract	12
Điều kiện tiên quyết	13
Tạo các folder/file cần có trong phần này:	14
(Vì Fabric 3.x) chạy External Chaincode (Caas)	14
Thực hiện các bước sau:	14
LAB 4.4. Setting up the IPFS network	17
Bootstrap các node (kết nối chúng với nhau):	18
LAB 4.5. Setting up the bank databases	19
LAB 4.6. Building the bank backend servers	21
- Backend là “cầu nối” giữa Fabric, IPFS, PostgreSQL và frontend; nó cung cấp các API: /customerinfo, /payment, /gettrans, cùng các method iwrite, submitTrans, updateBal, insertTrans.	21
LAB 4.7. Building the transaction listeners for the banks	23
Transaction listeners làm gì?	23
- Các bước thực hiện:	24
LAB 4.8. Creating the corporate remittance app frontend in React	25
•	Container.js: Là “khung” điều hướng giữa các phần của app; sẽ render các tab/section tương ứng (login, chuyển tiền, xem giao dịch).	27
•	AppLogin.js: Màn hình đăng nhập chọn Bank A / Bank B; sau khi chọn sẽ gọi method trong App.js để set tài khoản/kết nối đúng backend.	27
•	Transfer.js: Form tạo “payment request”: nhập bên nhận, số tiền, tiền tệ và đính kèm chứng từ (invoice/BOE/BOL). Gửi lên endpoint /payment của backend.	27
•	ViewTransactions.js: Bảng lịch sử incoming/outgoing; mỗi dòng có link tải chứng từ từ public/uploads/<hash>.txt.	27
LAB 4.9. Running all	27

 
TỔNG QUAN KIẾN TRÚC HỆ THỐNG

Trong Chapter 4 - “Corporate Remittances and Settlement”, hệ thống được xây dựng là một mạng blockchain cho chuyển tiền doanh nghiệp (Corporate Remittance Network).	
Hệ thống này gồm 5 nhóm thành phần chính, mỗi nhóm đảm nhận một vai trò cụ thể trong toàn bộ kiến trúc 
1. Mạng Blockchain (Hyperledger Fabric Network)
- Đây là trung tâm của hệ thống, nơi các giao dịch chuyển tiền được ghi nhận, xác thực và bảo mật.
- Thành phần chính:	
•	Orderer Node: Quản lý việc sắp xếp thứ tự và đóng gói các giao dịch thành block.
•	Peer Nodes (Bank Peers): Đại diện cho các ngân hàng trong mạng (ví dụ: Bank A, Bank B). Mỗi ngân hàng có peer riêng để xác thực và lưu trữ bản sao sổ cái.
•	Certificate Authority (CA): Cấp chứng chỉ định danh số cho các thành phần (admin, user, client).
•	Channel: Kênh riêng biệt cho giao dịch giữa các ngân hàng, bảo mật dữ liệu chỉ cho các bên liên quan.
•	Chaincode (Smart Contract): Logic nghiệp vụ của chuyển tiền, triển khai trên peer nodes.
 Đây là nơi toàn bộ giao dịch remittance được xử lý và ghi vào blockchain.
2. Cơ sở dữ liệu ngân hàng (PostgreSQL Databases)
- Mỗi ngân hàng trong hệ thống có một cơ sở dữ liệu riêng để lưu trữ thông tin khách hàng và giao dịch nội bộ.
- Thành phần chính:
•	Customer Table: lưu thông tin khách hàng (tên, ID, tài khoản, địa chỉ,…).
•	Transaction Table: ghi lịch sử giao dịch của từng khách hàng.
•	Balance Table: theo dõi số dư của từng tài khoản.
Cơ sở dữ liệu này giúp kết hợp dữ liệu on-chain (blockchain) với dữ liệu off-chain (nội bộ ngân hàng) để hiển thị và xử lý linh hoạt.
3. Hệ thống lưu trữ phi tập trung (IPFS – InterPlanetary File System)
Dùng để lưu trữ các tài liệu giao dịch, chẳng hạn như hóa đơn, chứng từ tuân thủ (compliance docs), biên lai,…
Thành phần chính:
•	IPFS Nodes: các node lưu trữ dữ liệu và kết nối thành mạng chia sẻ tệp.
•	Key File & Config File: cấu hình danh tính và thiết lập mạng IPFS.
•	Utility Scripts: tiện ích để upload, retrieve file từ IPFS.
IPFS đảm bảo các tệp không thể bị chỉnh sửa, tăng tính minh bạch trong kiểm toán.
4. Hệ thống Backend (Bank Server Applications)
- Đây là phần trung gian giữa blockchain, cơ sở dữ liệu, và giao diện người dùng.
- Chức năng chính:
•	Giao tiếp với blockchain thông qua SDK của Hyperledger Fabric.
•	Gửi và nhận yêu cầu giao dịch.
•	Kết nối và lưu dữ liệu vào PostgreSQL.
•	Gọi IPFS API để lưu hoặc truy xuất tài liệu.
•	Cung cấp RESTful API cho frontend.
- Ví dụ:
•	POST /payment-request → tạo yêu cầu chuyển tiền mới.
•	GET /transactions → truy xuất danh sách giao dịch.
•	PUT /balance → cập nhật số dư khách hàng.
 Backend giúp kết nối toàn bộ các lớp trong hệ thống lại với nhau.
5. Giao diện người dùng (Frontend – React App)
- Đây là ứng dụng web dành cho nhân viên ngân hàng và người dùng doanh nghiệp để tương tác với hệ thống.
- Thành phần chính:
•	Login Component: xác thực người dùng (Bank Admin hoặc Customer).
•	Transfer Component: tạo yêu cầu chuyển tiền.
•	View Transactions Component: xem lịch sử và trạng thái giao dịch.
•	Compliance Upload: tải lên và xem tài liệu từ IPFS.
Giao diện này giao tiếp với backend qua API, hiển thị dữ liệu on-chain và off-chain một cách trực quan.

Tóm lại, hệ thống trong Chapter 4 gồm:
•	Blockchain Layer Hyperledger Fabric (Orderer, Peers, CA, Channel, Chaincode): Xử lý và ghi nhận giao dịch trên sổ cái phân tán.
•	Database Layer (PostgreSQL): Lưu trữ dữ liệu khách hàng & số dư ngoài chuỗi.
•	Storage Layer IPFS: Lưu trữ tài liệu giao dịch phi tập trung.
•	Backend Layer (Node.js/Express APIs): Trung gian kết nối blockchain – database - frontend.
•	Frontend Layer (ReactJS App): Giao diện người dùng cho ngân hàng & khách hàng.
BỘ CÔNG CỤ ĐỂ CHẠY LAB CHAPTER 4

1. Hyperledger Fabric Core
•	Peer / Orderer / Tools (cryptogen, configtxgen, peer, orderer) Version: 2.2.x (LTS) (ví dụ 2.2.0 hoặc 2.2.5)
•	Docker images:
o	hyperledger/fabric-peer:2.2
o	hyperledger/fabric-orderer:2.2
o	hyperledger/fabric-tools:2.2
2. Hyperledger Fabric CA
•	Vẫn dùng bản cũ 1.4.x, vì Fabric-CA chưa lên v2.
o	Docker image: hyperledger/fabric-ca:1.4.9
3. Database cho peer state
•	CouchDB: 3.1.1 (theo docker-compose trong sách) Image: couchdb:3.1.1
4. Ngôn ngữ chaincode
•	Go (dùng để viết chaincode backend) Version: 1.14.x hoặc 1.15.x (không dùng bản quá mới vì có thể lỗi với Fabric 2.2)
•	Ngoài Go, chaincode cũng có thể viết bằng Node.js, nhưng sách chủ yếu demo bằng Go.
5. Node.js (cho backend & frontend)
•	Node.js v12.x LTS Đúng theo hướng dẫn trong sách (curl -sL https://deb.nodesource.com/setup_12.x)
•	npm đi kèm với Node.js
6. Docker & Docker Compose
•	Docker CE mới nhất (20.x trở lên).
•	Docker Compose v1.27.x hoặc v1.29.x (chưa cần bản 2.x).
7. Các công cụ khác
•	cURL (để test REST API).
•	Git (clone fabric-samples, project code).
LAB 4.0. Chuẩn bị môi trường Hyperledger Fabric

1. Cài Máy ảo Ubuntu:
Ubuntu 24.04.3 LTS: 
https://ubuntu.com/download/desktop/thank-you?version=24.04.3&architecture=amd64&lts=true	

2. Chuẩn bị môi trường (Ubuntu)
- Cập nhật hệ thống
sudo apt update 

- Cài đặt các công cụ cần thiết: curl, git, docker.io:
sudo apt install -y curl
sudo apt install -y git
sudo apt install -y docker.io

Kiểm tra:
curl --version
git --version
docker --version
Curl: 8.5.0
Git: 2.43.0
Docker: 2.28.2

- Cài docker-compose:

Tải file Compose" V2 mới nhất cho x86_64 và đặt vào folder plugin:
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -fSL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64" \
  -o /usr/local/lib/docker/cli-plugins/docker-compose

sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

Kiểm tra:
docker compose version    #Xem phiên bản docker compose

Docker-compose: v2.40.0

Thêm user của bạn vào nhóm docker để chạy docker không cần sudo:
sudo usermod -aG docker $USER

- Cài cài Node.js 22.x LTS trên Ubuntu
Thêm NodeSource repo cho Node.js 22.x:
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
Cài Node.js:
sudo apt-get install nodejs -y
Kiểm tra phiên bản:
nodejs -v
npm -v

Nodejs: V22.20.0
npm: 10.9.3
- Cài Go (Golang) trên Ubuntu 18.04/20.04 
	Trong Chapter 4, chaincode được viết bằng Go → nên bắt buộc phải cài Go.
	Ngoài ra, nhiều công cụ CLI của Fabric (như peer) cũng build dựa trên Go.
	Hyperledger Fabric 3.1.x tương thích nhất với: Go 1.24.0 
Tải Go 1.24
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
Giải nén vào /usr/local
sudo tar -C /usr/local -xzf go1.24.0.linux-amd64.tar.gz
Thêm Go vào PATH
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
Áp dụng thay đổi:
source ~/.profile
Kiểm tra phiên bản đã cài:
go version

Go: 1.24.0

- Cài Hyperledger Fabric 3.1.x
1.	Tải Fabric samples mới nhất:  
git clone https://github.com/hyperledger/fabric-samples.git

cd fabric-samples
 
Kết quả xuất hiện thư mục fabric-samples:
 

2.	Tải và giải nén toàn bộ bộ “Fabric core binaries” phiên bản 3.1.0 cho Linux/amd64 
curl -LO https://github.com/hyperledger/fabric/releases/download/v3.1.0/hyperledger-fabric-linux-amd64-3.1.0.tar.gz

tar -xzf hyperledger-fabric-linux-amd64-3.1.0.tar.gz -C .

Kiểm tra ngay không phụ thuộc PATH Fabric core binaries vừa tải:
./bin/peer version

 

3.	Tải và giải nén bộ Hyperledger Fabric CA phiên bản 1.5.10:
curl -LO https://github.com/hyperledger/fabric-ca/releases/download/v1.5.10/hyperledger-fabric-ca-linux-amd64-1.5.10.tar.gz

tar -xzf hyperledger-fabric-ca-linux-amd64-1.5.10.tar.gz -C .

Kiểm tra ngay không phụ thuộc PATH Fabric core binaries vừa tải:
./bin/fabric-ca-client version
 

4.	Thêm PATH để lần sau gõ peer ở đâu cũng nhận bản mới:

echo 'export PATH=$HOME/fabric-samples/bin:$PATH' >> ~/.bashrc
echo 'export FABRIC_CFG_PATH=$HOME/fabric-samples/config' >> ~/.bashrc
source ~/.bashrc

5.	Pull Docker images cho Hyperledger Fabric 3.1.x và Fabric CA 1.5.10

docker pull hyperledger/fabric-peer:3.1
docker pull hyperledger/fabric-orderer:3.1
docker pull hyperledger/fabric-ccenv:3.1
docker pull hyperledger/fabric-ca:1.5.10

# (tùy môi trường, Docker có thể tự kéo thêm baseos khi cần)
Kiểm tra kết quả các docker images đã được pull:
docker images "hyperledger/*" --format "{{.Repository}}:{{.Tag}}" | sort
	
 
Sau khi cài xong, bạn sẽ có Thư mục:
fabric-samples/
├── bin/
├── config/
└── ...
Các binary chính: peer, orderer, configtxgen, configtxlator, cryptogen, discover, osnadmin, fabric-ca-client
Các Docker images tương ứng (Fabric 3.1.x)
 
 

=> Đây là bộ toolset được sử dụng trong Chapter 4 để dựng mạng Bankchain.
 
LAB 4.1. Setting up the hyperledger fabric bankchain network

  dựng một mạng Fabric (gồm orderer, peer, CA, channel…) để làm nền cho các bước sau.

Thư mục dự án Bankchain tham khảo:
Tải code mẫu từ GitHub:
git clone https://github.com/PacktPublishing/Blockchain-Development-for-Finance-Projects.git

cd Blockchain-Development-for-Finance-Projects/Chapter4/bankchain
Có thể sao chép thư mục bankchain vào trong fabric-samples (để có sẵn binaries và config)
Trong phần LAB 4.1 cần các các file quan trọng:
1.	crypto-config.yaml file
2.	configtx.yaml file
3.	Các docker-compose files:
o	docker-compose-bankchain.yaml 
o	docker-compose-ca.yaml 
o	docker-compose-couch.yaml
Lưu ý: Các file trên đặt trong thư mục: fabric-samples/bankchain
Launching the network - Thực hiện 4 bước cho Fabric 3.x: 

Kết quả LAB 4.1:
•	Mạng Bankchain chạy với 1 Orderer + 2 Org (Bank1, Bank2).
•	Có channel bankchannel kết nối 2 ngân hàng.
•	Tất cả peer join channel thành công, anchor peer đã định nghĩa.
Một số lệnh kiểm tra Lab 4.1: 
1. Liệt kê tất cả service định nghĩa trong file compose để biết tên đúng:
docker compose -f docker-compose-bankchain.yaml config --services
2. Khởi động các peer:
docker compose -f docker-compose-bankchain.yaml up -d 
3. Khởi động các orderer:
docker compose -f docker-compose-bankchain.yaml up -d orderer.bankchain.com
4. Khởi động Cli:
docker compose -f docker-compose-bankchain.yaml up -d cli	
5. Xem trạng thái hiện tại của các container do compose quản lý:
docker compose -f docker-compose-bankchain.yaml ps
Ví dụ kết quả như sau là OK:
 

 
LAB 4.2. Creating blockchain identities for the banks

Trước khi gửi/đọc giao dịch, mỗi ngân hàng cần 2 Identities: admin (quản trị CA, tạo/xoá user, nhóm…) và bank user (gửi/đọc giao dịch). Ta sẽ viết 2 utilities Node.js: enrollAdmin-*.js (tạo admin) và registerUser-*.js (tạo user) cho banka và bankb
Các file cần có trong phần này: 
	Utilities: 
•	enrollAdmin-BankA.js
•	enrollAdmin-BankB.js
•	registerUser-BankA.js
•	registerUser-BankB.js
•	connection-banka.json
•	connection-bankb.json
connection-banka.json/connection-bankb.json là network/connection profile (hồ sơ kết nối) của Hyperledger Fabric dùng cho ứng dụng client (Node/Go/Java).
Nó cho SDK biết kết nối tới mạng như thế nào: địa chỉ CA/peer/orderer, MSP của tổ chức, chứng chỉ TLS, tên kênh… để có thể:
	Kết nối tới Fabric-CA và enroll/đăng ký danh tính admin, user.
	Dùng Gateway (fabric-network) để gửi giao dịch: chọn peer/orderer, bật discovery, xác thực TLS, v.v.
Những trường quan trọng thường có
	client.organization: tên tổ chức mặc định của ứng dụng.
	organizations[BankAMSP]: mspid, danh sách certificateAuthorities (và có thể liệt kê peers).
	certificateAuthorities[ca.banka...]: url, caName, tlsCACerts (đường dẫn hoặc PEM), (tuỳ chọn: registrar).
	(Khi giao dịch) peers, orderers, channels với endpoint/port/TLS cert.
Run utilities:
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
	npm i = viết tắt của npm install.
	fabric-ca-client và fabric-network là tên 2 package sẽ được cài.
	Kết quả: npm tải chúng về thư mục node_modules/, thêm mục vào dependencies trong package.json, và cập nhật package-lock.json.
Kết quả:
  
 
Now, our network is ready to use. We can submit transactions to the network, and query transactions that have been added to the blockchain ledger. Next, let's write and deploy our corporate remittance smart contract.
LAB 4.3. Building the corporate remittance contract
- To successfully carry out remittances on our blockchain network, we need to write and deploy a smart contract. The smart contract will define the transaction object, including the transaction parameters that will be captured in the channel ledger. The smart contract will allow authorized users to write transactions to the ledger and read transactions that have been written to the ledger.
- Các thư mục & file cần có:
~/fabric-samples/chaincode/corprem/
├─ lib/
│  └─ corprem.js        
├─ index.js             
├─ package.json         
├─ server.js            
└─ ccpack/
   └─ connection.json   

- Các bước thực hiện:

Quy trình đầy đủ triển khai corprem (Corporate Remittance) trên Fabric 3.x bằng CLI theo mô hình Chaincode-as-a-Service (CaaS). Mình hướng dẫn theo hướng chạy CLI trong cùng Docker network (không cần publish cổng ra host) vì ổn định và ít lỗi hơn.
	
	1. Mở CLI:

docker compose -f docker-compose-bankchain.yaml up -d cli
docker exec -it cli bash

•	Vào Network và lấy tên Network:	
cd ~/fabric-samples/bankchain

# Lấy tên network của các container (ví dụ: bankchain_default)
docker inspect peer0.banka.bankchain.com \
  --format '{{range $k,$v := .NetworkSettings.Networks}}{{println $k}}{{end}}'
		
		Kết quả thấy tên network là bankchain_bankchain-net:
 

•	Thay bằng tên bạn thấy ở bước trên (bôi vàng):
export NET=bankchain_bankchain-net

•	Mở 1 shell CLI trong network đó:
docker run --rm -it --network $NET \
  -v "$PWD:/work" -w /work \
  -e FABRIC_CFG_PATH=/work/config \
  hyperledger/fabric-tools:3.1 bash

Điều kiện tiên quyết
•  Các container đã chạy: orderer.bankchain.com, peer0/peer1 của BankA & BankB, cli, (CAs nếu cần).
•  Kênh bankchannel đã join tất cả peer, anchor peers đã cập nhật.
•  Trong docker-compose-bankchain.yaml, ở service cli có mount thư mục chaincode:
services:
  cli:
    ...
    volumes:
      - ./crypto-config:/opt/crypto
      - ./channel-artifacts:/opt/channel-artifacts
      - ./bin:/usr/local/bin
      - ./chaincode:/opt/chaincode          # <— thêm dòng này

Sau đó chạy lệnh:
docker compose -f docker-compose-bankchain.yaml up -d cli --force-recreate

Tạo các folder/file cần có trong phần này: 
~/fabric-samples/chaincode/corprem/
├─ lib/
├─ index.js
├─ package.json
├─ server.js
└─ ccpack/
   └─ connection.json   
(Vì Fabric 3.x) chạy External Chaincode (Caas)
- Tạo gói “external”:
Tạo thư mục mô tả kết nối và file connection.json:
mkdir -p ccpack
cat > ccpack/connection.json <<'JSON'
{
  "address": "host.docker.internal:9999",
  "dial_timeout": "10s",
  "tls_required": false
}
JSON

Thực hiện các bước sau:
1. Vào container CLI & set biến chung:

docker exec -it cli bash
export CHANNEL=bankchannel
export CCNAME=corprem
export CC_LABEL=corprem_1
export CC_PATH=/opt/chaincode/corprem
export ORDERER=orderer.bankchain.com:7050
export ORDERER_CA=/opt/crypto/ordererOrganizations/bankchain.com/orderers/orderer.bankchain.com/tls/ca.crt

# TLS root cert của peer0 hai bên
export BA_PEER0=peer0.banka.bankchain.com:7051
export BB_PEER0=peer0.bankb.bankchain.com:7051
export BA_TLS=/opt/crypto/peerOrganizations/banka.bankchain.com/peers/peer0.banka.bankchain.com/tls/ca.crt
export BB_TLS=/opt/crypto/peerOrganizations/bankb.bankchain.com/peers/peer0.bankb.bankchain.com/tls/ca.crt

2. Đóng gói chaincode:

peer lifecycle chaincode package /opt/channel-artifacts/${CCNAME}.tar.gz \
  --path ${CC_PATH} --lang node --label ${CC_LABEL}

3. Cài đặt lên mỗi org (ít nhất 1 peer/org):

BankA (peer0):
export CORE_PEER_LOCALMSPID=BankAMSP
export CORE_PEER_ADDRESS=${BA_PEER0}
export CORE_PEER_MSPCONFIGPATH=/opt/crypto/peerOrganizations/banka.bankchain.com/users/Admin@banka.bankchain.com/msp
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=${BA_TLS}

peer lifecycle chaincode install /opt/channel-artifacts/${CCNAME}.tar.gz
peer lifecycle chaincode queryinstalled | sed -n "s/Package ID: \(.*\), Label: ${CC_LABEL}/export PKGID=\1/p"
echo $PKGID

BankB (peer0):
export CORE_PEER_LOCALMSPID=BankBMSP
export CORE_PEER_ADDRESS=${BB_PEER0}
export CORE_PEER_MSPCONFIGPATH=/opt/crypto/peerOrganizations/bankb.bankchain.com/users/Admin@bankb.bankchain.com/msp
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=${BB_TLS}

peer lifecycle chaincode install /opt/channel-artifacts/${CCNAME}.tar.gz
# (chỉ cần packageID đã lấy ở bước BankA)

4. Approve cho từng org:

Chọn policy yêu cầu cả hai org đồng thuận:
export SIG_POLICY="AND('BankAMSP.peer','BankBMSP.peer')"

BankA approve:
export CORE_PEER_LOCALMSPID=BankAMSP
export CORE_PEER_ADDRESS=${BA_PEER0}
export CORE_PEER_MSPCONFIGPATH=/opt/crypto/peerOrganizations/banka.bankchain.com/users/Admin@banka.bankchain.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${BA_TLS}

peer lifecycle chaincode approveformyorg \
  --channelID ${CHANNEL} --name ${CCNAME} \
  --version 1.0 --sequence 1 \
  --package-id ${PKGID} \
  --signature-policy "${SIG_POLICY}" \
  --orderer ${ORDERER} --tls --cafile ${ORDERER_CA}

BankB approve:
export CORE_PEER_LOCALMSPID=BankBMSP
export CORE_PEER_ADDRESS=${BB_PEER0}
export CORE_PEER_MSPCONFIGPATH=/opt/crypto/peerOrganizations/bankb.bankchain.com/users/Admin@bankb.bankchain.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${BB_TLS}

peer lifecycle chaincode approveformyorg \
  --channelID ${CHANNEL} --name ${CCNAME} \
  --version 1.0 --sequence 1 \
  --package-id ${PKGID} \
  --signature-policy "${SIG_POLICY}" \
  --orderer ${ORDERER} --tls --cafile ${ORDERER_CA}

Kiểm tra readiness (ở một trong hai org):
peer lifecycle chaincode checkcommitreadiness \
  --channelID ${CHANNEL} --name ${CCNAME} \
  --version 1.0 --sequence 1 \
  --signature-policy "${SIG_POLICY}" --output json

=> Kết quả phải thấy cả BankAMSP và BankBMSP đều true.

5. Commit definition
Chạy từ bất kỳ org nào (ví dụ BankA), nhưng kèm cả 2 peerAddresses:
export CORE_PEER_LOCALMSPID=BankAMSP
export CORE_PEER_ADDRESS=${BA_PEER0}
export CORE_PEER_MSPCONFIGPATH=/opt/crypto/peerOrganizations/banka.bankchain.com/users/Admin@banka.bankchain.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${BA_TLS}

peer lifecycle chaincode commit \
  --channelID ${CHANNEL} --name ${CCNAME} \
  --version 1.0 --sequence 1 \
  --signature-policy "${SIG_POLICY}" \
  --orderer ${ORDERER} --tls --cafile ${ORDERER_CA} \
  --peerAddresses ${BA_PEER0} --tlsRootCertFiles ${BA_TLS} \
  --peerAddresses ${BB_PEER0} --tlsRootCertFiles ${BB_TLS}

peer lifecycle chaincode querycommitted --channelID ${CHANNEL} --name ${CCNAME}

6. Gọi thử (invoke/query)

Tạo remittance:
peer chaincode invoke -C ${CHANNEL} -n ${CCNAME} \
  -c '{"Args":["Create","R100","ACME","MEGABANK","2500","USD"]}' \
  --orderer ${ORDERER} --tls --cafile ${ORDERER_CA} \
  --peerAddresses ${BA_PEER0} --tlsRootCertFiles ${BA_TLS} \
  --peerAddresses ${BB_PEER0} --tlsRootCertFiles ${BB_TLS} \
  --waitForEvent

Lấy remittance:
peer chaincode query -C ${CHANNEL} -n ${CCNAME} \
  -c '{"Args":["Get","R100"]}'

Cập nhật trạng thái:
peer chaincode invoke -C ${CHANNEL} -n ${CCNAME} \
  -c '{"Args":["UpdateStatus","R100","SETTLED"]}' \
  --orderer ${ORDERER} --tls --cafile ${ORDERER_CA} \
  --peerAddresses ${BA_PEER0} --tlsRootCertFiles ${BA_TLS} \
  --peerAddresses ${BB_PEER0} --tlsRootCertFiles ${BB_TLS} \
  --waitForEvent

LAB 4.4. Setting up the IPFS network

- The IPFS network will allow the banks to share important compliance and AML documents that are required to process corporate remittance transactions. Our IPFS network will consist of two nodes. Each node will be controlled by a bank.
The banks will publish the documents to the IPFS network. Only authorized participants will be able to retrieve the documents from the network, using the document's hash signature.

- Các bước thực hiện:

Cài IPFS binary: (go-ipfs: phần mềm IPFS được viết bằng Go)
# tải gói go-ipfs mới nhất, giải nén và chép binary
tar -xzf go-ipfs_*.tar.gz
cd go-ipfs
sudo mv ipfs /usr/local/bin/ipfs
ipfs version

Khởi tạo (initialize) 2 node:
# Node 1
IPFS_PATH=~/.ipfs ipfs init

# Node 2 (thư mục dữ liệu khác)
IPFS_PATH=~/.ipfs2 ipfs init

Tạo key (private swarm) cho mạng:
# sinh khóa và đặt vào cả 2 IPFS_PATH (ví dụ dùng ipfs-swarm-key-gen)
ipfs-swarm-key-gen > ~/.ipfs/swarm.key
cp ~/.ipfs/swarm.key ~/.ipfs2/swarm.key

Cấu hình từng node (cổng API/Gateway khác nhau):
# Node 1: giữ mặc định (API :5001, Gateway :8080)
IPFS_PATH=~/.ipfs ipfs config Addresses.API /ip4/127.0.0.1/tcp/5001
IPFS_PATH=~/.ipfs ipfs config Addresses.Gateway /ip4/127.0.0.1/tcp/8080

# Node 2: đặt cổng khác (API :5002, Gateway :8081)
IPFS_PATH=~/.ipfs2 ipfs config Addresses.API /ip4/127.0.0.1/tcp/5002
IPFS_PATH=~/.ipfs2 ipfs config Addresses.Gateway /ip4/127.0.0.1/tcp/8081

Bootstrap các node (kết nối chúng với nhau):
# (a) Trên Node 1: xoá bootstrap mặc định và thêm chính Node 1 nếu cần
IPFS_PATH=~/.ipfs ipfs bootstrap rm --all
# Thêm địa chỉ bootstrap (ví dụ IP nội bộ của Node 1 và PeerID của Node 1)
IPFS_PATH=~/.ipfs ipfs bootstrap add /ip4/192.168.10.1/tcp/4001/ipfs/<PeerID_Node1>

# (b) Trên Node 2: trỏ bootstrap về Node 1
IPFS_PATH=~/.ipfs2 ipfs bootstrap rm --all
IPFS_PATH=~/.ipfs2 ipfs bootstrap add /ip4/192.168.10.1/tcp/4001/ipfs/<PeerID_Node1>

Chạy daemon cho từng node:
# Terminal 1
IPFS_PATH=~/.ipfs ipfs daemon

# Terminal 2
IPFS_PATH=~/.ipfs2 ipfs daemon

Kiểm thử mạng (add từ Node 1, cat từ Node 2):
# Tạo file mẫu trên máy
echo "hello IPFS" > file.txt

# Thêm từ Node 1 -> nhận CID (ghi lại CID_in_ra do lệnh trả về)
IPFS_PATH=~/.ipfs ipfs add file.txt
# ví dụ sách minh họa một CID như:
# QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN

# Đọc lại từ Node 2 bằng đúng CID
IPFS_PATH=~/.ipfs2 ipfs cat QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
LAB 4.5. Setting up the bank databases

- We need to set up databases for both Bank A and Bank B, which will hold the customer data and transaction data. We use postgresql

- Các bước thực hiện:

Cài PostgreSQL:
sudo apt install postgresql postgresql-contrib

CentOS/RHEL:
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl start postgresql

Tạo user & database cho 2 ngân hàng:
# chuyển sang user postgres
su - postgres

# tạo 2 database user, đặt mật khẩu khi được hỏi (vd: banka / bankb)
createuser banka --pwprompt
createuser bankb --pwprompt

# vào psql
psql

# tạo 2 database, mỗi cái thuộc sở hữu đúng user
CREATE DATABASE banka OWNER banka;
CREATE DATABASE bankb OWNER bankb;

Tạo Unix users tương ứng (để chạy app/server theo từng ngân hàng):
# trở lại root/sudo rồi tạo UNIX users
sudo adduser banka
sudo adduser bankb

# đặt mật khẩu (ví dụ trùng tên cho dễ thử nghiệm)
sudo passwd banka
sudo passwd bankb

•	Trên database banka:
su - banka
psql banka

-- bảng khách hàng
CREATE TABLE customers(
  user_id serial PRIMARY KEY,
  name VARCHAR NOT NULL,
  address VARCHAR NOT NULL,
  account VARCHAR NOT NULL,
  balance INTEGER NOT NULL
);

-- bảng giao dịch
CREATE TABLE transactions(
  transactions_id VARCHAR PRIMARY KEY,
  sname VARCHAR NOT NULL,
  saddress VARCHAR NOT NULL,
  saccount VARCHAR NOT NULL,
  sbank VARCHAR NOT NULL,
  rname VARCHAR NOT NULL,
  raddress VARCHAR NOT NULL,
  raccount VARCHAR NOT NULL,
  rbank VARCHAR NOT NULL,
  currency VARCHAR(4) NOT NULL,
  amount INTEGER NOT NULL,
  invhash VARCHAR NOT NULL,
  boehash VARCHAR NOT NULL,
  dochash VARCHAR NOT NULL,
  transtype VARCHAR NOT NULL
);

•	Lặp lại cho bankb:
su - bankb
psql bankb
-- chạy lại 2 lệnh CREATE TABLE như trên

Chèn dữ liệu mẫu cho khách hàng (mỗi bank 1 khách):
# Trên banka
su - banka
psql banka
INSERT INTO customers (name, account, address, balance)
VALUES('Acme','ACMEAC8829','New Delhi',1000);

# Trên bankb
su - bankb
psql bankb
INSERT INTO customers (name, account, address, balance)
VALUES('Apex','APXAC09002','Dubai',1000);
=> Đã có 2 CSDL (banka, bankb) với schema và dữ liệu mẫu để dùng cho các bước backend/listener ở những phần tiếp theo của Chapter 4.
LAB 4.6. Building the bank backend servers
- Backend là “cầu nối” giữa Fabric, IPFS, PostgreSQL và frontend; nó cung cấp các API: /customerinfo, /payment, /gettrans, cùng các method iwrite, submitTrans, updateBal, insertTrans.	

- Các bước thực hiện:

Tạo môi trường Node.js:
mkdir bank-backends && cd bank-backends
npm init -y

Mở package.json và điền dependencies như sách:
express, body-parser, multer (upload chứng từ), fabric-ca-client, fabric-network, fs, ipfs-http-client, pg. Sau đó cài:

npm install

Tạo file server: Tạo 2 file:
touch backend-BankA.js backend-BankB.js
 
Viết backend cho Bank A (backend-BankA.js):
•	Import module & khai báo client:
var express = require('express');
var app = express();
var multer = require('multer');
var bodyParser = require('body-parser');
const { FileSystemWallet, Gateway } = require('fabric-network');
const ipfsClient = require('ipfs-http-client');
const ipfs = ipfsClient('http://localhost:5001'); // IPFS client Bank A
const pg = require('pg');

•	Khai báo endpoints:
	/customerinfo: trả về name, address, account, balance từ DB. 
	/payment: nhận yêu cầu thanh toán + upload chứng từ (invoice/BOE/BOL), đẩy tài liệu lên IPFS, gọi giao dịch lên Fabric, ghi DB và cập nhật số dư; trả về “Transaction successfully submitted”. 
	/gettrans: trả về mảng giao dịch của khách hàng. 
Ví dụ phần xử lý /gettrans (truy vấn transactions theo account, trả JSON): 
•	Method tiện ích:
	iwrite(filepath): add file lên IPFS (client 5001) và trả về hash. 
	submitTrans(txID, trans, hasharray): submit giao dịch lên Fabric, dùng wallet BankA (~/fabric-samples/bankchain/wallet-BankA) & kiểm tra user user1 tồn tại trước khi submit. 
	updateBal(...): cập nhật số dư sau khi gửi. (nằm trong nhóm method ở mục “Writing a method to update the customer's balance”). 
	insertTrans(...): ghi giao dịch vào bảng transactions, kết thúc bằng log “Transaction successfully submitted”. 

•	Chạy server Bank A:
var server = app.listen(process.env.PORT || 8000, () => {
  console.log("App now running on port", server.address().port);
});
Tạo backend cho Bank B (sao chép & đổi cấu hình):
Sao chép backend-BankA.js → backend-BankB.js, sau đó chỉnh 5 điểm sau (sách liệt kê rõ):
•	Đổi IPFS client sang 5002:
const ipfs = ipfsClient('http://localhost:5002');
``` :contentReference[oaicite:16]{index=16}

2) Đổi **Postgres** sang DB **bankb**:  
```js
const conString = "postgres://bankb:bankb@localhost:5432/bankb";
``` :contentReference[oaicite:17]{index=17}

3) Đổi **port server** (tránh xung đột) sang **8001**:  
```js
var server = app.listen(process.env.PORT || 8001, () => {
  console.log("App now running on port", server.address().port);
});
``` :contentReference[oaicite:18]{index=18}

4) Đổi **đường dẫn connection profile** Fabric sang **bankb**:  
```js
const ccpPath = '~/fabric-samples/bankchain/connection-bankb.json';
``` :contentReference[oaicite:19]{index=19}

5) Đổi **wallet path** sang **wallet-BankB**:  
```js
const wallet = new FileSystemWallet('~/fabric-samples/bankchain/wallet-BankB');
``` :contentReference[oaicite:20]{index=20}

---

# 5) Chạy 2 backend
Mở **2 terminal**:
```bash
node backend-BankA.js   # chạy Bank A (port 8000)
node backend-BankB.js   # chạy Bank B (port 8001)
Gợi ý kiểm tra nhanh (sanity check):
•	Gọi POST /customerinfo bằng account của bạn để xem thông tin. (API này được mô tả ở danh sách endpoint). 
•	Gọi POST /gettrans để đọc lịch sử giao dịch (ví dụ đã có trong sách). 
•	Thực hiện POST /payment với chứng từ: server sẽ upload file → IPFS (iwrite) → submit Fabric (submitTrans) → updateBal → insertTrans và trả thông báo “Transaction successfully submitted”.
LAB 4.7. Building the transaction listeners for the banks

- We need to build a transaction listener, to detect incoming transactions to the bank's customers. We need one for Bank A and one for Bank B.
Transaction listeners làm gì?
•	Nghe event txCreated từ chaincode corprem (kênh bkchannel).
•	Khi có event, nếu trans.Rbank đúng bank đang chạy listener → cập nhật balance khách nhận trong bảng customers, insert giao dịch vào bảng transactions với transtype='Incoming', và tải chứng từ (invoice/BOE/BOL/khác) từ IPFS về storage nội bộ (iread()). 
•	Listener có 2 hàm: Transactionlisten() (nghe & xử lý), iread() (tải tài liệu từ IPFS).
Cấu trúc thư mục & các file cần có:
•	Thư mục listener (Node.js)
trans-listeners/
├─ package.json
├─ TransListener-BankA.js
└─ TransListener-BankB.js
Bạn tạo TransListener-BankA.js, rồi nhân bản thành TransListener-BankB.js và đổi cấu hình (IPFS 5002, DB bankb, connection profile connection-bankb.json, wallet BankB, điều kiện Rbank=='Bank B').
•	Hồ sơ mạng & ví Fabric (đã chuẩn bị từ bước trước):
~/fabric-samples/bankchain/
├─ connection-banka.json
├─ connection-bankb.json
├─ wallet-BankA/
│  ├─ admin/...
│  └─ user1/...
└─ wallet-BankB/
   ├─ admin/...
   └─ user1/...

Listener cần connection profile cho từng bank và wallet có user1/admin. (Các file/ thư mục này được tạo ở phần “Creating blockchain identities” và các utility enroll/register.) 
•	Thư mục lưu chứng từ tải từ IPFS (thuộc frontend portal):
/path/to/CorpRemApp/CorpRemApp/public/uploads/
Hàm iread() sẽ ghi file theo tên <hash>.txt vào thư mục public/uploads của app portal React. Bạn chỉ cần bảo đảm thư mục tồn tại và đường dẫn trong code trỏ đúng vào đây.
•	CSDL cho 2 ngân hàng (đã tạo ở bước trước)
•	IPFS nodes đang chạy: 
Không cần file mới, chỉ cần 2 daemon IPFS (Bank A cổng 5001, Bank B cổng 5002) đã set ở phần trước; listener dùng client IPFS tương ứng để get() tài liệu
- Các bước thực hiện:

Tạo project & cài thư viện (Bank A):
mkdir trans-listeners && cd trans-listeners
npm init -y
npm install express body-parser fabric-network ipfs-http-client pg fs

Tạo file cho Bank A:
touch TransListener-BankA.js

Viết code listener cho Bank A: TransListener-BankA.js

Chạy listener Bank A:
node TransListener-BankA.js

(Khi kết nối gateway OK sẽ log kiểu “Gateway Connected”, rồi listener active.)

Tạo & chỉnh listener cho Bank B:
•	Sao chép file Bank A → Bank B:
cp TransListener-BankA.js TransListener-BankB.js

Rồi sửa 5 điểm:
1.	IPFS client sang 5002
const ipfs = ipfsClient('http://localhost:5002') 
2.	Postgres trỏ DB bankb
const conString = "postgres://bankb:bankb@localhost:5432/bankb"; 
3.	Connection profile sang connection-bankb.json
const ccpPath = '~/fabric-samples/bankchain/connection-bankb.json'; 
4.	Wallet sang wallet-BankB
const wallet = new FileSystemWallet('~/fabric-samples/bankchain/wallet-BankB'); 
5.	Điều kiện lọc bank nhận trong listener → Bank B
if (trans.Rbank == 'Bank B') { ... } 
•	Chạy Bank B:
node TransListener-BankB.js

Kiểm thử nhanh (sanity):
•	Gửi một payment từ Bank A → Bank B qua backend /payment (bạn đã dựng ở phần trước). Listener Bank B nhận txCreated, thấy trans.Rbank == bankb → UPDATE số dư khách nhận, INSERT transactions (transtype='Incoming'), rồi tải 3 chứng từ về uploads/. 
•	Xem bảng customers và transactions trong bankb để xác nhận.
LAB 4.8. Creating the corporate remittance app frontend in React

- We need to create a frontend that will interact with the backend server and allow users to submit transactions and view submitted transactions.
The app should render three screens:
•	The Bank login screen
•	The Transfer screen
•	The View Transactions screen

The user logs in using the Bank login screen. They can either log in as Acme Inc or Apex Corp, by clicking on the Acme Inc. button or the Apex Corp button.

After logging in, they land on the Transfer screen. Here, they can initiate a new transfer request by filling in the details of the remittance, including the receiver's name, account number, bank and address, and the transaction amount and currency. They also need to upload the compliance documents, which include the invoice document, BOE/BOL, and any other document they want.
On clicking on Submit, a new remittance transfer request is initiated and sent to the backend server for processing. After the backend confirms the success of the transaction, the user is notified of the Tx Status on the screen.

On clicking on View Transactions, the app fetches all of the user's transactions from the database and renders it on the screen. The user can view all incoming or outgoing transactions. By default, all incoming transactions are shown. The user can view all transaction details, including the compliance documents. The screen renders a separate link to view each document. On clicking on the link, the document is downloaded from the server's local storage.

- Các file và folder cần có:

CorpRemApp/
├─ public/
│  ├─ index.html
│  └─ uploads/                    # chứa chứng từ tải từ IPFS (<hash>.txt)
│
├─ src/
│  ├─ App.js                      # chứa logic chính (methods, state, điều hướng)
│  ├─ index.js                    # render App vào DOM
│  ├─ Components/
│  │  ├─ Container.js             # khung điều hướng / layout chính
│  │  ├─ AppLogin.js              # form chọn ngân hàng (Bank A hoặc Bank B)
│  │  ├─ Transfer.js              # form chuyển tiền + upload chứng từ
│  │  └─ ViewTransactions.js      # hiển thị lịch sử giao dịch + link tải chứng từ
│  │
│  ├─ App.css                     # (tuỳ chọn) style chung
│  └─ Components.css              # (tuỳ chọn) style cho từng component
│
├─ package.json                   # định nghĩa dependencies React, axios,...
└─ README.md                      # mô tả dự án (tự động sinh bởi create-react-app)


Thành phần	Vai trò chính	Ghi chú
App.js	Khởi tạo state, định nghĩa các phương thức (setAccount, submitPayment, getTransactions, setBalance, …) và điều hướng giữa các component	Được trình bày trong mục “Writing methods in the App.js file”
Container.js	Component bao ngoài, hiển thị từng màn hình (Login, Transfer, Transactions)	“Creating the Container component”
AppLogin.js	Cho phép người dùng chọn ngân hàng → gọi setAccount() để kết nối đúng backend	“Creating the Login component”
Transfer.js	Form nhập dữ liệu chuyển tiền, đính kèm chứng từ → gọi API /payment	“Creating the Transfer component”
ViewTransactions.js	Hiển thị lịch sử giao dịch, có link tới file chứng từ trong public/uploads/	“Creating the View Transactions component”
public/uploads/	Nơi listener ghi các file <hash>.txt tải từ IPFS để frontend hiển thị	Mục “Displaying documents from IPFS”


- Các bước thực hiện:

Tạo dự án React + cấu trúc thư mục:
# 1.1 Tạo app React
npx create-react-app CorpRemApp

# 1.2 Tạo các thư mục/file cần dùng
cd CorpRemApp
mkdir -p src/Components public/uploads
touch src/Components/Container.js \
      src/Components/AppLogin.js \
      src/Components/Transfer.js \
      src/Components/ViewTransactions.js

Xây các component giao diện:
•	Container.js: Là “khung” điều hướng giữa các phần của app; sẽ render các tab/section tương ứng (login, chuyển tiền, xem giao dịch).
•	AppLogin.js: Màn hình đăng nhập chọn Bank A / Bank B; sau khi chọn sẽ gọi method trong App.js để set tài khoản/kết nối đúng backend. 
•	Transfer.js: Form tạo “payment request”: nhập bên nhận, số tiền, tiền tệ và đính kèm chứng từ (invoice/BOE/BOL). Gửi lên endpoint /payment của backend. 
•	ViewTransactions.js: Bảng lịch sử incoming/outgoing; mỗi dòng có link tải chứng từ từ public/uploads/<hash>.txt. 

Viết App.js (trái tim của frontend):

Kết nối frontend ↔ backend:
Frontend gọi đúng ba endpoint backend: /customerinfo (lấy hồ sơ + số dư), /payment (gửi yêu cầu), /gettrans (xem lịch sử). Các endpoint này đã được bạn dựng ở “Building the bank backend servers”. 

Hiển thị & tải chứng từ IPFS
•	Listener đã kéo file từ IPFS và ghi thành public/uploads/<hash>.txt.
•	Ở ViewTransactions.js, render link tải như: <a href={'uploads/' + tx.invhash + '.txt'} download>. 
LAB 4.9. Running all 

npm start
