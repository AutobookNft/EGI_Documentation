# Lezione 2: Il DNS è organizzato come un albero

### Come i nomi di dominio e i record DNS sono organizzati in una struttura ad albero

---

### Introduzione
Il DNS è organizzato in una struttura gerarchica simile a un albero, che permette una gestione efficiente e distribuita dei nomi di dominio. In questa lezione, esploreremo come i nomi di dominio e i record DNS sono strutturati in questo sistema ad albero e come funziona la navigazione attraverso questa gerarchia.

---

### 1. Struttura gerarchica del DNS
Il DNS è strutturato in una gerarchia a più livelli, con il livello più alto noto come "root" o "radice". Da questo livello, la gerarchia si ramifica in vari livelli di dominio, inclusi i domini di primo livello (TLD), i domini di secondo livello e così via.

- **Root (Radice):** Il livello più alto nella gerarchia DNS, rappresentato da un punto (.) alla fine di un nome di dominio, spesso omesso nelle URL comuni.
- **Domini di primo livello (TLD):** Questi includono domini generici come `.com`, `.org`, `.net` e domini nazionali come `.it` per l'Italia.
- **Domini di secondo livello:** Questi sono registrati sotto i TLD, come `example.com`.
- **Domini di terzo livello e sottodomini:** Questi sono sotto i domini di secondo livello, come `www.example.com` o `mail.example.com`.

---

### 2. Nomi di dominio e percorsi
Ogni nodo nell'albero DNS rappresenta un dominio, e l'intero percorso dal nodo alla radice rappresenta il nome di dominio completo (FQDN - Fully Qualified Domain Name). Ad esempio, nel nome di dominio `www.example.com`:

- **`www`** è un sottodominio del dominio `example.com`.
- **`example`** è un dominio di secondo livello sotto il TLD `.com`.
- **`com`** è un TLD sotto la radice.

---

### 3. Record DNS
I record DNS contengono informazioni su come un dominio deve essere gestito. Esistono vari tipi di record DNS, ciascuno con una funzione specifica:

- **A Record:** Mappa un nome di dominio a un indirizzo IPv4.
- **AAAA Record:** Mappa un nome di dominio a un indirizzo IPv6.
- **CNAME Record:** Alias per un altro nome di dominio.
- **MX Record:** Specifica i server di posta per un dominio.
- **TXT Record:** Contiene informazioni di testo per vari usi, come la verifica del dominio e l'autenticazione delle email.

---

### 4. Funzionamento della gerarchia DNS
Quando un client DNS cerca di risolvere un nome di dominio, la ricerca avviene attraverso la gerarchia DNS:

1. **Query alla radice:** La ricerca inizia dai server DNS della radice, che indirizzano la query ai server DNS dei TLD appropriati.
2. **Query ai TLD:** Il server TLD risponde con i server DNS autorevoli per il dominio di secondo livello.
3. **Query al dominio di secondo livello:** Infine, la query raggiunge il server DNS autorevole per il dominio specifico, che fornisce l'informazione richiesta (es. l'indirizzo IP).

---

### 5. Esempio pratico
Supponiamo di voler risolvere `www.example.com`. Ecco come si svolge il processo:

1. **Query alla radice:** Il resolver invia una query ai server DNS della radice chiedendo informazioni su `com`.
2. **Query al TLD:** Il server della radice risponde con i server DNS autorevoli per il TLD `.com`.
3. **Query al dominio di secondo livello:** Il resolver invia una query a uno dei server DNS autorevoli per `com`, chiedendo informazioni su `example.com`.
4. **Query al sottodominio:** Il server `com` risponde con i server DNS autorevoli per `example.com`.
5. **Risposta finale:** Il resolver invia una query a uno dei server DNS autorevoli per `example.com`, che risponde con l'indirizzo IP di `www.example.com`.

---

### Conclusione
Comprendere la struttura ad albero del DNS è fondamentale per navigare e gestire i nomi di dominio. Questa struttura gerarchica consente un'efficiente distribuzione e risoluzione dei nomi di dominio, facilitando la navigazione in Internet.
