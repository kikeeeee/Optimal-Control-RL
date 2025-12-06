# Predictive Deception: LLM-based Command Anticipation in SSH Honeypots

![Project Status](https://img.shields.io/badge/Status-Research_Prototype-blue)
![Python](https://img.shields.io/badge/Python-3.10+-yellow)
![Tech](https://img.shields.io/badge/Technology-LLM%20%7C%20RAG%20%7C%20Ansible-green)

Questo repository contiene l'implementazione ufficiale del framework **Predictive Deception**, un'architettura di sicurezza offensiva sviluppata nell'ambito del corso di Ingegneria Informatica dell'Università di Bologna.

Il progetto introduce un cambio di paradigma nella gestione degli honeypot SSH: dal logging passivo (reattivo) all'anticipazione comportamentale (proattiva), sfruttando **Large Language Models (LLM)** e **Retrieval-Augmented Generation (RAG)**.

---

## 📑 Indice

1. [Abstract](#-abstract)
2. [Architettura del Sistema](#-architettura-del-sistema)
3. [Componenti Core](#-componenti-core)
4. [Struttura del Repository](#-struttura-del-repository)
5. [Setup e Installazione](#-setup-e-installazione)
6. [Workflow Operativo](#-workflow-operativo)
7. [Risultati Sperimentali](#-risultati-sperimentali)
8. [Autori e Riferimenti](#-autori-e-riferimenti)

---

## 🔬 Abstract

Gli honeypot tradizionali (es. Cowrie) raccolgono intelligence registrando le azioni degli attaccanti *post-factum*. Questo approccio limita le capacità di inganno (deception) in tempo reale.
**Predictive Deception** supera questo limite implementando un ciclo OODA (Observe-Orient-Decide-Act) automatizzato:
1.  **Observe:** Intercetta lo stream di comandi in tempo reale.
2.  **Orient:** Recupera contesti storici simili da un database vettoriale (RAG).
3.  **Decide:** Predice la sequenza di prossimi comandi ($Top\text{-}k$) tramite LLM.
4.  **Act:** Genera e materializza artefatti "esca" (file, log, config) nel filesystem prima che l'attaccante li richieda.

---

## 🏗️ Architettura del Sistema

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
