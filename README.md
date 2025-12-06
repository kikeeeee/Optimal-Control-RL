# Predictive Deception: LLM-based Command Anticipation in SSH Honeypots

![Project Status](https://img.shields.io/badge/Status-Research_Prototype-blue)
![Python](https://img.shields.io/badge/Python-3.10+-yellow)
![Tech](https://img.shields.io/badge/Technology-LLM%20%7C%20RAG%20%7C%20Ansible-green)

Questo repository contiene l'implementazione ufficiale del framework **Predictive Deception**, un'architettura di sicurezza offensiva sviluppata nell'ambito del corso di Ingegneria Informatica dell'Università di Bologna.

Il progetto introduce un cambio di paradigma nella gestione degli honeypot SSH: dal logging passivo (reattivo) all'anticipazione comportamentale (proattiva), sfruttando **Large Language Models (LLM)** e **Retrieval-Augmented Generation (RAG)**.

---

## 📑 Indice

1. [Abstract](#-abstract)
2. [Core Concept](#-core-concept
3. [Architettura del Sistema](#-architettura-del-sistema)
4. [Struttura del Repository](#-struttura-del-repository)
5. [Setup e Installazione](#-setup-e-installazione)
6. [Workflow Operativo](#-workflow-operativo)
7. [Autori e Riferimenti](#-autori-e-riferimenti)

---

##  Abstract

Gli honeypot tradizionali (es. Cowrie) raccolgono intelligence registrando le azioni degli attaccanti *post-factum*. Questo approccio limita le capacità di inganno (deception) in tempo reale.
**Predictive Deception** supera questo limite implementando un ciclo OODA (Observe-Orient-Decide-Act) automatizzato:
1.  **Observe:** Intercetta lo stream di comandi in tempo reale.
2.  **Orient:** Recupera contesti storici simili da un database vettoriale (RAG).
3.  **Decide:** Predice la sequenza di prossimi comandi ($Top\text{-}k$) tramite LLM.
4.  **Act:** Genera e materializza artefatti "esca" (file, log, config) nel filesystem prima che l'attaccante li richieda.
---

## Core Concept
Il cuore del progetto è la transizione da una difesa passiva a una **difesa proattiva e adattiva**.

Gli honeypot tradizionali (es. Cowrie) si limitano a registrare i comandi dopo che sono stati eseguiti e spesso presentano un ambiente statico facilmente identificabile. Il nostro sistema di **Predictive Deception** inverte questo approccio:

1.  **Anticipazione in Tempo Reale:** Un modulo predittivo basato su LLM analizza la sequenza di comandi dell'attaccante mentre la sessione è in corso.
2.  **Memoria Storica (RAG):** Utilizzando la *Retrieval-Augmented Generation*, il modello consulta un database vettoriale (ChromaDB) contenente migliaia di sessioni di attacco reali (dataset CyberLab Honeynet) per migliorare la precisione della predizione.
3.  **Generazione Dinamica di Artefatti:** Prima ancora che l'attaccante prema invio sul prossimo comando, il sistema "immagina" cosa potrebbe chiedere (es. un file di configurazione, una password, una directory specifica) e **crea l'artefatto ingannevole nel filesystem reale**.
4.  **Coerenza Temporale:** Se l'attaccante interagisce con l'artefatto, questo rimane persistente; se la predizione era errata o il percorso cambia, il sistema ripulisce le "false piste" per mantenere l'ambiente coerente e credibile.
---

##  Architettura del Sistema

Il sistema opera all'interno di un ambiente virtualizzato (Vagrant) isolato, orchestrato via Ansible.

### Diagramma Logico

```mermaid
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
