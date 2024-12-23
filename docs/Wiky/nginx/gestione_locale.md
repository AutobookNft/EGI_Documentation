# Configurazione di Nginx Locale

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    client_max_body_size 100M;

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

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
}


```

### Modifiche e Verifiche da Fare

1. **Verifica del Path Alias:**
   Assicurati che il percorso `/home/fabio/nftflorence/public/js/` esista e sia accessibile. 


2. **Permessi e Proprietà:**
   Assicurati che i permessi e le proprietà delle directory siano corretti, in modo che Nginx possa accedervi senza problemi.

3. **Controlla i Log:**
   I log di Nginx e PHP possono fornire informazioni dettagliate sugli errori:
   - `/var/log/nginx/error.log`
   - `/var/log/nginx/access.log`
   - Il log di Laravel (solitamente in `storage/logs/laravel.log`).

5. **Riavviare Nginx:**
   Dopo aver apportato modifiche alla configurazione, è necessario riavviare Nginx:
   ```sh
   sudo systemctl restart nginx
   ```

### Passaggi Successivi

1. **Verifica che Nginx stia utilizzando il file di configurazione corretto:**
   ```sh
   sudo nginx -t
   ```
   Questo comando testerà la configurazione per eventuali errori.

2. **Riavvia il server Nginx:**
   ```sh
   sudo systemctl restart nginx
   ```

### Domande e Considerazioni

- **Errore 400 (Bad Request):** Questo errore potrebbe essere dovuto a una mancata corrispondenza tra il dominio configurato in Nginx e il dominio utilizzato nella richiesta. Assicurati che l'URL nel frontend corrisponda alla configurazione del server.
