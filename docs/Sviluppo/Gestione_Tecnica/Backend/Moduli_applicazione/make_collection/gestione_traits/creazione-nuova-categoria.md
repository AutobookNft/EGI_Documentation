# Procedura per il Salvataggio di una Nuova Categoria in Laravel con Livewire e Filament

Questa procedura descrive i passaggi necessari per il salvataggio di una nuova categoria nel sistema, gestendo le validazioni, la traduzione dei dati e il controllo dei termini vietati. La procedura è implementata all'interno di un componente Livewire e utilizza Filament per la gestione dei form.

### Input tramite Filament Form

Il seguente codice mostra il Filament form per la creazione di una nuova Category:

```php
public function categoryForm(Form $form): Form
    {

        // Rilevo se siamo in fase di test
        $duringTest = app()->environment('testing') ? false : true;

        // Nel caso delle categorie, il nome del campo corrisponde esattamente alla sigla della lingua
        $categoryField = $this->userLanguage;

        return $form->schema([
            TextInput::make($categoryField)
                ->label(__('traits.create_a_new_category'))
                ->required($duringTest)
                ->maxLength(30)
                ->minLength(3),
        ])->statePath('categoryData');
    }
```
### 1. Metodo `createNewCategory`
Il dato viene salvato da Livewire con il metodo createNewCategory
Il metodo `createNewCategory` gestisce il processo di creazione di una nuova categoria. Esegue diverse operazioni tra cui la verifica dei permessi, la validazione dell'input, la sanitizzazione dei dati, la traduzione e il salvataggio nel database.

```php
public function createNewCategory(): void
    {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Action: Dentro il metodo createNewCategory');

        // Verifica se l'utente ha il permesso di creare traits
        if (!Auth::user()->can('create own traits')) {
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: dentro if (!Auth::user()->can(\'create own traits\')');
            abort(403, 'Unauthorized action.');
        }

        // Rilevo se siamo in fase di test, permette di gestire la sospensionedella transazione DB durante la fase del test, altrimenti impossibile da eseguire
        $useTransaction = app()->environment('testing') ? false : true;

        try {

            // Valido il dato da scrivere nel db
            $validated = $this->validateInput($this->categoryField, 'trait_categories', $this->categoryForm->getState());
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: ValidatedInput: ', $validated);

            // Sanitizzo il dato prima di darlo in pasto al traduttore
            $sanitizedText = $this->sanitizeInput($validated[$this->categoryField]);
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: $sanitizedText: '. $sanitizedText);

            /** Verifico che non sia un termine proibito
            * PERCHè INIZIALIZZARE LOCALMENTE...
            * Inizializza il servizio ForbiddenTermService localmente piuttosto che assegnarlo a una proprietà nella mount.
            * Livewire non può serializzare oggetti complessi come servizi durante la gestione delle richieste AJAX.
            * Se tenti di assegnare un servizio a una proprietà nella mount, Livewire non sarà in grado di mantenere
            * lo stato dell'oggetto tra le richieste, causando un errore di "Call to a member function ... on null".
            * Inizializzando il servizio localmente, assicuri che sia sempre disponibile quando necessario. */
            $forbiddenTermService = app(ForbiddenTermService::class);
            $forbiddenTermService->checkForbiddenTerm($sanitizedText, $this->userLanguage);

            // Avvio la transazione atomica dell'operazione solo se non siamo in fase di test
            if ($useTransaction) {
                $this->serverTest=true;
                DB::beginTransaction();
            }

            // Aggiungo manualmente user_id, questa categoria è assegnata univocamente all'utente autenticato
            $validated['user_id'] = auth()->id();

            // Traduco il testo e salvo la categoria
            $arrayValidateTranslated = $this->processTranslation($validated, $sanitizedText, null);
            Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory Action: $arrayValidateTranslated: ', $arrayValidateTranslated);

            // Salvo la categoria tradotta in tutte le lingue supportate
            TraitCategory::create($arrayValidateTranslated);

            // tutto è andato bene, confermo la transazione
            if ($useTransaction) {
                $this->serverTest=false;
                DB::commit();
            }
            Log::channel('traits')->info('classe: TraitLibraryComponent metodo: createNewCategory action: dopo commit');

            // Aggiorno la pulsantiera delle categorie
            $this->categories = TraitCategory::all();

            // chiudo il form per la creazione della categoria
            $this->showFormCreateCategory = false;

            // notifico con SweetAlert2 che l'operazione è andata a buon fine
            $this->dispatch('success', [
                'message' => __('traits.category_added'),
            ]);

        } catch (ValidationException $e) {

            $failedRules = $e->validator->failed();

            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: message: ' .$e->getMessage());
            Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: failedRules: ', $failedRules);

            if (isset($failedRules[$this->categoryField])) {
                $errors = [];
                if (isset($failedRules[$this->categoryField]['Unique'])) {
                    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Failed rule: UNIQUE');
                    $errors["categoryData.$this->categoryField"] = ['unique' => __('errors.unique', ['attribute' => __('traits.category')])];
                    $this->dispatch('generic_error', [
                        'message' => __('errors.unique'),
                    ]);
                }
                if (isset($failedRules[$this->categoryField]['Required'])) {
                    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Failed rule: REQUIRED');
                    $errors["categoryData.$this->categoryField"] = ['required' => __('errors.required', ['attribute' => __('traits.category')])];
                    $this->dispatch('generic_error', [
                        'message' => __('errors.required', ['attribute' => __('traits.category')]),
                    ]);
                }

                // aggiorno l'errorBag
                $this->setErrorBag($errors);
            }

            // c'è stato qualche errore, annullo l'operazione
            if ($useTransaction) {
                $this->serverTest=false;
                DB::rollBack();
            }

        }catch (ForbiddenTermException $e){

            Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory. Action: dentro catch ForbiddenTermException' .$e->getMessage());

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

        Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory. Action: metodo edit eseguito con successo: ');
    }
```

