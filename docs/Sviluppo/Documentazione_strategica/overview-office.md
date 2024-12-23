Capisco, mi scuso per la confusione precedente. Riscriverò la descrizione dettagliata di tutti i moduli presenti in **Office**, includendo le modifiche che abbiamo concordato insieme, in particolare quelle relative alla tokenomics e alla distribuzione delle royalties. Procederò ad aggiornare le informazioni in modo accurato, formattandole in modo chiaro per facilitare la tua analisi.

---

# Moduli di Office

La piattaforma è costituita da tre siti principali:

1. **Office**: Contiene tutti gli strumenti necessari affinché i creator possano assemblare le loro **collection**.
2. **Marketplace (o E-commerce)**: Contiene tutte le gallerie dei creator; è ciò che l'utente visita per visualizzare e acquistare opere.
3. **Sito Istituzionale**: Presenta il progetto, il white paper, la roadmap, dati amministrativi pubblici, policy sulla privacy, ecc. Qui si presentano gli **EPP** (Environmental Protection Programs) e si creano i bandi.

**Nota**: In **Office**, l'insieme degli item che costituiscono gli **EGI** (Elemento Grafico Identificativo) è chiamato **collection**. Nel **Marketplace**, la stessa entità è chiamata **galleria**.

---

## Indice dei Moduli di Office

