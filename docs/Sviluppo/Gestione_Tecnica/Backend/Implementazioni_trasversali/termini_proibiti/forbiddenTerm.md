# Classe `ForbiddenTerm`

<span style={{color: 'red', fontWeight: 'bold'}}>
    # ATTENZIONE! Questa classe non è in uso!
    **Ho preferito tenere la gestione dei termini proibiti separata dalla validazione per poter avere un maggior controllo durante la fase di test.**
</span>



## Scopo
La classe `ForbiddenTerm` è una regola di validazione personalizzata in Laravel progettata per verificare se un dato input contiene termini proibiti. Questa regola utilizza la classe `ForbiddenTermChecker` per analizzare il contenuto inviato e determinare se contiene termini inappropriati, come parolacce o espressioni offensive, sfruttando l'API Perspective di Google.

## Funzionamento

### 1. Inizializzazione
**Trigger**: L'utente inserisce un contenuto che deve essere verificato.  
**Azione**: La regola di validazione `ForbiddenTerm` viene applicata all'attributo del modello che si desidera validare.

### 2. Verifica dell'Istanza di `ForbiddenTermChecker`
**Verifica**: La classe utilizza `app(ForbiddenTermChecker::class)` per ottenere un'istanza di `ForbiddenTermChecker`.  
**Controllo**: Se l'istanza è `null`, viene utilizzata la callback `$fail` per registrare un messaggio di errore.

### 3. Analisi del Contenuto
**Richiesta API**: Il testo viene inviato all'API di Pe rspective per l'analisi.  
**Ricezione Risposta**: L'API restituisce un punteggio di tossicità per ciascun attributo richiesto.

### 4. Valutazione dei Risultati
**Confronto**: I punteggi di tossicità vengono confrontati con soglie predefinite.  
**Condizione**: Se uno o più punteggi superano le soglie, la callback `$fail` viene invocata con un messaggio di errore.

## Diagramma del Flusso

```plaintext
[Input Utente] --> [validate (ForbiddenTerm)]
      |
      v
[Istanzia ForbiddenTermChecker]
      |
      |--[Null?]-- Yes --> [Utilizza $fail] --> [Ritorna False]
      |                 |
      |                 No
      v
[Analisi del Contenuto con API Perspective]
      |
      v
[Valutazione Punteggi]
      |
      |--[Supera Soglia?]-- Yes --> [Utilizza $fail per segnalare l'errore]
      |                    |
      |                    No
      v
[Nessun Termine Proibito Trovato]
```

### Implementazione

#### Classe `ForbiddenTerm`

La classe `ForbiddenTerm` implementa la regola di validazione personalizzata per verificare la presenza di termini proibiti.

```php
namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;
use App\Services\ForbiddenTermChecker;
use Illuminate\Support\Facades\Log;

class ForbiddenTerm implements ValidationRule
{
    protected $language;
    protected $forbiddenTermChecker;

    public function __construct($language = 'en')
    {
        $this->language = $language;
        $this->forbiddenTermChecker = app(ForbiddenTermChecker::class);
        Log::channel('tests')->error('ForbiddenTerm');
    }

    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        Log::channel('tests')->error('ForbiddenTerm. Validate');

        if (is_null($this->forbiddenTermChecker)) {
            Log::channel('tests')->error('ForbiddenTermChecker è null');
            $fail(__('errors.forbidden_term_checker_null'));
            return;
        }

        if ($this->forbiddenTermChecker->containsForbiddenWords($value, $this->language)) {
            Log::channel('tests')->info('ForbiddenTermChecker è true');
            $fail('termine vietato'); // Questo verrà passato come messaggio dell'eccezione
        }
    }
}
```

### Utilizzo

Per utilizzare la regola di validazione personalizzata `ForbiddenTerm`, è necessario includerla nelle regole di validazione del modello o del componente Livewire.

#### Esempio di Utilizzo nel Componente Livewire

