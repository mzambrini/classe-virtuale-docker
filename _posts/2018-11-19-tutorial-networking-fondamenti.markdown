---
layout: post
title:  "Docker tutorial - Fondamenti di networking"
date:   2018-11-19
author: "@mzambrini"
tags: [beginner, linux, operations, developer, networking]
categories: beginner
terms: 1
---

In questo laboratorio, saranno affrontate le tematiche di networking per Docker.

Creeeremo un paio di containers e vedremo come farli interagire tra di loro in base alle configurazioni di rete applicabili.

> **Difficoltà**: Facile

> **Durata**: Circa dieci minuti

> **Tasks**:
>

> * [Task 1: Il networking su Docker](#Task_1)
> * [Task 2: Creazione di due container](#Task_2)
> * [Task 3: Ping!](#Task_3)
> * [Task 4: Utilizzo del flag --link](#Task_4)
> * [Task 5: Creazione del bridge](#Task_5)

## <a name="Task_1"></a>Il networking su Docker

Docker fornisce tre differenti driver di rete di base:
*	**none**: Il container è isolato dal mondo esterno e dispone della sola interfaccia di loopback
*	**host**: Per i containers di tipo standalone, rimuove l’isolamento di rete tra container e docker host usando la rete dell’host direttamente
*	**bridge**: Il driver di rete predefinito. Se non si specifica un driver, questo è il tipo di rete che si sta creando. Le reti bridge vengono solitamente utilizzate quando le applicazioni vengono eseguite in contenitori autonomi che devono comunicare tra di loro. Non esiste un sistema DNS di risoluzione dei nomi. I containers possono comunicare tra di loro solo via indirizzo IP

Docker mette a disposizione la possibilità di creare reti bridged *user-defined*. 

Queste tipologie di rete hanno una serie di vantaggi rispetto a quella di default messa a disposizione da docker:
*	I bridges *user-defined* forniscono un miglior isolamento e interoperabilità tra i vari containers
*	I bridges *user-defined* forniscono un servizio automatizzato di DNS per la risoluzione dei nomi
*	I containers possono essere agganciati e sganciati dalla reti *user-defined*  al volo (senza fermare/riavviare il container)
*	Ogni rete *user-defined* permette di creare un bridge configurabile, cosa che non è possibile fare con il bridge di default
*	I containers linkati sul bridge di default condividono le variabili di ambiente(*)

## <a name="Task_2"></a>Creazione di due container

Passiamo ora ad utilizzare il comando run per creare e avviare il nostro container nginx.
Eseguiamo i seguenti comando:

```.term1
   docker run --name servizio-uno -d alpine:3.8 \
   /bin/sh -c "while true; do echo servizio-uno; sleep 20; done"
```

```.term1
   docker run --name servizio-due -d alpine:3.8 \
   /bin/sh -c "while true; do echo servizio-due; sleep 20; done"
```

### Effetti dei comandi
Abbiamo creato due containers a partire dalla stessa immagine. ciascuno di questi due container è impegnato in un loop infinito che stampa a intervalli di 10 secondi il nome del container stesso.

### Configurazioni di rete
Verifichiamo, per il primo container, le configurazioni di rete:

```.term1
   docker inspect servizio-uno
```
Questo comando ci fornisce, in formato **json** tutta una serie di informazioni sul container in esecuzione.
Nel nostro caso, sul finire del file json, ci sarà una sezione simile alla seguente:

```
 "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "a67dc4a400ddd2cc8876073c8aa6c7bc5eecfefcaa3be5f2a77a1b98699ebef7",
                    "EndpointID": "59ef9e44f74c5be83ef3c6c2c8717f7fa72fd2d9894185dac844dccd6bb2411d",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
```

Possiamo vedere che il container è stato assegnato al bridge di default e che il suo indirizzo IP è 172.17.0.2 (durante l'esercitazione questo valore potrebbe differire).

## <a name="Task_3"></a>Ping!
Proviamo ora ad eseguire un semplice comando ping:

```.term1
   docker exec servizio-due /bin/sh -c 'ping -c 4 www.google.com'
```

Il risultato sarà simile al seguente.

```
PING www.google.com (216.58.218.228): 56 data bytes
64 bytes from 216.58.218.228: seq=0 ttl=49 time=2.030 ms
64 bytes from 216.58.218.228: seq=1 ttl=49 time=2.687 ms
64 bytes from 216.58.218.228: seq=2 ttl=49 time=2.084 ms
64 bytes from 216.58.218.228: seq=3 ttl=49 time=1.967 ms

--- www.google.com ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 1.967/2.192/2.687 ms
```
Abbiamo, quindi, verificato come Google risponda correttamente al nostro comando ping.

Proviamo ora a pingare l'altro container:

```.term1
   docker exec servizio-due /bin/sh -c 'ping -c 4 servizio-uno'
```
Il risultato sarà solamente 

```
ping: bad address 'servizio-uno'
```

Ciò avviene perché, come già detto, sul bridge di default non vi è un servizio di risoluzione di nomi. 
Occorre conoscere l'indirizzo IP, ma tale informazione non è nota al momento della creazione del container.

## <a name="Task_4"></a>Utilizzo del flag *--link*

Una possibilità è quella di linkare esplicitamente un container ad un altro.
Facciamolo con il *servizio-tre*:


```.term1
   docker run --name servizio-tre --link servizio-uno -d alpine:3.8 \
   /bin/sh -c "while true; do echo servizio-tre; sleep 20; done"
```
eseguiamo il ping

```.term1
    docker exec servizio-tre \
   /bin/sh -c 'ping -c 4 servizio-uno'
```

L'output sarà più o meno il seguente:
```
PING servizio-uno (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=3.190 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.137 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.183 ms
64 bytes from 172.17.0.2: seq=3 ttl=64 time=0.185 ms

--- servizio-uno ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.137/0.923/3.190 ms
```
In questo caso, pur essendo attestato sul bridge di default, questo container è stato linkato esplicitamente al *container-uno*, quindi il ping funziona. 
Se però eseguiamo il ping sul *servizio-due* che **NON** abbiamo linkato:
```.term1
    docker exec servizio-tre \
   /bin/sh -c 'ping -c 4 servizio-due'
```
otteniamo di nuovo
```
ping: bad address 'servizio-due'
```

E' chiaro che una simile soluzione, seppur applicabile, può essere scomoda in presenza di un numero di container superiore.
La cosa migliore da fare è creare una rete di tipo bridge **user-defined**


## <a name="Task_5"></a>Creazione del bridge

Cominciamo a creare la nostra rete:
```.term1
   docker network create my-net
```
Con questo comando abbiamo creato una rete di nome *my-net* di tipo bridge (default).

Adesso aggiungiamo le nostre macchine servizio-uno e servizio-due a questa rete:

```.term1
   docker network connect my-net servizio-uno
   docker network connect my-net servizio-due
```
Ora eseguiamo il ping che prima falliva:

```.term1
   docker exec servizio-due /bin/sh -c 'ping -c 4 servizio-uno'
```
il risultato questa volta sarà poisitivo.

Per nostra curiosità possiamo verificare come le macchine *servizio-uno* e *servizio-due* appartengano alla rete *my-net* eseguendo il comando

```.term1
   docker network inspect my-net
```

Verrà stampato un file di configurazione in formato json simile al seguente:

```
[
    {
        "Name": "my-net",
        "Id": "38835db7823bada955e4b3f9b5918c8251feef1163609fa0dcc320a30447a4ba",
        "Created": "2018-11-20T11:54:11.107174759Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "21c7f69ab7cf182696ab464790e4a4f60c5d532f6a79d5bfde12e81daa484037": {
                "Name": "servizio-uno",
                "EndpointID": "b207395a465d2a73cedcf708157c5086129b187de9a5a607e9b246a5940e6754",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "f467a5d1571def5d01e2eed53a09b6aaddc470c93302aa27e64a14573a361825": {
                "Name": "servizio-due",
                "EndpointID": "8246afab9de0fb0adc85cab0f5d8e9d4ab037ff3419661d4794e50dba2fd6326",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

Nella sezione **Containers** si possono vedere i nostri due containers *servizio-uno* e *servizio-due*
