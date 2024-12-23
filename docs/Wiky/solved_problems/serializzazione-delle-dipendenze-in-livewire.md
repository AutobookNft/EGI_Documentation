# Problema della serializzazione delle Dipendenze in Livewire

### Descrizione Generica del Problema

Livewire utilizza un approccio unico per gestire i dati dei componenti tra le richieste del server. Ogni proprietà in un componente Livewire viene serializzata in JSON tra una richiesta e l'altra, quindi deserializzata da JSON nuovamente in PHP per la richiesta successiva. Questo processo di conversione bidirezionale presenta alcune limitazioni, poiché non tutti i tipi di oggetti possono essere facilmente convertiti in JSON e poi ricostruiti.

### Tipi di Proprietà Supportati

Livewire supporta tipi primitivi come stringhe, numeri interi, array, booleani e null. Inoltre, supporta alcuni tipi di oggetti PHP comuni utilizzati nelle applicazioni Laravel, come:

- Collezioni (`Illuminate\Support\Collection`)
- Collezioni Eloquent (`Illuminate\Database\Eloquent\Collection`)
- Modelli (`Illuminate\Database\Eloquent\Model`)
- DateTime (`DateTime`)
- Carbon (`Carbon\Carbon`)
- Stringable (`Illuminate\Support\Stringable`)

### Problema Specifico

Quando si cerca di utilizzare una classe personalizzata come proprietà in un componente Livewire, si incontra il problema che tale classe non è automaticamente serializzabile e deserializzabile. Di conseguenza, le istanze degli oggetti personalizzati possono andare perse tra le richieste, causando errori come "Call to a member function on null".

### Soluzione di Livewire 
```php
Wireable
```
Livewire fornisce una soluzione attraverso l'implementazione dell'interfaccia `Wireable`, che permette di definire come la classe deve essere serializzata e deserializzata.

### Esempio di Implementazione 

Per risolvere questo problema, è necessario implementare l'interfaccia `Wireable` nella classe personalizzata e definire i metodi `toLivewire` e `fromLivewire` che specificano come serializzare e deserializzare l'oggetto.

```php
use Livewire\Wireable;

class CustomClass implements Wireable
{
    protected $property;

    public function __construct($property)
    {
        $this->property = $property;
    }

    public function toLivewire()
    {
        return ['property' => $this->property];
    }

    public static function fromLivewire($value)
    {
        return new static($value['property']);
    }
}
```

# Problema Riscontrato con ForbiddenTermChecker

Durante lo sviluppo del componente `TraitLibraryComponent` in Livewire, abbiamo riscontrato un problema con la gestione della classe `ForbiddenTermChecker`. Quando questa classe veniva assegnata a una proprietà pubblica del componente, l'istanza della classe risultava `null` quando si tentava di utilizzarla nelle richieste successive.

### Dettagli del Problema

1. **Assegnazione della Classe alla Proprietà Pubblica:**
   ```php
   class TraitLibraryComponent extends Component
   {
       public $forbiddenTermChecker;

       public function mount(ForbiddenTermChecker $forbiddenTermChecker)
       {
           $this->forbiddenTermChecker = $forbiddenTermChecker;
           Log::channel('traits')->info('ForbiddenTermChecker assegnato');
       }
   }
   ```

2. **Errore Durante l'Uso della Proprietà:**
   Nonostante l'assegnazione corretta nel metodo `mount`, durante le richieste successive, l'istanza risultava `null`:
   ```php
   if ($this->forbiddenTermChecker->containsForbiddenTerm($sanitizedText, $this->userLanguage)) {
       throw new \App\Exceptions\ForbiddenTermException('The term contains forbidden words.');
   }
   ```
   Questo causava l'errore:
   ```
   Call to a member function containsForbiddenTerm() on null
   ```

### Motivo del Problema

Il problema si verifica perché Livewire serializza le proprietà del componente in JSON tra le richieste. Poiché la classe `ForbiddenTermChecker` non è serializzabile in JSON, l'istanza viene persa e la proprietà diventa `null` nelle richieste successive.

<u style={{color: 'green', fontWeight: 'bold'}}>[Vedi soluzione proposta da Livewire](#soluzione-di-livewire)</u>

### Soluzione Adottata

Per evitare la complessità aggiuntiva di implementare l'interfaccia `Wireable`, abbiamo deciso di istanziare la classe `ForbiddenTermChecker` direttamente all'interno dei metodi dove viene utilizzata. Questo approccio è più semplice e garantisce che l'istanza sia sempre disponibile quando necessaria.

#### Esempio di Implementazione
```php
public function createNewCategory(): void
{
    $forbiddenTermChecker = new ForbiddenTermChecker();

    if ($forbiddenTermChecker->containsForbiddenTerm($sanitizedText, $this->userLanguage)) {
        throw new \App\Exceptions\ForbiddenTermException('The term contains forbidden words.');
    }

    // Rest of the method...
}
```

### Vantaggi della Soluzione Adottata

- **Semplicità:** Non è necessario implementare l'interfaccia `Wireable` o gestire la serializzazione.
- **Leggibilità:** Il codice è più semplice e immediato da comprendere.
- **Isolamento:** Ogni metodo gestisce la propria istanza del `ForbiddenTermChecker`, rendendo il codice più modulare e meno soggetto a effetti collaterali.
