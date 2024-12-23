# Documentazione per la Classe `EGIImageService`

## Introduzione

La classe `EGIImageService` è responsabile della gestione delle immagini associate alle collezioni (EGI) nella piattaforma **FlorenceEGI**. Fornisce metodi per:

1. **Salvare** immagini nei servizi di hosting attivi.
2. **Rimuovere** immagini obsolete.
3. **Recuperare** il percorso delle immagini (con caching per ottimizzare le performance).
4. **Invalidare** la cache delle immagini.

La classe supporta più servizi di hosting configurati dinamicamente tramite il file `config/paths.php`. Questo permette di avere un sistema flessibile e scalabile per la gestione delle immagini.

---

## Spiegazione Discorsiva

Immagina di dover gestire le immagini delle collezioni di un marketplace, dove ogni collezione può avere un banner, un avatar e altre immagini associate. La classe `EGIImageService` è stata progettata per rendere questa gestione il più semplice e automatizzata possibile. 

Quando carichi un'immagine, ad esempio un banner, non devi preoccuparti di dove verrà salvata: è sufficiente chiamare `saveEGIImage()`, specificare l'ID della collezione, il nome del file e il file stesso. La classe si occuperà di salvarlo su tutti i servizi di hosting attivi configurati nel file `paths.php`.

Se in un secondo momento hai bisogno di rimuovere vecchie immagini, magari per evitare di occupare spazio inutile o per aggiornare il contenuto della collezione, puoi usare `removeOldImage()`. Ad esempio, se vuoi rimuovere tutti i vecchi banner con un certo prefisso, questa funzione scansionerà i percorsi di tutti i servizi attivi e cancellerà i file che corrispondono.

Un altro aspetto utile è la gestione della cache. Ogni volta che vuoi ottenere il percorso di un'immagine, invece di calcolarlo ogni volta (il che potrebbe richiedere richieste a servizi esterni), puoi usare `getCachedEGIImagePath()`. Questo metodo verifica se il percorso è già memorizzato nella cache e, se non lo è, lo calcola e lo salva per un tempo prestabilito. Questo migliora notevolmente le performance, specialmente quando hai molte immagini da caricare.

Inoltre, se aggiorni o rimuovi un'immagine e vuoi che le modifiche siano immediate, puoi usare `invalidateEGIImageCache()` per cancellare la cache associata a quell'immagine. Così facendo, la prossima volta che verrà richiesta, verrà ricalcolato il percorso corretto.

Questa classe è pensata per essere flessibile: puoi facilmente aggiungere nuovi servizi di hosting nel file di configurazione `paths.php` e la classe li gestirà automaticamente, senza bisogno di modifiche al codice.

In sintesi, `EGIImageService` ti semplifica la vita: salva, recupera, rimuove e ottimizza l'uso delle immagini senza che tu debba preoccuparti dei dettagli di implementazione.

---

## Configurazione (`config/paths.php`)



La configurazione dei percorsi e dei servizi di hosting è definita nel file `config/paths.php`. Ecco un esempio della struttura di configurazione:

```php
<?php

return [
    'default_hosting' => env('DEFAULT_HOSTING', 'Local'),

    'hosting' => [
        'Local' => [
            'url' => '/storage/',
            'disk' => 'public',
            'is_default' => true,
            'is_active' => true,
        ],
        'Digital Ocean' => [
            'url' => env('BUCKET_PATH_FILE_FOLDER_DO', 'https://frangettediskspace.fra1.digitaloceanspaces.com'),
            'disk' => 'do',
            'is_default' => false,
            'is_active' => false,
        ],
        'AWS' => [
            'url' => env('AWS_URL', 'https://aws.example.com'),
            'disk' => 's3',
            'is_default' => false,
            'is_active' => false,
        ],
        'IPFS' => [
            'url' => 'https://ipfs.io/ipfs/',
            'disk' => 'ipfs',
            'is_default' => false,
            'is_active' => false,
        ],
    ],

    'paths' => [
        'collections' => 'users_files/collections_{collectionId}/',
        'head' => [
            'root' => 'users_files/collections_{collectionId}/head/',
            'banner' => 'users_files/collections_{collectionId}/head/banner/',
            'card' => 'users_files/collections_{collectionId}/head/card/',
            'avatar' => 'users_files/collections_{collectionId}/head/avatar/',
            'EGI_asset' => 'users_files/collections_{collectionId}/head/EGI_asset/',
        ],
        'EGIs' => 'users_files/collections_{collectionId}/EGIs/',
        'user_data' => [
            'root' => 'users_files/users-data/',
            'documents' => 'users_files/users-data/documents/',
        ],
    ],
];
```

