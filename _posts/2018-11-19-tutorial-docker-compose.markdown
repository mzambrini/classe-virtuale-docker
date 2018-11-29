---
layout: post
title:  "Docker tutorial - Il tool docker-compose"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questo laboratorio, si vedrà come utilizzare il tool di composing (docker-compose) per creare sistemi di containers (su singolo host) connessi e operanti tra di loro.

Per lo scopo, utilizzeremo un semplice progetto consistente in un servizio REST implementato con Node.js e CouchDB.

> **Difficoltà**: Facile

> **Durata**: Circa quindici minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio e considerazioni sul tool docker-compose](#Task_1)
> * [Task 2: Analisi della struttura del progetto](#Task_2)
> * [Task 3: Creazione dei servizi](#Task_3)
> * [Task 4: Analisi dei logs](#Task_4)
> * [Task 5: Uno sguardo all'applicazione ](#Task_5)
> * [Task 6: Sincronizzazione dei containers](#Task_6)

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

### Il file  docker-compose.yml
Verifichiamo il contenuto del file docker-compose.yml
```.term1
   cat docker-compose.yml
```
ovvero:
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

#### Couchdb
E' il database utilizzato dall'applicazione.

Per gli scopi didattici del progetto non abbiamo necessità di utilizzare un *volume* specifico per gestire la persistenza; se rimuoviamo il container, i nostri dati vanno persi.

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **always (sempre)**

Si attesta sulla rete bridged **node-example**

___

#### Db-init
E' un container che ha come unico scopo quello di inizializzare il database; una volta terminato il compito si arresta da solo

La cartella di **context build** è *db-init*, subfolder della cartella in cui risiede il file *docker-compose.yml*

Il nome del file Dockerfile è proprio **Dockerfile** (in questo caso, questa istruzione poteva essere omessa in quanto valore default)

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **on failure (in caso di fallimento)** 

Dipende dal container **couchdb**

Si attesta sulla rete bridged **node-example**

___

#### Node-app
E' il container dell'applicazione REST Node.js vera e propria

La cartella di **context build** è *node-app*, subfolder della cartella in cui risiede il file *docker-compose.yml*

Anche per questo container il nome del file Dockerfile è proprio **Dockerfile**

Utilizza le variabili di ambiente presenti nel file esterno.
  
La sua policy di restart è **on failure (in caso di fallimento)** 

Espone la sua porta 3000 sull'omonima porta dell'host

Dipende dal container **couchdb**

Si attesta sulla rete bridged **node-example**

___


## <a name="Task_3"></a>Creazione dei servizi

A questo punto possiamo avviare lo stack di containers con il seguente comando:
```.term1
   docker-compose -p esempio up -d
```
### Analisi della sintassi del comando
* **docker-compose up**: il sottocomando che effettua le build, crea (o ricrea), avvia i containers di un servizi
* **-p esempio**: specifica il nome del progetto, di default è il nome del folder in cui viene eseguito il comando
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*

**Attenzione all'ordine dei flag!**
Il flag **-p** è relativo al comando docker-compose, il flag **-d** al sottocomando **up**,
l'istruzione "*docker-compose up -p esempio  -d*" non funziona!!!!

### Effetti del comando
docker-compose analizza e valida il file **docker-compose.yml**; crea tutte le strutture ivi dichiarate (volumi, immagini, reti, ecc...) e avvia i containers in base alle dipendenze dichiarate. 
Assegnerà la stringa **esempio** quale nome di progetto ed si distaccherà dai processi lasciandoli eseguire in background.

## <a name="Task_4"></a>Analisi dei logs
### Visualizzazione dei logs
Il tool docker-compose mette a disposizione anche un proprio sottocomando per visualizzare i logs di tutti i containers creati: 
```.term1
   docker-compose -p esempio logs -f -t --tail=20
```
### Analisi della sintassi del comando
* **docker-compose logs**: il sottocomando che visualizza i log di uno specifico progetto
* **-p esempio**: specifica il nome del progetto
* **-f**: si aggancia al sistema di logging (Ctrl + c per sganciarsi)
* **-t**: visualizza il timestamp del log
* **--tail=20**: il numero di linee (a partire dalla fine) di log visualizzato per ogni container, *--tail="all"* per stampare tutte le linee

### Effetti del comando
docker-compose si aggancia ai logs di tutti i containers definiti nel file *docker-compose.yml* e stampa le ultime venti righe di ciascuno di essi visualizzando il timestamp di ogni linea

### Apriamo l'applicazione
## <a name="Task_5"></a>Uno sguardo all'applicazione 
A questo punto il sistema dovrebbe essersi correttamente avviato.

Trattandosi di un'applicazione REST, possiamo visualizzare la schermata **[explorer dell'applicazione](/explorer){:data-term=".term1"}{:data-port="3000"}** 

Oppure eseguire una **[semplice invocazione di un metodo REST](/api/Banks/3150447){:data-term=".term1"}{:data-port="3000"}**  



## <a name="Task_6"></a>Sincronizzazione dei containers

In un sistema di containers connessi tra di loro è naturale che ci siano dipendenze: nel nostro caso, i containers **db-init** e **node-app** dipendono dal container **couchdb** come esplicitato dall'opzione **depends-on**.

docker-compose garantisce che i containers vengano sempre avviati nell'ordine di dipendenza; non esiste solo **depends_on** (come nel nostro caso) ma anche altre istruzioni quali **links**, **volumes_from** e **network_mode**.

Tutto bene, quindi?

**NO**

docker-compose  non attende che il container sia "pronto" ma solo che sia stato correttamente avviato.
Il problema di aspettare che una risorsa sia pronta, fa parte, in realtà, di un insieme di problematiche dei sistemi distribuiti.
Ogni singola applicazione deve essere *resiliente* a queste problematiche. Se il database va giù, l'applicazione che lo utilizza dovrebbe essere in grado di tentare una riconnessione.

Nel nostro esempio, la soluzione migliore, è eseguire questo controllo nel codice dell'applicazione, sia all'avvio che in caso di perdita della connessione per qualsiasi motivo. 

Un aiuto, per evitare di dovere scrivere il proprio codice da soli, ce lo forniscono alcune librerie:
* **[wait-for-it]https://github.com/vishnubob/wait-for-it**
* **[dockerize]https://github.com/jwilder/dockerize**
* **[wait-for (sh compatibile)]https://github.com/Eficode/wait-for**

In questo esempio, è stata utilizzata la libreria **dockerfile**

Se visualizziamo il Dockerfile del progetto node-app vedremo:
```.term1
   cat node-app/Dockerfile
```
questo:
```
FROM node:latest

ENV DOCKERIZE_VERSION v0.6.1

RUN apt-get update && apt-get install -y wget \
&& wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
&& tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
&& rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

WORKDIR /usr/src/app
COPY ./src/package*.json ./

RUN npm install

COPY ./src .

EXPOSE 3000

ENTRYPOINT exec dockerize -wait http://${COUCHDB_USER}:${COUCHDB_PASSWORD}@${COUCHDB_HOST}:${COUCHDB_PORT}/tests_empty  -timeout 120s npm start

```

Con la prima istruzione **RUN**, tra le altre cose, andiamo ad installare l'applicazione *dockerfile* nel nostro container.
Con l'istruzione **ENTRYPOINT** andiamo a invocare lo strumento che è sostanzialmente un wrapper al nostro metodo **npm start**

Nello specifico, il flag **-wait [indirizzo http] -timeout 120s** chiede a dockerize di attendere un esito positivo (*HTTPS 200 code*) dall'indirizzo HTTP fornito. Se questo non accade entro il timeout definito, lo script uscirà con un codice di errore facendo arrestare il container.

