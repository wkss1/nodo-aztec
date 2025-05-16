# 🛠️ Guía: Cómo correr un nodo sequencer de Aztec (Ubuntu + Docker)
cambiar

## 📋 Requisitos de hardware

| Componente | Recomendación             |
|------------|---------------------------|
| Sistema    | Ubuntu 20.04 o superior   |
| RAM        | 8 - 16 GB                 |
| CPU        | 4 - 6 núcleos             |
| Disco      | 550 GB SSD (crece hasta 1 TB) |

---

Esta guía corre un nodo sepolia usando docker y utiliza los clientes Geth y Lighthouse, si nunca antes levantaste un nodo, esta es una muy buena manera de iniciar con un paso a paso para entender todo al detalle.

## Sistema Operativo: Ubuntu

Puedes seguir esta guía en tu equipo local, o puedes tambien utilizar un VPS (alquilar un servidor virtual) para tener tu nodo, esto te permitira tener acceso directo a un RPC de Ethereum Sepolia, para desarrollardores, para correr otros nodos L2 sin tener que pagar por un RPC ($50-$250 por mes).

Por EJ. si deseas correr un nodo de algun L2 como puede ser Aztec, este nodo te pide tener un RPC para el cliente de consenso de L1 y un RPC para el cliente de ejecución del L1

Un RPC no es mas que una llamada que permite interactuar con el nodo, en el caso de un desarrollador, tener un RPC te permite usarlo directamente y tener acceso a la blockchain desde tu nodo sin depender de un proveedor pago o uno gratis que suelen estar limitados.

En el caso de Aztec o otros L2 necesitan tener un RPC del L1 para poder correr ya que el L2 depende del L1 y normalmente los RPC gratis no dan a basto por sus limitaciones. asi que si quieres ahorrarte $ y aprender como correr tu primer nodo que puede ser una profesion o ayudarte en tu perfil profesional. Esta guía es para ti

## Ethereum L1 (SEPOLIA)

El L1 de ethereum esta compuesto por dos redes o dos capas en paralelo, ethereum en realidad son 2 redes que hablan entre ellas, por eso en esta guía correremos los 2 clientes.

1. Geth para el cliente de ejecucion
2. Lighthouse para el cliente de consenso


## 1️⃣ Instalar dependencias y Docker

Usaremos docker para la instalacion del nodo, asi que con estos comando actualizamos el sistema y descargamos las dependencias (software que necesitas para poder correr el nodo)

### 🔧 Dependencias básicas

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```


### 🐳 Instalar Docker

Una ves listo, procedemos a descarga y probar Docker

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo docker run hello-world
sudo systemctl enable docker
sudo systemctl restart docker
```

---
## Aumenta la capacidad de transferencia de datos
Aqui vamos a modificar un archivo para asegurarnos que nuestros sockets por defecto tengan un limite mucho mayor de transferencia de datos, esto suele facilitar la instalacion y evitar posible errores.

Primero, abrimos el archivo con este comando.
```bash
sudo nano /etc/sysctl.conf
```

Luego agregamos al final del archivo sin modificar nada mas lo siguiente:

```bash
net.core.rmem_max=7500000
net.core.wmem_max=7500000
```
Luego de pegarlo, presiona "ctrl" + "X" para cerrar el archivo, de dira si quieres guardar le das Y (Y significa si) y luego le das enter.

Una vez cerrado el archivo, con este comando hacemos que los cambios realizados hagan efecto.

```bash
sudo sysctl -p
```

---

## 2️⃣ Crear directorios y archivo JWT

mkdir se utiliza para crear un directorio (es como una carpeta o una ubicacion) llamada ethereum y dentro de ese directorio, creamos uno llamado execution y uno llamado consensus, aqui estara la informacion de cada uno de los clientes.

```bash
mkdir -p ~/ethereum/execution
mkdir -p ~/ethereum/consensus
```


Este paso se encarga de crear el JWT que es un acceso seguro para que tus 2 clientes puedan hablar entre ellos y sincronizarse con Ethereum (Sepolia en este caso, sin embargo la instalacion es similar para Ethereum mainnet, solo cambiarian algunos parametros) como puedes ver se crea dentro del directorio "ethereum"


```bash
openssl rand -hex 32 > ~/ethereum/jwt.hex
```
Luego para verificar la creacion de este archivo JWT, usamos este comando (debe de regresarte un valor largo como por ejemplo: "96899fad43bf5d94363d67862bb4efe3c070c22f424fcf031dd335e71137776a")

```bash
cat ~/ethereum/jwt.hex
```

---

## 3️⃣ Crear archivo `docker-compose.yml`
Aqui procedemos a crear el archivo de docker compose, en este archivo va toda la informacion de los clientes de ejecucion y consenso de ethereum, como la red en la que queremos (sepolia), los puertos que usaremos, el token JWT que creamos y mas

