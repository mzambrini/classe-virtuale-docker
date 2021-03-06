---
layout: post
title:  "Maven tutorial - Usiamo docker per buildare le nostre applicazioni java"
date:   2018-12-07
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questo laboratorio si vedrà come effettuare le build maven dei nostri progetti java utilizzando un'opportuna immagine docker senza dovere installre maven.

> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio](#Task_1)
> * [Task 2: Build dei sorgenti](#Task_2)
> * [Task 3: Caching del repository locale](#Task_3)
> * [Task 4: Considerazioni finali](#Task_4)

## <a name="Task_1"></a>Cloning del progetto di esempio
Eseguiamo i comandi:
```.term1
   git clone https://github.com/mzambrini/tutorial-maven.git
   cd tutorial-maven
```
Con questi comandi abbiamo clonato il progetto GitHub sul nostro terminale virtuale e ci siamo posizionati all'interno della cartella di progetto.

La struttura del progetto è la seguente:

```
│   .gitignore
│   global-variables.env
│   pom.xml
│   README.md
│
├───.m2
│       settings-default.xml
│       settings-proxy.xml
│
├───src
│   ├───main
│   │   └───java
│   │       └───it
│   │           ├───brainmaxz
│   │           │   └───shazam
│   │           │       │   CodiciErrore.java
│   │           │       │   GestoreBundleMessaggi.java
│   │           │       │   GestoreMessaggiErrore.java
│   │           │       │   GestoreProperties.java
│   │           │       │   SystemProperties.java
│   │           │       │
│   │           │       ├───eccezione
│   │           │       │       ShazamEjbException.java
│   │           │       │       ShazamException.java
│   │           │       │
│   │           │       └───properties
│   │           │               GestoreBundleMessaggiImpl.java
│   │           │               GestoreMessaggiErroreImpl.java
│   │           │               PropertiesInjector.java
│   │           │
│   │           └───prova
│   │                   Prova.java
│   │
│   └───test
│       ├───java
│       │   └───it
│       │       └───aci
│       │           └───informatica
│       │               └───shazam
│       │                   └───properties
│       │                           TestGestoreBundleMessaggiImpl.java
│       │
│       └───resources
│               pref.properties
│               primo.properties
│               secondo.properties
│               terzo.properties
```

I file rilevanti sono i seguenti:
* **pom.xml**: il file utilizzato da maven per reperire tutte le informazioni necessarie alla build
* **global-variables.env**: il file utilizzato per passare al container di maven le variaibli di ambiente necessarie per accedere a internet
* **.m2/settings-default.xml**: il file xml di default per la configurazione di maven (nel nostro esempio non contiene nulla di sostanziale)
* **.m2/settings-proxy.xml**: il file xml per la configurazione di maven con le informazioni del proxy
* **src/main/java**: la cartella contenente i sorgenti java
* **src/test/java**: la cartella contenente i sorgenti java dei test unitari
* **src/test/resources**: la cartella contenente i file di properties necessari per i test unitari


Eseguiamo il seguente comando per visualizzare il contenuto del **pom.xml**
```.term1
   cat pom.xml
```

## <a name="Task_2"></a>Build dei sorgenti
Dall'analisi del file **pom.xml** si può desumere che dovrà essere utilizzata la versione **7** della *Java Virtual Machine*.
Questa è l'informazione che utilizzeremo per scegliere l'immagine maven più adatta alle nostre esigenze.

Se ci troviamo dietro un proxy aziendale, è necessario utilizzare un file **settings.xml** contenente la configurazione del proxy.
Conviene impostare una variabile di ambiente:
```.term1
   M2_SETTINGS=${PWD}/.m2/settings-proxy.xml
```
nel caso ciò non sia vero (non serve il proxy) possiamo usare il comando:
```.term1
   M2_SETTINGS=${PWD}/.m2/settings-default.xml
```
Eseguiamo ora il comando:
```.term1
   docker run --rm -v ${M2_SETTINGS}:/root/.m2/settings.xml -v ${PWD}:/usr/src/mymaven -w /usr/src/mymaven  maven:3-jdk-7-slim mvn clean install
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare ed avviare il container
* **--rm**: indica a Docker di rimuovere il container una volta terminata l'elaborazione
* **-v ${M2_SETTINGS}:/root/.m2/settings.xml**: mappa il file puntato dalla variabile *M2_SETTINGS* dell'host al file di settings del container
* **-v ${PWD}:/usr/src/mymaven**: mappa la cartella corrente dell'host alla cartella */usr/src/mymaven* del container
* **-w /usr/src/mymaven**: indica quale cartella di lavoro la cartella del container */usr/src/mymaven*
* **maven:3-jdk-7-slim**: l'immagine utilizzata per creare il container
* **mvn clean install**: il comando maven completo

### Effetti del comando
Docker recupera l’immagine *maven:3-jdk-7-slim* dall'Hub Docker e avvia un nuovo container.

La cartella corrente dell'host viene mappata alla cartella */usr/src/mymaven* del container; detta cartella è configurata anche come *working dir* (flag *-w*).

Viene utilizzato il file di settings puntato dalla variabile *M2_SETTINGS*

All'interno del container viene eseguito il comando *mvn clean install*.

Al termine dell'elaborazione, il container viene rimosso (flag *--rm*).

Il risultato dell'elaborazione sarà presente nella cartella *target* come è possibile vedere eseguendo il comando
```.term1
   ls target
```

### Commenti
In questo caso, Docker ha utilizzato un suo repository locale che è stato popolato all'interno del container.
Al termine dell'elaborazione, il container è stato distrutto insieme al repository locale.

Questa è la situazione più *pulita* possibile. Se una build non funziona, sicuramente non dipende da un repository locale corrotto (succede raramente, ma può succedere).
Tuttavia non è la soluzione più veloce. Ogni volta che effettuiamo una build bisogna riscaricare tutti i plugin utilizzati da maven.

## <a name="Task_3"></a>Caching del repository locale
E' possibile effetuare il caching del repository locale mappando semplicemente un volume alla cartella **/root/.m2** del container.
Nel nostro caso non andremo a creare un volume con il comando **docker volume create** ma utilizzeremo la cartella locale **.m2**.
Stavolta, però, dovremo settare esplicitamente il file di settings visto che mappiamo tutta la cartella che lo contiene.


Se ci troviamo dietro un proxy aziendale eseguiremo il comando:

```.term1
   cp ${PWD}/.m2/settings-proxy.xml ${PWD}/.m2/settings.xml
```
altrimenti:
```.term1
   cp ${PWD}/.m2/settings-default.xml ${PWD}/.m2/settings.xml
```

Il nuovo comando sarà
```.term1
  docker run --rm -v ${PWD}/.m2:/root/.m2 -v ${PWD}:/usr/src/mymaven -w /usr/src/mymaven  maven:3-jdk-7-slim mvn clean install
```
### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare ed avviare il container
* **--rm**: indica a Docker di rimuovere il container una volta terminata l'elaborazione
* **-v ${PWD}/.m2:/root/.m2**: mappa la sottocartella **.m2** della cartella corrente dell'host alla cartella */root/.m2* del container
* **-v ${PWD}:/usr/src/mymaven**: mappa la cartella corrente dell'host alla cartella */usr/src/mymaven* del container
* **-w /usr/src/mymaven**: indica quale cartella di lavoro la cartella del container */usr/src/mymaven*
* **maven:3-jdk-7-slim**: l'immagine utilizzata per creare il container
* **mvn clean install**: il comando maven completo

### Effetti del comando
Similmente al caso precedente, Docker recupera l’immagine *maven:3-jdk-7-slim* dall'Hub Docker e avvia un nuovo container.

La cartella corrente dell'host viene mappata alla cartella */usr/src/mymaven* del container; detta cartella è configurata anche come *working dir* (flag *-w*).

Inoltre, viene mappata la sottocartella *.m2* alla cartella del container */root/.m2*.

All'interno del container viene eseguito il comando *mvn clean install*.

Al termine dell'elaborazione, il container viene rimosso (flag *--rm*).

Il risultato dell'elaborazione sarà presente nella cartella *target*


### Commenti
Andando a curiosare nella cartella dell'host **.m2**, si potrà verificare la presenza del repository locale.

Il risultato dell'elaborazione sarà presente nella cartella *target* come è possibile vedere eseguendo il comando
```.term1
   ls .m2
```
Rieseguiamo ora lo stesso comando:
```.term1
  docker run --rm -v ${PWD}/.m2:/root/.m2 -v ${PWD}:/usr/src/mymaven -w /usr/src/mymaven  maven:3-jdk-7-slim mvn clean install
```

Si potrà notare facilmente una maggior velocità di esecuzione in quanto i plugin di maven e le dipendenze del progetto non dovranno esere scaricati nuovamente.

## <a name="Task_5"></a>Considerazioni finali
L'esempio che abbiamo visto è estremamente basico, ma abbiamo potuto capire come sia possibile utilizzare maven quale strumento di build senzala necessità di doverlo installare.

Inoltre, questo approccio è molto utile, in quanto **stateless**, ovvero una build può fallire (o, peggio ancora, non fallire quando dovrebbe!) perché il repository locale è corrotto o perché facciamo riferimento ad una dipendenza che non dovrebbe esistere ma che ritroviamo nel repository locale (succede spesso con le dipendenze snapshot).
Insomma, i motivi possono essere molteplici.

L'approccio stateless permette di escludere facilmente e velocemente questa tipologia di problematiche.