```php
namespace App\Http\Livewire;

use Livewire\Component;
use App\Rules\ForbiddenTerm;
use Illuminate\Support\Facades\Validator;

class TraitLibraryComponent extends Component
{
    public $categoryField;
    public $userLanguage = 'it';

    protected function validateInput($field, $table)
    {
        $validated = match ($table) {
            'traits_value_users' => $this->valueForm->getState(),
            'trait_key_commons' => $this->keyForm->getState(),
            'trait_categories' => $this->categoryForm->getState(),
            default => throw new \InvalidArgumentException(__('errors.table_not_exist')),
        };

        return Validator::make($validated, [
            $field => [
                'required',
                'string',
                'min:3',
                'max:30',
                'unique:' . $table . ',' . $field . ',NULL,id,user_id,' . auth()->id(),
                new ForbiddenTerm($this->userLanguage)
            ],
        ])->validate();
    }
}
```

### Eccezione sollevata per validazione fallita a causa del termine vietato

Quando l'utente inserisce un termine vietato, all'interno di `createNewCategory()`, viene sollevata l'eccezione `ValidationException`, essa contiene il messaggio presente nella regola personalizzata `ForbiddenTerm`, vedi qui sotto.

```php
if ($this->forbiddenTermChecker->containsForbiddenWords($value, $this->language)) {
    $fail('termine vietato'); // Questo messaggio verrà passato all'eccezione ValidationException
}
```


```php
public function createNewCategory(): void
    {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Action: Dentro il metodo createNewCategory');


        try {

            // Valido il dato da scrivere nel db. All'interno del metodo validateInput avviene anche il controllo per i termini vietati
            $validated = $this->validateInput($this->categoryField, 'trait_categories');
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: $validated: ', $validated);

            // Sanitizzo il dato prima di darlo in pasto al traduttore
            $sanitizedText = $this->sanitizeInput($validated[$this->categoryField]);
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: $sanitizedText: '. $sanitizedText);

            // Avvio la transazione atomica dell'operazione
            DB::beginTransaction();

            // Aggiungo manualmente user_id, questa categoria è assegnata univocamente all'utente autenticato
            $validated['user_id'] = auth()->id();

            // Traduco il testo e salvo la categoria
            $arrayValidateTranslated = $this->processTranslation($validated, $sanitizedText, null);
            Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory Action: $arrayValidateTranslated: ', $arrayValidateTranslated);

            // Salvo la categoria tradotta in tutte le lingue supportate
            TraitCategory::create($arrayValidateTranslated);

            // tutto è andato bene, confermo la transazione
            DB::commit();

            Log::channel('traits')->info('classe: TraitLibraryComponent metodo: createNewCategory action: dopo update');

            // Aggiorno la pulsantiera delle categorie
            $this->categories = TraitCategory::all();

            // chiudo il form per la creazione della categoria
            $this->showFormCreateCategory = false;

            // notifico con SweetAlert2 che l'operazione è andata a buon fine
            $this->dispatch('success', [
                'message' => __('traits.category_added'),
            ]);

        } catch (ValidationException $e) {

            Log::channel('tests')->info('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: dentro catch ValidationException' .$e->getMessage());

            if ($e->getMessage() === "termine vietato"){
                // notifico con SweetAlert2 che è la validazione non andata a buon fine
                $message = __('errors.forbidden_term_warning', [
                    'link' => "URL_DELLA_TUA_PAGINA_DELLE_NORME",
                ]);

                // Prelevo da $this->categoryForm->getState() il dato inserito nel form, non posso prelevarlo da $validated
                // in quanto, se c'è stata un eccezione sulla validazione, $validated non è stato creato
                $categoryFormData = $this->categoryForm->getState();

                $this->dispatch('forbiddenTermFound',[
                    'message' =>  $message,
                    'element' => $categoryFormData[$this->categoryField]
                ]);
            }
        }

        Log::channel('traits')->info('metodo edit eseguito con successo: ');
    }
```

### Conclusione

La classe `ForbiddenTerm` è una componente essenziale per garantire che i contenuti inseriti dagli utenti rispettino le linee guida della comunità. Integrando questa regola di validazione, puoi monitorare e gestire efficacemente i contenuti generati dagli utenti, proteggendo la comunità e mantenendo la reputazione della piattaforma.

## Tests

### ForbiddenTermTest

**Descrizione**: Questo test verifica la funzionalità della regola di validazione `ForbiddenTerm`. Il test assicura che la regola identifichi correttamente termini proibiti utilizzando il servizio `ForbiddenTermChecker` e che la validazione fallisca o passi correttamente in base al contenuto dell'input.

