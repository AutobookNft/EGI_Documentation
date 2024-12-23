# Gestione delle Collezioni

La sezione "Collections" nell'area amministrativa permette di gestire e visualizzare le collezioni di EGI (Ecological Goods Invent) associate all'account dell'utente.

 ![Mappa mentale della struttura Make Collection](/img/make_collection.svg "Struttura dettagliata della funzionalità Make Collection")

## Accesso alla Pagina

- **Navigazione**: La pagina è accessibile dal menu di navigazione laterale, sotto la voce "Collections"
- **Icona**: Rappresentata da un'icona a forma di portafoglio
- **Ordinamento**: Posizionata al numero 300 nella navigazione

## Struttura della Pagina

La pagina presenta una tabella con due colonne principali:

### 1. Informazioni sulla Collezione e Wallet

- Nome della collezione
- Indirizzo del wallet (versione abbreviata)
- Indirizzo del creatore (versione abbreviata)
- Pulsante per aprire/attivare la collezione

### 2. Thumbnail degli EGI

- Visualizzazione a griglia delle immagini o video degli EGI
- Indicatore dello stato di pubblicazione (pubblicato/non pubblicato)
- Link diretto all'editor dell'EGI

## Funzionalità

### Gestione delle Collezioni

- **Visualizzazione**: Mostra tutte le collezioni associate all'account dell'utente
- **Cambio Collezione Attiva**: Utilizzare il pulsante "Open" per cambiare la collezione correntemente attiva

### Informazioni sul Wallet

- **Copia Indirizzi**: Cliccando sugli indirizzi del wallet o del creatore, questi vengono copiati negli appunti
- **Identicon**: Visualizzazione di un'immagine unica generata dall'indirizzo del wallet

### Gestione degli EGI

- **Anteprima**: Visualizzazione delle thumbnail degli EGI con supporto per immagini e video
- **Stato di Pubblicazione**: Indicatore visivo per EGI pubblicati e non pubblicati
- **Modifica Rapida**: Cliccando su un'immagine si accede direttamente alla pagina di modifica dell'EGI

### Funzionalità Aggiuntive

- **Gestione Errori Immagine**: Sistema di fallback per gestire eventuali errori di caricamento delle immagini
- **Zoom**: Possibilità di ingrandire le immagini al passaggio del mouse (funzionalità da implementare nel CSS)

## Note Tecniche

- La pagina utilizza Livewire per il caricamento dinamico dei dati
- Le viste sono ottimizzate per la responsività, con layout flessibili per diversi dispositivi
- Implementazione di funzioni JavaScript per la copia degli indirizzi e la gestione degli errori delle immagini

### Gestione degli Script JavaScript

La gestione degli script JavaScript nella vista `collection-image.blade.php` presenta alcune particolarità:

- Viene definita una variabile globale `window.defaultCoverPath` utilizzando la configurazione dell'applicazione
- Le funzioni JavaScript principali sono incluse tramite un file separato

:::info Nota sull'Inclusione di File JavaScript
Per comprendere le ragioni dietro l'utilizzo di un file incluso per il codice JavaScript, si prega di consultare la documentazione specifica su questo problema:

[Problemi di Inclusione di File JavaScript](http://localhost:3000/frangette-docs/wiky/solved_problems/problemi-inclusione-file-javascript)

Questa soluzione è stata adottata per superare alcune limitazioni nell'inclusione diretta di script JavaScript nelle viste Blade.
:::

## Struttura del Codice

### Componenti Principali

1. **Pagina Filament**: `App\Filament\Pages\Collection.php`
   - Definisce la struttura generale della pagina
   - Gestisce la navigazione e i badge

2. **Componente Livewire**: `app/Livewire/Images.php`
   - Gestisce la logica della tabella principale

3. **Viste Blade**:
   - `resources/views/filament/app/pages/home.blade.php`: Vista principale
   - `resources/views/livewire/images.blade.php`: Contenitore per la tabella Livewire

### Colonne Personalizzate

1. **WalletCreatorAddress**:
   - File: `app/Tables/Columns/WalletCreatorAddress.php`
   - Vista: `resources/views/tables/columns/wallet-creator-address.blade.php`
   - Funzionalità: Visualizza informazioni sul wallet e permette la copia degli indirizzi

2. **CollectionImage**:
   - File: `app/Tables/Columns/CollectionImage.php`
   - Vista: `resources/views/tables/columns/collection-image.blade.php`
   - Funzionalità: Mostra le thumbnail degli EGI con indicatori di stato

  
