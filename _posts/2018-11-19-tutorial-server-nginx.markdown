---
layout: post
title:  "Docker tutorial - Configurazione di un server nginx"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---


In questo laboratorio, utilizzeremo l'immagine [Nginx](https://www.nginx.com/) per avviare un container che esponga il nostro contenuto HTML.

Nel nostro caso avremo una semplice paginetta con scritto (indovinate un po'?) 'Ciao mondo'.

Verranno  affrontate alcune modalità di mapping delle porte e di binding dei volumi.

> **Difficoltà**: Facile

> **Durata**: Circa quindici minuti

> **Tasks**:
>

> * [Task 1: Preparazione del file html](#Task_1)
> * [Task 2: Creazione e avvio del container nginx](#Task_2)
> * [Task 3: Fermo e rimozione del container](#Task_3)
> * [Task 4: Mapping randomico delle porte](#Task_4)
> * [Task 5: Utilizzo dei volumi](#Task_5)

## <a name="Task_1"></a>Preparazione del file html

Questo è un task di prerequisito; eseguiamo il comando:

```.term1
    mkdir data 
```
Questo comando crea la cartella *data* all'interno della cartella corrente.

Poi eseguiamo il seguente:

```.term1
cat << EOF > data/index.html
<!DOCTYPE html>
<html>
    <body>
        <h1>Ciao mondo</h1>
    </body>
</html>
EOF
```

Con questo comando andiamo a creare il file *index.html* all'interno della cartella *data*; questo file contenente semplici istruzioni HTML.

Per verificare il contenuto di detto file possiamo eseguire

```.term1
cat  data/index.html
```

## <a name="Task_2"></a>Creazione e avvio del container nginx

Passiamo ora ad utilizzare il comando run per creare e avviare il nostro container nginx.
Eseguiamo il seguente comando:

```.term1
   docker run --name nginx-server \
      -v ${PWD}/data:/usr/share/nginx/html:ro \
       -p 8080:80 -d nginx:1.14
```

### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare un nuovo container
* **--name nginx-server**: il nome da assegnare al container (univoco)
* **-v ${PWD}/data:/usr/share/nginx/html:ro**: indica di mappare la cartella dell'host *${PWD}/data* alla cartella del container */usr/share/nginx/html* con attributi di sola lettura *ro*
* **-p 8080:80**: indica a Docker di mappare la porta dell'host *8080* alla porta del container *80*
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*
* **nginx:1.14**: il nome dell’immagine da utilizzare

### Effetti del comando
Docker recupera l’immagine **nginx:1.14** (localmente o, se non presente dall’Hub Docker) e avvia un nuovo container assegnadogli il nome **nginx-server**.
Indica anche il mapping di una cartella dell'host in una del container e il mapping della porta *8080* dell'host sulla porta *80* del container.
L'esecuzione del container è in background, a schermo viene stampato il solo *container ID*
 
### Verifica del corretto funzionamento
Verifichiamo che il servizio del container sia accessibile dalla porta 8080 dell'host e che riporti il corretto cliccando su:
* **[Link al container nginx](/){:data-term=".term1"}{:data-port="8080"}**

## <a name="Task_3"></a>Fermo e rimozione del container
Passiamo ora ad analizzare il comando per fermare un container:

```.term1
   docker stop -t 20  nginx-server 
```

### Analisi della sintassi del comando
* **docker stop**: il subcomando che indica di fermare un container in esecuzione
* **-t 20**: il tempo di attesa (in secondi) prima di killare il container (default 10 secondi)
* **nginx:1.14**: indica il nome del container da stoppare; in alternativa si può specificare il container ID o una sua sottostringa(*)

### Effetti del comando
Docker invia il segnale di terminazione al container di nome *nginx-server*. Attende 20 secondi e, nel caso il processo non sia terminato, lo killa.

Il passo successivo è quello di rimuovere definitivamente il container. il comando da eseguire è:

```.term1
   docker rm nginx-server 
```
### Analisi della sintassi del comando
* **docker rm**: il subcomando che indica di rimuovere un container
* **nginx:1.14**: indica il nome del container da rimuovere; in alternativa si può specificare il container ID o una sua sottostringa(*)

### Effetti del comando
Docker invia il segnale di terminazione al container di nome *nginx-server*. Attende 20 secondi e, nel caso il processo non sia terminato, lo killa.


## <a name="Task_4"></a>Mapping randomico delle porte
Proviamo ora a ricreare un'istanza di nginx in maniera leggermente differente:

```.term1
   docker run --name nginx-server \
      -v ${PWD}/data:/usr/share/nginx/html:ro \
       -P -d nginx:1.14
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare un nuovo container
* **--name nginx-server**: il nome da assegnare al container (univoco)
* **-v ${PWD}/data:/usr/share/nginx/html:ro**: indica di mappare la cartella dell'host *${PWD}/data* alla cartella del container */usr/share/nginx/html* con attributi di sola lettura *ro*
* **-P**: indica a Docker di mappare tutte le porte esposte dal container a porte randomiche dell'host
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*
* **nginx:1.14**: il nome dell’immagine da utilizzare

### Effetti del comando
Docker recupera l’immagine **nginx:1.14** e avvia un nuovo container assegnadogli il nome **nginx-server**.
Mappa il volume la cartella dell'host *${PWD}/data* sulla cartella del container indicata.
A differenza del caso precedente, mappa **tutte** le porte **esposte** dal container su porte randomiche dell'host.

Possiamo fare una verifica delle porte esposte eseguendo il comando

```.term1
   docker port nginx-server 
```
A questo punto eliminiamo il container eseguendo il comando:

```.term1
   docker rm -f nginx-server 
```
Differentemente dal caso precedente, stiamo tentando di rimuovere un container in esecuzione.
Ciò non è normalmente possibile, a meno che, non si forzi l'operazione con il flag **-f** come nel nostro caso

## <a name="Task_5"></a>Utilizzo dei volumi

Creiamo ora un terzo container dichiarando esplicitamente un volume.

```.term1
   docker run --name nginx-server \
      -v data_volume:/usr/share/nginx/html \
       -p 8080:80 -d nginx:1.14
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare e avviare un nuovo container
* **--name nginx-server**: il nome da assegnare al container (univoco)
* **-v data_volume:/usr/share/nginx/html**: indica di mappare il volume *data_volume* alla cartella del container */usr/share/nginx/html*
* **-p 8080:80**: indica a Docker di mappare la porta dell'host *8080* alla porta del container *80*
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*
* **nginx:1.14**: il nome dell’immagine da utilizzare

### Effetti del comando
Docker recupera l’immagine **nginx:1.14** e avvia un nuovo container assegnadogli il nome **nginx-server**.
Mappa il volume *data_volume* sulla cartella del container indicata; se il volume non esiste, lo crea contestualmente alla creazione del container.
Mappa la porta *8080* dell'host sulla porta *80* del container ed esegue il container in background.

Verifichiamo ora l'accessibilità del container:
* **[Link al container nginx](/){:data-term=".term1"}{:data-port="8080"}**

Come possiamo vedere, non troviamo la nostra pagina (e questo ce lo aspettavamo), ma una pagina di default di nginx.

Se eseguiamo il comando:

```.term1
    docker volume ls
```
avremo un risultato simile a questo

```
DRIVER              VOLUME NAME
local               data_volume
```

Un volume è una struttura dati gestita direttamente da Docker; a differenza del mapping diretto su cartelle dell'host, garantisce una maggiore portabilità del container.
Possiamo anche usare l'istruzione **inspect** per visualizzarne le caratteristiche.
Usiamo il seguente comando:

```.term1
    docker volume inspect data_volume
```
avremo un risultato simile a questo

```
[
    {
        "CreatedAt": "2018-11-20T09:35:27Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/data_volume/_data",
        "Name": "data_volume",
        "Options": null,
        "Scope": "local"
    }
]
```
Se ora invochiamo il seguente comando:

```.term1
    ls /var/lib/docker/volumes/data_volume/_data
```
avremo il seguente risultato
```
50x.html    index.html
```

Ciò indica che, quando utilizziamo i volumi, Docker copia all'interno del volume i files/folders eventualmente specificati nel **Dockerfile** dell'immagine relativa nella specifica cartella del container mappata al volume.

Nel nostro caso, avendo mappato la cartella del container **/usr/share/nginx/html**, il contenuto originale di tale cartella è stato copiato (all'avvio del container) nel volume **data_volume**.
Se avessimo mappato, come nel primo caso di questo tutorial, un folder locale all'host, il contenuto della cartella **/usr/share/nginx/html** sarebbe stato sostituito con il contenuto del folder sull'host.


