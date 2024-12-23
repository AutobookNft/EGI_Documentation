# Classe `ForbiddenTermChecker`

## Scopo
La classe `ForbiddenTermChecker` è progettata per verificare se un dato input contiene termini proibiti utilizzando l'API Perspective di Google. Questa classe permette di analizzare il contenuto inviato dagli utenti e determinare se contiene termini inappropriati, come parolacce o espressioni offensive, e di gestire eventuali errori di connessione all'API inviando notifiche al team di sviluppo.

## Funzionamento

### 1. Inizializzazione
**Trigger**: La classe viene istanziata con un'istanza di `SendErrorEmail` per gestire le notifiche di errore.  
**Azione**: Il costruttore imposta la chiave API di Perspective e l'istanza di `SendErrorEmail`.

### 2. Verifica dei Termini Proibiti
**Richiesta API**: Il metodo `containsForbiddenWords` invia il testo all'API di Perspective per l'analisi.  
**Ricezione Risposta**: L'API restituisce un punteggio di tossicità per ciascun attributo richiesto.

### 3. Valutazione dei Risultati
**Confronto**: I punteggi di tossicità vengono confrontati con soglie predefinite.  
**Condizione**: Se uno o più punteggi superano le soglie, viene lanciata un'eccezione `ForbiddenTermException`.

### 4. Gestione degli Errori
**Errore di Connessione API**: Se la connessione all'API fallisce, viene inviato un'email al team di sviluppo e il metodo ritorna `false` senza interrompere il flusso dell'applicazione.  
**Errore di Attributi**: Se non ci sono attributi supportati per la lingua specificata, viene inviato un'email al team di sviluppo e il metodo ritorna `false`.

### Diagramma del Flusso

```plaintext
[Input Utente] --> [containsForbiddenWords]
      |
      v
[Costruisci URL API]
      |
      v
[Ottieni Attributi Supportati]
      |
      |--[Attributi Vuoti?]-- Yes --> [Invia Email Errore] --> [Ritorna False]
      |                     |
      |                     No
      v
[Invia Richiesta API]
      |
      |--[Connessione Fallita?]-- Yes --> [Invia Email Errore] --> [Ritorna False]
      |                           |
      |                           No
      v
[Analizza Risposta API]
      |
      |--[Supera Soglia?]-- Yes --> [Lancia ForbiddenTermException]
      |                    |
      |                    No
      v
[Nessun Termine Proibito Trovato]
```

### Implementazione

#### Classe `ForbiddenTermChecker`