### Spiegazione della Configurazione

- **`hosting`**: Contiene una lista dei servizi di hosting configurati. Ogni servizio ha le seguenti chiavi:
  - `url`: L'URL base per il servizio.
  - `disk`: Il nome del disco configurato in `config/filesystems.php`.
  - `is_default`: Indica se il servizio è il default.
  - `is_active`: Indica se il servizio è attivo.

- **`paths`**: Contiene i percorsi relativi per le immagini e altri file associati alle collezioni.

---

## Metodi della Classe `EGIImageService`

### 1. `removeOldImage`

**Descrizione:**
Rimuove i file esistenti con un determinato prefisso dai servizi di hosting attivi.

**Firma del Metodo:**
```php
public static function removeOldImage(string $prefix, int $collectionId, string $pathKey): bool
```

**Parametri:**
- `string $prefix`: Il prefisso dei file da rimuovere (es. "banner_").
- `int $collectionId`: L'ID della collezione.
- `string $pathKey`: La chiave di configurazione per il percorso.

**Ritorna:**
- `bool`: `true` se tutti i file sono stati rimossi con successo, `false` altrimenti.

---

### 2. `saveEGIImage`

**Descrizione:**
Salva un'immagine nei servizi di hosting attivi.

**Firma del Metodo:**
```php
public static function saveEGIImage(int $collectionId, string $filename, $file, string $pathKey): bool
```

**Parametri:**
- `int $collectionId`: L'ID della collezione.
- `string $filename`: Il nome del file da salvare.
- `$file`: Il file da salvare (es. istanza di `UploadedFile`).
- `string $pathKey`: La chiave di configurazione per il percorso.

**Ritorna:**
- `bool`: `true` se l'immagine è stata salvata con successo su almeno un servizio.

---

### 3. `getCachedEGIImagePath`

**Descrizione:**
Ottiene il percorso dell'immagine memorizzata nella cache. Se non esiste, lo calcola e lo salva in cache.

**Firma del Metodo:**
```php
public static function getCachedEGIImagePath(int $collectionId, string $filename, bool $isPublished, ?string $hostingService = null, string $pathKey = 'head.banner'): ?string
```

**Parametri:**
- `int $collectionId`: L'ID della collezione.
- `string $filename`: Il nome del file.
- `bool $isPublished`: Indica se l'immagine è pubblicata.
- `string|null $hostingService`: Il servizio di hosting da usare (opzionale).
- `string $pathKey

**Ritorna:**
- `string|null`: Il percorso dell'immagine o `null` se non trovata.

---

### 4. `invalidateEGIImageCache`

**Descrizione:**
Invalidare la cache dell'immagine specificata.

**Firma del Metodo:**
```php
public static function invalidateEGIImageCache(int $collectionId, string $filename, ?string $hostingService = null): void
```

**Parametri:**
- `int $collectionId`: L'ID della collezione.
- `string $filename`: Il nome del file.
- `string|null $hostingService`: Il servizio di hosting (opzionale).

**Ritorna:**
- `void`

---

## Esempi d'Uso con i Componenti Livewire

### `BannerImageUpload`

Gestisce l'upload e la rimozione dell'immagine del banner.

```php
public function saveImage()
{
    $filename = 'banner_image_' . uniqid() . '.' . $this->bannerImage->getClientOriginalExtension();
    EGIImageService::saveEGIImage($this->collectionId, $filename, $this->bannerImage, 'head.banner');
}
```

### `AvatarImageUpload`

Gestisce l'upload e la rimozione dell'immagine avatar.

```php
public function saveImage()
{
    $filename = 'avatar_image_' . uniqid() . '.' . $this->avatarImage->getClientOriginalExtension();
    EGIImageService::saveEGIImage($this->collectionId, $filename, $this->avatarImage, 'head.avatar');
}
```

---

## Note Aggiuntive

- **Caching**: Utilizza `Cache::remember` per migliorare le performance.
- **Log**: I log sono registrati nel canale `florenceegi` per un debug dettagliato.
- **Fallback**: Supporta fallback tra più servizi di hosting.

---

© **FlorenceEGI** | Documentazione sviluppata per **wikyDev**.