### 2. Metodo `validateInput`
Il metodo `validateInput` gestisce la validazione dell'input utilizzando un'istanza di `Validator`.

```php
private function validateInput($field, $table, $getState = null)
{

    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: validateInput Action: $field AND $table '.$field.' '.$table);

    // leggo il valore da verificare
    $value= $getState[$field];
    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: validateInput Action: $value) '.$value);

    // termino su quale tabella stiamo facendo la verifica
    $validated = match ($table) {
        'traits_value_users' => $this->valueForm->getState(),
        'trait_key_commons' => $this->keyForm->getState(),
        'trait_categories' => $this->categoryForm->getState(),
        default => throw new \InvalidArgumentException(__('errors.table_not_exist')),
    };
    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: validateInput Action: $validated: ', $validated);

    /*
    *    Spiegazione di unique:
    *
    *    unique: $table, $field:
    *    Questa parte indica che il campo deve essere unico nella tabella $table nella colonna $field.
    *    NULL,id: Specifica che stai ignorando un ID particolare, in questo caso NULL.
    *    Questo è utile se stai aggiornando un record e vuoi ignorare il record corrente.
    *    user_id,' . auth()->id(): Aggiunge una clausola WHERE che assicura che il controllo di unicità venga applicato solo ai record
    *    con lo stesso user_id dell'utente autenticato. In pratica, controlla che il campo sia unico tra i record dello stesso utente.
    */

    if ($table == 'traits_value_users') {
        $validator = Validator::make([$field => $value], [
            $field => [
                'required',
                new UniqueKeyValue($this->key_filter),
            ],
        ]);

    }elseif ($table == 'trait_key_commons') {
        $validator = Validator::make([$field => $value], [
            $field => 'required',
        ],
    );

    }elseif ($table == 'trait_categories') {
        $validator = Validator::make([$field => $value], [
            $field => [
                'required',
                'unique:' . $table . ',' . $field . ',NULL,id,user_id,' . auth()->id(),
            ],
        ],
    );
    }

    if ($validator->fails()) {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: validateInput Action: ERRORE DI VALIDAZIONE!');
    }

    return $validator->validate();

}
```

### 3. Metodo `checkForbiddenTerm`
Il metodo `checkForbiddenTerm` utilizza `ForbiddenTermChecker` per verificare se l'input contiene termini vietati.

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

    Log::channel('traits')->error('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: DENTRO IL METODO');

    // Inizializzo il ForbiddenTermChecker,
    // vedi: http://localhost:3000/frangette-docs/wiky/solved_problems/serializzazione-delle-dipendenze-in-livewire#soluzione-di-livewire

    $forbiddenTermChecker = app(ForbiddenTermChecker::class);

    // Verifica che $this->forbiddenTermChecker non sia null, può accadere se la classe non viene correttamente istanziata
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

        Log::channel('traits')->error('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: la classe non viene inizializzata correttamente');

        // Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
        $checkSendedEmail = $this->sendErrorEmail->sendErrorEmail($params);
        Log::channel('traits')->error('classe: TraitLibraryComponent. Metodo: checkForbiddenTerm Action: $checkSendedEmail '. $checkSendedEmail);
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

### 4. Metodo `sanitizeInput`
Il metodo `sanitizeInput` utilizza `HTMLPurifier` per pulire l'input.

