# Motivazioni per la Scelta del Precaricamento su DigitalOcean (DO) tramite Presigned URL

### Contesto e Obiettivi
Durante lo sviluppo del modulo di caricamento dei file per la nostra applicazione, l'obiettivo principale era fornire un'esperienza utente fluida e intuitiva, garantendo al contempo efficienza nel trasferimento dei dati. Abbiamo esplorato diverse strategie per implementare il caricamento dei file, incluse soluzioni basate su caricamento locale e precaricamento su DigitalOcean (DO) tramite Presigned URL. 

### Tentativi e Considerazioni
#### 1. Caricamento Locale sul Server:
   - **Vantaggi**: 
     - **Implementazione Semplice**: La gestione del caricamento direttamente sul server locale è inizialmente più semplice da implementare e non richiede configurazioni esterne.
     - **Controllo Completo**: Abbiamo il controllo completo sul processo di caricamento, gestione dei file e risposta immediata dal server.
   - **Problemi Riscontrati**:
     - **Sincronizzazione della Progress Bar**: Durante il caricamento di file di dimensioni maggiori, abbiamo notato che la progress bar non si sincronizzava correttamente con il caricamento effettivo dei file. Nonostante numerosi tentativi di correggere il problema, la progress bar non riusciva a riflettere accuratamente lo stato del caricamento, risultando in un'esperienza utente disallineata e poco fluida.
     - **Prestazioni Incoerenti**: La gestione del caricamento locale ha portato a prestazioni incoerenti, con alcuni file che venivano caricati rapidamente mentre altri causavano rallentamenti o comportamenti inaspettati.

#### 2. Caricamento su DigitalOcean (DO) con Presigned URL:
   - **Vantaggi**:
     - **Esperienza Utente Fluida**: Il caricamento su DO tramite Presigned URL ha offerto un'esperienza utente molto più fluida e piacevole. La progress bar si è mostrata sempre accurata e ben sincronizzata con il caricamento effettivo dei file.
     - **Riduzione del Carico sul Server**: Utilizzando Presigned URL, il caricamento avviene direttamente su DO, riducendo il carico sul server principale e migliorando le prestazioni complessive del sistema.
     - **Gestione di File di Grandi Dimensioni**: Questo approccio ha semplificato la gestione di file di grandi dimensioni, garantendo che il processo di caricamento rimanesse stabile e prevedibile.
   - **Sfide Affrontate**:
     - **Complessità Tecnica**: Implementare il caricamento su DO ha richiesto una configurazione più complessa, inclusa la gestione della generazione di Presigned URL e la sincronizzazione dei file caricati con il sistema principale.
     - **Maggiore Latenza Iniziale**: Sebbene il processo di caricamento possa sembrare più lento inizialmente a causa della generazione e gestione dei Presigned URL, questo è stato bilanciato dalla maggiore affidabilità e fluidità dell'intero processo.

### Conclusione
Dopo aver considerato attentamente i vari approcci, abbiamo deciso di adottare il precaricamento su DigitalOcean tramite Presigned URL come soluzione definitiva per il caricamento dei file nella nostra applicazione. Questa decisione è stata guidata principalmente dal miglioramento significativo dell'esperienza utente, che si è rivelata essere un fattore cruciale per l'efficacia del sistema. Nonostante la complessità tecnica aggiuntiva, i benefici a lungo termine in termini di prestazioni, affidabilità e interfaccia utente giustificano ampiamente la scelta di questa strategia. 

Aggiungere questo metodo alla nostra pipeline di sviluppo ha permesso di garantire che i nostri utenti abbiano una percezione del processo di caricamento che sia tanto fluida quanto affidabile, risolvendo i problemi precedentemente riscontrati con il caricamento locale.