### Struttura del Test

1. **Setup**:
    - Inizializzazione di un mock della classe `ForbiddenTermChecker`.

2. **Esecuzione dei Test**:
    - **Verifica che la validazione fallisca per un termine proibito**: Simula che il metodo `containsForbiddenWords` ritorni `true` e verifica che la validazione fallisca.
    - **Verifica che la validazione passi per un termine consentito**: Simula che il metodo `containsForbiddenWords` ritorni `false` e verifica che la validazione passi.

### Codice del Test

```php
namespace Tests\Unit;

use Tests\TestCase;
use App\Rules\ForbiddenTerm;
use App\Services\ForbiddenTermChecker;
use Illuminate\Support\Facades\Validator;
use Mockery;

// Inizializzazione prima di ogni test
protected function setUp(): void
{
    parent::setUp();
    // Creazione del mock per ForbiddenTermChecker
    $this->forbiddenTermChecker = Mockery::mock(ForbiddenTermChecker::class);
}

public function testValidationFailsForForbiddenTerm()
{
    // Configura il mock per ritornare true quando contiene termini proibiti
    $this->forbiddenTermChecker->shouldReceive('containsForbiddenWords')
        ->andReturn(true);

    // Crea una nuova istanza della regola ForbiddenTerm con il mock
    $rule = new ForbiddenTerm('en', $this->forbiddenTermChecker);

    // Crea un'istanza del Validator con il testo e la regola di validazione
    $validator = Validator::make(['input' => 'Sei un idiota'], ['input' => $rule]);

    // Verifica che la validazione fallisca
    $this->assertFalse($validator->passes());
    // Verifica che il messaggio di errore sia quello atteso
    $this->assertEquals(
        'errors.forbidden_term_warning',
        $validator->errors()->first('input')
    );
}

public function testValidationPassesForCleanInput()
{
    // Configura il mock per ritornare false quando non contiene termini proibiti
    $this->forbiddenTermChecker->shouldReceive('containsForbiddenWords')
        ->andReturn(false);

    // Crea una nuova istanza della regola ForbiddenTerm con il mock
    $rule = new ForbiddenTerm('en', $this->forbiddenTermChecker);

    // Crea un'istanza del Validator con il testo e la regola di validazione
    $validator = Validator::make(['input' => 'clean word'], ['input' => $rule]);

    // Verifica che la validazione passi
    $this->assertTrue($validator->passes());
}

// Pulizia dei mock dopo ogni test
protected function tearDown(): void
{
    Mockery::close();
    parent::tearDown();
}
```

### Dettaglio delle Funzioni di Test

1. **Setup**:
    - `setUp`: Prima di ogni test, viene creato un mock di `ForbiddenTermChecker` utilizzando Mockery.

2. **Test di Fallimento della Validazione**:
    - `testValidationFailsForForbiddenTerm`:
        - Configura il mock di `ForbiddenTermChecker` per ritornare `true` quando il metodo `containsForbiddenWords` viene chiamato.
        - Crea un'istanza della regola `ForbiddenTerm` con il mock.
        - Utilizza il validatore di Laravel per verificare un testo contenente un termine proibito.
        - Verifica che la validazione fallisca e che il messaggio di errore corrisponda a `'errors.forbidden_term_warning'`.

3. **Test di Successo della Validazione**:
    - `testValidationPassesForCleanInput`:
        - Configura il mock di `ForbiddenTermChecker` per ritornare `false` quando il metodo `containsForbiddenWords` viene chiamato.
        - Crea un'istanza della regola `ForbiddenTerm` con il mock.
        - Utilizza il validatore di Laravel per verificare un testo che non contiene termini proibiti.
        - Verifica che la validazione passi.

4. **Pulizia**:
    - `tearDown`: Dopo ogni test, chiude i mock utilizzando `Mockery::close`.

### Conclusione

Il test `ForbiddenTermTest` verifica che la regola di validazione `ForbiddenTerm` funzioni correttamente, identificando termini proibiti attraverso la classe `ForbiddenTermChecker`. Assicura che la validazione fallisca quando l'input contiene termini proibiti e passi quando l'input è pulito. Questo test è essenziale per garantire che la validazione dei termini proibiti sia implementata correttamente nel sistema.