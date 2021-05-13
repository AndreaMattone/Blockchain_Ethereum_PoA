# EthereumPoABlockchain

* **[Pre-requisites](#pre-requisites)**
* **[Setup](#setup)**
* **[Run](#run)**

## Getting Started

These instructions will allow you to connect to an ethereum private Proof Of Authority network. <https://hackernoon.com/setup-your-own-private-proof-of-authority-ethereum-network-with-geth-9a0a3750cda8>
If you clone this repo you have the pre-inizializated blockchain (with Node1 and Node2 Sealers and Node3 not a Sealer) in /myEthPoa and you can directly go to the "Run" module.

NOTE: You can also use WindowsPowerShell but Ubuntu WLS usually works better.



## Pre-requisites
* If you are in Windows install Ubuntu WSL from <https://ubuntu.com/wsl>
Once you have installed Ubuntu WSL go to 
```sh
C:\Users\userName\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\home\user\ 
```
Make a new desktop shortcut with this path, you are going to create the Blockchain folder to this path (because in WSL this is the default path).

Initialize the workspace creating "myEthPoa" folder in this path.

When you open the Ubuntu wls terminal with "ls" you should see the folder. 
ONLY if you dont see the folder go to
```sh
andrea@Ouezzane:~$ cd ..
andrea@Ouezzane:/home$ cd ..
andrea@Ouezzane:/$ ls
bin   dev  home  lib    lib64   media  opt   root  sbin  srv  tmp  var
boot  etc  init  lib32  libx32  mnt    proc  run   snap  sys  usr
andrea@Ouezzane:/$ cd home
andrea@Ouezzane:/home$ ls
andrea
andrea@Ouezzane:/home$ cd andrea
```


* Install go-ethereum 

* Install puppeth


## Setup

###  1]  Enviroment

Initialize the workspace

```sh
$> cd myEthPoa
$> mkdir node1 node2 node3
```

Create the accounts

```sh
$> mkdir myEthPoa
$> cd myEthPoa
$> mkdir node1 node2 node3
```

Accounts (wallet) have a private-public key to interact with the blockchain.
A Sealer node needs the private key to send transactions and the public key to identify itself in the network.
A Sealer node can vote other nodes to become Sealers in a PoA Blockchain.


For every node (in this case for node1)

```sh
myEthPoa$> geth --datadir node1/ account new
```

Let's set a password for the node
```sh
myEthPoa$> 
INFO [05-11|13:39:34.555] Maximum peer count ETH=50 LES=0 total=50
Your new account is locked with a password. Please give a password. Do not forget this password.
Password: pswnode1
RepeatPassword: pswnode1

Your new key was generated

Public address of the key:   0x146dCA62D5353bB20C116E3F8A14A4f975C3A173
Path of the secret key file: node1/keystore/UTC--2021-05-11T11-52-58.571600400Z--146dca62d5353bb20c116e3f8a14a4f975c3a173

- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```
Create a file node1/password.txt and save the password in it.
Repeat for every node.


Now create myEthPoa/accounts.txt save the public addresses in it.


###  2]  Create a genesis file
We are going to create the genesis file with puppeth. We need the genesis to initialize the blockchain. the genesis is the first block of the chain.

First open a new Ubuntu WLS terminal and run puppeth in the blockchain folder
```sh
myEthPoa$> puppeth
```
Puppeth terminal will open


```sh
Please specify a network name to administer (no spaces, hyphens or capital letters please)
> myethpoa

What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

What would you like to do? (default = create)
 1. Create new genesis from scratch
 2. Import already existing genesis
> 1

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 2

How many seconds should blocks take? (default = 15)
> 5
```

Now we are going to set the accounts that are able to Seal, the Sealer accounts.
In this example Node1 and Node2 are Sealers, Node3 is not.

```sh
Which accounts are allowed to seal? (mandatory at least one)
> 0x146dCA62D5353bB20C116E3F8A14A4f975C3A173
> 0xd087cCDB19b917bbb5880a623675CBde4308B124
> 0x

Which accounts should be pre-funded? (advisable at least one)
> 0x146dCA62D5353bB20C116E3F8A14A4f975C3A173
> 0xd087cCDB19b917bbb5880a623675CBde4308B124
> 0x

Should the precompile-addresses (0x1 .. 0xff) be pre-funded with 1 wei? (advisable yes)
>


```

```sh
Specify your chain/network ID if you want an explicit one (default = random)
> 1555
INFO [05-11|14:07:17.360] Configured new genesis block

What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> 2

 1. Modify existing configurations
 2. Export genesis configurations
 3. Remove genesis configuration
> 2

Which folder to save the genesis specs into? (default = current)
  Will create myethpoa.json, myethpoa-aleth.json, myethpoa-harmony.json, myethpoa-parity.json
>
```

Now we have our genesis file.



###  3]  Initialize nodes
Every node should initialize on the same genesis.

```sh
myEthPoa$> geth --datadir node1/ init myethpoa.json
myEthPoa$> geth --datadir node2/ init myethpoa.json
myEthPoa$> geth --datadir node3/ init myethpoa.json
```




###  4]  Create the bootnode
The bootnode allow the nodes to find each other in the network.
Nodes can have dinamic IPs, bootnode runs on a static IP so every node can use it to dispatch requests and find other nodes.

```sh
myEthPoa$> bootnode -genkey boot.key
```
This create boot.key that contains an enode value that identifies the bootnode.





## Run

###  5]  Start the bootnode service

In a Ubuntu wls terminal start the bootnode
```sh
myEthPoa$> bootnode -nodekey boot.key -verbosity 9
```

###  6]  Start the nodes

In a different Ubuntu wls terminal start the nodes (may be necessary run sudo before the command).

* identity: node name
* datadir: directory where the blockchain is stored
* syncmode: value that indicates the sinc option of the block (fast, full, light)
* port: listening port of the network
* rpc: start the http JSON-RPC interface
* rpcaddr: listening interface of the http-RPC server
* rpcport: listening port of the http-RPC server
* rpccorsdomain: list of the domains from where requests are accepted 
* rpcapi: list of API offered by http-RPC interface
* boonode: enode of the bootnode
* networkid: identifier of th network
* ipcpath path where you want to store the geth.ipc file (to attach)

Node1
```sh
geth --identity 'node' --allow-insecure-unlock --mine -unlock 0x146dCA62D5353bB20C116E3F8A14A4f975C3A173 --password node1/password.txt --datadir node1/ --syncmode 'full' --port 30311 --rpc --rpcaddr 127.0.0.1 --rpcport 8501 --rpccorsdomain "*" --rpcapi 'personal,db,eth,net,web3,txpool,miner,clique' --bootnodes 'enode://4eea9d83df5f8a674367d6efd7643c170c7821658d244445515144ea07bd39b9c7ad45dffdbf1966ec3e2954e0d6a9ce2376c1a9e6e5d9c0bbd14e52573035a6@127.0.0.1:0?discport=30301' --networkid 1555 --ipcpath "./node1/geth.ipc"
```

Node2
```sh
geth --identity 'node' --allow-insecure-unlock --mine -unlock 0xd087cCDB19b917bbb5880a623675CBde4308B124 --password node2/password.txt --datadir node2/ --syncmode 'full' --port 30312 --rpc --rpcaddr 127.0.0.1 --rpcport 8502 --rpccorsdomain "*" --rpcapi 'personal,db,eth,net,web3,txpool,miner,clique' --bootnodes 'enode://4eea9d83df5f8a674367d6efd7643c170c7821658d244445515144ea07bd39b9c7ad45dffdbf1966ec3e2954e0d6a9ce2376c1a9e6e5d9c0bbd14e52573035a6@127.0.0.1:0?discport=30301' --networkid 1555 --ipcpath "./node2/geth.ipc"
```

Node3 (NB no --mine because is not a Signer)
```sh
geth --identity 'node' --allow-insecure-unlock -unlock 0xe1Cd1937470F2598DED3CF4AFF49880A5709Da01 --password node3/password.txt --datadir node3/ --syncmode 'full' --port 30313 --rpc --rpcaddr 127.0.0.1 --rpcport 8503 --rpccorsdomain "*" --rpcapi 'personal,db,eth,net,web3,txpool,miner,clique' --bootnodes 'enode://4eea9d83df5f8a674367d6efd7643c170c7821658d244445515144ea07bd39b9c7ad45dffdbf1966ec3e2954e0d6a9ce2376c1a9e6e5d9c0bbd14e52573035a6@127.0.0.1:0?discport=30301' --networkid 1555 --ipcpath "./node3/geth.ipc"
```

Now your chain is online!


## Interact with nodes
###  7]  Interact with the nodes using clique

Open a new Ubuntu WLS terminal and enter the blockchian folder and run (may be necessary to use sudo)
```sh
myEthPoa$> cd node1 
myEthPoa$> geth attach geth.ipc
```




