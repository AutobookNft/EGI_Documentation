# Gestione termini proibiti

```php
namespace App\Services;

use App\Exceptions\ApiConnectionException;
use App\Exceptions\ForbiddenTermException;
use App\Mail\SendErrorEmail;
use App\Util\PerspectiveAttributes;
use Exception;
use Illuminate\Support\Facades\File;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Http;

class ForbiddenTermChecker
{
    protected $apiKey;
    protected $sendErrorEmail;

    public function __construct(SendErrorEmail $sendErrorEmail)
    {
        Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: __construct Action: Dentro il metodo');
        $this->apiKey = config('services.perspective.api_key');
        $this->sendErrorEmail = $sendErrorEmail;
    }

    /**
     * Verifica se l'input contiene termini vietati utilizzando l'API di Perspective.
     *
     * Questo metodo invia l'input all'API di Perspective per analizzarlo e determinare
     * se contiene termini vietati in base agli attributi di tossicità supportati per
     * la lingua specificata. Se l'API rileva termini vietati, viene lanciata un'eccezione.
     * In caso di errore di connessione all'API, viene inviato un'email al team di sviluppo
     * e il flusso non viene interrotto.
     *
     * @param string $input Il testo da analizzare.
     * @param string $language La lingua dell'input (predefinito: 'en').
     * @return bool Ritorna false se non ci sono termini vietati.
     * @throws ForbiddenTermException Se viene rilevato un termine vietato.
     * @throws Exception Per qualsiasi altro errore.
     */
    public function containsForbiddenWords($input, $language = 'en')
    {
        Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: Dentro il metodo');

        // Costruisci l'URL per l'API di Perspective
        $url = 'https://commentanalyzer.googleapis.com/v1alpha1/comments:analyze?key=' . $this->apiKey;

        Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: DOPO URL');

        // Ottieni gli attributi supportati per la lingua corrente
        $requestedAttributes = PerspectiveAttributes::getAttributesForLanguage($language);

        if (empty($requestedAttributes)) {
            // Prepara l'array dei dati per la email
            $params = [
                'to' => config('services.error_email.devteam_to'),
                'subject' => 'Nessun attributo di Perspective',
                'sysMessage' => 'Nessun attributo di Perspective per la lingua specificata.',
                'dynamicData' => [
                    'Classe' => 'ForbiddenTermChecker',
                    'Metodo' => 'containsForbiddenWords',
                    'language' => $language,
                    'requestedAttributes' => $requestedAttributes
                ],
           ];

            Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $params', $params);

            // Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
            $checkSendedEmail = $this->sendErrorEmail->sendErrorEmail($params);
            Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $checkSendedEmail '. $checkSendedEmail);
            return false; // Esco senza lanciare l'eccezione perché il flusso non deve fermarsi

        }

        Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $requestedAttributes: ', $requestedAttributes);

        $data = [
            'comment' => ['text' => $input],
            'languages' => [$language],
            'requestedAttributes' => $requestedAttributes
        ];

        try {
            $response = Http::post($url, $data);

            if ($response->failed()) {

                Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $response->failed()'. $response);

                // Prepara l'array dei dati per la email
                $params = [
                    'to' => config('services.error_email.devteam_to'),
                    'subject' => 'Error connecting to Perspective API',
                    'sysMessage' => 'Error connecting to Perspective API',
                    'dynamicData' => [
                        'Classe' => 'ForbiddenTermChecker',
                        'Metodo' => 'containsForbiddenWords',
                        'response' => $response,
                    ]
                ];

                Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $params', $params);

                // Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
                $checkSendedEmail = $this->sendErrorEmail->sendErrorEmail($params);
                Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: $checkSendedEmail '. $checkSendedEmail);
                return false; // Esco senza lanciare l'eccezione perché il flusso non deve fermarsi
            }

            // $response = $response->json();
            // Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: response: '. $response);

            // Controlla i punteggi di tossicità
            $threshold = 0.6;
            foreach ($requestedAttributes as $attribute => $value) {
                if ($response['attributeScores'][$attribute]['summaryScore']['value'] > $threshold) {
                    Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: foreach - if: '.json_encode($value));
                    return true;
                }
            }

            Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: NESSUN TERMINE VIETATO TROVATO');
            return false;

        } catch (Exception  $e) {

             Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: Exception $e: '. $e);
             throw $e; // Propaga l'eccezione

        }
    }

    /**
     * Check if the given input contains any forbidden term.
     *
     * @param  string  $input  The input to check.
     * @param  string  $language  The language of the input.
     * @return bool True if forbidden term is found, otherwise false.
     */
    // NOTA BENE:
    // ATTUALMENTE QUESTO METODO NON VIENE USATO, AL SUO POSTO SI USA containsForbiddenWords
    // Rimane comunque un'opzione, qualora si volesse usare un file di testo per i termini vietati
    public function containsForbiddenTerm($input, $language)
    {
        $path = resource_path("lang/$language/forbidden_words.txt");
        if (!File::exists($path)) {
            throw new \Exception('Language file for forbidden words not found.');
        }

        $fileContent = File::get($path);
        $forbiddenWords = array_map('trim', explode(',', $fileContent));

        $return = in_array(strtolower($input), $forbiddenWords);
        Log::channel('tests')->info('classe: ForbiddenTermChecker. Metodo: containsForbiddenTerm Action: $return: ' . $return);

        return $return;
    }
}

```

