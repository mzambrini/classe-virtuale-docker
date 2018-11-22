---
layout: post
title:  "Docker tutorial - Il tool docker-compose"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questo laboratorio, si vedrà come utilizzare il tool docker-compose per creare sistemi di containers (su singolo host) connessi e operanti tra di loro.

Per lo scopo, utilizzeremo un semplice progetto consistente in un servizio REST implementato con Node.js e CouchDB.

> **Difficoltà**: Facile

> **Durata**: Circa quindici minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio e considerazioni sul tool docker-compose](#Task_1)
> * [Task 2: Analisi della struttura del progetto](#Task_2)
> * [Task 3: Creazione del container con binding su cartella dell'host](#Task_3)
> * [Task 4: Creazione del container utilizzando un volume](#Task_4)

## <a name="Task_1"></a>Cloning del progetto di esempio e considerazioni sul tool docker-compose

Eseguiamo i comandi:
```.term1
   git clone https://github.com/mzambrini/tutorial-docker-compose.git
   cd tutorial-docker-compose
```
Con questo comando abbiamo clonato sul nostro terminale virtuale il progetto presente su GitHub e ci siamo posizionati all'interno della cartella di progetto.

### docker-compose
Docker compose è uno strumento utilizzato per definire, buildare, comporre ed eseguire applicazioni docker multi-container.
Le definizioni dei vari container, dei volumi, delle porte, ecc... sono contenute in uno o più files che utilizzano una grammatica di tipo [yaml](http://yaml.org/).
La specifica di tale file è consultabile su [questa pagina](https://docs.docker.com/compose/compose-file/)

## <a name="Task_2"></a>Analisi della struttura del progetto

### Struttura del progetto
Quella che segue è la struttura dei folder di progetto:
```
│   .gitignore
│   docker-compose.yml
│   global-variables.env
│   README.md
│
├───api
│       swagger.yaml
│
├───db-init
│   │   Dockerfile
│   │
│   └───work
│           entrypoint.sh
│           sample-data.json
│           subsample.json
│
└───node-app
    │   .dockerignore
    │   Dockerfile
    │
    └───src
        │   .editorconfig
        │   .eslintignore
        │   .eslintrc
        │   .gitignore
        │   .yo-rc.json
        │   package-lock.json
        │   package.json
        │   README.md
        │
        ├───client
        │       README.md
        │
        ├───common
        │   └───models
        │           bank.js
        │           bank.json
        │           banks.js
        │           banks.json
        │
        └───server
            │   component-config.json
            │   config.json
            │   datasources.json
            │   middleware.development.json
            │   middleware.json
            │   model-config.json
            │   server.js
            │
            └───boot
                    authentication.js
                    root.js
```

### contenuto del file docker-compose.yml

```.term1
   cat docker-compose.yml
```
il risultato sarà il seguente:
```yaml
version: '2'

services:
  couchdb:
    image: couchdb:2.2
    restart: always
    ports:
      - "5984:5984"
    env_file:
      - global-variables.env
    networks:
      - node-example

  db-init:
    build:
      context: ./db-init
      dockerfile: Dockerfile
    restart: on-failure
    env_file:
      - global-variables.env
    networks:
      - node-example
    depends_on:
      - couchdb

  node-app:
    build:
      context: ./node-app
      dockerfile: Dockerfile
    restart: on-failure
    ports:
      - "3000:3000"
    env_file:
      - global-variables.env
    networks:
      - node-example
    depends_on:
      - couchdb

networks:
  node-example:
    driver: bridge
```

### Variabili di ambiente
Le variabili di ambiente passate ai vari container sono state caricate per comodità nel file **global-variables.env** il cui contenuto possiamo leggere con il comando:

```.term1
   cat global-variables.env
```
ovvero:
```
COUCHDB_USER=couchdbadmin
COUCHDB_PASSWORD=couchdbpassword
COUCHDB_HOST=couchdb
COUCHDB_PORT=5984
```

### Servizi
Abbiamo tre servizi:

#### couchdb
E' il database utilizzato dall'applicazione.

Per gli scopi didattici del progetto non abbiamo necessità di utilizzare un *volume* specifico per gestire la persistenza; se rimuoviamo il container, i nostri dati vanno persi.

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **always (sempre)**

Si attesta sulla rete bridged **node-example**


#### db-init
E' un container che ha come unico scopo quello di inizializzare il database; una volta terminato il compito si arresta da solo

La cartella di **context build** è *db-init*, subfolder della cartella in cui risiede il file *docker-compose.yml*

Il nome del file Dockerfile è proprio **Dockerfile** (in questo caso, questa istruzione poteva essere omessa in quanto valore default)

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **on failure (in caso di fallimento)** 

Dipende dal container **couchdb**

Si attesta sulla rete bridged **node-example**


#### node-app
E' il container dell'applicazione REST Node.js vera e propria

La cartella di **context build** è *node-app*, subfolder della cartella in cui risiede il file *docker-compose.yml*

Anche per questo container il nome del file Dockerfile è proprio **Dockerfile**

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **on failure (in caso di fallimento)** 

Espone la sua porta 3000 sull'omonima porta dell'host

Dipende dal container **couchdb**

Si attesta sulla rete bridged **node-example**