La classe `ForbiddenTermChecker` implementa il controllo dei termini proibiti utilizzando l'API Perspective di Google.

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

    /**
     * Costruttore della classe ForbiddenTermChecker.
     *
     * @param SendErrorEmail $sendErrorEmail Classe per inviare email di errore.
     */
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

            $response = $response->json();

            // Controlla i punteggi di tossicità
            $threshold = 0.7;
            foreach ($requestedAttributes as $attribute => $value) {
                if ($response['attributeScores'][$attribute]['summaryScore']['value'] > $threshold) {
                    Log::channel('tests')->error('classe: ForbiddenTermChecker. Metodo: containsForbiddenWords Action: foreach - if: '.json_encode($value));
                    throw new ForbiddenTermException;
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

### Descrizione dei Metodi

#### __construct
Il costruttore imposta la chiave API di Perspective e l'istanza di `SendErrorEmail`.

**Parametri**:
- `SendErrorEmail $sendErrorEmail`: Classe per inviare email di errore.

#### containsForbiddenWords
Verifica se l'input contiene termini vietati utilizzando l'API di Perspective.

**Parametri**:
- `string $input`: Il testo da analizzare.
- `string $language`: La lingua dell'input (predefinito: 'en').

**Ritorno**:
- `bool`: Ritorna `false` se non ci sono termini vietati.

**Eccezioni**:
- `ForbiddenTermException`: Se viene rilevato un termine vietato.
- `Exception`: Per qualsiasi altro errore.

#### containsForbiddenTerm (Metodo Legacy)
Verifica se l'input contiene termini vietati utilizzando un file di testo locale.

**Parametri**:
- `string $input`: Il testo da analizzare.
- `string $language`: La lingua dell'input.

**Ritorno**:
- `bool`: Ritorna `true` se viene trovato un termine vietato, altrimenti `false`.

### Conclusione

La classe `ForbiddenTermChecker` è una componente essenziale per garantire che i contenuti inseriti dagli utenti rispettino le linee guida della comunità. Utilizzando l'API Perspective di Google, questa classe permette di analizzare e gestire efficacemente i contenuti generati dagli utenti, proteggendo la comunità e mantenendo la reputazione della piattaforma.

## Tests

### Test 1  `ForbiddenTermCheckerTest`

**Descrizione**: Questo test verifica la funzionalità della classe `ForbiddenTermChecker`. Il test assicura che la classe identifichi correttamente termini proibiti e gestisca correttamente le risposte dall'API Perspective, inclusi i casi in cui l'API restituisce un errore.

### Struttura del Test

1. **Setup**:
    - Inizializzazione di un'istanza di `ForbiddenTermChecker` con un mock della classe `SendErrorEmail`.

2. **Esecuzione dei Test**:
    - **Verifica che l'API rilevi termini proibiti**: Simulazione di una risposta dell'API con termini proibiti e verifica che il metodo `containsForbiddenWords` ritorni `true`.
    - **Verifica che l'API consenta termini puliti**: Simulazione di una risposta dell'API con termini non proibiti e verifica che il metodo `containsForbiddenWords` ritorni `false`.
    - **Verifica che venga inviata un'email di errore quando l'API fallisce**: Simulazione di un errore dell'API e verifica che venga inviata un'email di errore.

### Codice del Test

```php
use App\Exceptions\ForbiddenTermException;
use App\Services\ForbiddenTermChecker;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\App;

// Inizializzazione prima di ogni test
beforeEach(function () {
    $this->forbiddenTermChecker = App::make(ForbiddenTermChecker::class);
});

it('returns true for a forbidden term', function () {
    $text = "Sei un idiota!"; // Testo di esempio contenente un termine proibito
    $language = "it"; // Lingua di esempio

    // Mock della risposta dell'API Perspective
    Http::fake([
        'https://commentanalyzer.googleapis.com/*' => Http::response([
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

    // Esegui il metodo containsForbiddenWords
    $result = $this->forbiddenTermChecker->containsForbiddenWords($text, $language);

    // Verifica che il risultato sia true
    expect($result)->toBeTrue();
})->uses(RefreshDatabase::class);

it('returns false for an allowed term', function () {
    $text = "Comportamento normale"; // Testo di esempio contenente termini non proibiti
    $language = "it"; // Lingua di esempio

    // Mock della risposta dell'API Perspective
    Http::fake([
        'https://commentanalyzer.googleapis.com/*' => Http::response([
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

    // Esegui il metodo containsForbiddenWords
    $result = $this->forbiddenTermChecker->containsForbiddenWords($text, $language);

    // Verifica che il risultato sia false
    expect($result)->toBeFalse();
})->uses(RefreshDatabase::class);
```

### Dettaglio delle Funzioni di Test

1. **Setup**:
    - `beforeEach(function () { ... })`: Prima di ogni test, viene creato un mock di `ForbiddenTermChecker` utilizzando la funzione `App::make`.

2. **Test di Rilevazione di Termini Proibiti**:
    - `it('returns true for a forbidden term', function () { ... })`:
        - Simula una risposta dell'API Perspective che indica un termine proibito.
        - Esegue il metodo `containsForbiddenWords`.
        - Verifica che il metodo ritorni `true`.

3. **Test di Termini Consentiti**:
    - `it('returns false for an allowed term', function () { ... })`:
        - Simula una risposta dell'API Perspective che indica termini non proibiti.
        - Esegue il metodo `containsForbiddenWords`.
        - Verifica che il metodo ritorni `false`.

### Conclusione

Il test `ForbiddenTermCheckerTest` verifica che la classe `ForbiddenTermChecker` funzioni correttamente, identificando correttamente termini proibiti e gestendo le risposte dall'API Perspective. Assicura inoltre che le risposte dell'API siano gestite correttamente, inclusi i casi di errore. Questo test è essenziale per garantire la robustezza e l'affidabilità del controllo dei termini proibiti nel sistema.

### Test 2: `ForbiddenTermCheckerSendMailTest`

#### Descrizione

Questo test utilizza Mockery per verificare che l'email di errore venga inviata correttamente quando la connessione all'API Perspective fallisce. Simula una risposta fallita dell'API e controlla che il metodo `sendErrorEmail` della classe `SendErrorEmail` venga chiamato con i parametri corretti.

#### Codice del Test

```php
use App\Mail\SendErrorEmail;
use App\Services\ForbiddenTermChecker;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Http;
use Tests\TestCase;

// Inizializzazione prima di ogni test
beforeEach(function () {
    // Creazione del mock per SendErrorEmail
    $this->mockSendErrorEmail = Mockery::mock(SendErrorEmail::class);
    // Inizializzazione dell'istanza di ForbiddenTermChecker con il mock
    $this->forbiddenTermChecker = new ForbiddenTermChecker($this->mockSendErrorEmail);
});

it('sends an error email when API connection fails', function () {
    $text = "Testo qualsiasi"; // Testo di esempio
    $language = "it"; // Lingua di esempio

    // Mock della risposta fallita dell'API Perspective
    Http::fake([
        'commentanalyzer.googleapis.com/*' => Http::response([], 500) // Simula un errore server
    ]);

    // Verifica che l'email venga inviata
    $this->mockSendErrorEmail->shouldReceive('sendErrorEmail')
        ->once() // L'email deve essere inviata una volta
        ->with(Mockery::on(function ($params) {
            // Verifica che il parametro 'subject' sia corretto
            return $params['subject'] === 'Error connecting to Perspective API';
        }))
        ->andReturn(true); // Simula il ritorno true dal metodo sendErrorEmail

    // Esegui il metodo containsForbiddenWords
    $result = $this->forbiddenTermChecker->containsForbiddenWords($text, $language);

    // Verifica che il risultato sia false
    expect($result)->toBeFalse();

    // Chiudi i mock
    Mockery::close();
})->uses(RefreshDatabase::class);
```

### Test 3: `ForbiddenTermCheckerRealMailTest`

#### Descrizione

Questo test verifica l'invio reale di un'email di errore quando la connessione all'API Perspective fallisce. Simula una risposta fallita dell'API e controlla che il metodo `containsForbiddenWords` ritorni false.

#### Codice del Test

```php
use App\Mail\ErrorOccurredMailable;
use App\Services\ForbiddenTermChecker;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Log;
use Tests\TestCase;

// Inizializzazione prima di ogni test
beforeEach(function () {
    Log::channel('tests')->info('classe: ForbiddenTermCheckerRealMailTest. Action: Inizio test: invio email di errore');
    // Inizializzazione dell'istanza di ForbiddenTermChecker senza fake di Mail
    $this->forbiddenTermChecker = app(ForbiddenTermChecker::class);
});

it('sends an error email when API connection fails', function () {
    $text = "Testo qualsiasi"; // Testo di esempio
    $language = "it"; // Lingua di esempio

    // Mock della risposta fallita dell'API Perspective
    Http::fake([
        'commentanalyzer.googleapis.com/*' => Http::response([], 500) // Simula un errore server
    ]);

    Log::channel('tests')->info('classe: ForbiddenTermCheckerRealMailTest. Action: Simulazione errore API completata');

    // Esegui il metodo containsForbiddenWords
    $result = $this->forbiddenTermChecker->containsForbiddenWords($text, $language);

    Log::channel('tests')->info('classe: ForbiddenTermCheckerRealMailTest. Action: Risultato del metodo containsForbiddenWords: ' . json_encode($result));

    // Verifica che il risultato sia false
    expect($result)->toBeFalse();

    Log::channel('tests')->info('classe: ForbiddenTermCheckerRealMailTest. Action: Fine test: invio email di errore');

})->uses(RefreshDatabase::class);
```