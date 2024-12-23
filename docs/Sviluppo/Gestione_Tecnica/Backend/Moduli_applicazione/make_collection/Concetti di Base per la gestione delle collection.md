---
title: Concetti di Base per la Gestione delle Collection
tags:
  - makecollection
  - "#flussi"
  - "#wallets"
created: 2024-12-22
updated: 2024-12-22
---

## Scopo
Illustrare il processo per rendere attiva una collection e gestire la relazione tra collection, team e wallet.

---

## Contenuti

### 1. Concetti di Base
- Generazione automatica delle collection durante la registrazione di un nuovo utente.
- Creazione manuale di una nuova collection da parte di un utente.
- Relazione tra collection, team e gli [[Wallets]].

---

### 2. Flussi
#### Flusso 1: Registrazione di un Nuovo Utente
1. Quando un utente viene registrato:
   - Si crea automaticamente un **team**.
   - L'utente viene aggiunto come membro del [[Team Corrente]] con il [[ruolo]] di [[Creator]].
   - Viene creata una **collection** collegata al team appena generato.
2. Il team creato diventa il team corrente.

#### Flusso 2: Cambio del Team Corrente
- L'utente può cambiare il [[Team Corrente]] tramite la voce del menu **Switch Team**.

#### Flusso 3: Apertura di una Collection
- Dalla dashboard, l'utente può aprire le collection usando il menu **Open Collection**:
   - **Se c'è una sola collection attiva**: si apre la vista [[Collection Manager]].
   - **Se ci sono più collection**: si apre il [[Carousel Collection]] per selezionare la collection.

#### Flusso 4: Creazione di una Nuova Collection
- L'utente può creare una nuova collection dalla dashboard usando il menu **Create Collection**:
   - La nuova collection viene automaticamente associata al [[Team Corrente]].

---

### 3. Relazioni
- Una **collection** appartiene sempre a un [[team]]:
  - Ogni team ha come minimo 3 membri:
    - **Creator**: colui che ha creato la collection.
    - **Natan**: superadmin (id:1).
    - **EPP**: wallet del progetto ecologico (id:2).
  - La relazione tra **users** e **teams** è molti-a-molti, gestita tramite la tabella pivot **team_user**.
  - La tabella pivot **team_user** include:
    - Informazioni sui ruoli degli utenti nel team.
    - Gli [[Wallets]] associati.

---

### 4. Glossario
| **Termine**              | **Definizione**                                                 |
|---------------------------|-----------------------------------------------------------------|
| **Team Corrente**         | Il team attualmente selezionato dall'utente per operazioni.    |
| **Collection Manager**    | La vista per gestire i dettagli di una specifica collection.   |
| **Carousel Collection**   | Una vista che permette di selezionare una collection tra molte.|
| **Wallet**                | Un indirizzo Algorand associato al team o agli utenti.         |
| **Creator**               | Utente che ha creato la collection e il team.                 |

---

## Link Correlati
- [[team]]
- [[Wallets]]
- [Osservazioni e Decisioni Accesso alle Collection e Gestione Wallet](Osservazioni%20e%20Decisioni%20Accesso%20alle%20Collection%20e%20Gestione%20Wallet.md)
- [[Collection Manager]]
- [[Create Collection]]