```php
/*
*  HTMLpurifier:
*
*  è una libreria che permette di filtrare e pulire l'HTML,
*  in modo da evitare attacchi XSS. In questo caso, la libreria è stata
*  utilizzata per pulire i dati inseriti dall'utente nei form.
*
*  NOTA BENE:
*  La libreria è stata inizializzata all'interno del metodo e non nella funzione mount
*  in quanto la libreria non può essere serializzata e quindi non può essere passata a una property di
*  Livewire
*/
private function sanitizeInput($input)
{
    try {

        /**
         * Verifica se l'input è una stringa
         *
         * Questo controllo verifica che l'input sia una stringa. In un'applicazione web standard,
         * gli input dell'utente vengono generalmente ricevuti come stringhe attraverso i form HTML.
         * Tuttavia, ci sono scenari in cui l'input potrebbe non essere una stringa:
         *
         * 1. Attacchi di Manipolazione: Un utente malintenzionato potrebbe manipolare i dati inviati al server
         *    utilizzando strumenti come Postman, curl, o modificando i valori degli input tramite il console
         *    developer del browser, inviando un tipo di dato diverso da una stringa.
         *
         * 2. Problemi di Serializzazione: Errori nel processo di serializzazione/deserializzazione potrebbero
         *    causare l'invio di un tipo di dato non stringa.
         *
         * 3. Errori di Programmazione: Errori logici o di programmazione potrebbero portare all'invio di un
         *    tipo di dato non stringa alla funzione di sanitizzazione.
         *
         * In questi casi, è importante notificare il team di sviluppo per monitorare e rispondere prontamente
         * a potenziali problemi di sicurezza o di logica applicativa. Qui, inviamo un'email con i dettagli
         * dell'input non valido, inclusi il file e la linea del codice in cui si è verificato l'errore.
         */
        if (!is_string($input)) {
            Log::channel('traits')->error('Classe: TraitLibraryComponent. Metodo: processTranslation. Action: Un input non stringa è stato ricevuto durante la sanitizzazione.');
            throw new \InvalidArgumentException(__('errors.the_input_must_be_a_string'));
        }

        // Configurazione di HTMLPurifier
        $config = HTMLPurifier_Config::createDefault();

        /**
         * Questi sotto sono alcuni parametri di configurazione di HTMLPurifier:
         * p: Paragrafi
         * b: Testo in grassetto
         * a[href]: Link con l'attributo href
         * i: Testo in corsivo
         *
         * In questo caso non inserisco nessun parametro in quanto per CategorY, Kye, Value
         * non c'è bisogno di formattare il testo in nessun modo, quindi è inutile consentirlo
         * Un esempio di configurazione con alcuni parametri poteva essere: $config->set('HTML.Allowed', 'p,b,a[href],i');
         */

        $config->set('HTML.Allowed', ''); // Se non c'è nessuna configurazione significa che nessun HTML permesso.

        // Creazione del purificatore con la configurazione
        $purifier = new HTMLPurifier($config);

        // Sanitizzazione dell'input
        $sanitizedInput = $purifier->purify($input);

        return $sanitizedInput;

    } catch (\InvalidArgumentException $e) {

        // Prepara il mailable con i dati dell'errore
        $params = [
            'to' => config('services.error_email.devteam_to'),
            'subject' => 'Invalid Input Received',
            'sysMessage' => 'Un input non stringa è stato ricevuto durante la sanitizzazione.',
            'dynamicData' => [
                json_encode($input),
                'Classe' => 'TraitLibraryComponent',
                'Metodo' => 'sanitizeInput',
                ]
            ];

            // Invia un'email al team di sviluppo
            SendErrorEmail::sendErrorEmail($params);

        // Gestione degli errori di input non valido
        Log::channel('traits')->error('Classe: TraitLibraryComponent. Metodo: processTranslation. Action: Errore di input : ' . $e->getMessage());
        throw $e;

    } catch (\Exception $e) {

        // Prepara il mailable con i dati dell'errore
        $params = [
            'to' => config('services.error_email.devteam_to'),
            'subject' => 'Errore generico',
            'sysMessage' => 'Verificare la gestione di HTMLPurifier.',
            'dynamicData' => [
                json_encode($input),
                'Classe' => 'TraitLibraryComponent',
                'Metodo' => 'sanitizeInput',
                ]
            ];

            // Invia un'email al team di sviluppo
            SendErrorEmail::sendErrorEmail($params);

        // Gestione delle eccezioni generiche
        Log::channel('traits')->error('Classe: TraitLibraryComponent. Metodo: processTranslation. Action: Errore generico durante la sanitizzazione dell\'input: ' . $e->getMessage());
        throw $e;
    }
}
```

### 5. Metodo `processTranslation`
Il metodo `processTranslation` traduce il testo in diverse lingue utilizzando `GoogleTranslate`.

