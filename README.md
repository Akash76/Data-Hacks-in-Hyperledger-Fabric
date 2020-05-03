# HACKING DATA FROM HYPERLEDGER FABRIC BLOCKCHAIN

Hope you people are safe. This is Akash, working as Systems Engineer in TCS where I do Security Analysis for applications. Apart from that I am a blockchain developer mostly working on Hyperledger Fabric and Sawtooth.

Hyperledger Fabric has a complex architecture but is simple to control with the tools available.Internally it is having a good security and privacy with channels, policies etc., which fits it for enterprise Blockchain usecases. But externally it requires a lot of effort to deal with the security aspects of the network. It includes having a firewall protection, maintaining whitelisted domains to deal with APIs, providing security to the docker containers and so on. But point is hackers are smart enough to find a way to exploit and get access into anything. There are tons of ways in which a hacker can compromise a server. If a server is having a participant of fabric network, it could be a peer, orderer, couchDB etc, and if it is compromised, it will be a great loss to the business. Today in this article i would like to show how one can extract data from fabric ledger without being a user or admin.

### SETUP

For this article I am making use of fabcar that is part of fabric-samples provided by fabric. In that we can see that we have a 'InitLedger' function which will be invoked before a user can make use of it. It is initialising ledger with some data. It is this data that we are going to hack. so first lets clear the network and remove stopped containers and prune the docker volume and network so that there wont be any hurdles in the process. For that execute this in fabcar directory

```sh
$ ./networkDown.sh
```

Next lets start the network

```sh
$ ./startFabric.sh
```

By the time the script gets executed completely you will see the running containers as below.

<IMAGE>

Now if you see at the docker-compose file of peers and orderer you can see a volume mounted as orderer.example.com:/var/hyperledger/production/orderer in orderer and peer0.org1.example.com:/var/hyperledger/production in peer. Lets see what is there inside those paths.

### IN PEER VOLUME

In peers, ledger data gets stored in /var/hyperledger/production. production folder has 5 subfolders: **chaincodes**, **externalbuilder**, **ledgersData**, **lifecycle**, **transientStore**. Inside ledgersData we have six folders: **bookkeeper**, **chains**, **configHistory**, **couchdbRedoLogs**, **fileLock**, **historyLeveldb**, **ledgerProvider**, **pvtdataStore**. Inside chains we have two other folders: **chains**, **index**. chains folder has all the channel data, a folder with name mychannel will have complete ledger data of that channel in file **blockfile_000000**.

.   .
│   ├── chains
│   │   ├── chains
│   │   │   └── mychannel
│   │   │       └── blockfile_000000
.   .   .

### IN ORDERER VOLUME

In orderer, ledger data gets stored in /var/hyperledger/production/orderer folder. orderer has 3 folders: **chains**, **etcdraft**, **index**. chains has folders with name of channel and system-channel. All the folders have blockfile_000000. The **blockfile_000000** file in mychannel will have all the ledger data. If we have two or more channels, directories with their respective names will be created as it is orderer.

.
├── chains
│   ├── mychannel
│   │   └── blockfile_000000
│   └── system-channel
│       └── blockfile_000000
.

### DATA EXTRACTION

lets take blockfile_000000 from orderer. It can be seen that fabcar chaincode is using json.Marshal() function to convert data to bytes before writing data to ledger. I have written a small code in js to extract json from a given string. Copying block file to our desired location and executing the js code will give us the json data that is present in ledger as seen below.

```sh
$ node extract.js ./blockfile_000000
[ { make: 'Toyota',
    model: 'Prius',
    colour: 'blue',
    owner: 'Tomoko' },
  { make: 'Ford', model: 'Mustang', colour: 'red', owner: 'Brad' },
  { make: 'Hyundai',
    model: 'Tucson',
    colour: 'green',
    owner: 'Jin Soo' },
  { make: 'Volkswagen',
    model: 'Passat',
    colour: 'yellow',
    owner: 'Max' },
  { make: 'Tesla', model: 'S', colour: 'black', owner: 'Adriana' },
  { make: 'Peugeot',
    model: '205',
    colour: 'purple',
    owner: 'Michel' },
  { make: 'Chery', model: 'S22L', colour: 'white', owner: 'Aarav' },
  { make: 'Fiat', model: 'Punto', colour: 'violet', owner: 'Pari' },
  { make: 'Tata',
    model: 'Nano',
    colour: 'indigo',
    owner: 'Valeria' },
  { make: 'Holden',
    model: 'Barina',
    colour: 'brown',
    owner: 'Shotaro' } ]
```

Here we can observe data that is extracted is actually the one which we stored during initialising ledger. So here I didnt even enrolled Admin or a user to invoke QueryAllCars function but i am able to see the data that is stored in ledger. If a hacker is able to compromise the server in which either a peer or an orderer is stored, he can simply do what ever he want if he has some knowledge on fabric architecture. This may be very dangerous if it is a sensitive application.

### WHAT CAN WE DO?

https://github.com/yeasy/docker-compose-files/blob/master/hyperledger_fabric/v2.1.0/examples/chaincode/go/enccc_example/utils.go
