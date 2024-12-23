---
title: Wallets
tags:
  - wallets
  - makecollection
created: 2024-12-22
updated: 2024-12-22
---

## Scopo

Definire cosa sono e il funzionamento degli wallet

## Contenuti

### 1. Concetti di Base
- gli wallet sono l'entità che mette in relazione la collection alla blockchain ([[Algorand]])
- gli wallet sono gestiti all'interno della tabella team_user, la tabella pivot tra users e teams
- la tabella permette di scrivere l'address del wallet, la percentuale per il mint e per il rebind (mercato secondario)
- ci sono tre wallet che vengono aggiunti di default al momento della creazione del team:
	- quello del Creator
	- quello di Natan
	- quello di un EPP
- la modifica a un wallet è soggetta ad [[Natan/docs/Sviluppo/Gestione_Tecnica/Backend/Moduli_applicazione/make_collection/gestione_team/gestione_wallets/gestione_wallet|approvazione]] dal suo proprietario 

### 2. Flussi
- vedi [Gestione wallet](Natan/docs/Sviluppo/Gestione_Tecnica/Backend/Moduli_applicazione/make_collection/gestione_team/gestione_wallets/gestione_wallet)
- 

### 3. Relazioni
- [Indica come il documento si collega ad altri elementi o sistemi]

## Glossario

| **Termine** | **Definizione**                       |
| ----------- | ------------------------------------- |
| Rebind      | La transazione nel mercato secondario |

Sviluppo.
- [ ] [2024-12-22](2024-12-22.md)