```php
/**
 * Questo metodo prepara l'array associativo che contiene il nome dei campi e il valore tradotto da scrivere nel db.
 * --------------------------------------------------------------------------------------------------------------------
 * NOTA BENE: Le tabelle delle categorie, delle chiavi e dei valori sono state progettate in modo che i campi
 * delle lingue abbiano un prefisso che corrisponde alla sigla della lingua. Ad esempio, nella tabelle trait_categories
 * per la lingua inglese il campo si chiama en, per la lingua italiana il campo si chiama it, ecc.
 * Invece per le tabelle trait_keys_common e traits_value_user il campo per la lingua inglese si chiama key_en e value_en
 * --------------------------------------------------------------------------------------------------------------------
 * L'array così creato viene poi passato al metodo create per salvare i dati nel db.
 *
 * @param $validated: array che contiene i dati validati da inserire nel db
 * @param $sanitizedText: testo da tradurre e scrivere nell'array $validated
 * @param $prefix: campo che contiene il prefisso, per le categorie è null, per le chiavi è key, per i valori è value
 */
private function processTranslation($validated, $sanitizedText, $prefix = null)
{

    // $tr = new GoogleTranslate();
    $tr = $this->translator();

    if ($tr) {

        // setta il traduttore al riconsocimento automatico della lingua da cui tradurre
        $tr->setSource();

        foreach ($this->languages as $lang) {
            // setta il traduttore per la lingua in cui tradurre
            $tr->setTarget($lang);
            // traduce il testo
            $translatedText = $tr->translate($sanitizedText);
            // aggiunge il prefisso se c'è
            $field = $prefix ? $prefix . '_' . $lang : $lang;
            // aggiunge all'array il testo tradotto nell'array
            $validated[$field] = e($translatedText); //
            Log::channel('traits')->info("Traduzione per {$sanitizedText} in {$lang}: {$translatedText}");
        }

    } else {

        // Creo il log e aggiungo anche data e ora
        Log::channel('traits')->error("Classe: TraitLibraryComponent. Metodo: processTranslation. Action: traduttore non inizializzato" . now());

        // Prepara l'array dei dati per la email
        $params = [
            'to' => config('services.error_email.devteam_to'),
            'subject' => 'Errore in metodo nella traduzione dei traits',
            'sysMessage' => 'Il traduttore non è stato inizializzato correttamente',
            'dynamicData' => [
                'Classe' => 'TraitLibraryComponent',
                'Metodo' => 'processTranslation',
                'Data ora' => now(),
            ]
        ];

        // Avviso il team di sviluppo che Perspective non sta svolgendo il proprio lavoro
        $checkSendedEmail = $this->sendErrorEmail->sendErrorEmail($params);
        Log::channel('traits')->error('classe: TraitLibraryComponent. Metodo: processTranslation Action: $checkSendedEmail '. $checkSendedEmail);

    }

    return $validated;
}
```

### Test
Esempio di test per verificare il salvataggio di una nuova categoria e la gestione degli errori:

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;
use Livewire\Livewire;
use App\Livewire\Traits\TraitLibraryComponent;
use App\Models\TraitCategory;
use App\Models\TraitKeyCommon;
use App\Models\TraitsValueUser;
use Illuminate\Support\Facades\App;
use Illuminate\Support\Facades\Log;
use Mockery;
use Stichoza\GoogleTranslate\GoogleTranslate;
use Spatie\Permission\Models\Role;

/**
 * Class LibraryTraitsTest
 *
 * Questa classe di test verifica il funzionamento del componente Livewire TraitLibraryComponent.
 * I test coprono vari scenari di creazione e gestione delle categorie, chiavi e valori dei trait.
 * In particolare, i test si concentrano su:
 * - La creazione di nuove categorie, chiavi e valori con le relative traduzioni.
 * - La validazione dei campi richiesti durante la creazione.
 * - La gestione delle eccezioni e degli errori di validazione.
 * - L'integrazione con il traduttore GoogleTranslate.
 *
 * La classe utilizza RefreshDatabase per garantire che ogni test abbia un database pulito.
 */
class LibraryTraitsTest extends TestCase
{
    use RefreshDatabase;

    protected $user;
    protected $languages;
    protected $category_filter;
    protected $userLanguage;

    /**
     * Metodo setUp
     *
     * Questo metodo viene eseguito prima di ogni test e prepara l'ambiente di test.
     * In particolare, il metodo:
     * - Esegue le migrazioni del database.
     * - Esegue il seeder per i ruoli e i permessi.
     * - Crea un utente di test e gli assegna il ruolo di "Super Admin".
     * - Imposta la lingua dell'utente di test.
     *
     * @return void
     */
    protected function setUp(): void
    {
        parent::setUp();
        $this->artisan('migrate');
        $this->artisan('db:seed', ['--class' => 'RolesPermissionsTableSeeder']);

        $this->user = User::factory()->create();
        $this->languages = config('app.languages'); // Ottieni le lingue dalla configurazione

        // Creazione del ruolo "Super Admin" se non esiste
        if (!Role::where('name', 'Super Admin')->exists()) {
            Role::create(['name' => 'Super Admin']);
        }

        // Assegna il ruolo "Super Admin" all'utente creato
        $this->user->assignRole('Super Admin');

        $this->userLanguage = App::getLocale();
        Log::channel('tests')->info('Class: LibraryTraitsTest. Method. setUp. Action: $this->userLanguage'. $this->userLanguage);
    }

    /**
     * Questo test verifica che il metodo createNewCategory() funzioni correttamente quando si inserisce una nuova categoria con traduzioni.
     * La verifica include:
     * - Impostazione dell'utente autenticato per il test.
     * - Creazione di una nuova categoria utilizzando il modulo Livewire.
     * - Verifica che la categoria originale sia stata inserita nel database.
     * - Verifica che le traduzioni della categoria siano state inserite correttamente nel database.
     */
    public function test_create_new_category_with_translations_success()
    {
        $this->actingAs($this->user);

        Livewire::test(TraitLibraryComponent::class)
            ->set('categoryData.en', 'Test Category')
            ->call('createNewCategory')
            ->assertHasNoErrors();

        // Verifica che la categoria originale sia stata inserita
        $this->assertDatabaseHas('trait_categories', [
            'en' => 'Test Category',
            'user_id' => $this->user->id,
        ]);

        // Non uso un Mock della classe ma il vero traduttore per il test
        $translator = new GoogleTranslate();

        // Verifica la traduzione in tutte le lingue configurate
        $languages = config('app.languages');

        foreach ($languages as $lang) {
            $translatedText = $translator->setSource()->setTarget($lang)->translate('Test Category');
            $this->assertDatabaseHas('trait_categories', [
                $lang => e($translatedText),
                'user_id' => $this->user->id,
            ]);
        }

        Log::channel('tests')->info("Fine del test senza Mockery");
    }


