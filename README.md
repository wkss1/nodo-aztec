# 🛠️ Guía: Cómo correr un nodo sequencer de Aztec (Ubuntu + Docker)


# ¿Qué es un Nodo Secuenciador?

Un nodo secuenciador en Aztec Network es responsable de:

- Proponer bloques.
- Validar bloques propuestos por otros nodos.
- Votar en actualizaciones de la red.
- Estos nodos son esenciales para el funcionamiento de la red y permiten a los operadores participar activamente en la testnet.


## 📋 Requisitos de hardware

| Componente | Recomendación             |
|------------|---------------------------|
| Sistema    | Ubuntu 20.04 o superior   |
| RAM        | 16 GB                 |
| CPU        | 8 núcleos             |
| Disco      | 1 TB SSD              |

---

En esta guía iremos paso a paso de forma sencilla para montar un nodo sequenciador de Aztec L2 de forma local, tambien puedes seguir estos pasos en un VPS (virtual private server, o un servidor alquiulado en la nube)

Para esta guía necesitaremos tener RPCs de sepolia, Aztec es un L2 y para correr necesita estar en contacto con el L1 ya que los L2 dependen del L1, por eso necesitamos RPCs de un nodo de SEPOLIA que es el testnet de Ethereum L1!

Existen distintos RPC gratis que podemos usar, pero estan limitados y dan problemas para correr aztec, por eso es necesario tener un RPC sin limitaciones, para esto puedes pagar un rpc (entre $50-$250 por mes) o mejor aun, mas barato y aprenderas muchos mas si corres tu propio nodo de sepolia y usas tus propios RPC sin limitaciones y gratis!

Un RPC no es mas que una llamada que permite interactuar con el nodo, en el caso de un desarrollador, tener un RPC te permite usarlo directamente y tener acceso a la blockchain desde tu nodo sin depender de un proveedor pago o uno gratis que suelen estar limitados.

SI QUIERES CORRER TU PROPIO NODO DE SEPOLIA, SIGUE ESTA GUIA: https://github.com/wkss1/nodo-sepolia/tree/main

---

## Sistema Operativo: Ubuntu

Puedes seguir esta guía en tu equipo local, o puedes tambien utilizar un VPS (alquilar un servidor virtual) para tener tu nodo, si embargo tambien se puede hacer en equipos windows usando windows subsystem para acceder a ubuntu o mac con ciertos pasos extra. Hoy nos enfocamos en como hacerlo con Ubuntu.

---

## 1️⃣ Instalar dependencias y Docker

Usaremos docker para la instalacion del nodo, asi que con estos comando actualizamos el sistema y descargamos las dependencias (software que necesitas para poder correr el nodo)

### 🔧 Dependencias básicas

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```


### 🐳 Instalar Docker

Una ves listo, procedemos a descarga y probar Docker

```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world

