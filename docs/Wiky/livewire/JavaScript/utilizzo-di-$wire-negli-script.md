# Utilizzo di $wire negli script

Utilizzare `$wire` negli script  ti permette di interagire con i componenti Livewire direttamente dal codice JavaScript. `$wire` è un oggetto globale che rappresenta il tuo componente Livewire, e ti consente di chiamare metodi e aggiornare le proprietà del componente senza ricaricare la pagina.

### Ecco come funziona:

1. **Chiamare metodi del componente**: Puoi invocare metodi definiti nel componente Livewire direttamente da JavaScript.
2. **Aggiornare proprietà**: È possibile aggiornare le proprietà del componente da JavaScript.
3. **Ricevere risposte**: Le chiamate a `$wire` possono restituire promesse, permettendoti di gestire le risposte asincrone.

### Esempio:

Nel componente Livewire:

```php
class EsempioComponent extends Component
{
    public $nome;

    public function aggiornaNome($nuovoNome)
    {
        $this->nome = $nuovoNome;
    }

    public function render()
    {
        return view('livewire.esempio-component');
    }
}
```

Nella vista del componente:

```blade
<div>
    <input type="text" id="nomeInput">
    <button onclick="aggiornaNome()">Aggiorna Nome</button>

    <script>
        function aggiornaNome() {
            let nome = document.getElementById('nomeInput').value;
            $wire.aggiornaNome(nome).then(response => {
                console.log('Nome aggiornato:', response);
            });
        }
    </script>
</div>
```

In questo esempio, quando premi il pulsante, il valore dell'input viene inviato al metodo `aggiornaNome` del componente Livewire utilizzando `$wire`, e il nome viene aggiornato senza ricaricare la pagina.