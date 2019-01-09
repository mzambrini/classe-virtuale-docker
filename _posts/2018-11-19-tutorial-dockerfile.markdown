---
layout: post
title:  "Docker tutorial - Usiamo i dockerfiles"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer]
categories: beginner
terms: 1
---

In questo laboratorio, si vedrà come creare proprie immagini docker; il protagonistra principale di questo task sarà il **Dockerfile** un file di testo contenente tutte le istruzioni per generare la nostra immagine.

Saranno affrontate le problematiche più comuni che si possano incontrare in questi task e saranno esposte alcune best practices.

> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio](#Task_1)
> * [Task 2: Build e analisi dell'immagine](#Task_2)
> * [Task 3: Creazione del container con binding su cartella dell'host](#Task_3)
> * [Task 4: Creazione del container utilizzando un volume](#Task_4)

## <a name="Task_1"></a>Cloning del progetto di esempio
Eseguiamo i comandi:
```.term1
   git clone https://github.com/mzambrini/tutorial-dockerfile.git
   cd tutorial-dockerfile
```
Con questo comando abbiamo clonato il progetto su GitHub sul nostro terminale virtuale e ci siamo posizionati all'interno della cartella di progetto.
Eseguendo il comando:

Eseguiamo i comandi:
```.term1
   ls -la
```
visualizzermo un elenco di files; quelli che ci interessano sono i seguenti:
* **Dockerfile**: il file testuale (obbligatorio) contenente le istruzioni per creare la nostra immagine personalizzata
* **.dockerignore**: file di testo opzionale strutturalmente simile a *.gitignore*. Serve a definire il *context build* di Docker
* **entrypoint.sh**: un file di shell pensato per la nostra specifica immagine

Eseguiamo il seguente comando per visualizzare il contenuto del **Dockerfile**
```.term1
   cat Dockerfile
```
e il seguente per visualizzare il file di script **entrypoint.sh**
```.term1
   cat entrypoint.sh
```
Questo, come detto, è un file di script molto semplice che fa tre cose:
*  Stampa sullo **STDOUT** una stringa contenente il nome dell'utente che esegue lo script
*  Stampa sullo **STDOUT** la stringa rappresentante i parametri passati allo script
*  Stampa in append sul file **/result/output.txt** la stringa del passo precedente

## <a name="Task_2"></a>Build e analisi dell'immagine

Eseguiamo ora il comando:
```.term1
   docker build --build-arg USER=pincopalla --build-arg GROUP=gruppo -t mondo .
```
alla fine, avremo creato la nostra immagine **mondo**.

### Analisi della sintassi del comando
* **docker build**: il subcomando che indica di creare l'immagine
* **--build-arg USER=pincopalla**: indica di impostare il valore dell'argomento di build **USER** con il valore **pincopalla**
* **--build-arg GROUP=gruppo**: indica di impostare il valore dell'argomento di build **GROUP** con il valore **gruppo**
* **-t mondo**: imposta il nome (e opzionalmente) il tag dell'immagine da creare nel formato *nome:tag*
* **.(dot)**: la cartella di riferimento (in questo caso quella dove viene eseguito il comando

### Effetti del comando
Docker effettua la build dell'immagine assumendo come **docker context** la cartella dove è eseguito il comando.
L'immagine sarà taggata col nome **mondo**.

### Analisi dell'immagine
Per evidenziare la struttura a livelli di un'immagine docker eseguiamo il seguente comando:

```.term1
   docker history mondo
```
E' possibile notare non solo i **LAYERS** creati dalla nostra build, ma anche quelli presenti nell'immagine di partenza (clausola **FROM**)

### Un tool interessante per l'analisi delle immagini

Tra i tanti tools esistenti quale supporto a docker, è stato rilasciato in tempi recenti il tool **[dive](https://github.com/wagoodman/dive)**.
E' un'immagine docker che ci permette di analizzare in dettaglio la composizione di un'immagine docker.

Eseguiamo il seguente comando:
```.term1
   docker run --rm -it \
   -v /var/run/docker.sock:/var/run/docker.sock \
   wagoodman/dive:latest mondo
```
## <a name="Task_3"></a>Creazione del container con binding su cartella dell'host
Creiamo ora una cartella in locale e diamo i diritti in scrittura a tutti.
```.term1
   mkdir result
   chmod a+w result
```
Creiamo ora il container
```.term1
   docker run --rm -v ${PWD}/result:/result mondo 'Prima invocazione'
```
il risultato sarà il seguente:
```
Questo comando è stato eseguito dall'utente pincopalla
Prima invocazione
```
La prima riga riporta l'utente *unix* che ha eseguito il comando ed è lo **stesso utente** che abbiamo passato in fase di build dell'immagine.

La seconda riga riporta la stringa passata come argomento al container

Verifichiamo quello che ha scritto sul file locale **result/output.txt**:

```.term1
   cat result/output.txt
```
il contenuto sarà:
```
Prima invocazione
```
eseguiamo un nuovo comando docker:

```.term1
   docker run --rm -v ${PWD}/result:/result mondo 'Seconda invocazione'
```
eseguiamo

```.term1
   cat result/output.txt
```
il contenuto sarà:
```
Prima invocazione
Seconda invocazione
```
come previsto il container a scritto con modalità *append* sul file

## <a name="Task_4"></a>Creazione del container utilizzando un volume
Adesso utilizzeremo un volume creato esplicitamente. 
Ricordiamo che volume è una cartella del container cui si affida la persistenza dei dati indipendentemente dal ciclo di vita del container stesso.
I volumi sono gestiti automaticamente da docker stesso.

Creiamo il volume:
```.term1
   docker volume create result_data
```
e andiamo ad ispezionarne il contenuto
```.term1
   docker volume inspect result_data
```
avremo un risultato del tipo:
```
[
    {
        "CreatedAt": "2018-11-21T09:42:15Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/result_data/_data",
        "Name": "result_data",
        "Options": {},
        "Scope": "local"
    }
]
```
Il nostro volume è stato creato all'interno del folder **/var/lib/docker/volumes/result_data/_data**.

Ora creiamo il container:
```.term1
   docker run --rm -v result_data:/result mondo 'Prima scrittura sul volume'
```
e andiamo a verificare il contenuto
```.term1
   cat /var/lib/docker/volumes/result_data/_data/output.txt
```
il contenuto del file sarà

```
Prima scrittura sul volume
```
Notiamo che il container è stato rimosso (flag --**rm**) ma il volume, correttamente, è persistente.

Creiamo un nuovo container mappando lo stesso volume:
```.term1
   docker run --rm -v result_data:/result mondo 'Seconda scrittura sul volume'
```
verifichiamo il contenuto
```.term1
   cat /var/lib/docker/volumes/result_data/_data/output.txt
```
il contenuto del file sarà:
```
Prima scrittura sul volume
Seconda scrittura sul volume
```
### Note sui volumi
I volumi possono essere condivisi tra più container. Un pattern comune è quello di delegare un primo container per inizializzare i dati di un volume con un secondo container che li utilizzi.
