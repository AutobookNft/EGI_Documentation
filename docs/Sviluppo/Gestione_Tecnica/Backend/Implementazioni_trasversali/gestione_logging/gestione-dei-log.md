# Gestione dei Log nell'Applicazione

<span style={{ color: 'green', fontWeight: 'bold' }}>
    **v.1.0.0**
</span>
<span style={{ color: 'green', fontWeight: 'bold' }}>
    **Data ultimo aggiornamento: 2024 giugno 11**
</span>

### Introduzione

La gestione efficace dei log è essenziale per il monitoraggio e il debugging dell'applicazione. Questo documento descrive la configurazione dei log, la formattazione per una precisione maggiore dei timestamp e il processo per unire i log in un unico file.

### Formattazione dei Timestamp

Per ottenere timestamp con precisione a millisecondi, abbiamo personalizzato la configurazione dei log. La classe `CustomizeFormatter` è stata creata per formattare i timestamp con la precisione desiderata.

#### Classe CustomFormatter

```php
namespace App\Logging;

use Monolog\Formatter\LineFormatter;
use Monolog\LogRecord;

class CustomFormatter extends LineFormatter
{
    public function format(LogRecord $record): string
    {
        if (isset($record->datetime)) {
            $recordArray = $record->toArray();
            $recordArray['datetime'] = $record->datetime->format('Y-m-d H:i:s.u');
            $record = LogRecord::fromArray($recordArray);
        }
        return parent::format($record);
    }
}
```



#### Configurazione di logging.php

Nel file `config/logging.php`, abbiamo aggiunto la configurazione per utilizzare `CustomizeFormatter`,
vedi: 

```php
    'tap' => [CustomizeFormatter::class]
```

```php
'channels' => [
    ...

    'nft_transaction' => [
        'driver' => 'daily',
        'path' => storage_path('logs/NFT_transaction.log'),
        'level' => 'debug',
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'minting' => [
        'driver' => 'daily',
        'path' => storage_path('logs/minting.log'),
        'level' => 'debug',
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'upload' => [
        'driver' => 'daily',
        'path' => storage_path('logs/upload.log'),
        'level' => 'debug',
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'traits' => [
        'driver' => 'daily',
        'path' => storage_path('logs/traits.log'),
        'level' => 'debug',
        'tap' => [CustomizeFormatter::class],
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'nftflorence' => [
        'driver' => 'daily',
        'path' => storage_path('logs/nftflorence.log'),
        'level' => 'debug',
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'tests' => [
        'driver' => 'daily',
        'path' => storage_path('logs/tests.log'),
        'level' => 'debug',
        'tap' => [CustomizeFormatter::class],
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'bind_unbind_cover' => [
        'driver' => 'daily',
        'path' => storage_path('logs/bind_unbind_cover.log'),
        'level' => 'debug',
        'days' => 7,  // Numero di giorni per cui conservare i log
    ],

    'services' => [
        'driver' => 'daily',
        'path' => storage_path('logs/services-'.date('Y-m-d').'.log'),
        'level' => 'debug',
        'days' => 7,
    ],
],
```

### Unione dei Log
Con tanti file di log diversi, seguire il flusso del codice quando cerco un bug diventa molto complesso. Per questo motivo, scelgo di unire tutti i file di log in un file centralizzato.

Per unire i log da diversi file in un unico file centralizzato, utilizzo un semplice script PHP. Questo script legge i file di log specificati, ordina le righe in base ai timestamp e scrive il risultato in un nuovo file.

Decido di creare uno script personalizzato anziché usare una libreria di terze parti per diverse ragioni. Non ho bisogno di una gestione complessa; mi serve solo unire i file in un unico file centralizzato. Le librerie di terze parti richiedono l'installazione di un database e altre configurazioni, e possono avere costi elevati.

#### Funzioni PHP per Unire i Log

```php
<?php

// Funzione per ottenere i file di log da unire
function getLogFilesToMerge(array $channels): array {
    $currentDate = date('Y-m-d'); // Data corrente nel formato YYYY-MM-DD
    $logPath = storage_path('logs'); // Percorso dei log
    $logFiles = [];

    foreach ($channels as $channel) {
        $filePath = "$logPath/$channel-$currentDate.log"; // Costruisce il percorso del file di log
        if (file_exists($filePath)) {
            $logFiles[] = $filePath; // Aggiunge il file alla lista se esiste
        }
    }

    return $logFiles;
}

// Funzione per leggere e unire i log
function mergeLogs(array $files, string $outputFile): void {
    $logEntries = [];

    foreach ($files as $file) {
        $handle = fopen($file, 'r');
        if ($handle) {
            while (($line = fgets($handle)) !== false) {
                $logEntry = parseLogLine($line);
                if ($logEntry !== null) {
                    $logEntries[] = $logEntry;
                }
            }
            fclose($handle);
        }
    }

    // Ordina tutte le righe di log in base ai timestamp
    usort($logEntries, function ($a, $b) {
        return strcmp($a['timestamp'], $b['timestamp']);
    });

    // Scrivi le righe ordinate nel file di output
    $outputHandle = fopen($outputFile, 'w');
    foreach ($logEntries as $entry) {
        fwrite($outputHandle, $entry['line']);
    }
    fclose($outputHandle);
}

// Funzione per analizzare le righe di log
function parseLogLine(string $line): ?array {
    if (preg_match('/\[(\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d{6}\+\d{2}:\d{2})\]/', $line, $matches)) {
        return [
            'timestamp' => $matches[1],
            'line' => $line
        ];
    }
    return null;
}

// Esempio di uso delle funzioni
$logFiles = getLogFilesToMerge([
    'tests',
    'traits',
    'services',
]);

mergeLogs($logFiles, storage_path('logs/merged_file.log'));
```

### Logica dei Canali

Ogni sezione principale dell'applicazione avrà il suo canale dedicato, così come i servizi trasversali. I canali attualmente definiti sono:

- `tests`: Per i log relativi ai test dell'applicazione.
- `traits`: Per i log relativi alla gestione dei traits.

- `services`: Per i log dei servizi trasversali.

### Conclusione

La configurazione descritta consente di gestire i log in modo efficace, con timestamp precisi e la possibilità di unire i log in un unico file centralizzato per una più facile analisi. Questa struttura facilita il debugging e il monitoraggio dell'applicazione, mantenendo al contempo l'ordine e la chiarezza nei file di log.