```bash
cd ~/ethereum
nano docker-compose.yml
```

Pega el siguiente contenido:
El comando anterior te abre una ventana completamente vacia, lo cual esta bien por que recien creamos este archivo, ahora vamos a pegar aqui la información


```yaml
services:
  geth:
    image: ethereum/client-go:stable
    container_name: geth
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./execution:/data
      - ./jwt.hex:/data/jwt.hex
    command:
      - --sepolia
      - --http
      - --http.api=eth,net,web3
      - --http.addr=0.0.0.0
      - --authrpc.addr=0.0.0.0
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/data/jwt.hex
      - --authrpc.port=8551
      - --syncmode=snap
      - --datadir=/data

  lighthouse:
    image: sigp/lighthouse:latest
    container_name: lighthouse
    restart: unless-stopped
    depends_on:
      - geth
    network_mode: host
    volumes:
      - ./consensus:/data
      - ./jwt.hex:/data/jwt.hex
    command: >
      lighthouse bn
      --network sepolia
      --execution-endpoint http://localhost:8551
      --jwt-secrets=/data/jwt.hex
      --datadir=/data
      --checkpoint-sync-url=https://checkpoint-sync.sepolia.ethpandaops.io
      --http
      --http-address=0.0.0.0
      --http-port=5052
      --metrics
```


Ya pegado el texto, hacemos "CTRL" + "X" para cerrar, grabamos los cambios presionando Y. y luego Enter para cerrarlo y volver al terminal

NOTA: En este caso, en el docker compose no hemos colocado o "mapeado" puertos, ya que configuramos el contenedor en la red HOST, eso quiere decir que los nodos estan en nuestra red de host y no necesita mapear los puertos, esta directamente conectado a nuestro host local. (esto simplifica los pasos a cambio de isolacion, pero en nuestro caso esnun testnet y estamos practicando asi que nos nos afecta en nada, por el contrario nos facilita todo)

---

## 4️⃣ Comprobar puertos utilizados

Con este comando podemos verificar si los puertos estan siendo utilizados para evitar confilctos, si no tienes ningun otro servicio, entonces no estaran utilizados

```bash
netstat -tuln | grep -E '30303|8545|8546|8551|5052|9000'
```

---

## 5️⃣ Iniciar los nodos
Con este comando le decimos a docker que inicie usando el compose que recien creamos. esperamos que se cargue y ya tendremos los contenedores activos y el nodo estara corriendo
```bash
docker compose up -d
```

### 🔍 Ver contenedores:
Con este comando podemos ver los contenedores que estan ahora corriendo, y ver si estan funcionando bien, o si se estan reiniciando.

```bash
docker ps
```

### 🔍 Ver logs:
Los Logs son como el reporte, aqui podemos ver que esta sucediendo con nuestro nodo, si esta arrancando, si se esta sincronizando o si esta dando algun error y asi poder determinar que esta sucediendo o si algo debemos cambiar. (si seguiste todos los pasos tal cual, en tus logs deberias ver que esta sincronizando)

```bash
docker compose logs -f
```

---

## 6️⃣ Verificar sincronización

Con estos comando, podes ver si tu nodo esta corriendo o si esta sincronizado de una forma muy sencilla.

### 🧠 Nodo de ejecución (Geth)

```bash
curl -X POST -H "Content-Type: application/json" \
--data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
http://localhost:8545
```

- `"result":false` → ya está sincronizado tu cliente de ejecución
- Si devuelve un valor (bloques) → está en proceso de sincronización

### 🔗 Nodo de consenso (Lighthouse)

```bash
curl http://localhost:5052/eth/v1/node/syncing
```

- `"is_syncing": false` → si dice falso, tu cliente de consenso ya esta sincronizado.

El proceso de sincronizacion puede durar varias horas, si ambos comando anteriormente te funcionaron y te devolvieron un valor, quiere decir que esta funcionando bien, solo debes esperar que sincronize, tambien debes ver los Logs para saber si esta cargando bloques veras un porcentaje de sincronizacion o veras nuevos bloques (revisa si dice error o warn, de lo contrario todo bien). Tambien puedes usar GPT y enviarle el resultado de tus logs o preguntarle que esta pasando o que te ayude a entender algun error.

---

## 7️⃣ Configurar firewall
El firewall es lo que permite las conecciones en tu puertos, y en este caso nuestros cliente hablan entre ellos usando estos puertos, asi que debemos abrirlos o darle permiso con los siguientes comandos.

```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 8545/tcp
sudo ufw allow 5052/tcp
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
sudo ufw allow 9000/tcp
sudo ufw enable
```