### **Descrizione**:
Il metodo, `containsForbiddenWords`, utilizza l'API di Perspective per analizzare l'input e verificare la presenza di termini vietati. L'API di Perspective fornisce una serie di attributi di tossicità che vengono utilizzati per determinare se l'input contiene contenuti inappropriati.

### **Flusso di Lavoro**:
1. **Log di Ingresso**: Viene registrato un log per indicare l'inizio dell'analisi del metodo.
2. **Configurazione della Richiesta**: 
    - Costruzione dell'URL di richiesta utilizzando la chiave API.
    - Recupero degli attributi supportati per la lingua specificata tramite la classe `PerspectiveAttributes`.
    - Preparazione del payload della richiesta con il testo da analizzare, la lingua e gli attributi richiesti.
3. **Invio della Richiesta**:
    - Invio della richiesta all'API di Perspective utilizzando GuzzleHTTP.
4. **Gestione della Risposta**:
    - Se la richiesta fallisce, viene registrato un errore e viene inviato un'email al team di sviluppo senza interrompere il flusso.
    - Se la richiesta ha successo, la risposta viene decodificata e i punteggi di tossicità vengono verificati rispetto a una soglia predefinita (0.7).
    - Se uno qualsiasi dei punteggi supera la soglia, viene lanciata un'eccezione `ForbiddenTermException`.
5. **Gestione delle Eccezioni**:
    - In caso di errore di connessione all'API, viene inviato un'email di avviso al team di sviluppo, senza interrompere il flusso di lavoro.
    - Se viene lanciata un'eccezione diversa, questa viene propagata ulteriormente.

## **Dettagli Tecnici**:

### **Metodo: containsForbiddenWords**

- **Parametri**:
  - `string $input`: Il testo da analizzare.
  - `string $language`: La lingua dell'input (default: 'en').

- **Ritorno**:
  - `bool`: Ritorna false se non ci sono termini vietati.

- **Eccezioni**:
  - `ForbiddenTermException`: Se viene rilevato un termine vietato.
  - `Exception`: Per qualsiasi altro errore.

### **Classe di Utilità: PerspectiveAttributes**

La classe `PerspectiveAttributes` contiene una mappa degli attributi supportati per ciascuna lingua e un metodo per ottenere gli attributi supportati per una lingua specifica.
al 01/06/2024 le lingue gestite sono quelle riportate qui sotto. Nota che le ultime supportano solo l'inglese. 

```php
class PerspectiveAttributes
{
    public static $attributes = [
        'TOXICITY' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'de', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'es', 'sv'],
        'SEVERE_TOXICITY' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'sv'],
        'IDENTITY_ATTACK' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'sv'],
        'INSULT' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'sv'],
        'PROFANITY' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'sv'],
        'THREAT' => ['ar', 'zh', 'cs', 'nl', 'en', 'fr', 'hi', 'hi-Latn', 'id', 'it', 'ja', 'ko', 'pl', 'pt', 'ru', 'sv'],
        'SEXUALLY_EXPLICIT'=> ['en'],
    ];

    public static function getAttributesForLanguage($language)
    {
        $attributes = [];

        foreach (self::$attributes as $attribute => $languages) {
            if (in_array($language, $languages)) {
                $attributes[$attribute] = new \stdClass();
            }
        }

        return $attributes;
    }
}
```

### **Gestione degli Errori**

In caso di errore di connessione all'API di Perspective, viene inviato un'email al team di sviluppo contenente i dettagli dell'errore. Questo aiuta a mantenere il flusso dell'applicazione senza interrompere l'utente finale.

