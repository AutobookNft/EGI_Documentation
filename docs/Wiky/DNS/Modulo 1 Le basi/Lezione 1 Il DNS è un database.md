# Lezione 1: Il DNS è un database

## Obiettivi di progettazione del DNS e funzioni principali

---

### Introduzione
Il Domain Name System (DNS) è un componente fondamentale di Internet che consente la traduzione dei nomi di dominio in indirizzi IP, rendendo possibile la navigazione web come la conosciamo oggi. In questa lezione, esploreremo il DNS come un database, i suoi obiettivi di progettazione e le sue funzioni principali.

---

### 1. Cos'è il DNS?
Il DNS è un sistema gerarchico e distribuito che associa i nomi di dominio leggibili dall'uomo, come `www.example.com`, agli indirizzi IP numerici, come `192.0.2.1`, che i computer utilizzano per identificarsi l'un l'altro su una rete. 

---

### 2. Obiettivi di progettazione del DNS
Il DNS è stato progettato con alcuni obiettivi chiave in mente:

1. **Scalabilità:** Il sistema deve essere in grado di gestire un numero crescente di nomi di dominio e richieste di risoluzione.
2. **Affidabilità:** Deve garantire un'alta disponibilità e resistenza ai guasti.
3. **Distribuzione:** Il carico di gestione dei nomi di dominio è distribuito tra molteplici server per evitare colli di bottiglia.
4. **Estensibilità:** Il sistema deve permettere l'aggiunta di nuove funzionalità senza interrompere quelle esistenti.

---

### 3. Funzioni principali del DNS
Le funzioni principali del DNS includono:

1. **Risoluzione dei nomi:** Traduzione dei nomi di dominio in indirizzi IP.
2. **Routing delle email:** Utilizzo dei record MX (Mail Exchange) per instradare le email ai server di posta corretti.
3. **Autenticazione:** Implementazione di record come SPF, DKIM e DMARC per verificare l'autenticità delle email.
4. **Servizi aggiuntivi:** Utilizzo di vari record DNS per fornire servizi aggiuntivi, come il bilanciamento del carico e la distribuzione del traffico.

---

### 4. Componenti del DNS
Il DNS è composto da vari componenti che lavorano insieme per fornire il servizio di risoluzione dei nomi:

1. **Client DNS (Resolver):** L'applicazione o il sistema che richiede la risoluzione del nome di dominio.
2. **Server DNS:** I server che rispondono alle richieste dei client DNS, inclusi:
   - **Server DNS ricorsivi:** Inoltrano le richieste ad altri server DNS fino a trovare una risposta.
   - **Server DNS autorevoli:** Forniscono risposte definitive per i nomi di dominio di cui sono responsabili.

---

### 5. Come funziona la risoluzione dei nomi?
Quando un utente digita un nome di dominio nel browser, il processo di risoluzione DNS segue questi passaggi:

1. **Query ricorsiva:** Il client DNS invia una richiesta ricorsiva al resolver locale.
2. **Risoluzione iterativa:** Il resolver contatta i server DNS a partire dalla root, scendendo attraverso i TLD (Top-Level Domain) e i domini di secondo livello fino a trovare il server autorevole per il dominio richiesto.
3. **Risposta:** Il server autorevole risponde con l'indirizzo IP associato al nome di dominio, che viene poi memorizzato nella cache del resolver per future richieste.

---

### 6. Esempio pratico
Immaginiamo che un utente voglia visitare `www.example.com`. Il processo di risoluzione DNS potrebbe essere simile al seguente:

1. Il browser invia una richiesta DNS al resolver configurato.
2. Il resolver contatta un server root per trovare il server autorevole per il dominio `.com`.
3. Il server root risponde con l'indirizzo del server `.com` autorevole.
4. Il resolver contatta il server `.com` per trovare il server autorevole per `example.com`.
5. Il server `.com` risponde con l'indirizzo del server autorevole per `example.com`.
6. Il resolver contatta il server autorevole per `example.com`, che risponde con l'indirizzo IP di `www.example.com`.
7. Il browser utilizza questo indirizzo IP per stabilire una connessione al server web di `www.example.com`.

---

### Conclusione
Il DNS è un sistema complesso e ben progettato che svolge un ruolo cruciale nella navigazione su Internet. Comprendere i suoi obiettivi di progettazione e le sue funzioni principali è fondamentale per chiunque lavori nel campo dell'IT e della gestione delle reti.

---

Questo conclude la prima lezione sul DNS. Fammi sapere quando sei pronto per passare alla prossima lezione!