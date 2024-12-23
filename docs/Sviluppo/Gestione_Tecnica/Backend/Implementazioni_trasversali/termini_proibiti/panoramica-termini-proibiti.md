# Panoramica Introduttiva alla Gestione dei Termini Proibiti

<span style={{ color: 'green', fontWeight: 'bold' }}>
    **v.1.0.0**
</span>
<span style={{ color: 'green', fontWeight: 'bold' }}>
    **Data ultimo aggiornamento: 2024 giugno 11**
</span>

### Scopo del Controllo dei Termini Proibiti

La gestione dei termini proibiti è un elemento fondamentale per garantire che i contenuti generati dagli utenti siano appropriati e rispettosi. Questo controllo aiuta a prevenire l'uso di linguaggio offensivo, discriminatorio o inappropriato all'interno delle nostre piattaforme. Implementare un sistema di verifica dei termini proibiti è essenziale per mantenere un ambiente sicuro e accogliente per tutti gli utenti.

### Motivo del Controllo dei Termini Proibiti

1. **Protezione degli Utenti**: Assicurarsi che i contenuti pubblicati non contengano termini offensivi o dannosi, proteggendo così la comunità da potenziali abusi verbali.
2. **Conformità Legale**: Rispettare le normative e le linee guida legali che richiedono il monitoraggio e la rimozione di contenuti inappropriati.
3. **Miglioramento dell'Esperienza Utente**: Fornire un'esperienza utente positiva, priva di linguaggi volgari o offensivi, contribuendo a creare una comunità inclusiva e rispettosa.
4. **Mantenimento della Reputazione**: Preservare la reputazione della piattaforma come spazio sicuro e accogliente, incoraggiando la partecipazione degli utenti e la fiducia nel servizio offerto.

### Logica Utilizzata nel Controllo dei Termini Proibiti

Il sistema di gestione dei termini proibiti si basa su un'architettura che combina l'uso di servizi di analisi linguistica e logica personalizzata per verificare e gestire i contenuti generati dagli utenti. Ecco una panoramica della logica implementata:

1. **Inizializzazione del Servizio di Controllo**:
   - Ogni volta che un contenuto generato dall'utente deve essere verificato, il sistema inizializza il servizio `ForbiddenTermService`, che si interfaccia con servizi di analisi linguistica esterni (ad esempio, l'API Perspective).

2. **Analisi del Contenuto**:
   - Il testo fornito dall'utente viene inviato al servizio di analisi linguistica, che valuta il contenuto in base a vari attributi di tossicità specifici per la lingua in uso.
   - Gli attributi di tossicità includono, ma non sono limitati a, termini offensivi, attacchi personali, minacce e linguaggio esplicito.

3. **Valutazione dei Risultati**:
   - Il servizio di analisi restituisce un punteggio per ogni attributo di tossicità. Questi punteggi vengono confrontati con soglie predefinite per determinare se il contenuto contiene termini proibiti.
   - Se uno o più punteggi superano le soglie stabilite, il contenuto viene considerato inappropriato.

4. **Gestione degli Errori**:
   - Se durante il processo di analisi si verifica un errore (ad esempio, un problema di connessione con il servizio di analisi esterno), il sistema gestisce l'errore inviando una notifica via email al team di sviluppo.
   - Questo approccio garantisce che i problemi siano monitorati e risolti tempestivamente, senza interrompere l'esperienza utente.

5. **Notifica e Azioni Conseguenti**:
   - Se il contenuto viene considerato inappropriato, viene lanciata un'eccezione specifica (ad esempio, `ForbiddenTermException`), che può essere gestita per informare l'utente dell'errore e bloccare la pubblicazione del contenuto.
   - Inoltre, vengono loggati tutti gli eventi rilevanti per fornire una traccia completa delle operazioni e facilitare il debug e il monitoraggio.

### Implementazione del Servizio di Controllo dei Termini Proibiti

- **Inizializzazione del Servizio**:
  ```php
  $forbiddenTermService = app(ForbiddenTermService::class);
  ```

- **Verifica dei Termini Proibiti**:
  ```php
  $forbiddenTermService->checkForbiddenTerm($sanitizedText, $this->userLanguage);
  Log::channel('traits')->info('Classe: TraitLibraryComponent. Metodo: createNewCategory. Action: Controllo termini proibiti completato');
  ```

- **Motivo dell'Inizializzazione Locale del Servizio**:
  Il servizio `ForbiddenTermService` viene inizializzato localmente all'interno del metodo anziché come proprietà della classe, per evitare problemi di serializzazione con Livewire. Questo approccio garantisce che il servizio sia disponibile e correttamente istanziato ogni volta che il metodo viene chiamato.