```php
// Prepara il mailable con i dati dell'errore
$params = [
    'to' => config('services.error_email.devteam_to'),
    'subject' => 'Error connecting to Perspective API',
    'sysMessage' => 'Error connecting to Perspective API',
    'dynamicData' => $response->body(),
    'file' => __FILE__,
    'line' => __LINE__
];

// Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
SendErrorEmail::sendErrorEmail($params);
```

## **Test**

```php
<?php

use App\Exceptions\ForbiddenTermException;
use App\Services\ForbiddenTermChecker;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\App;

beforeEach(function () {
    $this->forbiddenTermChecker = App::make(ForbiddenTermChecker::class);
});

it('throws an exception for a forbidden term', function () {
    $text = "Sei un idiota!";
    $language = "it";

    // Mock della risposta dell'API Perspective
    Http::fake([
        'commentanalyzer.googleapis.com/*' => Http::response([
            'attributeScores' => [
                'TOXICITY' => [
                    'summaryScore' => [
                        'value' => 0.8 // Un valore sopra il threshold
                    ]
                ],
                'SEVERE_TOXICITY' => [
                    'summaryScore' => [
                        'value' => 0.6 // Valore per evitare undefined index error
                    ]
                ],
                'IDENTITY_ATTACK' => [
                    'summaryScore' => [
                        'value' => 0.5 // Valore per evitare undefined index error
                    ]
                ],
                'INSULT' => [
                    'summaryScore' => [
                        'value' => 0.9 // Un valore sopra il threshold
                    ]
                ],
                'PROFANITY' => [
                    'summaryScore' => [
                        'value' => 0.4 // Valore per evitare undefined index error
                    ]
                ],
                'THREAT' => [
                    'summaryScore' => [
                        'value' => 0.3 // Valore per evitare undefined index error
                    ]
                ]
            ]
        ], 200)
    ]);

    expect(fn() => $this->forbiddenTermChecker->containsForbiddenWords($text, $language))
        ->toThrow(ForbiddenTermException::class);
})->uses(RefreshDatabase::class);

it('does not throw an exception for an allowed term', function () {
    $text = "Comportamento normale";
    $language = "it";

    // Mock della risposta dell'API Perspective
    Http::fake([
        'commentanalyzer.googleapis.com/*' => Http::response([
            'attributeScores' => [
                'TOXICITY' => [
                    'summaryScore' => [
                        'value' => 0.4 // Un valore sotto il threshold
                    ]
                ],
                'SEVERE_TOXICITY' => [
                    'summaryScore' => [
                        'value' => 0.2 // Valore per evitare undefined index error
                    ]
                ],
                'IDENTITY_ATTACK' => [
                    'summaryScore' => [
                        'value' => 0.1 // Valore per evitare undefined index error
                    ]
                ],
                'INSULT' => [
                    'summaryScore' => [
                        'value' => 0.2 // Valore per evitare undefined index error
                    ]
                ],
                'PROFANITY' => [
                    'summaryScore' => [
                        'value' => 0.1 // Valore per evitare undefined index error
                    ]
                ],
                'THREAT' => [
                    'summaryScore' => [
                        'value' => 0.2 // Valore per evitare undefined index error
                    ]
                ]
            ]
        ], 200)
    ]);

    $this->forbiddenTermChecker->containsForbiddenWords($text, $language);

    expect(true)->toBeTrue(); // Se non viene lanciata un'eccezione, il test passa
})->uses(RefreshDatabase::class);
```

## checkForbiddenTerm

