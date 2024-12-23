# Libreria dei traits

### Funzionamento della Pulsantiera

Quando viene creato un nuovo utente, il sistema popola automaticamente le tabelle `trait_categories` e `traits_key_commons` con un set predefinito di categorie e chiavi. Il Creator può quindi creare la sua libreria personale di traits partendo da queste chiavi predefinite. Per esempio, se la chiave è "colore dello sfondo", il Creator può aggiungere il value: "rosso". Da quel momento in poi, nella libreria dei traits di quel Creator, sarà presente il trait "colore dello sfondo" => "rosso".
Per facilitare questo processo, è stata creata una <u style={{color: 'green', fontWeight: 'bold'}}>[pulsantiera](#funzionamento-della-pulsantiera)</u> gestita dal componente `TraitLibraryComponent`. Questo componente gestisce il colore dei pulsanti in base allo stato di selezione e permette la creazione di nuove Category / Key / Value.

### Abbinamento traits agli EGI
Nella terza sezione della pulsantiera, c'è quella che contiene i traits creati dal Creator. I traits possono essere abbinati agli EGI in due modi differenti:

1 - Idividualmente, dalla pagina `/editEgi/edit-egi-page?itemId={$itemId}`. 

#### Procedura
#### 1 - L'user utilizza il drag and drop per abbinare il trait all'EGI
#### 2 - nel div all'interno di `resources/views/livewire/collections/items-edit.blade.php` viene richiamato tramite dispatch il metodo `bindTraitToEgi`

    ```php
    <div class='{{ $ongoingDrop && $item->drop_id ? ' bg-gradient-to-br from-purple-500 to-pink-500' : 'bg-white' }} mr-[1px]'
    x-data="{ zoomLevel: 1, isDragOver: false, openEditEgi: false, hasTraits: false }" x-on:dragover.prevent="$event.preventDefault()" x-on:dragenter="isDragOver = true"
    x-on:dragleave="isDragOver = false" :class="{ 'bg-blue-200': isDragOver, 'bg-white': !isDragOver }"
    x-on:drop="$dispatch('bindTraitToEgi', {event: event.dataTransfer.getData('text/plain') })">
    ```

##### metodo `bindTraitToEgi`

    ```php
    #[On('bindTraitToEgi')]
    public function bindTraitToEgi($event)
    {
        // La tua logica qui
        Log::channel('traits')->info('Classe: ItemsEdit. Metodo: bindTraitToEgi. Action: $event: '.$event);
        $this->traitsValueUsersId = intval($event);
        Log::channel('traits')->info('Classe: ItemsEdit. Metodo: bindTraitToEgi. Action: $this->itemId: '.$this->itemId);

        // notifico con SweetAlert2 che l'operazione è andata a buon fine
        $this->dispatch('sureMergeTraitToEGI', [
            'confirmButtonText' => __('traits.yes_match_it'),
            'cancelButtonText' => __('label.cancel'),
            'title' => __('traits.apply_traits'),
            'message' => __('traits.sure_you_what_match_this_trait'),
            'result_title' => __('traits.trait_added'),
            'result_message' => __('traits.the_trait_was_matched'),
        ]);

    }
```
Al listener viene passato il parametro `sureMergeTraitToEGI`, questi si trova dentro il file notifications.js

```php
Livewire.on('sureMergeTraitToEGI', (text) => {
    console.log(text);
    Swal.fire({
        title: text[0]['title'],
        text: text[0]['message'],
        icon: 'question',
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: text[0]['confirmButtonText'],
        cancelButtonText: text[0]['cancelButtonText']
    }).then((result) => {
        if (result.isConfirmed) {
            // Invia un evento Livewire per chiamare il metodo mergeTraitToEGI
            Livewire.dispatch('mergeTraitToEGI');

            // Mostra un messaggio di successo
            Swal.fire({
                title: text[0]['result_title'],
                text: text[0]['result_message'],
                icon: 'success',
            });
        }
    });
});
```
Se l'utente conferma premendo il bottone di conferma, viene richiamato il listener `mergeTraitToEGI`
##### metodo `mergeTraitToEGI`
```php
/**
     * Questo metodo viene chiamato quando l'utente trascina un trait sopra un EGI
     * queste sono le variabili utilizzate:
     * var $this->traitsValueUsersId = id del trait
     * var $this->itemId = id dell'item
     * */
    #[On('mergeTraitToEGI')]
    public function mergeTraitToEGI()
    {

        Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: $this->itemId: '.$this->itemId);
        Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: $this->traitsValueUsersId: '.$this->traitsValueUsersId);
        Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: $this->current_team->id: '.$this->current_team->id);

        try {
            // Tentativo di eseguire operazioni di database che potrebbero fallire
            $trait = TraitsItem::where('traits_value_users_id', $this->traitsValueUsersId)
                ->where('teams_items_id', $this->itemId)
                ->first(); // first

            if ($trait === null) {

                Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: Auth::id(): '.Auth::id());
                // Questo blocco viene eseguito se firstOrFail() non trova il record
                $trait = new TraitsItem;
                $trait->teams_items_id = $this->itemId;
                $trait->traits_value_users_id = $this->traitsValueUsersId;
                $trait->team_id = $this->current_team->id;
                $trait->user_id = Auth::id();

                // Adesso devo trovare il valore del trait che l'utente ha scelto per abbinarlo all'EGI
                // Per capire meglio, si tratta di "valore" della coppia chiave/valore
                // Cerco il record nella tabella traits_value_users, usando l'id del trait scelto dall'utente
                $traitValueUser = TraitsValueUser::find($this->traitsValueUsersId);

                // Poi mi occorre trovare la chiave corrispondente a questo traitValueUser
                // quindi leggo il traits_key_common_id per cercare il record della chiave della coppia chiave/valore
                $traits_key_common_id = $traitValueUser->traits_key_common_id;

                // Leggo il record della tabella traits_key_common corrispondente all'id della key
                $traitKeyCommon = TraitKeyCommon::find($traits_key_common_id);

                Log::channel('traits')->info('$traits_key_common_id: '.$traitKeyCommon->key_en);
                Log::channel('traits')->info('$value: '.$traits_key_common_id);

                // Log::channel('traits')->info('$this->languages'.$this->languages);

                foreach ($this->languages as $lang) {

                    $keyField = 'key_'.$lang;
                    $valueField = 'value_'.$lang;

                    $trait->{$keyField} = $traitKeyCommon->{$keyField};
                    $trait->{$valueField} = $traitValueUser->{$valueField};

                    Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: assign:'.$keyField.' => '.$traitKeyCommon->{$keyField});
                    Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: assign:'.$valueField.' => '.$traitValueUser->{$valueField});

                }

                $trait->save();

                $this->notificationAction = 'success';
                $this->notitications();

                if (TraitsItem::where('teams_items_id', $this->itemId)->count() > 0) {
                    $this->hasTraits = true;
                }

            }

        } catch (\Exception $e) {
            // Questo blocco può catturare altri tipi di eccezioni generiche
            // Log dell'errore o gestione dell'eccezione

            Log::channel('traits')->info('Classe: ItemsEdit. Metodo: mergeTraitToEGI. Action: error:'.$e->getMessage());
            // Potresti voler aggiungere un messaggio di errore generico per l'utente
        }

    }
    ```



2 - oppure in modo massivo, selezionando più EGI nella pagina `/collection/bindtrait_to_EGI`

```php
<div class="z-10 grid grid-cols-12">
    <div class="z-30 col-span-12 mb-2 mr-1 mt-2 rounded-md bg-black" x-data="{ zoomLevel: 1, isDragOver: false, openEditEgi: false, hasTraits: false }"
        x-on:dragover.prevent="$event.preventDefault()" x-on:dragenter="isDragOver = true"
        x-on:dragleave="isDragOver = false" :class="{ 'bg-blue-200': isDragOver, 'bg-white': !isDragOver }"
        x-on:drop="$dispatch('bindTraitToEgi', {event: event.dataTransfer.getData('text/plain') })">
        >
        @if ($onlyOne)
            @isset($item)
                @include('livewire.traits.traits-items-grid')
            @endisset
        @else
            <div class="flex flex-wrap justify-between p-1">
                @foreach ($items as $item)
                    @include('livewire.traits.traits-items-grid')
                @endforeach
            </div>
        @endif
    </div>
</div>
```