Si tienes algun problema con esto (que no deberias), puedes usar:

```bash
sudo ufw enable
sudo ufw allow incoming
sudo ufw allow outgoing
```

Esto abrira todas las entradas y salidas en todos los puertos, puedes usarlo para verificar si alguna apertura de puertos te esta dando problemas aunque no es lo mas recomendable por seguridad, idealmente solo abrimos los puertos que necesitamos usar, igualmente en un ambiente de prueba o desarrollo y en un testnet no tenemos ningun problema.

---

## 8️⃣ Acceder a los endpoints RPC

Una ves que tu nodo esta corriendo y actualizado, ya puedes obtener los RPC de consenso (beaconchain) y de ejecucion. Estos RPC son para conectarse a tu nodo, entonces si eres desarrollador uedes usar este rpc, puedes configurar tu wallet para que use tu propio nodo, o si quieres correr otro nodo L2 que necesita hablar con un nodo L1, estos son los RPC que usarias.

En el caso de aztec en su testnet, necesita hablar con el L1 de ethereum en testnet tambien. por eso usamos estos rpc de sepolia testnet de Ethereum l1.

| Nodo       | Endpoint local            | Endpoint externo           |
|------------|---------------------------|-----------------------------|
| Geth       | `http://localhost:8545`   | `http://<IP_VPS>:8545`     |
| Lighthouse | `http://localhost:5052`   | `http://<IP_VPS>:5052`     |

- Endpoint local: usa localhost por que es cuando estas en la misma red
- Endpoint externo: es cuando quieres usarlo desde otra red externa no tu misma red local

Cuando estas en tu red interna, tambien puedes usar tu IP interno depende de la configuracion, este IP interno suele ser 192.168.1....  mientras que el IP externo sera diferente, no iniciara con 192.168.

Por ejemplo, si vas a correr un nodo de aztec en tu mismo servidor local donde tienes tu nodo de sepolia, usarias como URL de tu RPC algo asi:

- Geth: http://192.168.100.149:8545
- Lighthouse: http://192.168.100.149:5052
(192.168.100.149 deberas cambiarlo por tu propio IP privado)

8545: por que es el puerto donde esta el cliente de ejecución
5052: por que es el puerto donde esta el cliente de consenso

Es el mismo IP local del equipo o servidor pero son diferentes puertos. (tanto el ip interno como usar local host es lo mismo, pero si usaste localhost y te da algun error prueba con el IP o viceversa)
Ya que en el ejemplo de Aztec, asi este en tu propio equipo, Docker crea una red personalizada (bridge) para este nodo, por ende no podras usar localhost, deberas usar tu IP interno de tu equipo (192.168..)

---
## Encuentra el IP INTERNO de tu servidor o equipo:
Este comando te retornara varios segmentos de informacion de redes, tu IP Interno o privado, suele ser el que esta en el bloque enp5s0 o enp3s0 y veras una linea que diga inet 192.168...

```bash
ip addr show
```

Sera algo como: inet 192.168.100.149/24 (el IP es solo los digitos sin el /24)  (IP= 192.168.100.149)


## Encuentra el IP EXTERNO de tu servidor o equipo:
Si deseas acceder a tus RPC desde afuera de tu red LAN o local, debes usar el IP publico, asi lo obtenemos:

```bash
curl icanhazip.com
```
Este comando imprimer unicamente tu IP Externo o Pubico para que puedas acceder al nodo desde internet en cualquier otra red.

---

## Port Forwarding:

Cuando quieres acceder a tu nodo local desde una red externa, usaras tu IP externo y pondras :8548 (o el puerto que necesitas acceder segun el cliente al que quieres acceder), el temas es que necesitas configurar o hacer "port forward: de estos puertos, para que tu router o tu red local sepa que cuando llega un mensaje a ese puerto, debe enviarlo a tu equipo donde esta tu nodo especificamente.

Si estas usando un vps (o servidor virtual alquilado, este paso no es necesario ya que los VPS tienen ya un IP publico, pero si estas corriendolo de forma local en una red (como puede ser en tu casa con tu router) entonces si debes de entrar a la configuracion de tu router y hacer port forward de los puertos 8545 para Geth cliente de ejecución y puerto 5052 para Lighthouse cliente de consenso).

Puedes buscar en youtube como hacer port forward o ayudarte con GPT si nunca lo has echo pero es bastante sencillo. (el proceso es diferente depende de la marca de tu router)

Ya con tu nodo sincronizado y tus RPCs puedes proceder a usar tu nodo y hacer lo que quieras con el! 

---

¿Listo para usar tu nodo Sepolia para pruebas y desarrollo? 🚀  
- dudas: @inbestprogram
