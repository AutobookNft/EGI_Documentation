# Ruoli e permessi v2

## 🔍 **Analisi dei Ruoli e Permessi**

### 1. **Ruoli Definiti**

1. **Superadmin**
   - **Ambito**: Piattaforma
   - **Permessi**:
     - **CRUD globale** (su tutto tranne che sui wallet).

2. **Creator**
   - **Ambito**: Team
     - **Permessi**:
       - CRUD sui team.
       - Aggiunta e rimozione di membri.
       - Modifica ruoli e permessi (inclusi gli admin).
   - **Ambito**: Collection
     - **Permessi**:
       - CRUD sulle collection.
   - **Ambito**: EGI
     - **Permessi**:
       - CRUD sugli EGI.

3. **Admin**
   - **Ambito**: Team
     - **Permessi**:
       - Aggiungere e rimuovere membri (con sola lettura per altri admin).
       - Modificare i ruoli dei membri (con sola lettura per altri admin).
   - **Ambito**: Collection
     - **Permessi**:
       - Update e Read sulle collection.
   - **Ambito**: EGI
     - **Permessi**:
       - CRUD sugli EGI.

4. **Editor**
   - **Ambito**: Collection
     - **Permessi**:
       - Read sulle collection.
   - **Ambito**: EGI
     - **Permessi**:
       - Update e Read sugli EGI.

5. **Guest**
   - **Ambito**: Collection
     - **Permessi**:
       - Read sui dati di testata delle collection.
   - **Ambito**: EGI
     - **Permessi**:
       - Read sugli EGI.

---

### 📌 **Considerazioni sui Ruoli**

1. **Chiarezza Gerarchica**:  
   La gerarchia dei ruoli è chiara e segue una progressione logica, con:
   - **Superadmin** al vertice della piattaforma.
   - **Creator** come figura principale per la gestione dei team e delle collection.
   - **Admin** con compiti più limitati rispetto ai Creator.
   - **Editor** e **Guest** con permessi progressivamente più ristretti.

2. **Sovrapposizione tra Creator e Admin**:  
   - **Creator** può gestire i ruoli e i permessi degli Admin, mentre gli Admin possono solo leggerli. Questo mantiene un buon equilibrio di potere.

3. **Wallet**:  
   - La gestione dei wallet è ben delineata con una **policy** che consente solo al proprietario di modificarli, mantenendo sicurezza e controllo.

---

## 📝 **Riepilogo Ruoli e Permessi**

| **Ruolo**      | **Ambito**       | **Permessi**                                                                                       |
|----------------|------------------|----------------------------------------------------------------------------------------------------|
| **Superadmin** | Piattaforma      | CRUD su tutto tranne i wallet                                                                      |
| **Creator**    | Team             | CRUD sui team, gestione membri e ruoli                                                             |
|                | Collection       | CRUD sulle collection                                                                               |
|                | EGI              | CRUD sugli EGI                                                                                      |
| **Admin**      | Team             | Aggiungere/rimuovere membri (R sugli admin), modificare ruoli (R sugli admin)                     |
|                | Collection       | UR sulle collection                                                                                 |
|                | EGI              | CRUD sugli EGI                                                                                      |
| **Editor**     | Collection       | R sulle collection                                                                                  |
|                | EGI              | RU sugli EGI                                                                                        |
| **Guest**      | Collection       | R sui dati di testata delle collection                                                              |
|                | EGI              | R sugli EGI                                                                                         |

---

## 🔄 **Gestione dei Wallet**

1. **Solo il Proprietario del Wallet** può modificare il proprio wallet.
2. **Team ID e User ID** vengono associati al wallet per applicare correttamente le policy.
3. **Modifiche ai Wallet** richiedono approvazione del **Creator** del team, anche per modifiche effettuate dal Creator stesso.

---

## 📋 **Sequenze Chiave**

### 1. **Creazione di un Nuovo Utente**

1. Assegnare il ruolo di **Creator**.
2. Creare un **team personale** e renderlo il **current team**.
3. Registrare il nuovo utente come **Admin** del team.
4. Creare una **collection** associata al team.
5. Generare i tre **wallet di default**:
   - Natan (ID: 1)
   - EPP (ID: 2)
   - Creator (ID generato).

### 2. **Invito come Membro del Team**

1. Invio della richiesta con un ruolo via email da parte di **Creator** o **Admin**.
2. Accettazione dell’invito:
   - Se **non iscritto**: Registrazione e aggiunta automatica al team con il ruolo assegnato.
   - Se **già iscritto**: Accettazione dell’invito e aggiunta al team.
3. **Dashboard del Creator** mostra inviti non accettati per una gestione manuale.

### 3. **Gestione dei Wallet**

1. **Creazione del Wallet**:
   - Ogni membro può creare il proprio wallet.
   - Richiede approvazione del **Creator**.
2. **Modifiche alle Royalties**:
   - Richiedono approvazione del **Creator**.
3. **Dashboard del Creator** mostra le modifiche ai wallet in attesa di approvazione.

---

## 🚦 **Feedback e Prossimi Passi**

Questa struttura è chiara e ben organizzata. Vuoi approfondire qualche punto specifico o iniziare a definire l’implementazione pratica con **Spatie**?