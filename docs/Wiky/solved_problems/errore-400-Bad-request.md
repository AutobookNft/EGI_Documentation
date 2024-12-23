# Problema di Errore 400 - Bad Request Durante l'Upload di File di Grandi Dimensioni

**Sintomo:**
Durante l'upload di file di grandi dimensioni tramite Nginx e PHP-FPM, si verifica un errore HTTP 400 - Bad Request. Questo problema non si presenta quando si utilizza il server web integrato di Laravel.

**Causa:**
Il problema era dovuto alla configurazione del file `php.ini` per PHP-FPM, che limitava le dimensioni massime del file e del corpo delle richieste.

**Soluzione:**
Per risolvere il problema, è stato necessario modificare le impostazioni del file `php.ini` utilizzato da PHP-FPM per permettere l'upload di file di dimensioni maggiori. Di seguito sono elencati i passaggi eseguiti:

### **NOTA BENE: Si tratta di PHP-FPM quindi non quello all'interno di `etc/php/8.2/cli/php.ini`**

### **Ma quello all'interno di `/etc/php/8.2/fpm/php.ini`**

1. **Modifica delle Impostazioni di `php.ini`:**
   - Apri il file `php.ini` di PHP-FPM. Questo file si trova generalmente in `/etc/php/8.2/fpm/php.ini`, ma il percorso può variare in base alla versione di PHP e alla configurazione del sistema.
   - Modifica le seguenti direttive per permettere l'upload di file fino a 100MB:

   ```ini
   upload_max_filesize = 100M
   post_max_size = 100M
   ```

2. **Riavvio di PHP-FPM e Nginx:**
   Dopo aver modificato le impostazioni di `php.ini`, è necessario riavviare PHP-FPM e Nginx per applicare le modifiche:

   ```sh
   sudo systemctl restart php8.2-fpm
   sudo systemctl restart nginx
   ```

3. **Verifica della Configurazione di Nginx:**
   Assicurarsi che Nginx sia configurato correttamente per accettare file di grandi dimensioni. Aggiungi la direttiva `client_max_body_size` nel file di configurazione del sito in `/etc/nginx/sites-available/` o in un file di configurazione globale come `/etc/nginx/nginx.conf`:

   ```nginx
   server {
        listen 80;
        server_name localhost;

        root /home/fabio/nftflorence/public;

        index index.php index.html index.htm;

        # Configurazione per servire i file statici dalla cartella "js"
        location /js/ {
            alias /home/fabio/nftflorence/public/js/;
            try_files $uri $uri/ =404;
        }

        # Configurazione per gestire le richieste di caricamento file
        location /uploading-files {
            proxy_pass http://127.0.0.1:8000/uploading-files;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Imposta una pagina personalizzata per gli errori 404.
        error_page 404 /index.php;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        }

        location ~ /\.ht {
            deny all;
        }
    }
   ```

### Risultato
Dopo aver effettuato queste modifiche, l'errore 400 - Bad Request durante l'upload di file di grandi dimensioni è stato risolto, permettendo così l'upload di file fino a 100MB tramite Nginx e PHP-FPM.