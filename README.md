# Predictive Deception: LLM-based Command Anticipation in SSH Honeypots

[cite_start]Questo repository ospita l'implementazione di riferimento per il framework di **Predictive Deception**, un'architettura di sicurezza offensiva che integra Large Language Models (LLM) e Retrieval-Augmented Generation (RAG) all'interno di honeypot SSH ad alta interazione[cite: 20, 22].

[cite_start]Il progetto supera il modello reattivo tradizionale (logging passivo), introducendo un agente difensivo proattivo in grado di anticipare i comandi dell'attaccante e manipolare l'ambiente in tempo reale [cite: 14-16].

---

## 📋 Abstract e Obiettivi

[cite_start]Gli honeypot SSH tradizionali (es. Cowrie) operano secondo un paradigma reattivo: registrano le azioni dell'attaccante solo *dopo* che queste sono state eseguite[cite: 14]. [cite_start]Sebbene efficace per la threat intelligence a posteriori, questo approccio limita le capacità di inganno (deception) in tempo reale[cite: 15].

Questo lavoro propone un cambio di paradigma: **l'anticipazione comportamentale**.
[cite_start]Sfruttando la capacità predittiva di modelli LLM (CodeLlama, Gemini) potenziati da una memoria storica vettoriale (RAG su ChromaDB)[cite: 163, 220], il sistema:
1.  [cite_start]**Analizza** lo stream di comandi della sessione corrente in tempo reale[cite: 159].
2.  [cite_start]**Predice** la sequenza di azioni successive più probabili ($Top\text{-}k$)[cite: 171].
3.  [cite_start]**Genera e inietta** nel filesystem artefatti ingannevoli (file, config, log) coerenti con l'attacco previsto, *prima* che l'attaccante li richieda[cite: 21].

---

## 🏗️ Architettura del Sistema

[cite_start]Il sistema è implementato in un ambiente virtualizzato gestito via **Vagrant** e configurato tramite **Ansible**. [cite_start]L'architettura si divide in tre moduli logici principali[cite: 156]:

### 1. FakeShell (Interazione)
[cite_start]Uno script Python che sostituisce la shell di default (`/bin/bash`)[cite: 548].
* [cite_start]**Funzione:** Fornisce un prompt realistico (`user@hostname:/path$`) ed esegue i comandi reali tramite PTY, mantenendo l'interattività completa[cite: 552, 557].
* [cite_start]**Logging:** Intercetta e serializza ogni keystroke e comando in un log strutturato JSON (`/var/log/fakeshell.json`), fungendo da input per il motore predittivo[cite: 559].

### 2. Predictive Engine & RAG (Analisi)
[cite_start]Il modulo predittivo basato su LLM[cite: 162].
* [cite_start]**Input:** Riceve la *sliding window* degli ultimi $k$ comandi eseguiti[cite: 161].
* [cite_start]**Retrieval (RAG):** Interroga un database vettoriale (**ChromaDB**) contenente sessioni di attacco reali (dataset Cowrie), recuperando pattern storici simili per ridurre le allucinazioni e aumentare la coerenza[cite: 218, 220].
* [cite_start]**Output:** Produce una lista di comandi predetti ordinati per probabilità ($Top\text{-}k$)[cite: 171].

### 3. Defender Runtime (Deception Attiva)
[cite_start]Un demone Python che monitora i log della FakeShell e agisce sul filesystem[cite: 591].
* [cite_start]**Generazione Proattiva:** Per ogni comando predetto, genera dinamicamente un artefatto "esca" (decoy) e una sua descrizione[cite: 596].
* [cite_start]**Materializzazione:** Crea fisicamente i file nei percorsi previsti (es. `/etc/passwd`, `/var/log/auth.log`)[cite: 658].
* [cite_start]**Branching & Pruning:** Quando l'attaccante esegue un comando, il sistema mantiene gli artefatti del ramo corretto ed elimina istantaneamente quelli generati per le predizioni errate, garantendo la coerenza temporale dell'ambiente[cite: 598, 682].

---

## 🛠️ Requisiti Tecnici

Il progetto richiede un ambiente Linux/Unix per l'orchestrazione.

**Core Dependencies:**
* **Python 3.10+**
* [cite_start]**Vagrant & VirtualBox** (per l'ambiente honeypot isolato) [cite: 542]
* [cite_start]**Ansible** (per il provisioning automatizzato) 

[cite_start]**Librerie Python (Backend & ML)[cite: 753]:**
* [cite_start]`chromadb` - Database vettoriale per RAG.
* [cite_start]`sentence-transformers` - Generazione embeddings (`all-MiniLM-L6-v2`).
* [cite_start]`google-genai` - Interfaccia API per Gemini[cite: 627].
* [cite_start]`requests` - Interfaccia per modelli locali via Ollama[cite: 271].
* `scikit-learn`, `numpy`, `pandas` - Preprocessing e analisi dati.

---

## 📂 Struttura del Repository

[cite_start]L'organizzazione del codice segue una logica modulare per separare il provisioning infrastrutturale, il motore di inferenza e la gestione dei dati [cite: 717-760].

```bash
Predictive_deception/
│
├── chroma_storage/             # Database vettoriale persistente (ChromaDB)
│   ├── chroma.sqlite3
│   └── DB_checkpoint.txt
│
├── Honeypot/                   # Infrastructure as Code (Vagrant + Ansible)
│   ├── Vagrantfile             # Definizione VM
│   ├── playbook.yml            # Configurazione ruoli Ansible
│   ├── roles/
│   │   ├── defender/           # Logica di deception (Runtime + LLM integration)
│   │   │   └── files/defender.py
│   │   ├── fakeshell/          # Shell emulata con logging JSON
│   │   │   └── files/fakeshell.py
│   │   └── db_vettoriale/      # Setup storage vettoriale
│
├── inspectDataset/             # Pipeline ETL (Extract, Transform, Load)
│   ├── download_zenodo.py      # Download dataset CyberLab/Cowrie
│   ├── analyze_and_clean.py    # Normalizzazione comandi e pulizia rumore
│   └── merge_cowrie_datasets.py# Aggregazione Train/Test split
│
├── prompting/                  # Modulo di Inferenza e Valutazione
│   ├── core_rag.py             # Logica RAG e context retrieval
│   ├── core_topk.py            # Logica di prompting standard
│   ├── evaluate_gemini_rag.py  # Benchmark Gemini + RAG
│   ├── evaluate_ollama_rag.py  # Benchmark CodeLlama + RAG
│   └── utils.py                # Funzioni di utilità condivise
│
└── requirements.txt
