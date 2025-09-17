<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Trascrizione Tradotta: Controllo Ottimale e Reinforcement Learning - Ottimizzazione Nonlineare

**Corso del Prof. Giuseppe Notarstefano**
Dipartimento di Ingegneria dell'Energia Elettrica e dell'Informazione
Alma Mater Studiorum Università di Bologna
*Con un ringraziamento speciale a L. Sforni per il supporto nella preparazione delle slide*

## 1. Introduzione all'Ottimizzazione Non Vincolata

### Formulazione del Problema Base

L'ottimizzazione non vincolata si occupa di risolvere problemi della forma:

\$ \min_{z \in \mathbb{R}^d} \ell(z) \$

dove:

- \$ \ell: \mathbb{R}^d \rightarrow \mathbb{R} \$ è la **funzione di costo** da minimizzare
- \$ z \$ è il **vettore delle variabili decisionali**

Questo è il problema fondamentale dell'ottimizzazione matematica, dove si cerca il punto che rende minima una funzione obiettivo senza alcun vincolo esplicito sulle variabili.

### Tipologie di Minimi

Un punto \$ z^* \$ può essere classificato come diversi tipi di minimo a seconda della sua natura:

**Minimo Globale**: Il punto \$ z^* \$ è un minimo globale se \$ \ell(z^*) \leq \ell(z) \$ per tutti i punti \$ z \in \mathbb{R}^d \$. Questo rappresenta la migliore soluzione possibile in tutto lo spazio.

**Minimo Globale Stretto**: Si verifica quando \$ \ell(z^*) < \ell(z) \$ per tutti i punti \$ z \neq z^* \$, garantendo l'unicità della soluzione ottimale.

**Minimo Locale**: Esiste un intorno \$ \epsilon > 0 \$ tale che \$ \ell(z^*) \leq \ell(z) \$ per tutti i punti nella sfera \$ B(z^*, \epsilon) = \{z \in \mathbb{R}^d \mid \|z - z^*\| < \epsilon\} \$.

**Minimo Locale Stretto**: Come sopra, ma con disuguaglianza stretta per \$ z \neq z^* \$.

È importante notare che ogni minimo globale è anche locale, ma non viceversa. I massimi possono essere definiti in modo equivalente e sono semplicemente minimi della funzione \$ -\ell \$.

## 2. Notazioni Matematiche Fondamentali

### Il Gradiente

Per una funzione scalare \$ r: \mathbb{R}^d \rightarrow \mathbb{R} \$, il **gradiente** è il vettore delle derivate parziali:

\$ \nabla r(z) = $$
\begin{bmatrix} \frac{\partial r(z)}{\partial z_1} \\ \vdots \\ \frac{\partial r(z)}{\partial z_d} \end{bmatrix}
$$ \in \mathbb{R}^{d \times 1} \$

Il gradiente indica la direzione di massima crescita della funzione e la sua norma rappresenta il tasso di crescita in quella direzione.

### La Matrice Hessiana

Per la stessa funzione, la **matrice Hessiana** contiene le derivate seconde:

\$ \nabla^2 r(z) = $$
\begin{bmatrix} \frac{\partial^2 r(z)}{\partial z_1^2} & \cdots & \frac{\partial^2 r(z)}{\partial z_1 z_d} \\ \vdots & \ddots & \vdots \\ \frac{\partial^2 r(z)}{\partial z_d z_1} & \cdots & \frac{\partial^2 r(z)}{\partial z_d^2} \end{bmatrix}
$$ \in \mathbb{R}^{d \times d} \$

L'Hessiana è simmetrica quando le derivate seconde sono continue, poiché l'ordine di derivazione non influisce sul risultato.

### Gradiente di Funzioni Vettoriali

Per una funzione vettoriale \$ r: \mathbb{R}^d \rightarrow \mathbb{R}^m \$, il gradiente è:

\$ \nabla r(z) = $$
\begin{bmatrix} \frac{\partial r_1(z)}{\partial z_1} & \cdots & \frac{\partial r_m(z)}{\partial z_1} \\ \vdots & \ddots & \vdots \\ \frac{\partial r_1(z)}{\partial z_d} & \cdots & \frac{\partial r_m(z)}{\partial z_d} \end{bmatrix}
$$ \in \mathbb{R}^{d \times m} \$

Questa è la trasposta della matrice Jacobiana di \$ r \$.

## 3. Condizioni di Ottimalità

### Condizioni Necessarie del Primo e Secondo Ordine

**Condizione Necessaria del Primo Ordine**: Se \$ z^* \$ è un minimo locale di una funzione \$ \ell \$ continuamente differenziabile, allora necessariamente:

\$ \nabla \ell(z^*) = 0 \$

Questa condizione identifica i **punti stazionari**, che includono minimi, massimi e punti di sella.

**Condizione Necessaria del Secondo Ordine**: Se inoltre \$ \ell \$ è due volte continuamente differenziabile, allora:

\$ \nabla^2 \ell(z^*) \geq 0 \$

cioè l'Hessiana deve essere semidefinita positiva.

### Condizione Sufficiente del Secondo Ordine

Se un punto \$ z^* \$ soddisfa entrambe le condizioni:

1. \$ \nabla \ell(z^*) = 0 \$
2. \$ \nabla^2 \ell(z^*) > 0 \$ (definita positiva)

Allora \$ z^* \$ è garantito essere un **minimo locale stretto**.

## 4. Insiemi e Funzioni Convesse

### Insiemi Convessi

Un insieme \$ Z \subset \mathbb{R}^d \$ è **convesso** se per qualsiasi coppia di punti \$ z_A, z_B \in Z \$ e per ogni \$ \theta \in  \$, si ha:[^1]

\$ \theta z_A + (1-\theta) z_B \in Z \$

In altre parole, il segmento che congiunge due punti qualsiasi dell'insieme è completamente contenuto nell'insieme stesso.

### Funzioni Convesse

Su un insieme convesso \$ Z \$, una funzione \$ \ell: Z \rightarrow \mathbb{R} \$ è **convessa** se:

\$ \ell(\theta z_A + (1-\theta) z_B) \leq \theta \ell(z_A) + (1-\theta) \ell(z_B) \$

Una funzione è **concava** se \$ -\ell \$ è convessa, mentre è **strettamente convessa** se la disuguaglianza è stretta per \$ z_A \neq z_B \$ e \$ \theta \in (0,1) \$.

### Vincoli e Convessità

**Vincoli di Disuguaglianza**: L'insieme definito da \$ Z_{ineq} = \{z \in \mathbb{R}^d \mid g(z) \leq 0\} \$ è convesso se e solo se \$ g \$ è quasi-convessa. In particolare, se \$ g \$ è convessa, allora \$ Z_{ineq} \$ è convesso.

**Vincoli di Uguaglianza**: L'insieme \$ Z_{eq} = \{z \in \mathbb{R}^d \mid h(z) = 0\} \$ è convesso se e solo se \$ h \$ è una funzione affine. Tali insiemi rappresentano spazi lineari o iperpiani.

## 5. Metodi del Gradiente

### Algoritmo Base

Il metodo del gradiente segue la forma generale:

\$ z_{k+1} = z_k - \gamma_k \nabla \ell(z_k) \$

dove \$ \gamma_k > 0 \$ è il **passo** all'iterazione \$ k \$. L'idea è di muoversi nella direzione opposta al gradiente, che è la direzione di massima decrescita locale.

### Regole per la Selezione del Passo

**Regola di Armijo (Backtracking Line-Search)**:

1. Inizializzare con \$ \bar{\gamma}_0 > 0 \$, \$ \beta \in (0,1) \$, \$ c \in (0,1) \$
2. Mentre \$ \ell(z_k + \bar{\gamma}_i d_k) \geq \ell(z_k) + c\bar{\gamma}_i \nabla \ell(z_k)^T d_k \$:
    - \$ \bar{\gamma}_{i+1} = \beta \bar{\gamma}_i \$
3. Impostare \$ \gamma_k = \bar{\gamma}_i \$

Valori tipici sono \$ \beta = 0.7 \$ e \$ c = 0.5 \$.

**Passo Costante**: \$ \gamma_k = \gamma > 0 \$ per tutte le iterazioni.

**Passo Decrescente**: \$ \gamma_k \rightarrow 0 \$ con \$ \sum_{k=0}^{\infty} \gamma_k = \infty \$ per garantire progresso sostanziale. Una scelta tipica è \$ \gamma_k = \frac{1}{k^{\alpha}} \$ con \$ \frac{1}{2} < \alpha \leq 1 \$.

### Risultati di Convergenza

**Con Regola di Armijo**: Ogni punto limite della sequenza \$ \{z_k\} \$ è un punto stazionario, cioè soddisfa \$ \nabla \ell(\bar{z}) = 0 \$.

**Con Passo Costante o Decrescente**: Sotto l'assunzione di Lipschitz-continuità del gradiente \$ \|\nabla \ell(z) - \nabla \ell(y)\| \leq L\|z-y\| \$, ogni punto limite è stazionario.

### Osservazioni Importanti

- La convergenza garantisce solo il raggiungimento di punti stazionari, non necessariamente minimi globali
- Per problemi non convessi, si può dimostrare solo convergenza a punti stazionari
- Per problemi convessi, i punti stazionari sono automaticamente minimi globali
- L'esistenza di minimi può essere garantita assumendo che \$ \ell \$ sia **coerciva** (cioè \$ \lim_{\|z\| \rightarrow \infty} \ell(z) = \infty \$)


### Esempio: Programmi Quadratici

Per il problema quadratico:

\$ \min_{z \in \mathbb{R}^d} \frac{1}{2} z^T Q z + q^T z \$

con \$ Q = Q^T \geq 0 \$, il metodo del gradiente diventa:

\$ z_{k+1} = z_k - \gamma_k(Qz_k + q) \$

Questo caso particolare è particolarmente importante perché molti algoritmi di ottimizzazione si riducono localmente a problemi quadratici.

***

*Nota: Le slide presentate coprono i fondamenti teorici dell'ottimizzazione non vincolata, fornendo le basi matematiche necessarie per comprendere algoritmi più avanzati di controllo ottimale e reinforcement learning.*

<div style="text-align: center">⁂</div>

[^1]: OPTCON-RL_optimization_2025_09_16.pdf
