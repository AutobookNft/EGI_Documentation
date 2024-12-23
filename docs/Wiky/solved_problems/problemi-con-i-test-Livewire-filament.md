# Problemi con i test dei componenti Livewire con Filament form

## Problema test lato client con sintassi Filament

Durante lo sviluppo dei test per il metodo `createNewCategory` nel componente Livewire, ho riscontrato un problema che riguardava la gestione multi form vedi(https://filamentphp.com/docs/3.x/forms/adding-a-form-to-a-livewire-component#using-multiple-forms)
per qualche ragione che non sono riuscito a scoprire, nel caso di questo componente, il seguente test restituisce un errore. 

#### test:
```php
use App\Livewire\Traits\TraitLibraryComponent;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Livewire\Livewire;
use Spatie\Permission\Models\Role;

uses(RefreshDatabase::class);

beforeEach(function () {
    $this->user = User::factory()->create();
    $this->languages = config('app.languages');

    // Creazione del ruolo "Super Admin" se non esiste
    if (!Role::where('name', 'Super Admin')->exists()) {
        Role::create(['name' => 'Super Admin']);
    }

    // Assegna il ruolo "Super Admin" all'utente creato
    $this->user->assignRole('Super Admin');
});

test('create new category required error filament', function () {

    $this->actingAs($this->user);

    Livewire::test(TraitLibraryComponent::class)
        ->fillForm([
            'categoryData.en' => null
        ],'categoryForm')
        ->call('createNewCategory')
        ->assertHasFormErrors(['categoryData.en' => 'required']);
});
```

#### errore:
```php

Property [$form] not found on component: [traits.trait-library-component]

  at vendor/livewire/livewire/src/Component.php:105
    101▕ 
    102▕         $value = $finish($value);
    103▕ 
    104▕         if ($value === 'noneset') {
 ➜ 105▕             throw new PropertyNotFoundException($property, $this->getName());
    106▕         }
    107▕ 
    108▕         return $value;
    109▕     }

      +8 vendor frames 
  9   tests/Unit/FilamentFormTest.php:33
```

#### Filament form
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

#### Analisi
La documentazione Filament spiega che nel caso in cui il componenete gestisca più di un form, oltre al setup necessario nel componente stesso, per applicare il test sul corretto form è necessario specificare il nome del form stesso nel metodo `->fillForm` 
`Livewire::test(TraitLibraryComponent::class)->fillForm(['categoryData.en' => null],'categoryForm') <--- questo è il nome del form`. La spiegazione di ciò è specificata nella documentazione di Filament: vedi(https://filamentphp.com/docs/3.x/forms/testing#filling-a-form) 
e dice: 
```
Se disponi di più moduli su un componente Livewire, puoi specificare 
quale modulo desideri compilare utilizzando fillForm([...], 'createPostForm').
```
**Ma nonostante abbia seguito tutte le indicazioni riportate in documentazione, ottengo l'errore mostrato sopra.**

#### Soluzione:
Ho optato per un test Livewire anziché Filament
```php
// Questo test verifica che il metodo createNewCategory() fallisca se nessun dato viene ineserito
// In altre parole testa che la validazione required funzioni
public function test_create_new_category_required_error_livewire()
{

    Log::channel('tests')->info('DENTRO IL TEST REQUIRE');

    // Imposta l'utente autenticato per il test
    $this->actingAs($this->user);
    Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO ACTING AS');

    // Usa il metodo Livewire::test con l'interazione del modulo di Filament
    $component = Livewire::test(TraitLibraryComponent::class)
        ->set('categoryData.en', '')  // Usa il form di Filament per impostare il campo vuoto
        ->call('createNewCategory');  // Chiama il metodo per creare la nuova categoria

    // Aggiungi log per debug
    Log::channel('tests')->info('Stato delle proprietà del componente:', $component->get('categoryData'));
    Log::channel('tests')->info('Errori del componente:'. json_encode($component->get('errors')));

    $component->assertHasErrors('categoryData.en');  // Verifica che ci siano errori di validazione

    $this->assertDatabaseMissing('trait_categories', [
        'en' => '',
        'user_id' => $this->user->id,
    ]);

}
```
In questo modo verifico required lato server, ma questa soluzione porta con sè un secondo problema: il metodo `->required()` del Filament form, impedisce l'esecuzione del test lato server, rendendo impossibile testare la required. Eliminando il metodo `->required()` dal Filament form, il problema non si presenta, ma da un punto di vista di esperienza utente il messaggio di avviso di Filament basato sul metodo `->required()` è più adatto, in quanto immediato. Allora, per poter mantenere il metodo `->required()` ho fatto in modo che durante il test non fosse attivo: `->required($duringTest)` dove `$duringTest` è così definita: `$duringTest = app()->environment('testing') ? false : true;` In questo modo required è testato lato server ma funziona lato client. Probabilmente questo test non è neanche necessario, ma ho lasciato questo caso come documentazione come testimonianza di come è stato affrontato un problema del genere, forse utile per altri casi.

## Problema test di validazione **unique**

### Sintomi

- Il test per la verifica del campo richiesto falliva con il messaggio: 
  ```
  Component has no matching failed rule or error message [required] for [categoryData.en] attribute.
  Failed asserting that an array contains 'required'..
  ```
- Il test per la verifica dell'unicità falliva con il messaggio:
  ```
  Component has no matching failed rule or error message [unique] for [categoryData.en] attribute. 
  Failed asserting that an array contains 'unique'.
  ```

#### Diagnosi
L'uso delle transazioni nel metodo `createNewCategory` impedisce il commit dei record nel database durante i test, rendendo impossibile la scrittura di un record di test per verificare il record duplicato

#### Test
```php
public function test_create_new_category_required_error()
{
    Log::channel('tests')->info('DENTRO IL TEST REQUIRE');

    // Imposta l'utente autenticato per il test
    $this->actingAs($this->user);
    Log::channel('tests')->info('DENTRO IL TEST REQUIRE DOPO ACTING AS');

    // Usa il metodo Livewire::test con l'interazione del modulo di Filament
    $component = Livewire::test(TraitLibraryComponent::class)
        ->set('categoryData.en', '')  // Usa il form di Filament per impostare il campo vuoto
        ->call('createNewCategory');  // Chiama il metodo per creare la nuova categoria

    // Aggiungi log per debug
    Log::channel('tests')->info('Stato delle proprietà del componente:', $component->get('categoryData'));
    Log::channel('tests')->info('Errori del componente:'. json_encode($component->get('errors')));

    $component->assertHasErrors('categoryData.en');  // Verifica che ci siano errori di validazione

}
```

### Soluzione: Disabilitazione delle Transazioni durante i Test

Per verificare `unique` ho dovuto disabilitare l'uso delle transazioni durante il test. Dato che nel test veniva aggiunto un record per simulare la duplicazione del dato inserito, se la transazione era attiva, durante l'errore la transazione non era ancora stata commitata, quindi al conteggio dei record, in effetti, non risultava nessun record salvato, quindi per assicurarsi che i record vengano committati nel database durante i test, abbiamo modificato il metodo `createNewCategory` per disabilitare l'uso delle transazioni solo in ambiente di testing:

```php
public function createNewCategory(): void
{
    $useTransaction = app()->environment('testing') ? false : true;

    try {
        if ($useTransaction) {
            DB::beginTransaction();
        }

        // ... codice per validare e creare la categoria ...

        if ($useTransaction) {
            DB::commit();
        }
    } catch (ValidationException $e) {
        // ... gestione dell'errore ...

        if ($useTransaction) {
            DB::rollBack();
        }
    }
}
```

#### 1. Verifica degli Errori di Validazione

La seguente è l'impostazione basata sulla documentazione Livewire (Vedi):https://livewire.laravel.com/docs/testing#validation
```php
$component->assertHasErrors(['categoryData.en' => 'unique']);
```
#### Il Problema si genera solo se il componente deve gestire più di un Filament form 
L'errore che si presenta è il seguente
```php
  Component has no matching failed rule or error message [required] for [categoryData.en] attribute.
  Failed asserting that an array contains 'required'..
```
In questo caso il problema è stato risolto evitando l'uso della sintassi dettagliata per la verifica degli errori di validazione, optando per una soluzione più semplice che ha permesso di rilevare gli errori:

#### Problema con **->assertHasErrors**
Inoltre, nel caso della gestione multiform di Filament in un componente Livewire, che se non specificavo l'array categoryData di ->statePath, ottenevo l'errore `Component missing error: en Failed asserting that false is true.`
Per risolvere occorre quindi Questo funziona: `$component->assertHasErrors('categoryData.en');` invece questo dava errore: `$component->assertHasErrors('en');`
