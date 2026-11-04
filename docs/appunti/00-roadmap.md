# BluFit — Roadmap

App personale per corsa, palestra, corpo, nutrizione e alcol, con coach AI locale.
Stesso stack di Conto Chiaro 2.0, codice scritto da zero.

## Stack

| Livello | Tecnologia | Perché |
|---|---|---|
| Frontend | React 19 + TypeScript + Vite, React Router, TanStack Query, Recharts, lucide-react | Lo conosci già; PWA installabile su Mac e iPhone |
| Backend | Node + Express 5 + TypeScript, Prisma, Zod, argon2 | API tipizzata, validazione, login sicuro |
| Database | PostgreSQL 16 in Docker | Dati in casa tua, backup con pg_dump |
| AI | Servizio Python (FastAPI + LangChain + ChromaDB) con Ollama sul Mac | Schede e piani di corsa generati in locale |
| Ingresso | Caddy (prod) / proxy di Vite (dev) | Stessa origin, niente CORS |
| Rete | Tailscale (HTTPS) | iPhone fuori casa senza aprire porte |
| CI/CD | GitHub Actions | Ogni push: tipi, test, build Docker |

## Porte (per non scontrarsi con Conto Chiaro)

| Cosa | Conto Chiaro | BluFit |
|---|---|---|
| Postgres dev (sul Mac) | 5434 | **5435** |
| Backend dev | 3000 | **3100** |
| Vite dev | 5173 | **5180** |
| App prod (Caddy) | 8080 | **8090** |
| Servizio AI | 8000 (interno) | **8100** (interno) |

## Topbar (menu a tendina, niente sidebar)

Home · Corsa · Palestra · Corpo · Nutrizione · Coach AI (+ menu utente)

## Fasi

### Fase 0 — Fondamenta
- 0.1 Repo, struttura cartelle, Postgres in Docker  ← **siamo qui**
- 0.2 Backend: Express + TypeScript + Prisma, endpoint /api/health
- 0.3 Frontend: Vite + React, tema blu, topbar con tendine, routing
- 0.4 Login (utente singolo, account creati solo da script/inviti), sessioni con cookie
- 0.5 PWA: manifest, icone, service worker, installazione su iPhone
- 0.6 CI su GitHub Actions
- 0.7 Produzione: docker-compose.prod.yml, Caddy, Tailscale, backup giornalieri + offsite

### Fase 1 — Corsa (prima feature)
- Modello dati: corsa (data, tipo, distanza, durata, dislivello, FC media/max, cadenza, RPE, scarpe, note) + split per km
- Inserimento manuale e import GPX/FIT (dall'Apple Watch tramite export)
- Pagina corsa: mappa del percorso, grafici passo/FC/altimetria, split
- Statistiche: volume settimanale/mensile, passo medio, record personali (1k, 5k, 10k, mezza, maratona), stime gara
- Zone di frequenza cardiaca e di passo
- Scarpe: km percorsi e avviso usura

### Fase 2 — Piani di corsa (stile Runna)
- Obiettivo (distanza, data gara, tempo target), settimane, sedute pianificate
- Confronto pianificato vs eseguito, aderenza al piano

### Fase 3 — Palestra
- Catalogo esercizi con muscoli primari/secondari e attrezzatura (palestra / casa a corpo libero)
- Schede e sedute, log di serie × ripetizioni × kg, recuperi
- Pagina allenamento con **figura umana SVG** che colora i muscoli lavorati
- 1RM stimato, volume per gruppo muscolare, progressione

### Fase 4 — Corpo
- Peso, misure, % grasso, trend con media mobile

### Fase 5 — Nutrizione e alcol
- Pasti, calorie e macro, obiettivi giornalieri
- Bevande alcoliche: grammi di alcol, unità, stima tasso (Widmark), totali settimanali

### Fase 6 — Coach AI
- Ollama in locale + LangChain + ChromaDB (conoscenza su esercizi e metodologia)
- Genera schede (palestra/casa) e piani di corsa a partire dai tuoi dati

### Fase 7 — Dashboard
- Home con riepilogo settimana, carico di allenamento, obiettivi

## Metodo di lavoro
Per ogni passo: file da scrivere con spiegazione → comandi da lanciare → controllo che funzioni → commit.
Gli appunti di ogni passo stanno in `docs/appunti/NN-titolo.md` (il prontuario).