    /**
    *    Il test ha diversi scopi:
    *
    *    1. Validazione dei Dati di Input: Assicura che l'input vuoto per il campo value non superi la validazione required.
    *    2. Integrità del Database: Verifica che il valore non venga memorizzato nel database se la validazione fallisce.
    *
    *    Contesto Completo: Crea tutte le entità correlate necessarie per simulare correttamente il contesto in cui il metodo createNewValue viene chiamato.
    *    1. In sostanza, questo test garantisce che:
    *    2. Il sistema non accetti input non validi.
    *    3. I dati non validi non siano memorizzati nel database.
    *
    *    Il metodo createNewValue gestisca correttamente la validazione e le entità correlate.
    */
    public function test_create_new_category_required_error()
    {

        Log::channel('tests')->info('DENTRO IL TEST REQUIRE');

        // Imposta l'utente autenticato per il test
        $this->actingAs($this->user);
        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO ACTING AS');

        // Usa il metodo Livewire::test con l'interazione del modulo di Filament
        $component = Livewire::test(TraitLibraryComponent::class)
            ->set("categoryData.$this->userLanguage", '')  // Usa il form di Filament per impostare il campo vuoto
            ->call('createNewCategory');  // Chiama il metodo per creare la nuova categoria

        // Aggiungi log per debug
        Log::channel('tests')->info('Stato delle proprietà del componente:', $component->get('categoryData'));
        Log::channel('tests')->info('Errori del componente:'. json_encode($component->get('errors')));

        $component->assertHasErrors("categoryData.$this->userLanguage");  // Verifica che ci siano errori di validazione

        $this->assertDatabaseMissing('trait_categories', [
            'en' => '',
            'user_id' => $this->user->id,
        ]);

    }

    /**
     * Questo test verifica che il metodo createNewCategory() gestisca correttamente un errore di unicità
     * quando si tenta di creare una categoria con un nome già esistente per lo stesso utente.
     * La verifica include:
     * - Impostazione dell'utente autenticato per il test.
     * - Creazione di una categoria esistente associata all'utente autenticato.
     * - Controllo che la categoria "Existing Category" sia stata creata correttamente nel database.
     * - Tentativo di creare una nuova categoria con lo stesso nome utilizzando il componente Livewire.
     * - Verifica che ci siano errori di validazione per la categoria duplicata.
     * - Verifica che non sia stata creata una seconda categoria con lo stesso nome per lo stesso utente.
     *
     * Note:
     * - Il test utilizza log per il debug del processo di creazione della categoria e della verifica degli errori di validazione.
     * - Verifica la presenza di un solo record della categoria "Existing Category" per l'utente autenticato nel database.
     */
    public function test_create_new_category_unique_error()
    {
        $this->actingAs($this->user);

        TraitCategory::create([
            'en' => 'Existing Category',
            'user_id' => $this->user->id,
        ]);

        // Verifica che la categoria "Existing Category" sia stata creata correttamente
        $this->assertDatabaseHas('trait_categories', [
            'en' => 'Existing Category',
            'user_id' => $this->user->id,
        ]);

        $component = Livewire::test(TraitLibraryComponent::class)
            ->set('categoryData.en', 'Existing Category')  // Usa il form di Filament per impostare il campo vuoto
            ->call('createNewCategory');  // Chiama il metodo per creare la nuova categoria

        // Aggiungi log per debug
        Log::channel('tests')->info('Stato delle proprietà del componente:', $component->get('categoryData'));
        Log::channel('tests')->info('Errori del componente:'. json_encode($component->get('errors')));

        $component->assertHasErrors('categoryData.en');  // Verifica che ci siano errori di validazione
        $this->assertCount(1, TraitCategory::where('en', 'Existing Category')->where('user_id', $this->user->id)->get());
    }


