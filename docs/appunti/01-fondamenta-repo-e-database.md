# 01 — Fondamenta: repo, struttura e database

**Obiettivo:** alla fine di questo passo hai un repository Git `blufit` (anche su GitHub)
e un PostgreSQL acceso in Docker, pronto per il backend del passo 0.2.

---

## 1. Struttura delle cartelle

```
blufit/
├── backend/          ← API Node/Express (passo 0.2)
├── frontend/         ← app React (passo 0.3)
├── ai-service/       ← coach AI in Python (fase 6)
├── ops/              ← script di backup e infrastruttura (passo 0.7)
├── docs/appunti/     ← questo prontuario
├── docker-compose.yml
├── .env.example
├── .env              ← NON va su GitHub
├── .gitignore
└── README.md
```

Da Terminale:

```bash
cd ~/Desktop/blufit
mkdir -p backend frontend ai-service ops
```

> **Perché un "monorepo"?** Frontend, backend e AI cambiano spesso insieme (es. aggiungi un
> campo alla corsa → tocchi database, API e pagina). Un solo repo = un solo commit coerente.

---

## 2. `.gitignore`

Dice a Git cosa **non** salvare mai. Crea `blufit/.gitignore`:

```gitignore
# Dipendenze e build (si rigenerano)
node_modules/
dist/
build/

# Segreti: password e chiavi restano solo sul Mac
.env
.env.prod
!.env.example
!.env.prod.example

# Backup del database: dati personali, mai su GitHub
backups/
*.dump

# Python
__pycache__/
.venv/
*.pyc

# Sistema / editor
.DS_Store
*.log
```

> Le righe con `!` sono eccezioni: i file `.example` (senza segreti) vanno su GitHub,
> così sai sempre quali variabili servono.

---

## 3. `.env.example` e `.env`

Le **variabili d'ambiente** tengono fuori dal codice ciò che cambia tra macchine o è segreto.
Docker Compose legge automaticamente il file `.env` nella stessa cartella.

Crea `blufit/.env.example`:

```dotenv
# ---- Database di SVILUPPO ----
POSTGRES_USER=blufit
POSTGRES_PASSWORD=cambiami
POSTGRES_DB=blufit_dev
# Porta sul Mac: 5435, perché 5434 è già di Conto Chiaro (dev)
DB_PORT=5435
```

Poi copialo e metti una password vera:

```bash
cp .env.example .env
openssl rand -base64 24     # genera una password casuale: incollala in .env al posto di "cambiami"
```

> Evita nella password i caratteri `@ / : ?` — finiranno dentro un URL (`DATABASE_URL`) e lo
> romperebbero. Se `openssl` li produce, rigenera.

---

## 4. `docker-compose.yml` (solo database, per ora)

Crea `blufit/docker-compose.yml`:

```yaml
# =====================================================================
# BluFit — stack di SVILUPPO
# Per ora solo PostgreSQL; backend e frontend in dev girano sul Mac
# (npm run dev) per avere il ricaricamento istantaneo.
#
# Avvio:   docker compose up -d
# Stato:   docker compose ps
# Stop:    docker compose stop        (i dati restano)
# ATTENZIONE: "docker compose down -v" cancella anche i dati.
# =====================================================================

name: blufit-dev          # prefisso di container e volumi: separati da Conto Chiaro

services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      TZ: Europe/Rome
    ports:
      # 127.0.0.1 = raggiungibile solo dal Mac, non dalla rete di casa
      - "127.0.0.1:${DB_PORT:-5435}:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  pgdata:                 # volume Docker: i dati sopravvivono a stop e riavvii
```

**Come leggerlo**
- `image` → Postgres 16 in versione *alpine* (leggera). Stessa versione di Conto Chiaro.
- `${POSTGRES_USER}` → valore preso da `.env`. `${DB_PORT:-5435}` = "usa DB_PORT, se manca 5435".
- `ports "127.0.0.1:5435:5432"` → porta 5435 del Mac → porta 5432 *dentro* il container.
- `volumes pgdata` → dove Postgres scrive i dati; è gestito da Docker, non sta nella cartella.
- `healthcheck` → Docker sa quando il DB è davvero pronto (servirà al backend per aspettarlo).

---

## 5. `README.md`

Breve, per te tra sei mesi:

```markdown
# BluFit

App personale di allenamento: corsa, palestra, corpo, nutrizione, coach AI.

## Avvio in sviluppo
1. `cp .env.example .env` e imposta la password
2. `docker compose up -d`
3. Appunti e roadmap: `docs/appunti/`
```

---

## 6. Accendi e verifica

Docker Desktop deve essere aperto.

```bash
cd ~/Desktop/blufit
docker compose up -d
docker compose ps
```

Attendi che `STATUS` diventi `healthy`, poi entra nel database:

```bash
docker compose exec db psql -U blufit -d blufit_dev -c "select version();"
```

✅ Se vedi `PostgreSQL 16.x ...` il database funziona.

Controlla anche che Conto Chiaro non sia stato toccato:

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Devi vedere `blufit-dev-db-1` su `127.0.0.1:5435` e i container `contochiaro-*` come prima.

---

## 7. Git e GitHub

```bash
git init -b main
git status                 # .env NON deve comparire nell'elenco!
git add .
git commit -m "chore: struttura progetto e PostgreSQL di sviluppo"
```

Su github.com crea un repo **privato** `blufit` (senza README, ce l'hai già), poi:

```bash
git remote add origin https://github.com/malliamatteo99-afk/blufit.git
git push -u origin main
```

> Le cartelle vuote (`backend/`, `frontend/`…) Git non le salva: compariranno quando ci
> metteremo dentro i primi file. È normale.

---

## Checklist
- [ ] Cartelle create
- [ ] `.gitignore`, `.env.example`, `.env` (password vera), `docker-compose.yml`, `README.md`
- [ ] `docker compose ps` → `healthy`
- [ ] `select version();` risponde
- [ ] `.env` non è nel commit
- [ ] Repo privato su GitHub con il primo push

**Prossimo passo — 0.2:** backend Express + TypeScript + Prisma, primo endpoint `/api/health`
che verifica anche la connessione al database.
