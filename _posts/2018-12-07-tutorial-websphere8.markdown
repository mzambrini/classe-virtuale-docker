---
layout: post
title:  "WebSphere tutorial - Usiamo docker per le nostre applicazioni WebSphere (parte 1)"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questa prima parte del laboratorio, si vedrà come utilizzare Docker per creare un container WebSphere che possa eseguire la nostra applicazione JavaEE.
Sarà scaricato un semplice progetto da GitHub e verrà buildata l'immagine definita nel Dockerfile.
Il Dockefile è caratterizzato dal fatto di avere una doppia clausola FROM.


> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio](#Task_1)
> * [Task 2: Build dell'immagine con WebSphere 8.5.5.13](#Task_2)
> * [Task 3: Creazione del container (versione 8)](#Task_3)
> * [Task 4: Verifica e test](#Task_4)

## <a name="Task_1"></a>Cloning del progetto di esempio
Eseguiamo i comandi:
```.term1
   git clone https://github.com/mzambrini/tutorial-websphere.git
   cd tutorial-websphere
```
Con questi comandi abbiamo clonato il progetto GitHub sul nostro terminale virtuale e ci siamo posizionati all'interno della cartella di progetto.

La struttura del progetto è la seguente:

```
│   .dockerignore
│   .gitignore
│   Dockerfile8
│   Dockerfile9
│   README.md
│
├───artifacts
│       JSFSample.ear
│
├───config
│       install.json
│
└───scripts
        apply-configuration.sh
```
I file rilevanti sono i seguenti:
* **Dockerfile8**: il file testuale contenente le istruzioni per creare la nostra immagine personalizzata a partire dalla versione 8.5.5.13 di IBM
* **Dockerfile9**: il file testuale contenente le istruzioni per creare la nostra immagine personalizzata a partire dalla versione 9.0.0.6 di IBM
* **JSFSample.ear**: il file ear rappresentante l'applicaizone che vogliamo deployare
* **install.json**: il file json contenente le istruzioni per la deploy
* **apply-configuration.sh**: il file di script che si occuperà di installare la nostra applicazione sul server WebSphere


Eseguiamo il seguente comando per visualizzare il contenuto del **Dockerfile**
```.term1
   cat Dockerfile
```
e il seguente per visualizzare il file di script **apply-configuration.sh**
```.term1
   cat scripts/apply-configuration.sh
```

## <a name="Task_2"></a>Build dell'immagine con WebSphere 8.5.5.13

Eseguiamo ora il comando:
```.term1
   docker build -t my-was8-app -f Dockerfile8 .
```
alla fine del processo, avremo creato la nostra immagine **my-was8-app**.

Questa immagine conterrà l'ambiente WebSphere Application Server 8.5.5.13 con la nostra applicazione **JSFSample.ear** correttamente deployata.

### Analisi della sintassi del comando
* **docker build**: il subcomando che indica di creare l'immagine
* **-t my-was8-app**: imposta il nome (e opzionalmente) il tag dell'immagine da creare nel formato *nome:tag*
* **-f Dockerfile8**: indica il Dockerfile da utilizzare nel caso non abbia il nome di default (*Dockerfile* appunto)
* **.(dot)**: la cartella di riferimento (in questo caso quella dove viene eseguito il comando

### Effetti del comando
Docker effettua la build dell'immagine assumendo come **docker context** la cartella dove è eseguito il comando.
L'immagine sarà taggata col nome **my-was8-app**.

## <a name="Task_3"></a>Creazione del container (versione 8)
Creiamo il container
```.term1
  docker run --name was8 -p9060:9060 -p9080:9080 -d my-was8-app
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare il container
* **--name was8**: imposta il nome da assegnare al container
* **-p9060:9060**: espone la porta 9060 del container sulla porta 9060 dell'host
* **-p9080:9080**: espone la porta p9080 del container sulla porta 9043 dell'host
* **-d**: esegue il container in background
* **my-was8-app**: il nome dell'immagine di partenza

### Effetti del comando
Docker crea il container a poartire dall'immagine **my-was8-app**.
Il nome assegnato al container sarà **was8** e saranno esposte le tre porte **9060** e **9080**.
Il container sarà eseguito in background.

Per verificare che il server sia partito correttamente, possiamo digitare il comando
```.term1
  docker logs -f was8
```

Quando il server avrà terminato la fase di *bootstrap* potremo premere **CTRL+C** per abortire il comando e ritornare al prompt.


## <a name="Task_4"></a>Verifica e test

Una volta che il server sarà inizializzato sarà possibile accedere alla:
*  **[console amministrativa](/ibm/console/){:data-term=".term1"}{:data-port="9060"}**
*  **[applicazione](/SampleAjax/index.faces){:data-term=".term1"}{:data-port="9080"}**

Per fermare e rimuover il container eseguiremo il comando
```.term1
  docker rm -f was8
```
