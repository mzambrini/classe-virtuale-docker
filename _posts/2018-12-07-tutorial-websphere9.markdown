---
layout: post
title:  "WebSphere tutorial - Usiamo docker per le nostre applicazioni WebSphere (parte 2)"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questa seconda parte del laboratorio, si vedrà come passare dei **build args** durante la fase di costruzione dell'immagine.
Sfruttando, infatti, il costrutto **ARG** definito all'interno del Dockerfile, possiamo decidere di installare la nostra applicazione su una versione differente di WebSphere per verificare, ad esempio, la compatibilità.


> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio](#Task_1)
> * [Task 2: Build dell'immagine con WebSphere 9.0.0.8](#Task_2)
> * [Task 3: Creazione del container (versione 9)](#Task_3)
> * [Task 4: Verifica e test](#Task_4)
> * [Task 5: Considerazioni finali](#Task_5)

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
   cat Dockerfile9
```
e il seguente per visualizzare il file di script **apply-configuration.sh**
```.term1
   cat scripts/apply-configuration.sh
```

## <a name="Task_2"></a>Build dell'immagine con WebSphere 9.0.0.8

Eseguiamo ora il comando:
```.term1
   docker build -t my-was9-app -f Dockerfile9 .
```
alla fine del processo, avremo creato la nostra immagine **my-was9-app**.

Questa immagine conterrà l'ambiente WebSphere Application Server 9.0.0.8 con la nostra applicazione **JSFSample.ear** correttamente deployata.

### Analisi della sintassi del comando
* **docker build**: il subcomando che indica di creare l'immagine
* **-t my-was9-app**: imposta il nome (e opzionalmente) il tag dell'immagine da creare nel formato *nome:tag*
* **-f Dockerfile9**: indica il Dockerfile da utilizzare nel caso non abbia il nome di default (*Dockerfile* appunto)
* **.(dot)**: la cartella di riferimento (in questo caso quella dove viene eseguito il comando

### Effetti del comando
Docker effettua la build dell'immagine assumendo come **docker context** la cartella dove è eseguito il comando.
La build sarà effettuata seguendo le istruzioni presenti nel file **Dockerfile9**.
L'immagine sarà taggata col nome **my-was9-app**.

## <a name="Task_3"></a>Creazione del container (versione 9)
Creiamo il container
```.term1
  docker run --name was9 -p9060:9060 -p9080:9080 -d my-was9-app
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare il container
* **--name was9**: imposta il nome da assegnare al container
* **-p9060:9060**: espone la porta 9060 del container sulla porta 9060 dell'host
* **-p9080:9080**: espone la porta p9080 del container sulla porta 9043 dell'host
* **-d**: esegue il container in background
* **my-was9-app**: il nome dell'immagine di partenza

### Effetti del comando
Docker crea il container a poartire dall'immagine **my-was9-app**.
Il nome assegnato al container sarà **was9** e saranno esposte le tre porte **9060** e **9080**.
Il container sarà eseguito in background.

Per verificare che il server sia partito correttamente, possiamo digitare il comando
```.term1
  docker logs -f was9
```

Quando il server avrà terminato la fase di *bootstrap* potremo premere **CTRL+C** per abortire il comando e ritornare al prompt.

## <a name="Task_4"></a>Verifica e test

Una volta che il server sarà inizializzato sarà possibile accedere alla:
*  **[console amministrativa](/ibm/console/){:data-term=".term1"}{:data-port="9060"}**
*  **[applicazione](/SampleAjax/index.faces){:data-term=".term1"}{:data-port="9080"}**

Per fermare e rimuover il container eseguiremo il comando
```.term1
  docker rm -f was9
```
## <a name="Task_5"></a>Considerazioni finali
Nel nostro esempio, abbiamo utilizzato due distinti Dockerfiles in quanto il funzionamento delle due immagini di base di WebSphere è differente.

Oltre a cambiare il nome dell'immagine di base, infatti, cambia anche il comando **CMD** da invocare.

Per casistiche più semplici, laddove cambi *solamente* il nome dell'immagine di base, è possibile utilizzare il comando **ARG** che va a specificare una variabile di build.

Ad esempio nel seguente estratto avremmo


```Dockerfile
ARG IMAGE=node:10
FROM ${IMAGE}

...
...
...
```

Se invochiamo la build senza specificare il flag **--build-arg**, verrà utilizzata l'immagine base node:10