    /**
     * Questo test verifica che il metodo createNewKey() crei correttamente una nuova chiave con traduzioni in tutte le lingue supportate.
     * La verifica include:
     * - Impostazione dell'utente autenticato per il test.
     * - Creazione di una categoria associata all'utente autenticato.
     * - Creazione di una nuova chiave attraverso il componente Livewire.
     * - Controllo che non ci siano errori di validazione.
     * - Verifica che la chiave originale sia stata inserita correttamente nel database.
     * - Utilizzo del traduttore reale per verificare che le traduzioni della chiave siano presenti in tutte le lingue configurate.
     *
     * Note:
     * - Il test utilizza log per il debug del processo di creazione della chiave e della verifica delle traduzioni.
     * - Verifica la presenza delle traduzioni della chiave nel database.
     */
    public function test_create_new_key_with_translations_success()
    {
        $this->actingAs($this->user);

        Log::channel('traits')->info('Metodo: test_create_new_key_with_translations_success - Inizio metodo');

        // Crea una categoria prima di aggiungere una chiave
        $category = TraitCategory::create([
            'en' => 'Test Category',
            'user_id' => $this->user->id,
        ]);
        Log::channel('traits')->info('Metodo: test_create_new_key_with_translations_success - Categoria creata');

        Livewire::test(TraitLibraryComponent::class)
            ->set('keyData.key_en', 'Test Key') // Imposta il valore della chiave nel form
            ->set('category_filter', $category->id) // Imposta la categoria correlata alla chiave
            ->call('createNewKey')
            ->assertHasNoErrors();

        Log::channel('traits')->info('Metodo: test_create_new_key_with_translations_success - Chiave creata');

        // Verifica che la chiave originale sia stata inserita
        $this->assertDatabaseHas('trait_key_commons', [
            'key_en' => 'Test Key',
            'user_id' => $this->user->id,
            'trait_category_id' => $category->id,
        ]);

        Log::channel('traits')->info('Metodo: test_create_new_key_with_translations_success - Chiave verificata');

        // Non uso un Mock della classe ma il vero traduttore per il test
        $translator = new GoogleTranslate();

        // Verifica la traduzione in tutte le lingue configurate
        $languages = config('app.languages');

        foreach ($languages as $lang) {
            $translatedText = $translator->setSource()->setTarget($lang)->translate('Test Key');
            $this->assertDatabaseHas('trait_key_commons', [
                "key_{$lang}" => e($translatedText),
            ]);
        }

        Log::channel('tests')->info("Fine del test senza Mockery");
    }



    /**
     * Questo test verifica che il metodo createNewKey() fallisca se nessun dato viene inserito.
     * La verifica include:
     * - Impostazione dell'utente autenticato per il test.
     * - Creazione di una categoria associata all'utente autenticato.
     * - Tentativo di inserire una nuova chiave con un valore vuoto attraverso il componente Livewire.
     * - Controllo che ci siano errori di validazione per il campo chiave vuoto.
     * - Controllo che il valore vuoto non sia stato inserito nel database.
     *
     * Note:
     * - Il test utilizza log per il debug dello stato delle proprietà del componente e degli errori di validazione.
     * - Verifica che il valore vuoto non sia presente nel database.
     */
    public function test_create_new_key_required_error()
    {
        // Imposta l'utente autenticato per il test
        $this->actingAs($this->user);

        // Crea una categoria prima di aggiungere una chiave
        $category = TraitCategory::create([
            'en' => 'Test Category',
            'user_id' => $this->user->id,
        ]);

        // Usa il metodo Livewire::test con l'interazione del modulo di Filament
        $component = Livewire::test(TraitLibraryComponent::class)
            ->set("keyData.key_$this->userLanguage", '')  // Usa il form di Filament per impostare il campo vuoto
            ->set('category_filter', $category->id) // Assicurati di impostare category_filter
            ->call('createNewKey');  // Chiama il metodo per creare la nuova categoria

        // Aggiungi log per debug
        Log::channel('tests')->info('Stato delle proprietà del componente:', $component->get('keyData'));
        Log::channel('tests')->info('Errori del componente:'. json_encode($component->get('errors')));

        // Verifica che ci siano errori di validazione
        $component->assertHasErrors("keyData.key_$this->userLanguage");

        // Verifica che il valore vuoto non sia stato inserito nel database
        $this->assertDatabaseMissing('trait_key_commons', [
            'key_en' => '',
            'user_id' => $this->user->id,
            'trait_category_id' => $category->id,
        ]);
    }


    /**
     * Questo test verifica che il metodo createNewValue() funzioni correttamente.
     * La verifica include:
     * - Creazione di una categoria e di una chiave associate all'utente autenticato.
     * - Inserimento di un nuovo valore attraverso il componente Livewire.
     * - Controllo che il valore originale sia stato correttamente inserito nel database.
     * - Controllo che il valore sia stato tradotto correttamente in tutte le lingue configurate e salvato nel database.
     *
     * Note:
     * - Non viene utilizzato un mock della classe di traduzione, ma il vero traduttore per il test.
     * - Le entità HTML nel testo tradotto vengono decodificate per garantire la correttezza della traduzione nel database.
     */
    public function test_create_new_value_with_translations_success()
    {
        $this->actingAs($this->user);

        // Crea una categoria prima di aggiungere una chiave
        $category = TraitCategory::create([
            'en' => 'Test Category',
            'user_id' => $this->user->id,
        ]);

        // Crea una chiave prima di aggiungere un valore
        $key = TraitKeyCommon::create([
            'key_en' => 'Test Key',
            'trait_category_id' => $category->id,
            'user_id' => $this->user->id,
        ]);

        Livewire::test(TraitLibraryComponent::class)
            ->set('valueData.value_en', 'Test Value')
            ->set('key_filter', $key->id)
            ->call('createNewValue')
            ->assertHasNoErrors();

        // Verifica che il valore originale sia stato inserito
        $this->assertDatabaseHas('traits_value_users', [
            'value_en' => 'Test Value',
            'user_id' => $this->user->id,
            'traits_key_common_id' => $key->id,
        ]);

        // Non uso un mock della classe di traduzione ma il vero traduttore per il test
        $translator = new GoogleTranslate();

        // Verifica la traduzione in tutte le lingue configurate
        $languages = config('app.languages');

        foreach ($languages as $lang) {
            $translatedText = $translator->setSource()->setTarget($lang)->translate('Test Value');
            $decodedTranslatedText = e($translatedText); // Decodifica le entità HTML

            $this->assertDatabaseHas('traits_value_users', [
                'value_' . $lang => $decodedTranslatedText,
            ]);
        }

        Log::channel('tests')->info("Fine del test senza Mockery");
    }


