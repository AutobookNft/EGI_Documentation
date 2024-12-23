# Mini Corso sulla Configurazione di Nginx

Benvenuto a questo mini corso su come configurare Nginx! Utilizzeremo i file di configurazione che hai fornito come esempio per spiegare passo dopo passo le varie sezioni e direttive. Inoltre, ci concentreremo sulla gestione degli utenti in Nginx.

## Introduzione a Nginx

Nginx è un server web ad alte prestazioni che può essere utilizzato come server HTTP, reverse proxy, e server di posta elettronica. La sua configurazione è altamente modulare e si basa su file di configurazione strutturati in blocchi.

## Struttura dei File di Configurazione

I file di configurazione di Nginx sono tipicamente situati in `/etc/nginx/` e sono suddivisi in:

- **nginx.conf**: Il file principale di configurazione.
- **conf.d/**: Cartella per file di configurazione aggiuntivi.
- **sites-available/** e **sites-enabled/**: Cartelle per la configurazione dei server virtuali.

## Analisi del File `nginx.conf`

Cominciamo analizzando il file principale `nginx.conf`.

### Sezione Globale

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
```

- **user www-data;**: Specifica l'utente e il gruppo con cui Nginx verrà eseguito. Questo è importante per i permessi di accesso ai file e alla sicurezza.
- **worker_processes auto;**: Imposta il numero di processi worker. `auto` permette a Nginx di determinare automaticamente il numero ottimale basato sulle CPU disponibili.
- **pid /run/nginx.pid;**: Specifica il file PID per il processo master di Nginx.
- **include /etc/nginx/modules-enabled/*.conf;**: Include i file di configurazione dei moduli abilitati.

### Sezione `events`

```nginx
events {
    worker_connections 768;
    # multi_accept on;
}
```

- **worker_connections 768;**: Imposta il numero massimo di connessioni simultanee che ogni processo worker può gestire.
- **multi_accept on;**: (Commentato) Permetterebbe al processo worker di accettare più connessioni nuove alla volta.

### Sezione `http`

La sezione `http` contiene la maggior parte della configurazione relativa al server web.

#### Impostazioni di Base

```nginx
client_max_body_size 100M;
sendfile on;
tcp_nopush on;
types_hash_max_size 2048;
include /etc/nginx/mime.types;
default_type application/octet-stream;
```

- **client_max_body_size 100M;**: Imposta la dimensione massima del corpo delle richieste client (utile per l'upload di file).
- **sendfile on;**: Abilita l'uso di `sendfile()` per un trasferimento efficiente dei file.
- **tcp_nopush on;**: Migliora le prestazioni per i file statici.
- **types_hash_max_size 2048;**: Imposta la dimensione massima dell'hash per i tipi MIME.
- **include /etc/nginx/mime.types;**: Include i tipi MIME standard.
- **default_type application/octet-stream;**: Tipo MIME predefinito.

#### Log

```nginx
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```

- **access_log** e **error_log**: Specificano i percorsi dei log di accesso ed errore.

#### Gzip Compression

```nginx
gzip on;
```

- **gzip on;**: Abilita la compressione gzip per ridurre la dimensione delle risposte.

#### Inclusione di Altri File di Configurazione

```nginx
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

- Include ulteriori configurazioni e i server virtuali abilitati.

### Definizione dei Server (`server` Blocks)

I blocchi `server` definiscono le configurazioni per i server virtuali.

#### Primo Server (`localhost`)

```nginx
server {
    listen 80;
    server_name localhost;
    ...
}
```

- **listen 80;**: Ascolta sulla porta 80 (HTTP).
- **server_name localhost;**: Nome del server.

#### Root e Indice

```nginx
root /home/fabio/nftflorence/public;
index index.php index.html index.htm;
charset utf-8;
```

- **root**: Directory radice per i file del sito.
- **index**: File di indice predefiniti.
- **charset utf-8;**: Imposta il set di caratteri per garantire la corretta visualizzazione.

#### Gestione dei File Statici

```nginx
location /js/ {
    alias /home/fabio/nftflorence/public/js/;
    try_files $uri $uri/ =404;
}
```

- **location /js/**: Gestisce le richieste alla directory `/js/`.
- **alias**: Specifica un percorso alternativo nel file system.
- **try_files**: Tenta di servire il file richiesto.

#### Proxy Pass

```nginx
location /uploading-files {
    proxy_pass http://127.0.0.1:8000/uploading-files;
    ...
}
```

- **proxy_pass**: Inoltra le richieste a un altro server o applicazione (utile per applicazioni backend).

#### Gestione degli Errori

```nginx
error_page 404 /index.php;
```

- **error_page 404 /index.php;**: Reindirizza le pagine 404 a `index.php`.

#### PHP Processing

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.2-fpm.sock;
}
```

- **location ~ \.php$**: Gestisce i file PHP.
- **fastcgi_pass**: Invia le richieste PHP a PHP-FPM.

#### Sicurezza

```nginx
location ~ /\.ht {
    deny all;
}
```

- **deny all;**: Nega l'accesso a file che iniziano con `.ht`.

### Secondo Server (`florenceegi`)

Il secondo server è simile, con alcune differenze.

```nginx
server {
    listen 8015;
    server_name florenceegi;
    root /home/fabio/florenceegi/public;
    index index.php index.html;
    ...
}
```

- **listen 8015;**: Ascolta sulla porta 8015.
- **server_name florenceegi;**: Nome del server.

#### Configurazione di PHP Personalizzata

```nginx
location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
}
```

- Qui si utilizza PHP 8.3 e si specificano parametri FastCGI personalizzati.

#### Header di Sicurezza

```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options "nosniff";
```

- **add_header**: Aggiunge header HTTP per migliorare la sicurezza.

## Gestione degli Utenti in Nginx

Una parte fondamentale della configurazione di Nginx riguarda l'utente con cui il server viene eseguito.

### Direttiva `user`

```nginx
user www-data;
```

- Questa direttiva specifica l'utente e il gruppo sotto cui Nginx opera dopo l'avvio. È importante per:

  - **Permessi di File**: L'utente deve avere accesso ai file nel `root` specificato.
  - **Sicurezza**: Eseguire Nginx con un utente non privilegiato riduce i rischi in caso di compromissione.

### Come Gestire gli Utenti

1. **Verifica dell'Utente**: Assicurati che l'utente specificato esista nel sistema.

   ```bash
   id www-data
   ```

2. **Permessi delle Directory**: La directory `root` e i file devono avere permessi che permettono all'utente di leggere (e scrivere, se necessario).

   ```bash
   chown -R www-data:www-data /path/to/your/root
   ```

3. **Socket PHP-FPM**: Se usi PHP-FPM, il socket Unix deve avere permessi adeguati.

   - Verifica che `php-fpm` sia configurato per utilizzare lo stesso utente o che il socket sia accessibile.

4. **Sicurezza**:

   - Evita di eseguire Nginx come `root`.
   - Limita i permessi solo a ciò che è necessario.

### Esempio Pratico

Nel tuo secondo server:

```nginx
fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
```

- Assicurati che il socket `/var/run/php/php8.3-fpm.sock` sia accessibile dall'utente `www-data`.

## Consigli Finali

- **Test della Configurazione**: Dopo aver modificato i file, testa la configurazione con:

  ```bash
  sudo nginx -t
  ```

- **Ricarica Nginx**: Se il test ha esito positivo, ricarica Nginx:

  ```bash
  sudo systemctl reload nginx
  ```

- **Logging**: Monitora i log di accesso ed errore per identificare eventuali problemi.

## Risorse Aggiuntive

- [Documentazione Ufficiale di Nginx](https://nginx.org/en/docs/)
- [Guida a Nginx su DigitalOcean](https://www.digitalocean.com/community/tags/nginx)

## Conclusione

Spero che questo mini corso ti abbia aiutato a comprendere meglio come configurare Nginx e gestire gli utenti. La gestione corretta degli utenti è fondamentale per la sicurezza e il funzionamento appropriato del server web. Se hai altre domande, non esitare a chiedere!