1. [Make Collection](#make-collection)
2. [Make Item](#make-item)
3. [Make Traits](#make-traits)
4. [Make Team](#make-team)
5. [Make Wallet](#make-wallet)
6. [Make EPP](#make-epp)
7. [Make Utilities](#make-utilities)
8. [Make Drop](#make-drop)
9. [Biographies Builder](#biographies-builder)

---

## Make Collection

### Descrizione

Il modulo **Make Collection** consente ai creator di creare e gestire le loro **collection**. Ogni collection include:

- **Nome**: Il titolo della collection.
- **Floor Price**: Il prezzo minimo di vendita degli item.
- **Utility**: Servizi o prodotti associati agli item.
- **EPP Associato**: L'Environmental Protection Program scelto.

Ogni collection è rappresentata da un elemento digitale chiamato **EGI Asset**, che ne permette lo scambio come singola entità.

### Ruoli e Proprietà

- **Creator**: L'utente che crea la collection; è il proprietario iniziale (**Owner**).
- **Piattaforma (Natan)**: La piattaforma stessa.
- **EPP**: L'ente beneficiario associato alla collection.

**Nota**: Abbiamo semplificato la struttura riducendo il numero di attori coinvolti, come concordato.

### Distribuzione delle Quote (Modificata)

Abbiamo semplificato la distribuzione delle quote per rendere il sistema più gestibile.

#### Prima Vendita (Mint)

- **Creator**: **70%**
- **EPP**: **20%**
- **Piattaforma (Natan)**: **10%**

#### Vendite Secondarie (Re-Sale)

- **Creator**: **5%** (royalty sulle vendite future)
- **EPP**: **1%**
- **Piattaforma (Natan)**: **1%**

### Immagini della Collection

Ogni collection deve avere tre immagini assegnate:

1. **Banner**: Utilizzato nella galleria visitata dagli utenti.
2. **Avatar**: Immagine rappresentativa della collection.
3. **EGI**: L'elemento digitale che rappresenta la collection come **EGI Asset**.

### Collection di Default

- Ogni creator ha una collection di default (solitamente la prima creata).
- Serve come destinazione predefinita per il trasferimento degli item.

---

## Make Item

### Descrizione

Il modulo **Make Item** permette ai creator di creare e gestire gli **item**, ovvero gli elementi che contengono tutte le informazioni sull'**EGI**.

### Processo di Creazione

1. **Accesso dal Make Collection**: Il creator accede al modulo Make Item dalla propria collection.
2. **Upload dei Contenuti Digitali**: Utilizza un gestore di upload per caricare gli elementi digitali.
3. **Configurazione dell'Item**:

   - **Nome**: Assegnato di default, modificabile dal creator.
   - **Prezzo**: Impostato secondo il floor price della collection o personalizzato.
   - **EPP Associato**: Ereditato dalla collection.

### Caratteristiche Principali

- **CRUD dei Dati**: Creazione, lettura, aggiornamento e cancellazione degli item.
- **Gestione delle Immagini**:

  - Visualizzazione dell'immagine dell'EGI con possibilità di zoom.
  - Non è previsto l'editing grafico all'interno della piattaforma.

- **Trasferimento degli Item**:

  - **Trasferimento Interno**: Spostamento tra collection dello stesso creator.
  - **Trasferimento Esterno**: Spostamento verso la collection di un altro creator (funzionalità avanzata da implementare in futuro).

### Note Tecniche

- **Struttura Dati**:

  - Gli item devono contenere tutti i valori presenti nella collection per essere indipendenti.
  - La tabella `collection_item` gestisce la relazione tra item e collection.

---

## Make Traits

### Descrizione

Il modulo **Make Traits** gestisce i **traits** (tratti distintivi) degli EGI, facilitando l'attribuzione e la categorizzazione.

### Struttura dei Traits

- **Categoria**: Aiuta nella ricerca; non viene esportata come metadata.
- **Nome**
- **Valore**

### Implementazione

- **Archivio dei Traits**:

  - **Globale**: Set di traits predefiniti disponibili per tutti i creator.
  - **Personale**: Traits creati e mantenuti dal singolo creator.

- **Interfaccia Utente**:

  - Pulsanti per selezionare le categorie.
  - Visualizzazione dei traits esistenti e possibilità di aggiungerne di nuovi.
  - Sistema drag-and-drop per assegnare traits agli EGI.

### Ottimizzazione Tecnica

- **Performance Migliorata**:

  - Eliminazione dell'uso di Filament a causa della lentezza riscontrata.
  - Sviluppo dell'interfaccia con HTML, JavaScript, PHP e Tailwind CSS.

---

## Make Team

### Descrizione

Il modulo **Make Team** gestisce i team di creator che collaborano su una collection.

### Struttura e Relazioni

- **Separazione delle Tabelle**:

  - Creazione di una tabella dedicata per le collection, separata dal team.
  - Relazione uno a uno tra collection e team.

- **Tabella Pivot `team_user`**:

  - Permette di creare gruppi di lavoro per ogni collection.
  - Gestione dei ruoli e dei permessi all'interno del team.

### Implementazione

- **Libreria Spatie**:

  - Utilizzata per la gestione avanzata dei permessi e dei ruoli.
  - Permette una gestione modulare e sicura delle autorizzazioni.

- **Interfaccia Utente**:

  - Rifatta senza l'utilizzo di Filament.
  - Sviluppata con HTML, JavaScript, PHP e Tailwind CSS per migliorare le performance.

### Gestione della Cessione dell'EGI Asset

- **User Type: Owner**:

  - Alla vendita dell'EGI Asset, l'ID dell'Owner viene aggiornato nella tabella pivot `team_user`.
  - Il nuovo Owner diventa destinatario delle royalty destinate all'Owner.

---

## Make Wallet

### Descrizione

Il modulo **Make Wallet** gestisce i **wallet** degli utenti, destinatari dei fondi derivanti dalle transazioni degli EGI.

### Funzionalità

- **Wallet Unico per Utente**:

  - Ogni utente ha un wallet associato al proprio account sulla piattaforma.
  - Il Creator admin può decidere come suddividere le entrate in termini di FIAT o criptovaluta (se supportata in futuro).

- **Destinatari dei Fondi**:

  - Attualmente, i fondi sono destinati direttamente agli utenti registrati.
  - Possibilità futura di aggiungere destinatari esterni, tenendo conto delle complessità legali e tecniche.

### Considerazioni

- **Gestione Semplificata**:

  - Evitare la complessità di gestire multipli wallet o wallet esterni nella fase iniziale.
  - Focalizzarsi sulla sicurezza e sulla conformità legale nella gestione dei fondi.

---

## Make EPP

### Descrizione

Il modulo **Make EPP** gestisce gli **Environmental Protection Programs** (EPP), gli enti benefici associati alle collection.

### Caratteristiche

- **Registrazione degli EPP**:

  - Gli EPP sono registrati come utenti con `type=EPP`.
  - Ogni EPP ha una collection dedicata contenente:

    - Contratti
    - Iniziative
    - Testimonianze
    - Immagini e video degli eventi

- **Scelta dell'EPP**:

  - I Creator possono scegliere a quale EPP associare la loro collection tra quelli disponibili.

### Settori degli EPP

1. **Aquatic Plastic Removal**: Interventi per la rimozione della plastica dai corpi idrici.
2. **Appropriate Restoration Forestry**: Iniziative di riforestazione sostenibile.
3. **Bee Population Enhancement**: Progetti per l'aumento della popolazione delle api.

### Quota Destinata agli EPP (Modificata)

- **Prima Vendita (Mint)**: **20%** delle entrate viene destinato all'EPP associato.
- **Vendite Secondarie (Re-Sale)**: **1%** delle royalties viene destinato all'EPP.

### Inclusività

- **Registrazione di Altri Enti Benefici**:

  - Enti come l'ospedale Meyer di Firenze possono registrarsi come `user_type = ente benefico`.
  - Possono ricevere le quote destinate agli EPP senza obbligo di selezionare un EPP esistente.

---

## Make Utilities

### Descrizione

Il modulo **Make Utilities** gestisce le **utilities**, ovvero servizi o prodotti associati agli EGI.

### Tipologie

- **Fisiche**: Prodotti tangibili da consegnare all'acquirente.
- **Immateriali**: Servizi, abbonamenti, consulenze, omaggi.

### Considerazioni

- **Obbligatorietà**:

  - Se l'EGI non rappresenta qualcosa di fisico, è obbligatorio aggiungere una utility.

- **Durata delle Utilities**:

  - Le utilities immateriali dovrebbero avere una durata che non deprezzi l'EGI.
  - Evitare scadenze brevi a meno che l'EGI sia pensato per una durata limitata.

### Implementazione

- **Interfaccia Utente**:

  - Semplificata per consentire ai creator di aggiungere e gestire le utilities.
  - Possibilità di scegliere da una lista di utilities predefinite o di crearne di personalizzate.

- **Gestione delle Utilities**:

  - Documentazione chiara per gli acquirenti su come usufruire delle utilities.
  - Meccanismi per la consegna o l'attivazione delle utilities.

---

## Make Drop

### Descrizione

Il modulo **Make Drop** gestisce le **Drop**, eventi speciali per promuovere le opere e generare interesse tra gli acquirenti.

### Caratteristiche

- **Creazione di una Drop**:

  - **Durata**: Data di inizio e fine dell'evento.
  - **Titolo e Tema**: Ogni Drop può avere un tema specifico, spesso legato alle tematiche ambientali o a eventi speciali.
  - **Regole di Partecipazione**: Criteri che gli item devono soddisfare per essere inclusi.

- **Partecipazione dei Creator**:

  - I creator possono proporre i loro item per la Drop direttamente dal modulo Make Collection.
  - Visualizzazione di un banner che annuncia la Drop e possibilità di includere gli item con un semplice clic.

### Frequenza delle Drop

- **Cadenza Annuale**:

  - **4 Drop fisse** ogni anno, sempre negli stessi periodi, per creare aspettativa e abitudine tra gli utenti.

### Vantaggi

- **Marketing a Basso Costo**:

  - Le Drop fungono da catalizzatore per aumentare il traffico e le vendite senza richiedere grandi investimenti pubblicitari.

- **Supporto ai Creator**:

  - Opportunità per aumentare la visibilità e le vendite delle loro opere.

### Implementazione

- **Interfaccia Utente**:

  - Revisione grafica per rendere il modulo più intuitivo e accattivante.
  - Sezione dedicata nel Marketplace per evidenziare le Drop attive.

---

## Biographies Builder

### Descrizione

Il modulo **Biographies Builder** consente ai creator di scrivere la loro biografia in modo dettagliato e strutturato.

### Funzionalità

- **Struttura a Capitoli**:

  - Possibilità di suddividere la biografia in capitoli o episodi.
  - Ogni capitolo include:

    - **Titolo**
    - **Data Inizio** e **Data Fine**
    - **Contenuto**: Testo libero con eventuale formattazione.

- **Flessibilità**:

  - Il creator può scegliere di scrivere una biografia generale o approfondire specifici periodi della sua vita artistica.

### Importanza nel Marketplace

- **Valore Aggiunto per gli Acquirenti**:

  - Le biografie permettono agli acquirenti di comprendere il background, le ispirazioni e il percorso artistico del creator.

- **Incremento del Valore delle Opere**:

  - Una biografia dettagliata può aumentare l'interesse e la percezione del valore delle opere.

### Implementazione

- **Interfaccia Utente**:

  - Editor di testo semplice e intuitivo, con possibilità di formattazione di base.
  - Possibilità di aggiungere immagini correlate ai capitoli (se appropriato).

- **Visualizzazione nel Marketplace**:

  - Sezione dedicata alla biografia del creator nella sua galleria.
  - Accesso facile per gli utenti interessati a conoscere di più sull'artista.

---

# Conclusione

I moduli descritti costituiscono il cuore operativo della piattaforma **Office**, fornendo ai creator gli strumenti necessari per creare, gestire e promuovere le loro opere, in linea con le modifiche e le semplificazioni che abbiamo concordato.

---

## Prossimi Passi

1. **Analisi Dettagliata di Ogni Modulo**:

   - Valutare le specifiche tecniche e funzionali per ciascun modulo.
   - Definire le priorità di sviluppo in base all'importanza e alla complessità.

2. **Sviluppo Progressivo**:

   - Iniziare con i moduli essenziali come Make Collection, Make Item e Biographies Builder.
   - Implementare gradualmente gli altri moduli in base alla roadmap stabilita.

3. **Ottimizzazione e Testing**:

   - Assicurarsi che ogni modulo sia efficiente e privo di problemi di performance.
   - Condurre test interni e raccogliere feedback per migliorare l'usabilità.

4. **Integrazione con il Marketplace e il Sito Istituzionale**:

   - Garantire una coerenza grafica e funzionale tra i diversi siti.
   - Assicurarsi che i dati siano sincronizzati correttamente tra Office e Marketplace.

5. **Preparazione per il Lancio**:

   - Finalizzare i moduli e condurre test approfonditi.
   - Pianificare le strategie di marketing e comunicazione per il lancio della piattaforma.

---

Se hai bisogno di ulteriori dettagli o desideri apportare modifiche alle descrizioni, fammelo sapere. Sono qui per aiutarti a sviluppare una documentazione completa e utile per il tuo progetto.