    // Questo test verifica che il metodo createNewCategory() fallisca se nessun dato viene ineserito
    // In altre parole testa che la validazione required funzioni
    public function test_create_new_value_required_error()
    {

        Log::channel('tests')->info('DENTRO IL TEST REQUIRE');

        // Imposta l'utente autenticato per il test
        $this->actingAs($this->user);

        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO ACTING AS');

        // Crea una categoria prima di aggiungere una chiave
        $category = TraitCategory::create([
            'en' => 'Test Category',
            'user_id' => $this->user->id,
        ]);
        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO CATEGORI');

        // Crea una chiave prima di aggiungere un valore
        $key = TraitKeyCommon::create([
            'key_en' => 'Test Key',
            'trait_category_id' => $category->id,
            'user_id' => $this->user->id,
        ]);
        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO CHIAVI');

        $field = "value_$this->userLanguage";

        // Usa il metodo Livewire::test con l'interazione del modulo di Filament
        Livewire::test(TraitLibraryComponent::class)
            ->set($field, '')  // Usa il form di Filament per impostare il campo vuoto
            ->set('key_filter', $key->id)
            ->call('createNewValue')  // Chiama il metodo per creare la nuova categoria
            ->assertHasErrors('valueData.'.$field);  // Verifica che ci siano errori di validazione
        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO CHIAMATA');

        // Verifica che il valore nulla non sia stato inserito
        $this->assertDatabaseMissing('traits_value_users', [
            "value_$this->userLanguage" => '',
            'user_id' => $this->user->id,
            'traits_key_common_id' => $key->id,
        ]);
        Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO VERIFICA');

    }

    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}

