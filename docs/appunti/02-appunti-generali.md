# **APPUNTI GENERALI**

## ***.gitignore***

Il ? per Git non vuol dire "forse". E' un jolly che sta per un carattere qualsiasi. L'eccezione si scrive con il punto esclamativo all'inziio: !.env.example (per esempio).

Mettendo l'asterisco alla fine di un qualcosa (.dump.*), ignoro tutti i file che iniziano con .dump. Se si vuole ignorare tutti i file che finiscono con .dump, l'asterisco va prima di tutto *.dump.

Per quanto riguarda Python, va sempre ignorata la possibile spazzatura che viene creata. In particolare vengono create le cartelle __pycache__/, i file *.pyc e l'ambiente virtuale .venv/.

I file automatici che vengono creati dal Mac sono normalmente .DS_Store e i log *.log.

Il trucco è il seguente:
- *: indica qualsiasi sequenza di caratteri ➡️ *.log prende errori.log, app.log...
- ?: indica un solo carattere qualsiasi
- !: all'inizio indica un'eccezione, cioè "questo invece tienilo"

## ***.env.example***

1. I nomi delle variabili: L'immagine ufficiale di Postgres cerca nomi precisi: POSTGRES_USER, POSTGRES_PASSWORD e POSTGRES_DB.
2. Per la porta del Mac usare sempre DB_PORT
3. Il nome del database non è un segreto: il segnaposto serve solo per i valori segreti, cioè la password. Il nome del database si può tranquillamente mettere, così si capisce subito che è quello di sviluppo. Per la password basta un segnaposto corto come "Cambiami".

## ***.env***

Per creare una copia di .env.example rinominandolo .env, devo utilizzare il comando da terminale "cp .env.example .env".
Per la creazione di una password randomica di 24 carattere devo utilizzare il comando da terminale "openssl rand -hex 24".

## ***docker-compose.yml***

E' la ricetta che dice a Docker quali container accendere e come. Per ora ne avrà uno solo: il database. Backend e frontend, in sviluppo, li faremo girare direttamente sul Mac con npm run dev, così ogni modifica si vede all'istante. 

Due domande:
1. Il formato è YAML. Rispetto a JSON, cosa usa YAML per capire cosa sta "dentro" cosa, al posto delle parentesi graffe?
2. Nel compose scriveremo POSTGRES_USER: ${POSTGRES_USER}. Da dove prende Docker il valore dentro ${...}?

Risposte:
1. YAML capisce cosa sta "dentro" cosa grazie all'indentazione, cioè agli spazi a inizio riga, proprio come Python. Tre regole d'oro per YAML:
    - chiave: valore, con uno spazio dopo i due punti
    - 2 spazi per ogni livello, e mai il tasto Tab: YAML va in errore
    - il trattino a inizio riga significa che è l'elemento di una lista
2. Docker Compose legge il file .env che sta nella stessa cartella del docker-compose.yml e sostituisce ogni ${NOME} con il valore corrispondente. L'analogia con il dizionario Python è azzeccata: è proprio una raccolta di coppie chiave ➡️ valore.

Il docker-compose.yml viene scritto così:

name: il nome del progetto, blufit-dev. Serve a non mescolare container e volumi con quelli creati in altre situazioni

services:
  db:
    image: postgres:16-alpine # Tra nome dell'immagine e versione ci vogliono i due punti
    restart: unless-stopped # Gli altri valori che si possono mettere sono "no", "always" "on-failure"
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      TZ: Europe/Rome
    ports: 
      - "127.0.0.1:${DB_PORT:-5435}:5432" Deve essere una lista, perchè un container può avere più porte. Meglio anche metterla tra virgolette, perché i numeri con i : a volte YAML li interpreta in modo strano
    volumes: 
      - pgdata:/var/lib/postgresql/data Deve essere una lista con il nome del volume (in questo caso pgdata è il nome e successivamente si inserisce il percorso)
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      Il test normalmente mostra ["executable", "arg"] ed è il segnaposto della documentazione. Il comando vero andrà a sostiuire executable com "CMD-SHELL" che vuol dire "esegui questa riga in una shell" e l'argomento invece è il comando effettivo.
      interval: 5s ridurre i valori di interval, timeout e retries
      timeout: 3s
      retries: 10
      start_period: 30s

volumes: qui dichiariamo i volumi e la dichiarazione è un dizionario. Il nome va su una riga sua, seguito dai due punti e senza valore
  pgdata:

Dopo aver salvato il file, occorre controllare che Docker Desktop sia attivo e successivamente mandare da terminale questi tre comandi:
- docker compose config --quiet && echo "YAML ok"
- docker compose up -d
- docker compose ps
Il primo comando controlla soltanto che il file sia scritto bene, senza avviare niente. 

## ***Primo git***

Per fare il primo git effettivo occorre fare i seguenti comandi da terminale:
- git init -b main
- git status
- git add .
- git commit -m "..."
- git remote add origin "link del profilo github"
- git push -u origin main