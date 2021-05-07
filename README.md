# Setup
Instalación de hyperledger v2.2.x.

## Requisitos

## Instalar Docker
Los siguientes pasos son requeridos para la instalación de docker. Referencia: https://docs.docker.com/engine/install/ubuntu/

```bash
# Agregar clave GPG oficial de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# configurar el repositorio estable
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# instalar docker engine
apt update
apt install docker-ce docker-ce-cli containerd.io

# comprobar la versión de docker
docker --version
```

## Instalar Docker-Compose

Referencia https://docs.docker.com/compose/install/

```bash
# instalar docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Agregar permisos de ejecución
sudo chmod +x /usr/local/bin/docker-compose

# comprobar la versión de docker compose
docker-compose --version
```

## Instalar Golang
Hyperledger Fabric usa Golang internamente. Versión de Go 1.14.x o superior es requerida.

```bash 
# Descargar y extraer el binario de go
sudo wget -c https://dl.google.com/go/go1.14.9.linux-amd64.tar.gz -O - | tar -xz -C /usr/local

# Asignar ruta al binario
echo 'export PATH="$PATH:/usr/local/go/bin:/root/fabric-samples/bin"' >> $HOME/.profile

# Definir el espacio de trabajo para fabric
echo 'export GOPATH="$HOME"' >> $HOME/.profile

# Para aplicar los datos inmediatamente ejecutar
source $HOME/.profile

# comprobar la versión de golang
go version

# comprobar si las variables de entorno fueron correctamente asignadas
printenv | grep PATH
```

## Instalar node.js

```bash
# instalar node.js
apt-get install -y nodejs

# comprobar la versión de node
node -v
```

## Instalar Imágenes de docker y binarios

```bash
mkdir fabric
cd fabric

# Si desea instalar la última versión disponible
# curl -sSL https://bit.ly/2ysbOFE | bash -s

# En los ejemplos se utilizará la versión 2.2
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.1 1.4.9

# comprobar imágenes de docker
docker images

# comprobar que el bin este funcionando correctamente
peer version

#Aclaración: si el peer version no funciona verificar que las variables de entorno de GOLANG estén correctamente declaradas

```

## Interactuando con la red

Para probar que la instalación resulto exitosa, interactuamos con la red de la siguiente forma

```bash
# navegamos hacia la carpeta base de la red de prueba
cd fabric-samples/test-network

# levanta la red de prueba
./network.sh up createChannel -c mychannel

# despliega el contrato por default de la red
./network.sh deployCC -c channel1

# comprobar que los contenedores esten funcionando
docker ps
docker-compose -f docker/docker-compose-test-net.yaml ps
```

### Variables de entorno para el nodo Org1

```bash
# crear un archivo env para Org1
nano org1.sh

export FABRIC_CFG_PATH=$HOME/fabric-samples/config/
export CHANNEL_NAME="mychannel"

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

# actualizar el archivo env
source ./org1.sh

# comprobar las variables de entorno
printenv | grep CORE
```

#### Iniciar el ledger con datos
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```

#### Consultando la red

```bash
# Obtener todos los datos disponibles
peer chaincode query -C $CHANNEL_NAME -n basic -c '{"Args":["GetAllAssets"]}' | jq .

# Obtener un dato en particular
peer chaincode query -C $CHANNEL_NAME -n basic -c '{"Args":["ReadAsset","asset1"]}' | jq .
```

#### Crear
```bash

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset7","green","10","Roland","500"]}'
```

#### Actualizar
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"UpdateAsset","Args":["asset7","green","10","Roland","600"]}'
```

#### Transferir posesión
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset7","Joana"]}'
```

#### Eliminar
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"DeleteAsset","Args":["asset1"]}'
```

### Cambiar hacia el nodo Org2
Podemos cambiar el entorno de trabajo hacia el nodo Org2 peer0.org2.example.com configurando las siguientes variables de entorno

```bash 

# crear un archivo env
nano org2.sh

# Variables de entorno para Org2
export FABRIC_CFG_PATH=$HOME/fabric-samples/config/
export CHANNEL_NAME="mychannel"

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

# actualizar el archivo env
source ./org2.sh
```

## Apagar la red
```bash
./network.sh down
```