```
### Checklist per `createNewCategory`

#### Verifica dei Permessi

#### Canale dei log `Traits`

1. **Controllo dei permessi:**
   - Verifica che l'utente abbia il permesso di creare traits.
   - C'è un Log degli eventi per tracciare il controllo dei permessi.

#### Validazione dell'Input

2. **Validazione del campo:**
   - Usa il metodo `validateInput` per validare il campo della categoria.
   - Ci sono log per di verificare per ogni riga di codice significativa.

#### Sanitizzazione dell'Input

3. **Sanitizzazione dell'input:**
   - Sanitizza l'input usando il metodo `sanitizeInput`.
   - C'è una verifica che controlla se l'input non foisse di tipo stringa (può accadere solo se c'è un intervento malevolo o per un errore di programmazione), in questo caso, oltre a scrivere un log, viene anche spedida una mail al devTeam.
   - In caso di un qualsiasi altro errore generico, viene spedita una mail con il dettaglio dell'errore e viene anche scritto un log.
   - La configurazione di HTMLPurifier è stabilita in modo tale da impedire l'inserimento di qualasiasi tag HTML.

#### Controllo dei Termini Proibiti

4. **Controllo dei termini proibiti:**
   - Inizializza il servizio `ForbiddenTermService` localmente all'interno del metodo.
   - Usa il metodo `checkForbiddenTerm` del servizio per verificare la presenza di termini proibiti.
   - Aggiunto log per tracciare il controllo dei termini proibiti.
   - [Vedi la documentazione relativa a `ForbiddenTermChecker`](../termini_proibiti/panoramica-termini-proibiti)

#### Transazione Atomica

5. **Gestione delle transazioni:**
   - Avvia una transazione se non siamo in fase di test.
   - Aggiungi log per verificare l'inizio della transazione.

#### Traduzione dell'Input

6. **Traduzione del testo:**
   - Usa il metodo `processTranslation` per tradurre il testo in tutte le lingue supportate.
   - Aggiungi log per verificare le traduzioni.

#### Salvataggio nel Database

7. **Salvataggio della categoria:**
   - Salva la categoria tradotta nel database.
   - Aggiungi log per verificare il salvataggio.

#### Commit/Rollback della Transazione

8. **Commit/Rollback della transazione:**
   - Conferma la transazione se tutto va bene, altrimenti annulla la transazione in caso di errore.
   - Aggiungi log per tracciare il commit o il rollback.

#### Aggiornamento della Vista e Notifiche

9. **Aggiornamento della vista:**
   - Aggiorna la lista delle categorie.
   - Chiudi il form per la creazione della categoria.

10. **Notifiche:**
    - Notifica l'utente con un messaggio di successo usando SweetAlert2.
    - Aggiungi log per tracciare le notifiche.

#### Gestione delle Eccezioni

11. **Gestione delle eccezioni di validazione:**
    - Gestisci le eccezioni di validazione.
    - Aggiorna l'errorBag con gli errori di validazione.
    - Aggiungi log per tracciare gli errori di validazione.

12. **Gestione delle eccezioni di termini proibiti:**
    - Gestisci le eccezioni di termini proibiti.
    - Notifica l'utente con un messaggio di errore.
    - Aggiungi log per tracciare le eccezioni di termini proibiti.

## Metodo: `createNewCategory`

Il metodo `createNewCategory` consente di creare una nuova categoria di traits per l'utente autenticato. Include vari passaggi per la validazione, sanitizzazione, traduzione e salvataggio dell'input, oltre alla gestione delle transazioni e delle eccezioni.

### Codice del Metodo

```php
public function createNewCategory(): void
{
    Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Action: Dentro il metodo createNewCategory');

    // Verifica se l'utente ha il permesso di creare traits
    if (!Auth::user()->can('create own traits')) {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: dentro if (!Auth::user()->can(\'create own traits\')');
        abort(403, 'Unauthorized action.');
    }

    // Rilevo se siamo in fase di test, permette di gestire la sospensionedella transazione DB durante la fase del test, altrimenti impossibile da eseguire
    $useTransaction = app()->environment('testing') ? false : true;

    try {
        // Valido il dato da scrivere nel db
        $validated = $this->validateInput($this->categoryField, 'trait_categories', $this->categoryForm->getState());
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: ValidatedInput: ', $validated);

        // Sanitizzo il dato prima di darlo in pasto al traduttore
        $sanitizedText = $this->sanitizeInput($validated[$this->categoryField]);
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: $sanitizedText: '. $sanitizedText);

        // verifico che non sia un termine proibito
        $this->checkForbiddenTerm($sanitizedText, $this->userLanguage);

        // Avvio la transazione atomica dell'operazione solo se non siamo in fase di test
        if ($useTransaction) {
            $this->serverTest=true;
            DB::beginTransaction();
        }

        // Aggiungo manualmente user_id, questa categoria è assegnata univocamente all'utente autenticato
        $validated['user_id'] = auth()->id();

        // Traduco il testo e salvo la categoria
        $arrayValidateTranslated = $this->processTranslation($validated, $sanitizedText, null);
        Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory Action: $arrayValidateTranslated: ', $arrayValidateTranslated);

        // Salvo la categoria tradotta in tutte le lingue supportate
        TraitCategory::create($arrayValidateTranslated);

        // tutto è andato bene, confermo la transazione
        if ($useTransaction) {
            $this->serverTest=false;
            DB::commit();
        }
        Log::channel('traits')->info('classe: TraitLibraryComponent metodo: createNewCategory action: dopo commit');

        // Aggiorno la pulsantiera delle categorie
        $this->categories = TraitCategory::all();

        // chiudo il form per la creazione della categoria
        $this->showFormCreateCategory = false;

        // notifico con SweetAlert2 che l'operazione è andata a buon fine
        $this->dispatch('success', [
            'message' => __('traits.category_added'),
        ]);

    } catch (ValidationException $e) {
        $failedRules = $e->validator->failed();

        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: message: ' .$e->getMessage());
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory Action: failedRules: ', $failedRules);

        if (isset($failedRules[$this->categoryField])) {
            $errors = [];
            if (isset($failedRules[$this->categoryField]['Unique'])) {
                Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Failed rule: UNIQUE');
                $errors["categoryData.$this->categoryField"] = ['unique' => __('errors.unique', ['attribute' => __('traits.category')])];
                $this->dispatch('generic_error', [
                    'message' => __('errors.unique'),
                ]);
            }
            if

 (isset($failedRules[$this->categoryField]['Required'])) {
                Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Failed rule: REQUIRED');
                $errors["categoryData.$this->categoryField"] = ['required' => __('errors.required', ['attribute' => __('traits.category')])];
                $this->dispatch('generic_error', [
                    'message' => __('errors.required', ['attribute' => __('traits.category')]),
                ]);
            }

            // aggiorno l'errorBag
            $this->setErrorBag($errors);
        }

        // c'è stato qualche errore, annullo l'operazione
        if ($useTransaction) {
            $this->serverTest=false;
            DB::rollBack();
        }

    } catch (ForbiddenTermException $e) {
        Log::channel('traits')->info('classe: TraitLibraryComponent. Metodo: createNewCategory. Action: dentro catch ForbiddenTermException' .$e->getMessage());

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

    Log::channel('traits')->info('metodo edit eseguito con successo: ');
}
```

### Note

- **Logging**: Utilizza il logging in modo estensivo per tracciare ogni fase del processo. Questo è utile per il debug e la manutenzione.
- **Gestione delle Eccezioni**: Gestisce le eccezioni di validazione e di termini proibiti separatamente, notificando l'utente con messaggi appropriati.
- **Transazioni**: Usa transazioni per garantire l'atomicità dell'operazione di creazione della categoria.
- **Traduzioni**: Traduzioni del testo in tutte le lingue supportate utilizzando il servizio di traduzione.

Questa checklist e documentazione ti aiuteranno a mantenere il metodo `createNewCategory` ben organizzato e documentato, facilitando la comprensione e la manutenzione futura del codice.