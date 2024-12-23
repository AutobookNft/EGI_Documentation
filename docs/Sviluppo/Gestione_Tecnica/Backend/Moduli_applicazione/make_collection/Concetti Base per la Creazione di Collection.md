# Concetti base make collection

## Struttura del database
Ogni utente creator può avere infiniti team, un team può avere infiniti utenti associati. Quest'ultima affermazione è vera solamente da un punto di vista logico, ma non da un punto di vista funzionale, infatti i membri di un team non possono essere più di dodici, compresi quelli default.

Per realizzare la relazione molti a molti necessaria, viene utilizzata la tabella team_user. 
### Gestione degli wallet

La tabella team_user contiene anche i dati necessari alla gestione degli wallet.


### Implementazione della Gestione dei Team con Jetstream

Nel progetto è stato integrato il modulo Jetstream per sfruttare la gestione avanzata dei team. Questa implementazione fornisce una struttura dati robusta e scalabile, fondamentale per l'organizzazione delle collezioni e degli EGI (Eco Gate Invest).

#### Struttura delle Tabelle

- **Tabella `teams`:**
  La tabella `teams` rappresenta la struttura dati di base e contiene i dati delle collezioni. Ogni team corrisponde a una collezione gestita sulla piattaforma.

- **Tabella `team_users`:**
  Questa tabella pivot associa gli utenti ai team, gestendo l'appartenenza degli utenti ai vari team. È fondamentale per definire quali utenti hanno accesso a specifiche collezioni.

- **Tabella `team_items`:**
  La tabella `team_items` contiene i dati relativi agli EGI all'interno di ciascuna collezione. Ogni record rappresenta un singolo EGI, inclusi tutti i suoi attributi e metadata.

#### Vantaggi dell'Implementazione con Jetstream

1. **Scaffolding Avanzato:**
   L'uso di Jetstream permette di sfruttare uno scaffolding avanzato, eliminando la necessità di riscrivere molteplici funzionalità di base. Questo consente di focalizzarsi sull'aggiunta dei campi specifici necessari e sulla modifica dei vari CRUD (Create, Read, Update, Delete) per adattarli alle esigenze del progetto.

2. **Gestione delle Collezioni:**
   Quando un creator (utente) effettua il login, ha accesso immediato alla collezione di default o all'ultima collezione a cui si è connesso. Questo è gestito attraverso il campo `current_team` presente nella tabella `users` di Jetstream, che memorizza l'ultimo team selezionato dall'utente autenticato.

#### Processo di Accesso degli Utenti

- **Login e Selezione del Team:**
  Al momento del login, l'utente viene automaticamente connesso alla collezione indicata nel campo `current_team`. Questo permette una navigazione fluida e immediata all'interno delle collezioni gestite dall'utente.

- **Gestione Dinamica delle Collezioni:**
  Gli utenti possono passare facilmente da una collezione all'altra, con la garanzia che il contesto della collezione attualmente selezionata viene mantenuto grazie all'architettura di Jetstream.

#### Conclusione

L'integrazione di Jetstream con la gestione dei team ha permesso di creare una struttura dati efficiente e scalabile per la gestione delle collezioni e degli EGI. Questo approccio non solo ottimizza il processo di sviluppo, ma garantisce anche una gestione user-friendly e dinamica delle collezioni per i creator, migliorando l'esperienza utente complessiva sulla piattaforma.