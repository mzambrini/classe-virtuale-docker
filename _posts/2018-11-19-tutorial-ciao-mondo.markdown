---
layout: post
title:  "Docker tutorial - Ciao mondo"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questo laboratorio, verranno mostrati i comandi basilari di Docker e un semplice workflow di tipo *build-ship-run*. 
Saranno utilizzate immagine già esistenti per creare dei containers e cominciare a muoversi tra i differenti comandi e sottocomandi messi a disposizione da docker.

> **Difficoltà**: Facile

> **Durata**: Circa quindici minuti

> **Tasks**:
>

> * [Task 1: Ciao mondo](#Task_1)
> * [Task 2: Ciao mondo (versione demonizzata)](#Task_2)
> * [Task 3: Visualizzazione dei logs](#Task_3)
> * [Task 4: Lista dei containers](#Task_4)
> * [Task 5: Esecuzione di comandi all'interno di un container](#Task_5)

## <a name="Task_1"></a>Ciao mondo

In questo task vedremo all'opera il comando *docker run* utilizzato per creare e, contestualmente, avviare un container.
La sintassi generica del sottocomando in questione è la seguente:

```.term1
    docker run [options] image [command] [args]
```

Solo il parametro *image* è obbligatorio ed è specificato nel formato *repository:tag*.

### Avvio del container

Eseguire il seguente comando

```.term1
    docker run --rm ubuntu:18.04 /bin/echo 'Hello world' 
```
Il risultato sarà simile al seguente:

```
Unable to find image 'ubuntu:18.04' locally
18.04: Pulling from library/ubuntu
473ede7ed136: Pull complete
c46b5fa4d940: Pull complete
93ae3df89c92: Pull complete
6b1eed27cade: Pull complete
Digest: sha256:29934af957c53004d7fb6340139880d23fb1952505a15d69a03af0d1418878cb
Status: Downloaded newer image for ubuntu:18.04
Hello world
```

### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare un nuovo container
* **--rm**: indica che una volta arrestato, il container sarà rimosso
* **ubuntu:18.04**: il nome dell’immagine da utilizzare
* **/bin/echo ‘Hello World’**: il comando eseguito all’interno del container

### Effetti del comando
Docker recupera l’immagine ubuntu:18.04 (localmente o, se non presente dall’Hub Docker) e avvia un nuovo container indicandogli, come comando, di stampare la scritta *‘Hello World’*. Fatto questo, ferma il container e, in virtù del flag *--rm*, lo rimuove.  
Notare che, nel caso dell’esecuzione in questione, l’immagine *ubuntu:18.04* non era presente localmente, è stata scaricata dall'Hub Docker.

## <a name="Task_2"></a>Ciao mondo (versione demonizzata)

In questo esempio utilizzeremo nuovamente l'immagine ubuntu ma con delle differenze.

### Avvio del container
Eseguiamo il seguente comando:

```.term1
    docker run -d --name test-ubuntu --rm ubuntu:18.04 /bin/sh \
    -c "while true; do echo hello world; sleep 1; done"

```
Il risultato sarà simile al seguente:

```
23c28e50a10220c048552747fa4a40eb813ac939e861549f2b9b1ebdbde1d6eb
```

La stringa riportata sul terminale è il *container ID* associato al container appena creato e avviato.
Rappresenta un identificativo univoco: non possono esistere due differenti containers aventi lo stesso identificativo.

### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare un nuovo container
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*
* **--name test-ubuntu**: il nome da assegnare al container (univoco)
* **--rm**: indica che una volta arrestato, il container sarà rimosso
* **ubuntu:18.04**: il nome dell’immagine da utilizzare
* **/bin/sh -c "while true; do echo hello world; sleep 1; done"**:

### Effetti del comando
Docker recupera l’immagine ubuntu:18.04 (in questo caso localmente perché l'abbiamo già utilizzata) e avvia un nuovo container.
Questa volta, il comando passatogli altro non è che un loop infinito all'interno del quale stampa la stringa *hello world* a intervalli di un secondo.

A questo container viene assegnato il nome **test-ubuntu**.
Il parametro **–d** indica a Docker di distaccarsi (detach) dal container e di eseguirlo in background; in questi casi viene visualizzato sul terminale il solo *container ID*.
Anche in questo caso, quando il container sarà arrestato, verrà contestualmente rimosso.

## <a name="Task_3"></a>Visualizzazione dei logs
### Ma l'output dove lo troviamo?
Nell'esempio [precedente](#Task_1), abbiamo istruito Docker ad avviare il container in background; per questo motivo, Non possiamo vedere l'output generato dal container.
Quando vogliamo ispezionare i log di un container, Docker mette a disposizione il sottocomando *logs*.

Eseguiamo il seguente comando:

```.term1
    docker logs --tail 10 -t test-ubuntu
```
Il risultato sarà simile al seguente:

```
2018-11-19T10:09:47.406419781Z hello world
2018-11-19T10:09:48.407423751Z hello world
2018-11-19T10:09:49.408492521Z hello world
2018-11-19T10:09:50.409587791Z hello world
2018-11-19T10:09:51.411378057Z hello world
2018-11-19T10:09:52.412325828Z hello world
2018-11-19T10:09:53.413842297Z hello world
2018-11-19T10:09:54.414346471Z hello world
2018-11-19T10:09:55.416053039Z hello world
2018-11-19T10:09:56.417147611Z hello world

```
### Analisi della sintassi del comando
* **docker logs**: il subcomando che indica di stampare i logs di un container
* **--tail 10**: indica a Docker di stampare le ultime 10 righe del log con *"all"* le stampa tutte)
* **-t**: stampa anche il timestamp
* **test-ubuntu**: indica il nome del container; in alternativa si può specificare il container ID o una sua sottostringa(*)

### Effetti del comando
Docker recupera il log per il container di nome *test-ubuntu* e stampa a video le ultime 10 righe corredate di timestamp


## <a name="Task_4"></a>Lista dei containers

Per visualizzare e avere informazioni sui containers presenti all'interno di un sistema Docker, è messo a disposizione il sottocomando *ps*.
Eseguiamo il seguente comando:

```.term1
    docker ps -a -s
```
Il risultato sarà simile al seguente:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES               SIZE
5feb635ab5d8        ubuntu:18.04        "/bin/sh -c 'while t…"   38 seconds ago      Up 37 seconds                           test-ubuntu         0B (virtual 85.8MB)
```

### Analisi della sintassi del comando
* **docker ps**: il subcomando che indica di visualizzare i containers
* **-a**: indica a Docker di visualizzare tutti containers, anche quelli non attivi
* **-s**: stampa la dimensione totale del file

### Effetti del comando
Docker stampa a video una serie di informazioni su tutti i containers presenti nel sistema. Tra queste informazioni è presente anche la dimensione del file.

## <a name="Task_5"></a>Esecuzione di comandi all'interno di un container

Può accadere che si voglia eseguire un comando all'interno di un container in esecuzione in modalità interattiva, come se si aprisse un terminale.
Con Docker è possibile utilizzando il sottocomando *exec*.
Facendo sempre riferimento all'esempio [Ciao mondo (versione demonizzata)](#Task_1), vogliamo aprire una sessione su questo container.

Eseguiamo il seguente comando:

```.term1
    docker exec -i -t test-ubuntu bash
```

Il risultato sarà simile al seguente:

```
root@b26ffa9af2b7:/#
```
Questo è un prompt dei comandi eseguito all'interno del container. Qui possiamo eseguire i comandi interpretabili dalla shell bash.
potremmo, ad esempio, eseguire il comando:

```.term2
    ls
```
che riporterà un output del tipo 

```
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

### Analisi della sintassi del comando
* **docker exec**: il subcomando che indica di eseguire un comando su un container
* **-i**: richiede di mantenere aperto lo STDIN del container
* **-t**: richiede di allocare una pseudo-TTY
* **test-ubuntu**: indica il nome del container; in alternativa si può specificare il container ID o una sua sottostringa(*)
* **bash**: il comando da eseguire

### Effetti del comando
Docker si è connesso al container avente nome **test-ubuntu**; ha allocato una TTY e mantiene aperto lo STDIN (standard input) per ricevere i comandi.
Esegue il comando **bash** che apre una shell interattiva.
Sarà possibile uscire da tale shell con il comando **exit**