sudo systemctl enable docker
sudo systemctl restart docker
```

---

## 2️⃣ Instala la herramienta de Aztec

Esta es la herramienta que usamos para correr el nodo de Aztec, vamos a instalarla. en el proceso de instalacion te preguntara si quieres continuar, y debes darle Y para aceptar (Y significa si)

```bash
bash -i <(curl -s https://install.aztec.network)
```
Luego de instalarla, creamos el PATH

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc

source ~/.bashrc
```

Una vez echo esto, es necesario REINICIAR EL TERMINAL para aplicar los cambios.

Luego si todo salio bien, escribe en tu terminal el comando Aztec y te debera retornar las opciones, asi verificamos que se instalo correctamente.

```bash
aztec
```

---

## 3️⃣ Actualiza Aztec a la última version:

```bash
aztec-up latest
```

Este comando actualizara la herramienta a la ultima version disponible, lo cual es necesario para evitar errores mas adelante.

---

## 4️⃣ Obtener RPC URLs

Como mencionamos, para correr aztec es necesario tener RPCs de Sepolia L1, por esto debemos primero obtener el URL de estos RPCs.

Necesitamos 2 RPCs:
- 1 para el cliente de ejecucion
- 1 para el consenso o beaconchain

Estos son los RPCs que  necesitamos darle a nuestro nodo de Aztec, aqui tienes opciones.

- Crea tu propio nodo y usa tus RPCs con esta GUIA: https://github.com/wkss1/nodo-sepolia/tree/main
- Paga algun proveedor de RPC como Chainstack, Alchemy etc. y simplemente usa esos RPCs para tu nodo
- Los RPCs gratis no te serviran ya que estan limitados.

Una vez que ya tenemos los RPC URLs que vamos a utilizar podemos seguir!

Si pagas algun proveedor de RPC, en su pagina encontraras los URLs para estos RPCs, deberas buscar uno para Execution y otro para Consensus o Beaconchain.


Si corres tu nodo local, tu URL sera: http://localhost:XXXX o tu IP pirvado: 192.168.1.1:XXXX

- donde XXXX sera el puerto que utiliza dicho cliente.
- o si usas 192.168.... debes remplazarlo por tu propio IP privado.
- Si corres tu nodo pero no en la misma red local donde esta tu nodo de aztec, deberas usar tu IP PUBLICO envez de local host o el ip privado

GUIA PARA CORRER TU NODO DE SEPOLIA Y OBTENER LOS RPC URLs: https://github.com/wkss1/nodo-sepolia/tree/main

---

## 5️⃣ Genera tus llaves de Ethereum


Es necesario tener una cuenta o wallet de ethereum sepolia para correr el nodo de Aztec, vamos a necesitar el ADDRESS y la PRIVATE KEY.

Creas una cuenta NUEVA sin fondos en metamask, guarda tu address y copia y guarda tambien el private key.


- Luego necesitamos tener ETH Sepolia o ETH de prueba en la red de sepolia en este wallet que recien creamos, con 0.1 ETH esta mas que bien.


### Puedes obtenerlo en diferentes Faucets:

- https://faucetlink.to/sepolia

---

## 6️⃣ 🔍 Encuentra tu IP Publico:

Otra cosa que necesitamos para correr el nodo de sepolia, es saber cual es nuestro IP Publico del equipo donde estamos corriendo el nodo, podemos encontrarlo con el siguiente comando. (este es el IP publico y se utiliza para que otros servicios fuera de tu red local puedan acceder a tu nodo)

```bash
curl ipv4.icanhazip.com
```
Guardalo junto con tu addres y tu clave privada que las necesitaremos mas adelante.

---

## 🔍 Abrir Puertos y Firewall:

Es necesario abrir estos puertos en tu firewall para que el nodo pueda comunicarse de forma correcta.

```bash
# Firewall
ufw allow 22
ufw allow ssh
ufw enable

# Sequencer
ufw allow 40400
ufw allow 8080
```

Si tienes algun problema con esto (que no deberias), puedes usar:

```bash
sudo ufw enable
sudo ufw allow incoming
sudo ufw allow outgoing
```

Esto abrira todas las entradas y salidas en todos los puertos, puedes usarlo para verificar si alguna apertura de puertos te esta dando problemas aunque no es lo mas recomendable por seguridad, idealmente solo abrimos los puertos que necesitamos usar, igualmente en un ambiente de prueba o desarrollo y en un testnet no tenemos ningun problema.

---

## 7️⃣ Ahora, Porfin! a correr el NODO! 

En este caso no vamos a usar docker compose, si no que lo correremos directamente en el terminal con este comando para simplificar el proceso, al hacerlo de esta manera tu nodo queda corriendo en tu terminal y en cierta forma "se bloquea el terminal" ya que solo veras los outputs del nodo y no quieres cerrarlo por que cierras el proceso, queremos dejarlo activo!

Podemos hacerlo normal y luego abrir otra ventana del terminal para en esta segunda ventana seguir trabajando y dejar la ventana inicial con el nodo abierto viendo los Logs y revisandolo.

Tambien si prefieres, podemos usar SCREEN que es un comando que nos permite abrir una ventana vacia para correr el nodo aqui y luego salir de esta ventana y que el nodo siga corriendo en el fondo, asi no tener que dejar abierta la ventana del nodo.

Cualquier forma funciona! si prefieres mantenerlo simple, solo hazlo sin SCREEN en tu ventana principal del terminal y luego abres otra ventana para seguir trabajando!

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls RPC_URL  \
  --l1-consensus-host-urls BEACON_URL \
  --sequencer.validatorPrivateKey YourPrivateKey \
  --sequencer.coinbase YourevmAddress \
  --p2p.p2pIp IP
```

  REEMPLAZA los siguientes valores:

- RPC_URL & BEACON_URL: Aqui colocar los URL a tus RPC (paso 4)
- YourPrivateKey: Aqui la clave privada de tu wallet creado (Paso 5)
- YourAddress: La dirección del wallet creado (Paso 5)
- IP: Tu IP Publico (Paso 6)


### 🧠 Verifica tu nodo y la Sincronización

Si estas corriendo el nodo sin screen y tienes abierta la ventana con los logs, recuerda no cerrarla y en esta ventana puedes ver los resultados que van arrojando los Logs, aqui veras si el nodo se esta actualizando y saliendo nuevos bloques, al inicio debes darle tiempo, si no te arroja ningun error y sigue "pensando" todo esta perfecto solo dale tiempo.

En los Logs veras cuando ya este actualizado el numero de bloque por el cual va tu nodo, y para asegurarte que esta sincronizado, puedes visitar el explorador de bloques de aztec, ver el numero de bloque actual de la red y verificar si tu nodo esta en ese mismo bloque!

- https://aztecscan.xyz/

Si corriste el nodo con SCREEN o simplemente quieres abrir los LOGS desde otro terminal para ver como va, usa estos comando:

```bash
docker ps
```

Este comando te devolvera los contenedores de docker que estan corriendo en este momento, si tienes aztec activo lo veras aqui, si no te sale aztec aqui, no lo estas corriendo bien.

Tambien te dira el ID del contenedor y el Nombre del contenedor.

- Puede ser algo como: 


CONTAINER ID   IMAGE                        COMMAND                  CREATED        STATUS        PORTS                NAMES                                                                                                                           
169f0e72e866   aztecprotocol/aztec:latest   "node --no-warnings …"   24 hours ago   Up 24 hours   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp, 0.0.0.0:40400->40400/tcp, [::]:40400->40400/tcp, 0.0.0.0:40400->40400/udp, [::]:40400->40400/udp                       aztec-start-18af871c


-En este caso el ID es: 169f0e72e866
-Y el Nombre es: aztec-start-18af871c

Los tuyos seran diferentes, pero una vez que lo tienes, puedes usar el comando de DOCKER LOGS para abrir los Logs de ese contenedor y ver como va la sincronización o si dio algún error:

```bash
docker logs <NOMBRE o ID de tu contenedor de aztec>
```

Si el nodo esta sincronizado veras algo asi, y el numero de bloque coincidira con el bloque actual en el explorador de bloques de aztec (https://aztecscan.xyz/)

EJEMPLO: 
```bash
[21:07:22.126] INFO: p2p:libp2p_service:peer_manager Connected to 99 peers {"connections":99,"maxPeerCount":100,"cachedPeers":102,"medianScore":-2.8432599355836136e-32}
[21:07:37.927] INFO: archiver Downloaded L2 block 46516 {"blockHash":{},"blockNumber":46516,"txCount":1,"globalVariables":{"chainId":11155111,"version":150103439,"blockNumber":46516,"slotNumber":74569,"timestamp":1747429656,"coinbase":"0xc425a109e331c83f2e46261cd5505aaac8da9a4a","feeRecipient":"0x0000000000000000000000000000000000000000000000000000000000000000","feePerDaGas":0,"feePerL2Gas":351610}}
[21:08:22.137] INFO: p2p:libp2p_service:peer_manager Connected to 100 peers {"connections":100,"maxPeerCount":100,"cachedPeers":140,"medianScore":-2.558933942025252e-32}
[21:08:26.640] INFO: archiver Updated proven chain to block 46508 {"provenBlockNumber":46508}
[21:08:50.392] INFO: archiver Downloaded L2 block 46517 {"blockHash":{},"blockNumber":46517,"txCount":1,"globalVariables":{"chainId":11155111,"version":150103439,"blockNumber":46517,"slotNumber":74571,"timestamp":1747429728,"coinbase":"0x0000000000000000000000000000000000000000","feeRecipient":"0x0000000000000000000000000000000000000000000000000000000000000000","feePerDaGas":0,"feePerL2Gas":295720}}
```


## 8️⃣ Reclama tu ROL en el Discord de Aztec

Una vez sincronizado tu nodo, solo debemos obtener lgunos valores de nuestro nodo e ingresarlo en el discord en el canal de (OPERATORS | START HERE)

En este canal debes escribir en el chat lo siguiente:
/operator start


cuando lo hagas te saldra en tu campo para escribir en el chat un mensaje con 3 campos vacios:

- Address: aqui deberas poner la dirección de tu wallet que usaste en el nodo
- Block Number: Aqui deberas poner el numero de bloque que obtendremos
- Prove: Aqui pondras el codigo de la prueba del bloque nque obtendremos

---

### Obtener los valores en tu nodo:

Con este comando obtendremos el numero del ultimo bloque provado por el nodo (el bloque provado no es lo mismo que el ultimo bloque sincronizado, sera un bloque con un numero menor al ultimo sincronizado, esta bien)

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

El comando utiliza LOCALHOST:8080 ya que 8080 es el puerto que usa AZTEC y LOCALHOST por que estoy corriendo el comando desde la misma maquina donde corro mi nodo, si no lo estas haciendo de forma local, remplaza LOCALHOST con tu IP y manten el puerto 8080.

Luego necesitamos Obtener el codigo de la prueba de ese numero de bloque que obtuvimos:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```

En este comando debes REEMPLAZAR donde dice BLOCK_NUMBER y poner alli el numero de bloque que obtuvimos en el comando anterior. (ten en cuenta que BLOCK_NUMBER aparece 2 veces en el comando y debes remplazarlo en ambos)

Ya con los valores de ADDRESS, BLOCK y PROVE. puedes ir al discord y seguir los pasos que vimos anteriormente para obtener el rol!
  
![Screenshot 2025-05-16 at 3 21 36 PM](https://github.com/user-attachments/assets/81c2bd3a-6842-4f83-91ec-aa2e9d656538)


Ya aqui estamos listos, y si quieres obtener el siguiente ROL en su discord, solo debes tener tu nodo activo por mas tiempo y eventualmente podras subir de nivel y reclamar tu rol de GUARDIAN!

### Guardian:
Es un Rol en el Discord de Aztec que obtienen aquellos queya tienen rol de Aprentice que tienen su nodo activo y cumple con cierto tiempo de tenerlo estable. Cada cierto tiempo se toma un snapshot para ir identificando aquellos que cumplen el requisito. Para obtenerlo debes correr tu nodo de forma estable y constante.

### Sentinel: 
Es un Rol en el Discord de Aztec del cual no hay mucha informacion, mas que si vez uno sabes que es un dios de los nodos.

---

## Port Forwarding:

Cuando quieres acceder a tu nodo local desde una red externa (o para que tu nodo pueda ser descubierto por otros), usaras tu IP externo y pondras :40400 (o el puerto que necesitas acceder segun el nodo), el tema es que necesitas configurar o hacer "port forward: de estos puertos, para que tu router o tu red local sepa que cuando llega un mensaje del exterior a ese puerto, debe enviarlo a tu equipo donde esta tu nodo especificamente.

Si estas usando un vps (o servidor virtual alquilado, este paso no es necesario ya que los VPS tienen ya un IP publico, pero si estas corriendolo de forma local en una red (como puede ser en tu casa con tu router) entonces si debes de entrar a la configuracion de tu router y hacer port forward del puerto 40400.

Puedes buscar en youtube como hacer port forward o ayudarte con GPT si nunca lo has echo pero es bastante sencillo. (el proceso es diferente depende de la marca de tu router)

---

## 7️⃣ Regístrate como Validador (opcional)

Si quieres ir mas alla, puedes también registrarte como validador de Aztec

ya con tu nodo corriendo y sincronizado, solo debes utilizar este comando si quieres registrarte como validador!
(no es necesario para el rol)

```bash
aztec add-l1-validator \
  --l1-rpc-urls RPC_URL \
  --private-key your-private-key \
  --attester your-validator-address \
  --proposer-eoa your-validator-address \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111
```

REEMPLAZA (RPC_URL), (your-validator-address) & 2x(your-validator-address), lo demas lo dejas tal cual y  continuamos!

Recibiras un mensaje de este estilo:

```bash
Adding validator (xxxxxx83d3442508ad63f3afa7f4e874xxxx, xxxxxd3442508ad63f3afa7f4e874b78269xxxx [forwarder: 0x871e7294B54dA07cFd71A95b6e2E66d86BcE41f8]) to rollup 0x8D1cc702453fa889f137DBD5734CDb7Ee96B6Ba0
[06:34:36.706] INFO: cli Adding validator (xxxx3d3442508ad63f3afa7f4e874b7826xxxxx, xxxx3d3442508ad63f3afa7f4e874b782xxxx [forwarder: 0x871e7294B54dA07cFd71A95b6e2E66d86BcE41f8]) to rollup 0x8D1cc702453fa889f137DBD5734CDb7Ee96B6Ba0
Transaction hash: xxx066cfd1d3a0ec29adfe3fe4ac0b11fb91bfbe049f268179eb9xxxxx
[06:34:37.864] INFO: cli Transaction hash: xxxx66cfd1d3a0ec29adfe3fe4ac0b11fb91bfbe049f268179eb9ee9def27xxx
```

Se estan abriendo 5 espacios al dia para validadores en la red, por eso es muy probable que si intentas hacerlo recibas errores, ya que esta muy competitivo lograr ser validador, esto taambien requiere mayor conocimiento tecnico!

- Ahora mismo se abren las 5 plazas al dia a las UTC 21: 49 :12 

Pero no solo se trata de ser el mas rapido, sino tambien de el que paga mas!

Estamos en un testnet en sepolia y los ETH no tienen "valor" o almenos monetario, pero aun asi se estan registrando transferencias en la red donde usuarios ponen un fee altisimos de ETH para lograr que su transaccion se procese primero y asegurarse un espacio como validador, mientras la mayoria de usuarios que no entienden estos temas, solo mandan un comando que hace la transaccion con el gas promedio.

Para esto debemos buscar la documentacion de aztec y ver los comando CLI que tenemos disponibles, y en este caso nos interesa usa el siguiente comando al momento de registrarnos como validador: 

```bash
--archiver.maxGwei XXXXX \
--archiver.fixedPriorityFeePerGas XXXXX
```
Estos 2 comandos debemos incluirlos cuando usamos el comando para Registrar el validador en el paso 7!
y en XXXXX pondriamos el valor que queremos usar el cual suplantara al valor de gas base

---

Si queremos ser mas Degens, podemos tambien crear un script que en un timestamp especifico corra el comando por nosotros con un fee mas elevado y usando algo como foundry para enviar la transacción manual o de forma directa a la blockchain a travez de nuestro RPC, asi asegurarnos de ser los primero en llegar!


 # Video con el paso a paso: https://www.youtube.com/watch?v=6C7xb2kKAfw
---

¿Listo para usar tu nodo Aztec? 🚀  
gAztec

@inbestprogram
https://x.com/inbestprogram
