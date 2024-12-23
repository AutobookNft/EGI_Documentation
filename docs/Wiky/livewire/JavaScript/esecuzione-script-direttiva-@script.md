# Esecuzione di script con la direttiva @script

## La direttiva `@script`

La direttiva `@script` in Livewire permette di eseguire codice JavaScript specifico per un componente. Quando vuoi eseguire uno script JavaScript solo quando il componente è renderizzato, utilizzi questa direttiva all'interno del tuo componente Livewire. Questo consente di incorporare il codice direttamente nella vista del componente, garantendo che venga eseguito nel momento giusto, evitando problemi di sincronizzazione o esecuzione prematura. Ad esempio, puoi utilizzare `@script` per inizializzare un plugin JavaScript solo quando il componente è caricato.

### Ecco un esempio:

```blade
<div>
    <!-- Contenuto del componente -->
    @script
    console.log('Il componente è stato renderizzato');
    @endscript
</div>
```

In questo modo, il messaggio nel console log apparirà solo quando il componente è stato completamente caricato.