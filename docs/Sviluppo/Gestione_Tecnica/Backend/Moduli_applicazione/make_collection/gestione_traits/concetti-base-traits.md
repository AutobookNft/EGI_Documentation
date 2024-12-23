# Concetti base trait

## Logica di Gestione dei Traits 

<span style={{color: 'green', fontWeight: 'bold'}}>
    **v.1.0.0**
</span>
<span style={{color: 'green', fontWeight: 'bold'}}>
    **Data ultimo aggiornamento: 2024 maggio 24**
</span>

#### <u style={{color: 'green', fontWeight: 'bold'}}> [Visualizza il componente PHP su GitHub](https://github.com/AutobookNft/egiflorence/blob/master/app/Livewire/Traits/TraitLibraryComponent.php) </u>
#### <u style={{color: 'green', fontWeight: 'bold'}}> [Visualizza la vista Blade su GitHub](https://github.com/AutobookNft/egiflorence/blob/master/resources/views/livewire/traits/trait-library-component.blade.php)</u>



### Traits del Progetto Frangette

Ogni Creator ha a sua disposizione una libreria preconfigurata di categorie e chiavi che può utilizzare per creare i suoi traits. Quando un Creator vuole associare uno o più traits a un EGI, può farlo in modo massivo, selezionando più EGI nella pagina `/collection/bindtrait_to_EGI`, oppure individualmente nella pagina `/editEgi/edit-egi-page?itemId={$itemId}`. In entrambi i casi si apre una <u style={{color: 'green', fontWeight: 'bold'}}>[pulsantiera](#funzionamento-della-pulsantiera)</u> dove il Creator può selezionare una categoria, una chiave e poi creare il suo trait. Una volta creato, il trait entra a far parte della libreria personale del Creator e può essere associato all'EGI tramite drag and drop.

### Struttura dei Traits

I traits nel progetto Frangette sono costituiti da una coppia chiave/valore. Per facilitare la ricerca della chiave corretta, queste sono raggruppate in categorie. Tuttavia, l'introduzione della categoria serve solo durante la creazione del trait; una volta che l'EGI viene mintato, il file metadata conterrà solo la coppia chiave/valore.

### Categorie e Chiavi Fornite di Default

Quando viene creato un nuovo utente, gli viene assegnato automaticamente un set predefinito di categorie e chiavi. Questa libreria preconfigurata si trova nel file `database/data/traits_cat_key.json`. Per ulteriori dettagli, consultare la sezione [Assegnazione Automatica Categorie e Chiavi](implementazione-logica-assegnazione-automatica-categorie-chiavi-traits).

### Tabelle Coinvolte

Le tabelle principali utilizzate per gestire i traits sono le seguenti:

- `trait_categories`
- `traits_key_commons`
- `traits_value_users`
- `traits_items` (tabella pivot)

### Dati di Configurazione

Nel file `config/app.php`, sono definite due chiavi di configurazione cruciali per la gestione delle lingue:

```php
'locale' => 'it',
'languages' => ['it', 'en', 'es', 'pt', 'fr', 'de'],
```

La chiave `languages` stabilisce quali lingue devono essere gestite dall'applicazione.

### Esempio di Utilizzo

1. **Assegnazione di Traits Massiva:**
   - Accedere alla pagina `/collection/bindtrait_to_EGI`.
   - Selezionare più EGI e aprire la libreria di pulsanti.
   - Selezionare la categoria e la chiave desiderata, creare il trait e associarlo agli EGI tramite drag and drop.

2. **Assegnazione di Traits Individuale:**
   - Accedere alla pagina `/editEgi/edit-egi-page?itemId={$itemId}`.
   - Aprire la libreria di pulsanti.
   - Selezionare la categoria e la chiave desiderata, creare il trait e associarlo all'EGI tramite drag and drop.

### Tabella `trait_categories`
Questa tabella contiene le categorie dei traits, senza chiavi esterne.

```php
Schema::create('trait_categories', function (Blueprint $table) {
    $table->id();
    $table->string('en')->nullable(); // Inglese
    $table->string('it')->nullable(); // Italiano
    $table->string('de')->nullable(); // Tedesco
    $table->string('fr')->nullable(); // Francese
    $table->string('es')->nullable(); // Spagnolo
    $table->string('pt')->nullable(); // Portoghese
    $table->string('nl')->nullable(); // Olandese
    $table->string('ru')->nullable(); // Russo
    $table->string('ja')->nullable(); // Giapponese
    $table->string('zh')->nullable(); // Cinese
    $table->string('ko')->nullable(); // Coreano
    $table->string('ar')->nullable(); // Arabo
    $table->string('hi')->nullable(); // Hindi
    $table->timestamps();
});
```

### Tabella `traits_key_commons`
Questa tabella archivia le chiavi dei traits, suddivise per utente e categorie.

```php
Schema::create('trait_key_commons', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('user_id');
    $table->foreignId('trait_category_id')
        ->constrained(table: 'trait_categories', indexName: 'trait_key_commons_trait_category_id_foreign')
        ->onUpdate('cascade')
        ->onDelete('cascade');
    $table->string('key_en')->nullable()->index(); // Inglese
    $table->string('key_it')->nullable()->index(); // Italiano
    $table->string('key_de')->nullable(); // Tedesco
    $table->string('key_fr')->nullable(); // Francese
    $table->string('key_es')->nullable(); // Spagnolo
    $table->string('key_pt')->nullable(); // Portoghese
    $table->string('key_nl')->nullable(); // Olandese
    $table->string('key_ru')->nullable(); // Russo
    $table->string('key_ja')->nullable(); // Giapponese
    $table->string('key_zh')->nullable(); // Cinese
    $table->string('key_ko')->nullable(); // Coreano
    $table->string('key_ar')->nullable(); // Arabo
    $table->string('key_hi')->nullable(); // Hindi
    $table->timestamps();
});
```

### Tabella `traits_value_users`
Questa tabella archivia i traits creati dagli utenti. Contiene campi per le traduzioni in diverse lingue.

```php
 Schema::create('traits_value_users', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('traits_key_common_id');
    $table->unsignedBigInteger('user_id');
    $table->string('value_en')->nullable()->index(); // Inglese
    $table->string('value_it')->nullable()->index(); // Italiano
    $table->string('value_de')->nullable(); // Tedesco
    $table->string('value_fr')->nullable(); // Francese
    $table->string('value_es')->nullable(); // Spagnolo
    $table->string('value_pt')->nullable(); // Portoghese
    $table->string('value_nl')->nullable(); // Olandese
    $table->string('value_ru')->nullable(); // Russo
    $table->string('value_ja')->nullable(); // Giapponese
    $table->string('value_zh')->nullable(); // Cinese
    $table->string('value_ko')->nullable(); // Coreano
    $table->string('value_ar')->nullable(); // Arabo
    $table->string('value_hi')->nullable(); // Hindi

    // Aggiungi altri campi lingua se necessario
    $table->timestamps();

    });
```

### Tabella `traits_items`

La tabella `traits_items` è una tabella pivot che mette in relazione la tabella `teams_items`, contenente tutti i dati degli EGI, con i traits. Questa tabella contiene sia la chiave che il valore dei traits, gestendoli in modo statico e indipendente dalle tabelle `trait_key_commons` e `traits_value_users`. 

#### Motivazioni della Scelta

La scelta di inserire direttamente i valori dei traits nella tabella `traits_items` è stata presa per migliorare le prestazioni delle query. Inizialmente, la gestione dei traits era basata sulle relazioni con le altre tabelle, ma si è rivelata complessa e inefficiente, causando lentezza nelle operazioni e difficoltà nella gestione. 

Memorizzando i valori dei traits direttamente nella tabella `traits_items`, non solo si semplificano le query, ma si garantisce anche che i traits associati a un EGI rimangano immutabili. Questo significa che, anche se le chiavi o i valori nelle tabelle `trait_key_commons` e `traits_value_users` vengono modificati o eliminati, i traits degli EGI rimangono intatti.

#### Struttura della Tabella

```php
Schema::create('traits_items', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('traits_value_users_id');
    $table->unsignedBigInteger('teams_items_id');
    $table->unsignedBigInteger('team_id')->index();
    $table->unsignedBigInteger('user_id')->index();
    $table->string('key_en')->nullable()->index(); // Chiave in Inglese
    $table->string('value_en')->nullable(); // Valore in Inglese
    $table->string('key_it')->nullable()->index(); // Chiave in Italiano
    $table->string('value_it')->nullable(); // Valore in Italiano
    $table->string('key_de')->nullable()->index(); // Chiave in Tedesco
    $table->string('value_de')->nullable(); // Valore in Tedesco
    $table->string('key_fr')->nullable()->index(); // Chiave in Francese
    $table->string('value_fr')->nullable(); // Valore in Francese
    $table->string('key_es')->nullable()->index(); // Chiave in Spagnolo
    $table->string('value_es')->nullable(); // Valore in Spagnolo
    $table->string('key_pt')->nullable()->index(); // Chiave in Portoghese
    $table->string('value_pt')->nullable(); // Valore in Portoghese
    $table->string('key_nl')->nullable()->index(); // Chiave in Olandese
    $table->string('value_nl')->nullable(); // Valore in Olandese
    $table->string('key_ru')->nullable()->index(); // Chiave in Russo
    $table->string('value_ru')->nullable(); // Valore in Russo
    $table->string('key_ja')->nullable()->index(); // Chiave in Giapponese
    $table->string('value_ja')->nullable(); // Valore in Giapponese
    $table->string('key_zh')->nullable()->index(); // Chiave in Cinese
    $table->string('value_zh')->nullable(); // Valore in Cinese
    $table->string('key_ko')->nullable()->index(); // Chiave in Coreano
    $table->string('value_ko')->nullable(); // Valore in Coreano
    $table->string('key_ar')->nullable()->index(); // Chiave in Arabo
    $table->string('value_ar')->nullable(); // Valore in Arabo
    $table->string('key_hi')->nullable()->index(); // Chiave in Hindi
    $table->string('value_hi')->nullable(); // Valore in Hindi
    $table->timestamps();
});
```

### Vantaggi di Questa Struttura

1. **Prestazioni Migliorate:** Riduce la complessità delle query, migliorando la velocità di accesso e visualizzazione dei dati.
2. **Immutabilità:** I traits associati a un EGI rimangono costanti, anche se le chiavi o i valori originari vengono modificati o eliminati.
3. **Gestione Semplificata:** Evita la necessità di gestire complesse relazioni tra più tabelle, semplificando il codice e la manutenzione.

Questa implementazione garantisce che i dati siano facilmente accessibili e che le operazioni di lettura e scrittura siano eseguite in modo efficiente, migliorando l'esperienza utente complessiva.


### Tabella `teams`
La tabella `teams` contiene le collezioni. La gestione dei team è utilizzata per sfruttare la logica di Jetstream.  
<u
style={{color: 'green', fontWeight: 'bold'}}>
    [Vedi Concetti Base Makecollection](Concetti%20Base%20per%20la%20Creazione%20di%20Collection.md)
</u>

## Gestione Statica delle Lingue

La piattaforma è progettata per supportare sei lingue: italiano, inglese, francese, spagnolo, portoghese e tedesco. Questo supporto multilingue è fondamentale per garantire che i traits degli EGI (Eco Gate Invest) siano accessibili e comprensibili in diversi contesti linguistici.

### Perché i Traits Vengono Tradotti

I traits sono "oggetti" creati dagli utenti, che li scrivono nella loro lingua madre. Consideriamo il caso di Luca Rossi, un utente italiano che crea una collezione di EGI con descrizioni e traits in italiano. Quando questi EGI vengono visualizzati su un marketplace compatibile con Algorand, potrebbero esserci problemi di comprensione per gli utenti che non parlano italiano.

Ogni marketplace gestisce le traduzioni dei propri contenuti, ma non necessariamente quelle dei contenuti generati dagli utenti. Pertanto, se Luca Rossi scrive i suoi traits in italiano, questi saranno visualizzati in italiano su qualsiasi marketplace, anche se l'interfaccia del marketplace è in un'altra lingua, come l'inglese.

### Soluzione Implementata da Frangette

Per risolvere questo problema, la piattaforma Frangette traduce automaticamente i traits nelle lingue supportate al momento della loro creazione. Queste traduzioni vengono salvate nel database con i nomi dei campi che corrispondono alle sigle delle lingue (es. `it` per italiano, `en` per inglese, ecc.).

Quando una collezione viene mintata, i traits tradotti vengono inclusi nel file metadata in formato JSON. Questo formato è ampiamente supportato e facilmente leggibile da vari sistemi. Di conseguenza, quando un marketplace visualizza gli EGI di Luca Rossi, può accedere ai traits tradotti e mostrarli nella lingua preferita dall'utente del marketplace.

### Esempio Pratico

Immaginiamo che Luca Rossi abbia creato un EGI con un trait chiamato "colore" e che l'abbia descritto come "rosso". Quando questo trait viene salvato nel database, Frangette genera automaticamente le traduzioni:

- Italiano: "rosso"
- Inglese: "red"
- Francese: "rouge"
- Spagnolo: "rojo"
- Portoghese: "vermelho"
- Tedesco: "rot"

Questi valori vengono salvati nei rispettivi campi nel database (`color_it`, `color_en`, `color_fr`, ecc.).

Quando la collezione di Luca viene mintata, il file metadata JSON potrebbe apparire così:

```json
{
    "traits": {
        "color": {
            "it": "rosso",
            "en": "red",
            "fr": "rouge",
            "es": "rojo",
            "pt": "vermelho",
            "de": "rot"
        }
    }
}
```

### Vantaggi di Questa Implementazione

1. **Accessibilità Multilingue:** Gli EGI possono essere facilmente compresi da utenti di diverse lingue senza richiedere ulteriori sforzi di traduzione da parte dei marketplace.
2. **Uniformità dei Contenuti:** Garantisce che i traits siano visualizzati in modo coerente e nella lingua corretta su tutti i marketplace compatibili con Algorand.
3. **Esperienza Utente Migliorata:** Gli utenti possono vedere i contenuti nella loro lingua preferita, migliorando l'esperienza di navigazione e comprensione.

Questa soluzione assicura che i traits creati dagli utenti siano sempre accessibili e comprensibili, indipendentemente dalla lingua del marketplace su cui vengono visualizzati.

