# Classe CollectionFiles

## Panoramica

La classe `CollectionFiles` è un componente Livewire cruciale per la gestione del caricamento e dell'elaborazione di file per la creazione di EGI (Ecological Goods Invent) all'interno del nostro sistema. Questa classe gestisce l'intero processo di upload, dalla validazione dei file alla creazione di record nel database e al salvataggio dei file nello storage.

## Funzionalità Principali

1. Caricamento di file multipli
2. Validazione dei file caricati
3. Creazione di record EGI nel database
4. Salvataggio dei file nello storage (DigitalOcean Spaces)
5. Gestione della cache e degli eventi Livewire
6. Supporto per l'annullamento dell'upload

## Struttura della Classe

```php
class CollectionFiles extends Component
{
    use HasUtilitys;
    use WithFileUploads;
    use WithPagination;

    // Proprietà pubbliche e protette
    // ...

    // Metodi pubblici e protetti
    // ...
}
```

### Traits Utilizzati

- `HasUtilitys`: Fornisce funzionalità utility comuni
- `WithFileUploads`: Gestisce il caricamento dei file in Livewire
- `WithPagination`: Supporta la paginazione in Livewire

## Metodi Principali

### `mount()`

Inizializza la classe impostando i valori iniziali come il team corrente e il floor price.

### `store()`

Metodo principale che gestisce il processo di salvataggio dei file. Esegue le seguenti operazioni:
1. Verifica l'autorizzazione dell'utente
2. Genera un ID univoco per l'upload
3. Processa i file caricati
4. Gestisce eventuali errori

### `singleStore($file, $path_image)`

Elabora un singolo file, creando il record EGI corrispondente e salvandolo nello storage.

### `saveFileToSpaces($path_image, $tempPath, $extension)`

Salva il file nello storage DigitalOcean Spaces e nel disco pubblico locale.

## Flusso di Lavoro

1. L'utente seleziona i file da caricare
2. Il metodo `store()` viene chiamato
3. Ogni file viene processato da `singleStore()`
4. I record EGI vengono creati nel database
5. I file vengono salvati nello storage
6. La cache viene invalidata e l'interfaccia utente viene aggiornata

## Gestione degli Errori

La classe utilizza un sistema di logging esteso per tracciare errori e informazioni importanti. Gli errori vengono registrati nel canale di log 'upload'.

## Configurazione

La classe dipende da diverse configurazioni:
- `AllowedFileType.collection.allowed`: Tipi di file consentiti
- `AllowedFileType.collection.max_size`: Dimensione massima del file
- `app.bucket_root_file_folder`: Cartella root per lo storage dei file

Assicurarsi che queste configurazioni siano correttamente impostate nel file `config/app.php` e in altri file di configurazione pertinenti.

## Best Practices per l'Utilizzo

1. **Validazione**: Assicurarsi che i tipi di file consentiti e le dimensioni massime siano appropriati per il vostro caso d'uso.
2. **Sicurezza**: La classe implementa controlli di autorizzazione, ma assicuratevi che siano in linea con le policy di sicurezza del progetto.
3. **Performance**: Per upload di grandi dimensioni, considerare l'implementazione di un sistema di coda per elaborare i file in background.
4. **Manutenzione**: Aggiornare regolarmente le dipendenze e rivedere la logica di crittografia in `my_advanced_crypt()`.

## Note per lo Sviluppo Futuro

1. Considerare l'implementazione di un sistema di chunking per file di grandi dimensioni.
2. Valutare l'aggiunta di funzionalità di compressione delle immagini.
3. Implementare test unitari e di integrazione per garantire la robustezza della classe.

## Dipendenze

- Laravel Framework
- Livewire
- DigitalOcean Spaces SDK (per lo storage)

Assicurarsi che tutte le dipendenze siano correttamente installate e configurate nel progetto.

## Conclusione

La classe `CollectionFiles` è un componente critico per la funzionalità di upload dei file nel nostro sistema. Mantenere questa classe aggiornata e ottimizzata è essenziale per garantire un'esperienza utente fluida e sicura.

Per domande o chiarimenti, contattare il team di sviluppo principale.