```php
/**
 * Verifica se l'input contiene termini vietati utilizzando il ForbiddenTermChecker.
 *
 * Questo metodo inizializza il ForbiddenTermChecker e verifica se l'input contiene termini vietati.
 * Se la classe ForbiddenTermChecker non viene istanziata correttamente, viene inviato un avviso via email
 * al team di sviluppo e l'operazione viene interrotta senza lanciare un'eccezione.
 * Se vengono rilevati termini vietati, viene lanciata un'eccezione ForbiddenTermException.
 *
 * @param string $input Il testo da analizzare.
 * @param string $language La lingua dell'input.
 * @return bool Ritorna false se la classe ForbiddenTermChecker è null o se l'email di errore è stata inviata con successo.
 * @throws ForbiddenTermException Se viene rilevato un termine vietato.
 */
public function checkForbiddenTerm($input, $language)
{
    // Inizializzo il ForbiddenTermChecker
    $forbiddenTermChecker = app(ForbiddenTermChecker::class);

    // Verifica che $forbiddenTermChecker non sia null
    if (is_null($forbiddenTermChecker)) {
        Log::channel('traits')->error('Classe: TraitLibraryComponent. Metodo: createNewCategory. Action: $forbiddenTermChecker è nullo');
        
        // Prepara l'array dei dati per la email
        $params = [
            'to' => config('services.error_email.devteam_to'),
            'subject' => 'Errore ForbiddenTermChecker',
            'sysMessage' => 'app(ForbiddenTermChecker::class) ritorna null, controllare il servizio ForbiddenTermChecker.',
            'dynamicData' => [
                'Classe' => 'TraitLibraryComponent',
                'Metodo' => 'checkForbiddenTerm',
            ]
        ];

        Log::channel('tests')->error('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: la classe non viene inizializzata correttamente');

        // Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
        $checkSendedEmail = $this->sendErrorEmail->sendErrorEmail($params);
        Log::channel('tests')->error('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: $checkSendedEmail '. $checkSendedEmail);
        return false; // Esco senza lanciare l'eccezione perché il flusso non deve fermarsi
    }

    // Controllo parole proibite
    if ($forbiddenTermChecker->containsForbiddenWords($input, $language)) {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: dentro la if di containsForbiddenWords');
        // Se la parola è proibita, lancio un'eccezione che si propagata fino alla visualizzazione del messaggio per l'utente
        throw new ForbiddenTermException;
    }
}
```

## Approfondiamo a cosa serve il metodo  `checkForbiddenTerm`

### Panoramica

La funzione `checkForbiddenTerm` nella classe `TraitLibraryComponent` è progettata per verificare se un determinato input contiene termini proibiti utilizzando il servizio `ForbiddenTermChecker`. In caso di errore nell'inizializzazione della classe `ForbiddenTermChecker`, viene inviato un avviso al team di sviluppo.

#### Dettagli della Funzione

1. **Inizializzazione del `ForbiddenTermChecker`**:
   - Utilizza il metodo `app(ForbiddenTermChecker::class)` per istanziare la classe `ForbiddenTermChecker`.
   - Se l'istanza è `null`, viene loggato un errore e viene inviata un'email di avviso al team di sviluppo.

2. **Verifica dell'Istanza di `ForbiddenTermChecker`**:
   - Se l'istanza di `ForbiddenTermChecker` è `null`, viene loggato un messaggio di errore.
   - Viene preparato un array `$params` con i dettagli dell'errore, incluso il destinatario dell'email, il soggetto, il messaggio di sistema e i dati dinamici.
   - Viene utilizzata la funzione `sendErrorEmail` per inviare l'email di errore.
   - La funzione ritorna `false` senza lanciare un'eccezione per non interrompere il flusso dell'applicazione.

3. **Verifica dei Termini Proibiti**:
   - Se l'istanza di `ForbiddenTermChecker` è valida, viene chiamato il metodo `containsForbiddenWords` per verificare i termini proibiti.
   - Se vengono trovati termini proibiti, viene lanciata un'eccezione `ForbiddenTermException`.

#### Parametri della Funzione

- **$input**: Il testo che deve essere analizzato.
- **$language**: La lingua del testo.

### Esempio di Utilizzo. Vedi createNewCategory, all'interno della classe TraitLibraryComponent

```php
$input = "Testo da verificare";
$language = "it";

try {
    
    $this->checkForbiddenTerm($input, $language);

} catch (ForbiddenTermException $e) {
    
    // L'eccezione gestirà l'emissione dell'evento di errore
    $message = __('errors.forbidden_term_warning', [
        'link' => "URL_DELLA_TUA_PAGINA_DELLE_NORME",
    ]);
    $title =__('errors.exception.NotAllowedTermException');

    // Emissione dell'evento di errore
    $this->dispatch('forbiddenTermFound', [
        'title' => $title,
        'message' => $message
    ]);
}
```

#### Log degli Errori e Notifiche

- Gli errori di inizializzazione della classe `ForbiddenTermChecker` vengono loggati nel canale `traits`.
- Le notifiche di errore vengono inviate via email al team di sviluppo utilizzando la funzione `sendErrorEmail`.

#### Note Importanti

- Assicurarsi che la configurazione dell'email nel file `config/services.php` sia corretta.
- Verificare che l'istanza di `ForbiddenTermChecker` venga correttamente istanziata per evitare problemi nell'invio delle notifiche di erro

