---
layout: post
title:  "dotNet tutorial - Usiamo docker per buildare le nostre applicazioni dotNet"
date:   2018-12-07
author: "@mzambrini"
tags: [beginner, linux, operations, developer, dotNet]
categories: beginner
terms: 1
---

In questo laboratorio si vedrà come effettuare la build di progetti dotNet utilizzando le immagini docker messe a disposizione da Microsoft.

> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Cloning del progetto di esempio](#Task_1)
> * [Task 2: Analisi del Dockerfile](#Task_2)
> * [Task 3: Creazione ed esecuzione del container](#Task_3)
> * [Task 4: Considerazioni finali](#Task_4)

## <a name="Task_1"></a>Cloning del progetto di esempio
Eseguiamo i comandi:
```.term1
   git clone https://github.com/mzambrini/dotnet-docker.git
   cd dotnet-docker
```

Con questi comandi abbiamo clonato il progetto GitHub sul nostro terminale virtuale e ci siamo posizionati all'interno della cartella di progetto.
Il progetto in questione è la fork del progetto originale che si trova all'indirizzo https://github.com/dotnet/dotnet-docker.
Contiene tutti i **Dockerfile** utili per generare le differenti immagini docker.

Include anche una cartella **samples** contenente due applicazioni: *aspnetapp* e *dotnetapp*.

Noi ci interesseremo della prima, una semplice applicazione web di esempio.
Eseguiamo il comando seguente per portarci nella cartella del progetto di interesse
```.term1
   cd samples/aspnetapp
```
## <a name="Task_2"></a>Analisi del Dockerfile
 Visualizziamo il contenuto del Dockerfile:
```.term1
   cat Dockerfile
```
Il contenuto sarà simile al seguente:

```dockerfile
FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out


FROM microsoft/dotnet:2.2-aspnetcore-runtime AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

Come si può notare, viene adottato l'approccio **multi-stage** (più clausole **FROM** in un unico Dockerfile).

La prima clausola *FROM* utilizza l'immagine dell'SDK: sfruttando i file di tipo **.sln** e **.csproj** si provvede a ripristinare le dipendenze e gli strumenti del progetto utilizzando il comando **dotnet restore**.

A seguire si provvede a copiare i sorgenti dell'applicazione e a compilarla e distribuirla con il comando **dotnet publish -c Release -o out**.

Nella seconda clausola **FROM** si parte dall'immagine di runtime delle applicazioni **ASP**; il compilato della build viene copiato nella cartella **/app** del container.
Viene anche impostato il corretto entrypoint.


## <a name="Task_2"></a>Build dell'immagine docker
Provvediamo ora ad eseguire la build dell'immagine con il comando:

```.term1
   docker build -t aspnetapp .
```

### Analisi della sintassi del comando
* **docker build**: il subcomando che indica di costruire l'immagine a partire dal Dockerfile
* **-t aspnetapp**: indica il nome assegnato all'immagine
* **. (dot)**: indica la context-root della build, ovvero la cartella corrente

### Effetti del comando
Docker effettua la build dell'immagine a partire dal Dockerfile. Notare come, nella cartella di progetto, esistano più file di tipo Dockerfile. non avendo specificato il nome del Dockerfile, viene utilizzato il default **Dockerfile** appunto.

## <a name="Task_3"></a>Creazione ed esecuzione del container
A questo punto creiamo un container a partire dall'immagine appena buildata:
```.term1
   docker run --name asp-sample --rm -p 8000:80 -d aspnetapp
```

### Analisi della sintassi del comando
* **docker run**: il subcomando che indica di creare ed avviare il container
* **--name asp-sample**: indica il nome del containe rche andiamo a creare
* **--rm**: indica a Docker di rimuovere il container una volta terminata l'elaborazione
* **-p 8000:80**: mappa la porta **80** del container sulla porta **8000** dell'host
* **-d**: indica a Docker di eseguire il container in background e di stampare a schermo il *container ID*
* **aspnetapp**: l'immagine utilizzata per creare il container, quella creata poco sopra

### Effetti del comando
Docker utilizza l’immagine *aspnetapp* e avvia un nuovo container.

La porta *8000* dell'host viene mappata sulla *80* del container.

Al termine dell'elaborazione, il container viene rimosso (flag *--rm*).

una volta avviato il container, il client docker si distacca da questo eseguendolo in background (flag *--d*).

Con il comando seguente verifichiamo che il server sia stato correttamente avviato
```.term1
   docker logs asp-sample
```

Verifichiamo ora l'accessibilità del container:
* **[Link al container nginx](/){:data-term=".term1"}{:data-port="8000"}**

## <a name="Task_4"></a>Considerazioni finali
Abbiamo visto come sia possibile utilizzare docker per compilare, testare e avviare applicazione sviluppate con la piattaforma dotNet.
In questo caso abbiamo eseguito l'esempio relativo ad un'applicazione web, ma, all'interno di questo stesso progetto, è disponibile anche l'applicazione di tipo console.

Microsoft ha messo a disposizione tutta una serie di immagini docker di tipo linux e microsoft per facilitare l'adozione della metodica DevOps anche per progetti di tipo dotNet.

Nel caso specifico di GitLab, sarà possibile definire le differenti pipeline potendo utilizzare dei runner di tipo docker e non essere costretti (al contrario) a preparare batterie di macchine configurate con le diverse versioni dell'SDK di dotNet in un contesto di runner di tipo shell.