# Collaborazione fra creator

La logica che hai descritto permette di ottenere una collaborazione fluida tra due **Creator** come **Alberto** e **Franco** per la gestione condivisa di una **Collection**. Ecco un'analisi dettagliata di come questa logica funziona e come soddisfa l'obiettivo di facilitare la collaborazione e la divisione dei guadagni.

---

## 🎯 **Obiettivo: Collaborazione tra Creator con Guadagni Condivisi**

### 1. **Scenario di Base**
- **Alberto** e **Franco** sono due Creator con i propri **Team personali** e **Collection** separate.
- Vogliono collaborare a una nuova **Collection condivisa**.

### 2. **Passaggi della Collaborazione**

1. **Invito nel Team**:
    - **Alberto** (o **Franco**) invita l’altro Creator a unirsi al proprio **Team**.
    - Questo viene gestito tramite la tabella `team_user` che registra l’associazione tra il **Team** e il nuovo **membro** (l'altro Creator).

2. **Creazione di un Wallet per il Nuovo Membro**:
    - Quando il nuovo Creator accetta l’invito, viene creato un **Wallet** specifico per lui, associato al **Team**.
    - Questo consente di tracciare i guadagni individuali e stabilire le **royalty** per ciascun Creator.

3. **Creazione della Collection Condivisa**:
    - Viene creata una nuova **Collection** associata al **Team** in cui entrambi sono membri.
    - La **Collection** può essere configurata con specifiche **royalty** per ciascun Creator.

4. **Divisione dei Guadagni**:
    - I guadagni derivanti dalla vendita degli **EGI** o degli asset della **Collection** possono essere automaticamente distribuiti in base ai **wallet** associati al **Team**.
    - Le **royalty** sono definite nel momento della creazione dei wallet nel **Team** e sono personalizzabili per ogni membro.

---

## 🔄 **Flusso Operativo**

1. **Alberto** invita **Franco** nel suo **Team**:
    - **Franco** viene aggiunto alla tabella `team_user` con il ruolo di **Collaboratore** o **Creator**.

2. **Creazione del Wallet per Franco**:
    ```php
    $team->wallets()->create([
        'user_id' => $franco->id,
        'user_role' => 'Creator',
        'wallet' => $this->generateFakeAlgorandAddress(),
        'royalty_mint' => config('app.creator_royalty_mint'),
        'royalty_rebind' => config('app.creator_royalty_rebind'),
    ]);
    ```

3. **Creazione della Collection**:
    ```php
    $collection = Collection::create([
        'team_id' => $team->id,
        'collection_name' => 'Alberto e Franco Collection',
        'creator_id' => $alberto->id,
        'description' => 'Collection condivisa tra Alberto e Franco.',
    ]);
    ```

4. **Gestione dei Permessi**:
    - Utilizzando **Spatie**, puoi assegnare i permessi specifici ai membri del **Team**:
      ```php
      $franco->assignRole('creator');
      ```

5. **Distribuzione dei Guadagni**:
    - Quando avviene una vendita, i guadagni vengono spartiti secondo i wallet configurati nel **Team**.

---

## ✅ **Vantaggi di Questa Logica**

1. **Flessibilità**:
   - Ogni **Collection** può essere associata a un **Team** diverso, permettendo configurazioni di collaborazione dinamiche.

2. **Tracciabilità**:
   - Ogni membro del **Team** ha il proprio **Wallet**, garantendo una chiara divisione dei guadagni.

3. **Scalabilità**:
   - È facile aggiungere nuovi **Collaboratori** e nuovi wallet senza modificare la struttura di base.

4. **Semplicità**:
   - La procedura è intuitiva: invito → creazione wallet → creazione Collection → distribuzione guadagni.

Questa soluzione copre perfettamente il tuo obiettivo di facilitare le collaborazioni tra Creator e garantire una gestione chiara e trasparente dei guadagni! 🚀