Predictive Deception: LLM-based Command Anticipation in SSH Honeypots

Questo repository contiene l'implementazione ufficiale del framework Predictive Deception, un'architettura di sicurezza offensiva sviluppata nell'ambito del corso di Ingegneria Informatica dell'Università di Bologna.

Il progetto introduce un cambio di paradigma nella gestione degli honeypot SSH: dal logging passivo (reattivo) all'anticipazione comportamentale (proattiva), sfruttando Large Language Models (LLM) e Retrieval-Augmented Generation (RAG).

📑 Indice

Abstract

Architettura del Sistema

Componenti Core

Struttura del Repository

Setup e Installazione

Workflow Operativo

Risultati Sperimentali

Autori e Riferimenti

🔬 Abstract

Gli honeypot tradizionali (es. Cowrie) raccolgono intelligence registrando le azioni degli attaccanti post-factum. Questo approccio limita le capacità di inganno (deception) in tempo reale.
Predictive Deception supera questo limite implementando un ciclo OODA (Observe-Orient-Decide-Act) automatizzato:

Observe: Intercetta lo stream di comandi in tempo reale.

Orient: Recupera contesti storici simili da un database vettoriale (RAG).

Decide: Predice la sequenza di prossimi comandi ($Top\text{-}k$) tramite LLM.

Act: Genera e materializza artefatti "esca" (file, log, config) nel filesystem prima che l'attaccante li richieda.

🏗️ Architettura del Sistema

Il sistema opera all'interno di un ambiente virtualizzato (Vagrant) isolato, orchestrato via Ansible.

Diagramma Logico

graph TD
    A[Attacker SSH Session] -->|Input Command| B(FakeShell)
    B -->|Log JSON| C{Defender Runtime}
    C -->|Query| D[RAG Module]
    D -->|Retrieval| E[(ChromaDB)]
    D -->|Context + History| F[LLM Inference]
    F -->|Prediction Top-k| C
    C -->|Generate Artifacts| G[Filesystem Injection]
    G -->|Interaction| A
    C -->|Pruning| G


Il Ciclo di Deception Adattiva

Il sistema implementa un meccanismo di Branching & Pruning:

Per ogni comando eseguito $C_t$, il modello predice $k$ possibili comandi futuri ($C_{t+1}^1, ..., C_{t+1}^k$).

Il Defender genera $k$ rami di deception (es. crea file falsi per ogni previsione).

Quando l'attaccante esegue effettivamente $C_{t+1}$, il sistema identifica il ramo corretto.

Pruning: Gli artefatti dei rami errati vengono eliminati istantaneamente per mantenere la coerenza ambientale.

🧩 Componenti Core

1. FakeShell (Honeypot/roles/fakeshell)

Script Python che emula un terminale /bin/bash ad alta interazione.

Funzionalità: Gestione PTY, supporto pipe/redirezioni, prompt dinamico (user@hostname:cwd$).

Output: Logging strutturato in /var/log/fakeshell.json contenente timestamp, IP, utente, CWD e comando raw.

2. Predictive Engine (prompting/)

Modulo responsabile dell'inferenza. Supporta due modalità:

CodeLlama (Locale): Via Ollama API. Ideale per ambienti air-gapped.

Gemini 1.5 Flash (Cloud): Via Google GenAI API. Prestazioni superiori in reasoning complesso.

RAG Integration: Utilizza sentence-transformers (all-MiniLM-L6-v2) per convertire le sessioni in embeddings e ricercarle su ChromaDB.

3. Defender Runtime (Honeypot/roles/defender)

Demone di controllo che agisce come "Dungeon Master".

Monitora il log della FakeShell in tailing (tail -f).

Gestisce la logica di materialize_defense_artifacts (scrittura file) e cleanup_other_branches (pulizia).

📂 Struttura del Repository

Predictive_deception/
├── chroma_storage/             # Storage persistente per il Vector DB (SQLite3)
│   └── chroma.sqlite3          # Contiene gli embeddings delle sessioni Cowrie
│
├── Honeypot/                   # Infrastructure as Code (IaC)
│   ├── Vagrantfile             # Definizione VM (Network, Risorse)
│   ├── playbook.yml            # Playbook Ansible per il provisioning
│   └── roles/
│       ├── defender/           # Logica Runtime e Deception
│       ├── fakeshell/          # Emulatore Shell e Logging
│       └── db_vettoriale/      # Setup dipendenze RAG sulla VM
│
├── inspectDataset/             # Pipeline ETL (Extract, Transform, Load)
│   ├── download_zenodo.py      # Downloader automatico dataset CyberLab
│   ├── analyze_and_clean.py    # Parsing, normalizzazione regex e pulizia
│   └── merge_cowrie_datasets.py# Aggregazione e split Train/Test
│
├── prompting/                  # Core Logico LLM + RAG
│   ├── core_rag.py             # Indicizzazione e Retrieval (ChromaDB)
│   ├── core_topk.py            # Logica di prompting standard (Zero-shot)
│   ├── evaluate_gemini_*.py    # Script di benchmark per Gemini
│   ├── evaluate_ollama_*.py    # Script di benchmark per CodeLlama
│   └── utils.py                # Funzioni di supporto (parsing output)
│
└── requirements.txt            # Dipendenze Python


🛠️ Setup e Installazione

Prerequisiti

Host: Linux/macOS (consigliato) o Windows WSL2.

Software: Python 3.10+, Vagrant, VirtualBox, Ansible.

Hardware: * Consigliata GPU NVIDIA per RAG veloce (se eseguito localmente).

Minimo 8GB RAM per la VM Honeypot.

1. Installazione Dipendenze

Creare un virtual environment e installare le librerie necessarie:

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt


2. Configurazione Variabili d'Ambiente

Creare un file .env nella root del progetto:

GOOGLE_API_KEY="la_tua_chiave_gemini"
OLLAMA_BASE_URL="http://localhost:11434" # Se si usa CodeLlama


🔄 Workflow Operativo

Il ciclo di vita del progetto si divide in 4 fasi distinte.

Fase 1: Acquisizione e Pulizia Dati

Scaricamento e normalizzazione dei log di attacco reali (dataset Cowrie da Zenodo).

# Scarica i log grezzi
python3 inspectDataset/download_zenodo.py --n 50

# Pulisce, normalizza i comandi e crea dataset TRAIN/TEST
python3 inspectDataset/merge_cowrie_datasets.py


Fase 2: Creazione Knowledge Base (RAG)

Indicizzazione vettoriale del dataset di training per abilitare il retrieval.

# Genera embeddings e popola ChromaDB
python3 prompting/core_rag.py \
  --index \
  --dataset output/cowrie_TRAIN.jsonl \
  --persist-dir chroma_storage/


Fase 3: Valutazione Modelli (Benchmark)

Prima del deployment, valutare l'accuratezza predittiva.

# Esempio: Valutazione CodeLlama con RAG su dataset di test
python3 prompting/evaluate_ollama_rag.py \
  --sessions output/cowrie_TEST.jsonl \
  --index-file output/cowrie_TRAIN.jsonl \
  --k 5 --rag-k 3


Fase 4: Deployment Honeypot

Avvio dell'ambiente di produzione simulato.

cd Honeypot

# 1. Provisioning della VM (richiede ~10 min al primo avvio)
vagrant up --provision

# 2. Accesso (Simulazione Attaccante)
ssh -p 2222 user@127.0.0.1
