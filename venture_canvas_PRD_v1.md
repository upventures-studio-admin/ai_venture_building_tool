# Venture Building Canvas — Product Requirements Document

**Upventure · v1.0 · 2025**

---

| Campo | Valore |
|---|---|
| Documento | Product Requirements Document – v1.0 |
| Prodotto | Venture Building Canvas |
| Owner | Upventure |
| Stato | Draft – In revisione interna |
| Data | 2025 |
| Utenti target | Team interni Upventure, Clienti di Upventure |
| Documenti di riferimento | App Brief v1.0 · Node Library v1.0 |

---

## Indice

1. [Product Overview](#1-product-overview)
2. [Obiettivi di Prodotto e Metriche di Successo](#2-obiettivi-di-prodotto-e-metriche-di-successo)
3. [Utenti e Personas](#3-utenti-e-personas)
4. [Requisiti Funzionali](#4-requisiti-funzionali)
5. [Libreria dei Nodi](#5-libreria-dei-nodi)
6. [Configurazione LLM e Integrazione AI](#6-configurazione-llm-e-integrazione-ai)
7. [Template Predefiniti e Canvas](#7-template-predefiniti-e-canvas)
8. [Persistenza e Versioning](#8-persistenza-e-versioning)
9. [Nodi di Output e Formati di Esportazione](#9-nodi-di-output-e-formati-di-esportazione)
10. [Requisiti Non Funzionali](#10-requisiti-non-funzionali)
11. [Note Architetturali](#11-note-architetturali)
12. [Roadmap e Milestones](#12-roadmap-e-milestones)
13. [Questioni Aperte e Decisioni da Prendere](#13-questioni-aperte-e-decisioni-da-prendere)

---

## 0. Registro delle Modifiche

| Versione | Data | Modifiche | Autore |
|---|---|---|---|
| 1.0 | 2025 | Prima stesura del PRD da App Brief + Node Library | Upventure |

---

## 1. Product Overview

### 1.1 Descrizione del Prodotto

Venture Building Canvas è un'applicazione cloud-based progettata da Upventure per supportare lo sviluppo strutturato di nuove idee di business e iniziative di innovazione. A differenza di strumenti generici di whiteboarding (Miro, FigJam, Notion), ogni elemento del canvas è un **nodo intelligente** che esegue azioni concrete tramite LLM, trasformando il brainstorming in un flusso di lavoro strutturato, collaborativo e documentabile.

### 1.2 Visione e Posizionamento

> **Visione:** Diventare il sistema operativo del venture building per Upventure e i suoi clienti: uno strumento in cui ogni fase del processo innovativo è eseguita, tracciata e documentata in modo standardizzato e scalabile.

Il prodotto si posiziona come un **canvas esecutivo**, non documentativo.

| Dimensione | Competitor (Miro, Notion, FigJam) | Venture Building Canvas |
|---|---|---|
| Funzione del canvas | Documentazione e whiteboarding | Esecuzione: ogni nodo fa qualcosa |
| Intelligenza AI | Add-on generico opzionale | Nativa: ogni nodo è un agente LLM |
| Metodologia | Agnostica | Embedded: venture building Upventure |
| Output | Post-it, testo, diagram | Documenti strutturati pronti all'uso |
| Personalizzazione | Template visivi | Libreria di nodi con prompt configurabili |
| IP metodologico | Nessuno | Accumulato nella libreria nodi proprietaria |

### 1.3 Contesto Strategico

Upventure opera come partner di innovation e venture building per aziende corporate e startup. Il processo attuale si basa su prompt operativi avanzati eseguiti manualmente su LLM generici. Questo approccio ha tre criticità:

- **Mancanza di struttura:** ogni facilitatore esegue il processo in modo diverso
- **Non scalabile:** il tempo operativo per progetto è elevato e non si riduce con l'esperienza
- **IP non capitalizzato:** i prompt e la metodologia non vengono standardizzati né accumulati

Il Venture Building Canvas risolve queste criticità trasformando la metodologia Upventure in un prodotto software riproducibile, scalabile e vendibile.

---

## 2. Obiettivi di Prodotto e Metriche di Successo

### 2.1 Obiettivi di Business

| ID | Obiettivo | Razionale |
|---|---|---|
| OB-01 | Ridurre il tempo operativo per progetto | Standardizzare il processo permette di abbattere il tempo medio di analisi e documentazione |
| OB-02 | Scalare la metodologia Upventure | Ogni nuovo facilitatore opera al livello dei senior grazie ai nodi pre-configurati |
| OB-03 | Creare IP metodologico proprietario | La libreria nodi diventa un asset aziendale che si arricchisce ad ogni progetto |
| OB-04 | Differenziare l'offerta di mercato | Nessun competitor offre uno strumento simile |
| OB-05 | Abilitare nuovi modelli di revenue | Il canvas può essere offerto come SaaS ai clienti, generando revenue ricorrente |

### 2.2 Obiettivi di Prodotto (OKR)

| Objective | Key Result 1 | Key Result 2 |
|---|---|---|
| Rendere il processo di venture building eseguibile in autonomia | 100% dei nodi della catena principale disponibili al lancio | ≥ 3 template predefiniti attivi |
| Garantire qualità e affidabilità dell'output LLM | 0 nodi che producono output non strutturato senza notifica | Tempo medio risposta nodo ≤ 30s (P95) |
| Abilitare collaborazione e tracciabilità | Versioning completo con rollback funzionante | Auto-save con latenza ≤ 2 secondi |
| Supportare personalizzazione e crescita della libreria | Editor nodo custom disponibile al lancio | Import/export nodi in JSON funzionante |

### 2.3 Metriche di Successo (KPI)

| KPI | Descrizione | Target 3 mesi | Target 12 mesi |
|---|---|---|---|
| Time-to-canvas | Tempo dalla registrazione al primo nodo eseguito | < 10 min | < 5 min |
| Completion rate | % canvas avviati che raggiungono almeno NODE-BS-01 | — | ≥ 60% |
| Nodi custom creati | Numero di nodi personalizzati creati dai workspace | ≥ 5 | ≥ 50 |
| Canvas per workspace | Media di canvas attivi per organizzazione | ≥ 2 | ≥ 5 |
| Output generati | Documenti esportati (PDF, PPTX, Docs) | ≥ 10 | ≥ 200 |
| NPS interno | Net Promoter Score team Upventure | ≥ 7 | ≥ 8 |
| LLM error rate | % chiamate LLM con errore o output non strutturato | < 5% | < 2% |

---

## 3. Utenti e Personas

### 3.1 Utenti Target

- **Team interni Upventure:** facilitatori, innovation manager, consulenti che usano il canvas per erogare il servizio ai clienti
- **Clienti di Upventure:** team di innovazione corporate, PMI, startup che usano il canvas in autonomia o in co-facilitazione

### 3.2 Personas

---

#### Marco – Innovation Facilitator Senior
*Team interno Upventure · 5+ anni esperienza*

Marco gestisce 3-4 progetti di venture building in parallelo. Conosce a fondo la metodologia Upventure ma perde tempo a "reinventare la ruota" ad ogni progetto: copia prompt, ricrea documenti, spiega il processo ai colleghi junior.

**Obiettivi:**
- Completare una fase di venture building in metà tempo
- Onboardare un junior in meno di un'ora
- Avere output sempre strutturati e coerenti
- Personalizzare i prompt per verticali specifici

**Pain points:**
- I prompt non sono versionati né condivisi sistematicamente
- Gli output LLM cambiano formato ad ogni sessione
- Non c'è tracciabilità di cosa è stato fatto e quando
- Il cliente vede il processo come "magia nera"

---

#### Giulia – Innovation Manager Corporate
*Cliente Upventure · Responsabile innovazione PMI manifatturiera*

Giulia deve portare avanti un'iniziativa di innovazione interna ma non ha competenze di venture building. Lavora con Upventure come partner ma vuole essere più autonoma nel processo.

**Obiettivi:**
- Avviare un progetto di innovazione senza dipendere sempre da un consulente
- Produrre documentazione presentabile al management
- Capire dove si trova nel processo in ogni momento
- Avere output esportabili per presentazioni interne

**Pain points:**
- Non sa da dove iniziare senza una guida step-by-step
- Teme di "saltare passaggi importanti"
- Gli strumenti generici (Miro, Notion) non la guidano
- Fatica a spiegare al management cosa sta facendo e perché

---

#### Luca – Founder / Startup
*Cliente Upventure · Co-founder early-stage*

Luca sta costruendo una startup e usa Upventure per la fase di discovery e validazione. Ha energia e visione ma tende a saltare i passaggi analitici per arrivare subito alla soluzione.

**Obiettivi:**
- Validare l'idea con dati reali prima di costruire
- Produrre un pitch deck credibile in poco tempo
- Capire il mercato e i competitor in modo strutturato
- Iterare velocemente sulle ipotesi

**Pain points:**
- Tende a partire dalla soluzione invece che dal problema
- Non sa come fare una market research rigorosa
- Il pitch deck viene rifatto da zero ad ogni iterazione
- Confonde output LLM "bello" con output valido

---

### 3.3 User Journey Principale

| # | Step | Azione utente | Output sistema |
|---|---|---|---|
| 1 | Onboarding | Crea account, configura workspace, inserisce API keys LLM | Workspace creato, LLM default configurato |
| 2 | Nuovo Canvas | Sceglie template o parte da canvas vuoto | Canvas con nodi pre-collegati caricato |
| 3 | Input iniziale | Aggiunge nodo Input e inserisce la challenge iniziale | Nodo Input salvato nel canvas |
| 4 | Primo nodo LLM | Esegue NODE-PD-01 (Problem Discovery) | LLM elabora, output strutturato nel nodo |
| 5 | Revisione | Legge l'output, fa modifiche o re-esegue | Versione aggiornata salvata automaticamente |
| 6 | Catena | Propaga output a MR-01, BS-01, GTM-01 | Ogni nodo riceve input dal precedente |
| 7 | Output finale | Aggiunge nodo Output, seleziona formato PPTX | Pitch deck generato e scaricabile |

---

## 4. Requisiti Funzionali

> **Scala di priorità:** P0 = bloccante per il lancio · P1 = primo sprint post-lancio · P2 = enhancement · P3 = backlog

### 4.1 Canvas a Navigazione Libera

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-C-01 | Infinite Canvas | Canvas infinito con pan/zoom tramite mouse, trackpad e shortcut da tastiera | P0 |
| REQ-C-02 | Aggiunta nodi | Drag-and-drop o doppio click dal pannello libreria | P0 |
| REQ-C-03 | Spostamento nodi | Nodi spostabili liberamente; posizione salvata in auto-save | P0 |
| REQ-C-04 | Ridimensionamento | Ogni nodo è ridimensionabile tramite handle ai bordi | P1 |
| REQ-C-05 | Rimozione nodi | Eliminazione singola o multipla con conferma | P0 |
| REQ-C-06 | Selezione multipla | Rettangolo di selezione (lasso) o Shift+click | P1 |
| REQ-C-07 | Grouping | Nodi raggruppabili in cluster tematici con label personalizzabile | P2 |
| REQ-C-08 | Minimap | Minimap nell'angolo inferiore per orientamento in canvas grandi | P2 |
| REQ-C-09 | Snap to grid | Allineamento opzionale a griglia | P3 |
| REQ-C-10 | Keyboard shortcuts | Cmd+Z undo, Cmd+D duplicate, Delete remove, Space+drag pan | P1 |

### 4.2 Sistema di Nodi

#### Anatomia di un Nodo

Ogni nodo è composto da sette elementi fissi indipendentemente dal tipo:

| Componente | Descrizione | Dettaglio tecnico |
|---|---|---|
| Header | Nome, tipo, stato di esecuzione, LLM selezionato | Badge stato: idle / running / done / error |
| Input Panel | Area dove confluiscono i dati in ingresso | Testo libero + output propagati da nodi collegati |
| Prompt Editor | Campo di configurazione del prompt LLM | Textarea con variabili `{input}`, `{context}`; precompilato da libreria |
| Execution Controls | Pulsanti di controllo esecuzione | Run, Stop, Re-run, toggle manuale/cascata |
| Output Panel | Area dove viene visualizzato il risultato LLM | Testo strutturato, JSON, Markdown con copy/download |
| LLM Config | Selettore modello e parametri | Dropdown provider/modello, temperatura, max_tokens; override del default workspace |
| Link Ports | Porte di ingresso e uscita per i link | Input port (sinistra/top), output port (destra/bottom) |

#### Tipi di Nodo

| Tipo | Simbolo | Comportamento | Configurabile |
|---|---|---|---|
| LLM Node | ◈ | Esegue un prompt su un modello AI. Input testuale o da link. Output: testo, JSON, Markdown | Prompt, modello, temperatura, max_tokens, system prompt |
| Input Node | ▶ | Nodo di ingresso dati. Non esegue LLM. L'utente inserisce testo, incolla documenti, carica file | Label, placeholder, tipo input (testo/file/URL) |
| Output Node | ◉ | Assembla gli output dei nodi collegati e genera un documento finale nel formato selezionato | Formato (PDF/PPTX/Docs/MD/JSON), template, titolo |
| Custom Node | ⬡ | Nodo creato dall'utente. Uguale a LLM Node ma creato da zero. Salvabile in libreria | Tutti i parametri LLM Node + metadata libreria |

#### Requisiti Nodi

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-N-01 | Struttura uniforme | Ogni nodo espone sempre tutti e 7 i componenti dell'anatomia | P0 |
| REQ-N-02 | Stato esecuzione visivo | idle (grigio), running (pulsante animato), done (verde), error (rosso) | P0 |
| REQ-N-03 | Prompt con variabili | Supporto `{input}`, `{context}` con sostituzione dinamica all'esecuzione | P0 |
| REQ-N-04 | Copia nodo | Duplicazione mantenendo tutte le configurazioni | P1 |
| REQ-N-05 | Export nodo singolo | Export come `.json` con schema completo | P1 |
| REQ-N-06 | Note sul nodo | Note testuali visibili hovering | P2 |
| REQ-N-07 | Collapse nodo | Riduzione a sola header per risparmiare spazio | P2 |
| REQ-N-08 | Wizard gate | I nodi wizard espongono UI step-by-step con stop obbligatorio dopo ogni fase | P0 |

### 4.3 Sistema di Link e Passaggio Dati

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-L-01 | Link direzionale | Direzione esplicita (sorgente → destinazione) con freccia | P0 |
| REQ-L-02 | Modalità automatica | Output iniettato nell'input del nodo destinazione appena disponibile | P0 |
| REQ-L-03 | Modalità manuale | Utente propaga l'output con click esplicito sul link | P0 |
| REQ-L-04 | Modalità visiva | Link solo organizzativo: non trasferisce dati | P1 |
| REQ-L-05 | Configurazione per link | Pannello configurazione accessibile con doppio click | P0 |
| REQ-L-06 | Preview dati | Tooltip hovering con anteprima del dato in transito | P2 |
| REQ-L-07 | Multi-input | Un nodo riceve link da più sorgenti; contenuti concatenati con separatori | P1 |
| REQ-L-08 | Multi-output | Output propagato a più nodi destinazione contemporaneamente | P1 |
| REQ-L-09 | Eliminazione link | Delete su link selezionato | P0 |
| REQ-L-10 | Validazione cicli | Il sistema impedisce link circolari che creerebbero loop infiniti | P0 |

### 4.4 Esecuzione dei Nodi

| Modalità | Comportamento | Quando usarla |
|---|---|---|
| **Manuale** | L'utente avvia ogni nodo individualmente con Run. Nessuna propagazione automatica | Sessioni riflessive, workshop con clienti, iterazione su singole fasi |
| **A Cascata** | Quando un nodo completa, i nodi collegati in modalità automatica vengono eseguiti in sequenza | Generazione rapida di report, automazione flussi ripetitivi, demo |

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-E-01 | Run singolo | Pulsante Run su ogni nodo | P0 |
| REQ-E-02 | Run canvas | Pulsante "Run All" per cascata dall'intero canvas | P0 |
| REQ-E-03 | Stop esecuzione | Pulsante Stop per interruzione in corso | P0 |
| REQ-E-04 | Re-run | Re-esecuzione con sovrascrittura output previo confirm | P0 |
| REQ-E-05 | Progress indicator | Progress bar globale durante cascata (nodi completati / totale) | P1 |
| REQ-E-06 | Gestione errori LLM | Stato "error" con messaggio descrittivo e suggerimento azione | P0 |
| REQ-E-07 | Timeout | Timeout configurabile (default 120s); allo scadere → stato error | P0 |
| REQ-E-08 | Retry automatico | Max 3 retry con backoff esponenziale su errori transitori | P1 |
| REQ-E-09 | Log esecuzione | Per ogni run: timestamp, modello, token input/output, durata, stato | P1 |
| REQ-E-10 | Wizard stop | I nodi wizard attendono conferma esplicita prima di procedere al passo successivo | P0 |

---

## 5. Libreria dei Nodi

### 5.1 Struttura della Libreria

La libreria ha architettura a due livelli:

| Dimensione | Libreria Pubblica | Libreria Privata (Workspace) |
|---|---|---|
| Visibilità | Accessibile a tutti gli utenti | Solo ai membri del workspace |
| Contenuto | Nodi predefiniti Upventure + nodi condivisi con approvazione | Nodi custom + versioni modificate di nodi pubblici |
| Modifica | Sola lettura (forking permesso) | Modifica completa dai membri |
| Import/Export | Import JSON → personalizzazione locale | Export JSON per portabilità |
| Governance | Approvazione Upventure per pubblicazione | Gestita dall'admin del workspace |

### 5.2 Libreria Base v1.0 – Nodi Predefiniti

| Node ID | Nome | Fase | Input chiave | Output chiave | LLM default |
|---|---|---|---|---|---|
| NODE-PD-01 | Problem Discovery – Analisi Completa | Fase 1 | Challenge, interviste, desk research | 10 sezioni, Problem Statement, SWOT | Opus / GPT-4 |
| NODE-MR-01 | Ricerca di Mercato – Arricchimento Dati | Fase 2 | Problem Statement, cliente pagante, stime TAM | TAM/SAM/SOM ricalcolati, competitor map, data gaps | Opus / Gemini |
| NODE-BS-01 | Brainstorming Soluzione – Wizard 5 Step | Fase 3 | Problem Discovery, Market Research | Direzioni strategiche, MVP, Stack, Piano B, Rischi | Opus / GPT-4 |
| NODE-GTM-01 | Strategia & Go-to-Market – Wizard 4 Step | Fase 4 | Solution Definition, PD, MR | GTM options, Business Model, Roadmap, Stress test | Opus / GPT-4 |
| NODE-PITCH-01 | Pitch Deck – Wizard 10 Slide | Fase 5 | PD + MR + BS + GTM (tutti obbligatori) | 10 slide con headline, visual suggestion, speaker notes | Opus / GPT-4 |

### 5.3 Schede Nodo Dettagliate

---

#### NODE-PD-01 — Problem Discovery – Analisi Completa

| Campo | Valore |
|---|---|
| Tipo | LLM Node – Esecuzione singola (non wizard) |
| LLM default | Claude Opus 4 / GPT-4 Thinking |
| Input obbligatorio | Testo della challenge o dell'idea di business |
| Input opzionale | Risposte interviste stakeholder · Note di desk research |
| Fasi interne | P1 Riscrittura neutra · P2 Impatto · P3 Cause/sottocause · P4 Validazione evidenze · P5 Contesto · P6 Stakeholder · P7 Problem Statement · P8 Data Enrichment · P9 Alternative pre-soluzione · P10 SWOT alternative |
| Regole hard | No invenzione dati · Inferenze solo da fonti autorevoli (McKinsey/BCG/OECD/CE/WEF) degli ultimi 2-3 anni · No progettazione soluzioni · Gap dati segnalati esplicitamente |
| Output struttura | Documento 10 sezioni · Problem Statement validato (1-3 righe) · Tabella SWOT alternative · Data enrichment con range impatto € e formula |
| Output verso | NODE-MR-01 · NODE-BS-01 · NODE-GTM-01 |
| Wizard gate | No. Se input insufficienti: max 6 domande di chiarimento prima di procedere |

---

#### NODE-MR-01 — Ricerca di Mercato – Arricchimento Dati

| Campo | Valore |
|---|---|
| Tipo | LLM Node – Esecuzione singola (non wizard) |
| LLM default | Claude Opus / GPT-4 Thinking / Gemini 1.5 Pro |
| Input obbligatorio | Problem Statement (da NODE-PD-01) · Cliente pagante (ruolo, settore, geo) |
| Input opzionale | Stime TAM/SAM/SOM del team · Elenco competitor noti |
| Output richiesti | 1. Nuovi dati verificati (fonte/anno/geo) · 2. TAM ricalcolato (formula + delta) · 3. SAM ricalcolato · 4. SOM ricalcolato · 5. Competitor enrichment (pricing + URL) · 6. Data gaps · 7. Raw search log |
| Fonti ammesse | CE · OECD · World Bank · IEA · Eurostat · ISTAT · ENEA · WEF · McKinsey · BCG · Deloitte · PwC · Associazioni settore (ultimi 3 anni) |
| Regole hard | Ogni sezione aggiunge dati nuovi · No opinioni senza numeri · No numeri senza fonte · Calcoli espliciti obbligatori · Dati inesistenti dichiarati con motivazione |
| Output verso | NODE-BS-01 · NODE-GTM-01 · NODE-PITCH-01 |

---

#### NODE-BS-01 — Brainstorming Soluzione – Wizard 5 Step

| Campo | Valore |
|---|---|
| Tipo | LLM Node – WIZARD STEP-BY-STEP (stop obbligatorio dopo ogni step) |
| LLM default | Claude Opus 4 / GPT-4 Thinking |
| Input obbligatorio | Problem Discovery (NODE-PD-01) · Market Research (NODE-MR-01) |
| Wizard gate | Problem Statement chiaro + MR con geo/unità economica/competitor/TAM. Se mancano → max 6 domande, nessun step eseguito |
| Step 1 | Direzioni strategiche (2-3): nome, cliente pagante, meccanismo valore, alternativa attuale, difficoltà di copia 12-24m, trade-off. Selezione 1-2 direzioni + scarto motivato. **STOP.** |
| Step 2 | MVP per direzione scelta: descrizione, in scope (max 3), out of scope (max 3), flusso minimo (5 step), dati critici (max 3), integrazioni. **STOP.** |
| Step 3 | Stack tecnologico: max 6 componenti con ruolo, vincolo tecnico principale, dipendenza più rischiosa. **STOP.** |
| Step 4 | Piano B: versione minima testabile, metrica + timeframe, criterio successo/fallimento. **STOP.** |
| Step 5 | Rischi e dipendenze: top 3 rischi specifici, top 2 assunzioni critiche, punto di rottura, prima cosa da validare. **STOP.** |
| Regole hard | No dati inventati · No pitch language · Stack solo da Step 3 in poi · Procedere solo con conferma esplicita utente |
| Output verso | NODE-GTM-01 (Solution Definition) · NODE-PITCH-01 |

---

#### NODE-GTM-01 — Strategia & Go-to-Market – Wizard 4 Step

| Campo | Valore |
|---|---|
| Tipo | LLM Node – WIZARD STEP-BY-STEP |
| LLM default | Claude Opus 4 / GPT-4 Thinking |
| Input obbligatorio | Solution Definition (NODE-BS-01) |
| Input preferibile | Problem Discovery · Market Research |
| Wizard gate | Solution Definition con cliente pagante/problema/azione chiave/risultato misurabile/alternativa attuale + MR con geo/unità economica/competitor/TAM |
| Step 1 | GTM Options (2-3): nome, cliente prioritario, use case, canale ingresso, motivo acquisto ora, ostacolo adozione, alternative oggi, trade-off. Selezione + scarto motivato. **STOP.** |
| Step 2 | Business Model: unità economica venduta, chi paga e come, pricing (range+logica), driver costo (max 3), leva marginalità, 2 assunzioni critiche. **STOP.** |
| Step 3 | Roadmap Now/Next/Later: 2 milestone per fase + cosa dimostrano + KPI minimo per fase. **STOP.** |
| Step 4 | Stress test strategico: top 3 rischi GTM, top 2 assunzioni critiche, punto di rottura, prima validazione di mercato. **STOP.** |
| Regole hard | Solution Definition vincolante (non modificare) · No tecnologia/stack/UX · No dati inventati |
| Output verso | NODE-PITCH-01 |

---

#### NODE-PITCH-01 — Pitch Deck – Wizard 10 Slide

| Campo | Valore |
|---|---|
| Tipo | LLM Node – WIZARD STEP-BY-STEP + OUTPUT NODE |
| LLM default | Claude Opus 4 / GPT-4 Thinking |
| Input obbligatorio | Problem Discovery · Market Research · Solution Definition · Strategia & GTM · Nome progetto |
| Input opzionale | Data evento · Logo disponibile (sì/no) · Lingua (IT default / EN) |
| Wizard gate | Tutti e 4 i documenti obbligatori devono contenere i campi minimi. Se mancano → max 6 domande |
| Struttura 10 slide | 1. Tweet pitch · 2. Problema · 3. Soluzione · 4. Prodotto · 5. Mercato (TAM/SAM/SOM) · 6. Competitor · 7. Business Model · 8. Financial (12m) · 9. Team · 10. Chiusura |
| Output per slide | Headline · Contenuto essenziale (parole chiave) · Visual suggestion · Speaker notes |
| Regole hard | Max 10 slide · No dati inventati (→ "NON DIMOSTRABILE") · No nuove feature non nei template · Stile emozionante e assertivo |
| Connessione output | Collegare a nodo Output per generare PPTX / PDF / Google Slides |

---

### 5.4 Gestione Libreria – Requisiti Funzionali

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-LIB-01 | Browsing libreria | Pannello laterale con ricerca, filtri per categoria/fase/tipo e anteprima | P0 |
| REQ-LIB-02 | Aggiunta al canvas | Doppio click o drag-and-drop dalla libreria | P0 |
| REQ-LIB-03 | Fork nodo pubblico | Fork nel workspace privato per personalizzazione | P1 |
| REQ-LIB-04 | Crea nodo custom | Editor completo: nome, descrizione, prompt, LLM default, schema I/O, categoria | P0 |
| REQ-LIB-05 | Salva in libreria privata | Nodo custom creato nel canvas salvabile nella libreria privata | P0 |
| REQ-LIB-06 | Pubblica in libreria pubblica | Proposta pubblicazione con approvazione Upventure | P2 |
| REQ-LIB-07 | Export JSON nodo | Export `.json` con schema completo | P1 |
| REQ-LIB-08 | Import JSON nodo | Import `.json` nella libreria privata del workspace | P1 |
| REQ-LIB-09 | Versioning nodi | Ogni modifica crea nuova versione; versioni precedenti consultabili | P2 |
| REQ-LIB-10 | Nodi base non eliminabili | NODE-PD-01..PITCH-01 non eliminabili, solo forkabili | P0 |

---

## 6. Configurazione LLM e Integrazione AI

### 6.1 Modelli Supportati

| Provider | Modelli | Punti di forza | Use case consigliato |
|---|---|---|---|
| Anthropic (Claude) | Claude Opus 4 · Claude Sonnet 4 · Claude Haiku | Ragionamento complesso · Sicurezza · Lunga finestra contesto | Analisi strategica, Problem Discovery, nodi wizard |
| OpenAI (GPT-4) | GPT-4o · GPT-4 Turbo · o1 / o3 | Testi creativi · Bilanciamento qualità/velocità · Ragionamento avanzato | Brainstorming, redazione pitch, contenuti creativi |
| Google (Gemini) | Gemini 1.5 Pro · Gemini 1.5 Flash | Ricerca real-time · Integrazione Google · Velocità su task brevi | Market Research con web search, output verso Google Workspace |

### 6.2 Gerarchia di Configurazione LLM

La configurazione segue una gerarchia a 3 livelli. Il livello più specifico ha sempre priorità:

| Livello | Scope | Cosa configura | Override |
|---|---|---|---|
| 3 | Workspace (globale) | LLM default per tutti i nodi del workspace; API keys | Può essere sovrascritto da livello 2 o 1 |
| 2 | Canvas (locale) | LLM default per tutti i nodi di un canvas specifico | Può essere sovrascritto da livello 1 |
| 1 | Nodo (specifico) | LLM per il singolo nodo: provider, modello, temperatura, max_tokens, system prompt | Livello massimo di specificità |

### 6.3 Requisiti Tecnici LLM

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-LLM-01 | API key management | API key cifrate a riposo, mai esposte nel frontend | P0 |
| REQ-LLM-02 | Streaming risposta | Risposte streamed token per token nel pannello output | P0 |
| REQ-LLM-03 | Context window | Gestione automatica: se l'input supera i limiti, tronca con notifica | P0 |
| REQ-LLM-04 | Cost tracking | Token input/output e costo stimato in USD per ogni esecuzione | P1 |
| REQ-LLM-05 | Cost limits | Limiti di spesa mensili per workspace e per utente | P2 |
| REQ-LLM-06 | Model fallback | Fallback su modello alternativo se il primario non è disponibile | P2 |
| REQ-LLM-07 | Prompt templating | Variabili dinamiche: `{input}`, `{context}`, `{node_name}`, `{canvas_name}` | P0 |
| REQ-LLM-08 | System prompt globale | System prompt globale iniettato in tutte le chiamate a livello workspace | P2 |
| REQ-LLM-09 | Output parser | Validazione JSON strutturato; errore specifico se JSON malformato | P1 |

---

## 7. Template Predefiniti e Canvas

### 7.1 Template Disponibili al Lancio

| Template | Descrizione | Nodi inclusi | Utente target |
|---|---|---|---|
| Blank Canvas | Canvas vuoto senza nodi pre-caricati | Nessuno | Facilitatori senior |
| Catena Principale Upventure | Flusso completo PD→MR→BS→GTM→PITCH con nodi collegati in sequenza | NODE-PD-01, MR-01, BS-01, GTM-01, PITCH-01 | Utenti nuovi, clienti |
| Lean Startup | Validazione ipotesi, MVP, metriche di trazione (Build-Measure-Learn) | Input + PD-01 + BS-01 (steps 2-4) + Output | Startup, founder |
| Design Thinking | 5 fasi: empatia, definizione, ideazione, prototipo, test | Input ×2 + PD-01 + BS-01 (step 1-2) + Output | Innovation manager |
| Jobs to Be Done | Analisi del job statement, gains/pains, value proposition | Input + PD-01 + custom JTBD node + GTM-01 | Consulenti, PM |
| Business Model Canvas | 9 blocchi BMC con nodo LLM per ciascuno | 9× custom nodes + Output | Corporate, acceleratori |
| Innovation Sprint | Template compresso per sessioni intensive 1-2 giorni | Input + PD-01 light + BS-01 (step 1) + Output | Workshop, eventi |

### 7.2 Requisiti Template

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-T-01 | Selezione template | Dialog di selezione con preview visiva al momento della creazione | P0 |
| REQ-T-02 | Blank canvas | Opzione canvas vuoto sempre disponibile | P0 |
| REQ-T-03 | Template proprietari | I workspace possono salvare template custom dal canvas corrente | P1 |
| REQ-T-04 | Export template | Export `.json` e re-import in altro workspace | P2 |
| REQ-T-05 | Template pubblici | Template Upventure visibili a tutti; custom solo al workspace owner | P1 |
| REQ-T-06 | Anteprima template | Preview con nodi inclusi, flusso, stima tempi e output attesi | P1 |

---

## 8. Persistenza e Versioning

### 8.1 Cloud Storage e Auto-save

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-P-01 | Auto-save | Ogni modifica salvata automaticamente entro 2 secondi | P0 |
| REQ-P-02 | Indicatore stato | Header mostra: "Saved" / "Saving..." / "Error" | P0 |
| REQ-P-03 | Salvataggio output LLM | Output di ogni esecuzione persistito anche dopo refresh | P0 |
| REQ-P-04 | Multi-tab safety | Modifiche in una tab propagate alle altre con notifica | P1 |
| REQ-P-05 | Offline warning | Banner avviso e coda modifiche al ripristino connessione | P1 |

### 8.2 Versioning Completo

| Funzionalità | Specifica |
|---|---|
| Snapshot automatici | Ogni modifica significativa crea uno snapshot con: timestamp ISO, autore, tipo di modifica, hash del contenuto |
| Snapshot manuali | L'utente crea snapshot manuali con nome personalizzato (es. "Post-workshop 12 marzo") |
| History panel | Lista cronologica snapshot con: data, autore, tipo evento, nome (se manuale) |
| Visualizzazione versione | Click su snapshot → canvas in stato di quella versione in read-only |
| Rollback | "Restore this version" con conferma; auto-snapshot della versione corrente prima del rollback |
| Diff visualizer | Confronto visivo tra due versioni: aggiunti (verde), rimossi (rosso), modificati (giallo) |
| Tracking multi-utente | Modifiche di utenti diversi tracciate separatamente con colori e label per autore |
| Retention | Snapshot automatici: 90 giorni · Snapshot manuali: nessuna scadenza |

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-V-01 | Snapshot automatici | Ogni 30s o su ogni evento discreto (add/remove/run) | P0 |
| REQ-V-02 | Snapshot manuali | Funzione "Save checkpoint" con campo nome opzionale | P0 |
| REQ-V-03 | History panel | Lista cronologica con filtri per autore e tipo evento | P0 |
| REQ-V-04 | Rollback | Restore a versione precedente con confirm e auto-snapshot preventivo | P0 |
| REQ-V-05 | Diff visualizer | Confronto visivo tra due versioni selezionate | P2 |
| REQ-V-06 | Named versions | Snapshot manuali con nome custom | P1 |
| REQ-V-07 | Author tracking | Modifiche tracciate per autore in ambienti collaborativi | P1 |

---

## 9. Nodi di Output e Formati di Esportazione

### 9.1 Funzionamento dei Nodi Output

I nodi Output raccolgono gli output dei nodi collegati, li assemblano secondo un template e li convertono in un documento finale. Non effettuano chiamate AI: eseguono solo aggregazione e formattazione.

**Flusso:** Nodi LLM upstream → output testuali → Nodo Output raccoglie e assembla → documento formattato → download/upload

### 9.2 Formati Supportati

| Formato | Descrizione | Template | Integrazione |
|---|---|---|---|
| Google Docs | Documento formattato con titoli, sezioni, tabelle. Pronto per editing collaborativo | Template one-pager, report strutturato | Google Drive API – upload diretto |
| PDF | Documento statico per stampa e distribuzione formale | Template branded Upventure, layout a colonne | Generazione server-side via headless Chrome |
| Slides (PPTX) | Presentazione per pitch e workshop | Template pitch 10 slide, template workshop | Export PPTX + upload Google Slides |
| Markdown | Testo strutturato per Notion, Obsidian, GitHub | Documento piatto con heading H1/H2/H3 | Copy-to-clipboard + download `.md` |
| JSON strutturato | Output machine-readable per API integration | Schema fisso per tipo di nodo | Download `.json` + webhook opzionale |

### 9.3 Requisiti Nodi Output

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-OUT-01 | Selezione formato | Dropdown selezione formato prima dell'esecuzione | P0 |
| REQ-OUT-02 | Preview documento | Anteprima nel browser prima del download | P1 |
| REQ-OUT-03 | Download diretto | Documento scaricabile con un click in formato nativo | P0 |
| REQ-OUT-04 | Google Drive upload | Upload diretto su Google Drive per Docs/Slides | P1 |
| REQ-OUT-05 | Re-generazione | Ri-generazione dopo modifica input senza ri-eseguire nodi LLM | P1 |
| REQ-OUT-06 | Naming automatico | Naming automatico: canvas name + data + tipo; rinominabile | P1 |
| REQ-OUT-07 | Multi-formato | Generazione multipla di formati contemporaneamente | P2 |
| REQ-OUT-08 | Template custom | Upload template branded (logo, colori) per PDF e PPTX | P2 |

---

## 10. Requisiti Non Funzionali

### 10.1 Performance

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-NF-01 | Latenza canvas load | ≤ 3 secondi per canvas con fino a 50 nodi (broadband) | P0 |
| REQ-NF-02 | Pan/zoom fluido | 60fps su hardware standard (MacBook Pro, Chrome) | P0 |
| REQ-NF-03 | Auto-save latency | Completamento salvataggio ≤ 2 secondi dalla modifica | P0 |
| REQ-NF-04 | LLM response start | Primo token streaming entro 5 secondi dall'invio | P0 |
| REQ-NF-05 | Canvas scalabilità | Usabile senza degradazione visiva fino a 100 nodi attivi | P1 |

### 10.2 Sicurezza

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-SEC-01 | Autenticazione | Login email/password + OAuth Google · 2FA opzionale per enterprise | P0 |
| REQ-SEC-02 | Autorizzazione workspace | Dati di un workspace mai accessibili ad altri workspace | P0 |
| REQ-SEC-03 | API key encryption | AES-256 a riposo · TLS 1.3 in transito | P0 |
| REQ-SEC-04 | Audit log | Tutte le azioni significative loggate con timestamp e utente | P1 |
| REQ-SEC-05 | GDPR compliance | Conformità GDPR · Data residency EU configurabile per enterprise | P1 |
| REQ-SEC-06 | Content isolation | Output LLM non usati per addestrare modelli di terze parti | P0 |

### 10.3 Accessibilità e Usabilità

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-UX-01 | Onboarding guidato | Wizard onboarding: primo canvas in ≤ 5 min | P1 |
| REQ-UX-02 | Responsive | Desktop (1440px+) e tablet (768px+) · Mobile fuori scope v1 | P1 |
| REQ-UX-03 | Dark mode | Supporto dark mode con rilevamento preferenze sistema | P3 |
| REQ-UX-04 | Localizzazione | Interfaccia in italiano (default) e inglese | P2 |
| REQ-UX-05 | Help inline | Tooltip contestuali e documentazione inline per ogni nodo | P2 |

### 10.4 Affidabilità

| REQ-ID | Titolo | Descrizione | Priorità |
|---|---|---|---|
| REQ-REL-01 | Uptime | SLA ≥ 99.5% su base mensile (downtime max ~3.6h/mese) | P0 |
| REQ-REL-02 | Backup | Backup automatico ogni 24h · Retention 30 giorni | P0 |
| REQ-REL-03 | Graceful degradation | Se LLM provider irraggiungibile → errore chiaro + suggerimento alternativo | P1 |
| REQ-REL-04 | No data loss | Modifiche non salvate recuperate al ricaricamento dopo crash | P0 |

---

## 11. Note Architetturali

> Questa sezione raccoglie vincoli e indicazioni per il team di sviluppo. Non costituisce una specifica tecnica completa.

### 11.1 Componenti Principali

| Componente | Responsabilità | Note tecniche |
|---|---|---|
| Canvas Engine | Rendering e interazione del canvas infinito, gestione nodi e link | React Flow o Rete.js consigliati; performance critica su 100+ nodi |
| Node Executor | Gestione esecuzione nodi: coda, retry, timeout, streaming | Queue asincrona; WebSocket per streaming; stato in-memory + persistenza |
| LLM Gateway | Astrazione sui provider LLM (Anthropic/OpenAI/Google) | Interfaccia unificata; rate limiting; cost tracking; API key vault |
| Versioning Engine | Snapshot automatici e manuali, diff, rollback | Event sourcing pattern; storage efficiente (delta, non full-copy) |
| Document Generator | Generazione output nei formati supportati | Google Docs API · pptxgenjs · PDF via headless Chrome |
| Library Service | CRUD nodi, gestione visibilità pub/priv, import/export JSON | Schema validation JSON nodi; approvazione flow per libreria pubblica |
| Auth & Workspace | Autenticazione, gestione workspace, permessi | JWT + refresh token; RBAC a livello workspace; OAuth Google |
| Real-time Sync | Sincronizzazione canvas multi-utente | CRDT (Yjs) o OT per conflict resolution; WebSocket per presenza |

### 11.2 Data Model – Entità Principali

| Entità | Campi chiave |
|---|---|
| Workspace | `id`, `name`, `plan`, `members[]`, `api_keys{}`, `llm_default_config`, `created_at` |
| Canvas | `id`, `workspace_id`, `name`, `template_id`, `nodes[]`, `edges[]`, `version_history[]`, `created_at`, `updated_at` |
| Node | `id`, `canvas_id`, `type` (llm\|input\|output\|custom), `position{x,y}`, `size{w,h}`, `config{prompt,llm,temperature,max_tokens}`, `input_content`, `output_content`, `state` (idle\|running\|done\|error), `created_at` |
| Edge | `id`, `canvas_id`, `source_node_id`, `target_node_id`, `mode` (auto\|manual\|visual), `created_at` |
| NodeDefinition | `id`, `name`, `description`, `category`, `type`, `default_prompt`, `default_llm`, `input_schema`, `output_schema`, `visibility` (public\|private), `workspace_id`, `version`, `tags[]` |
| CanvasVersion | `id`, `canvas_id`, `snapshot_data` (JSON), `author_id`, `event_type`, `name` (opzionale), `created_at` |
| Execution | `id`, `node_id`, `canvas_id`, `llm_provider`, `model`, `tokens_in`, `tokens_out`, `cost_usd`, `duration_ms`, `state`, `error_message`, `created_at` |

---

## 12. Roadmap e Milestones

### 12.1 Release Plan

#### M1 — Alpha Interna (0-8 settimane)

**Funzionalità:**
- Canvas infinito con pan/zoom
- Nodi: LLM, Input, Output
- Sistema di link con modalità auto/manuale
- Esecuzione manuale e a cascata
- 5 nodi base (NODE-PD-01…PITCH-01) hardcoded
- Streaming LLM (Claude, GPT-4)
- Auto-save cloud
- Autenticazione base

**REQ inclusi:** Tutti i P0 di tutte le sezioni

---

#### M2 — Beta Chiusa (9-16 settimane)

**Funzionalità:**
- Versioning completo con rollback
- Libreria nodi (pubblica + privata)
- Editor nodo custom
- Import/export JSON nodi
- 3 template predefiniti
- Nodi Output: PDF, PPTX, Markdown
- Wizard gate per nodi BS-01, GTM-01, PITCH-01
- Log esecuzione e cost tracking

**REQ inclusi:** Tutti i P1 di tutte le sezioni

---

#### M3 — v1.0 GA (17-24 settimane)

**Funzionalità:**
- Google Docs/Drive integration
- Tutti i 7 template
- Gestione workspace completa (RBAC)
- Diff visualizer versioni
- Onboarding wizard
- Minimap e grouping nodi
- Libreria pubblica con approvazione
- GDPR compliance + audit log

**REQ inclusi:** P1-P2 rimanenti

---

#### M4 — v1.x Future (Post GA)

**Funzionalità:**
- Mobile/tablet optimization
- Dark mode
- Real-time multi-utente (CRDT)
- Template marketplace
- Webhook e integrazioni esterne
- Cost limits e billing avanzato
- Nuovi nodi (validazione ipotesi, financial modeling, due diligence)

**REQ inclusi:** P2-P3

---

### 12.2 Priorità Requisiti – Riepilogo

| Priorità | Release target | Criteri |
|---|---|---|
| **P0** | M1 – Alpha | Bloccante per il lancio. Il prodotto non è utilizzabile senza questa funzionalità |
| **P1** | M2-M3 – Beta/GA | Importante per adozione e value proposition. Da includere entro la GA |
| **P2** | M3-M4 | Enhancement significativo. Migliora l'esperienza ma non blocca il lancio |
| **P3** | M4+ | Nice to have. Backlog da valutare in base al feedback degli utenti |

---

## 13. Questioni Aperte e Decisioni da Prendere

| # | Questione | Dettaglio | Responsabile |
|---|---|---|---|
| Q-01 | Modello di pricing per clienti esterni | Il canvas sarà offerto come SaaS a pagamento? Freemium? Per-seat o per-workspace? | CEO / Product |
| Q-02 | Gestione API key clienti | I clienti usano le proprie API key LLM o Upventure mantiene un pool centralizzato? | CTO / CEO |
| Q-03 | Libreria pubblica – governance | Chi approva i nodi pubblici? Quale processo di review? Esiste un SLA di approvazione? | Product / Ops |
| Q-04 | Collaborazione real-time | La v1.0 supporta editing simultaneo da più utenti o solo editing sequenziale? | CTO |
| Q-05 | Integrazione Notion/Attio | I canvas e gli output devono essere sincronizzati con Notion o Attio (CRM)? | Product |
| Q-06 | Retaining del contesto LLM | I nodi wizard mantengono la conversazione in una singola sessione LLM o ogni step è una chiamata indipendente? | CTO |
| Q-07 | Canvas sharing con clienti | È necessario condividere un canvas in read-only con un cliente esterno senza account? | Product / Sales |
| Q-08 | Limiti di canali LLM | Ci sono restrizioni su quali LLM usare per dati con NDA o vincoli di confidenzialità? | Legal / CTO |

---

*© 2025 Upventure · Venture Building Canvas PRD v1.0 · Documento riservato ad uso interno*
