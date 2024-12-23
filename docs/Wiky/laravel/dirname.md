# dirname(__DIR__)

### `__DIR__`: 
È una costante magica di PHP che restituisce il percorso della directory del file corrente.

### `dirname(__DIR__)`: 
Restituisce il percorso della directory padre di __DIR__.

Vediamo alcuni esempi pratici per capire come funzionano `__DIR__` e `dirname(__DIR__)`.

### Esempio di `__DIR__`

La costante magica `__DIR__` restituisce il percorso della directory del file corrente.

Supponiamo di avere la seguente struttura di directory:

<span>La funzione `dirname()` restituisce la directory padre del percorso passato come argomento. Quando passiamo `__DIR__` a `dirname()`, otteniamo la <strong>directory padre della directory</strong> in cui si trova il file corrente.</span>

### Struttura della Directory

Immaginiamo la seguente struttura di directory:

```
/var/www/myapp
├── index.php
└── app
    └── bootstrap
        └── app.php
```

### Esempio di Codice e Output

#### File: `app/bootstrap/app.php`

```php
// File: /var/www/myapp/app/bootstrap/app.php

echo "__DIR__ output: " . __DIR__ . "\n";
echo "dirname(__DIR__) output: " . dirname(__DIR__) . "\n";
```

Quando esegui `app.php`, ottieni:

1. `__DIR__` restituisce il percorso della directory corrente (`/var/www/myapp/app/bootstrap`).
2. `dirname(__DIR__)` restituisce la directory padre della directory corrente (`/var/www/myapp/app`).

#### Output Atteso:

```
__DIR__ output: /var/www/myapp/app/bootstrap
dirname(__DIR__) output: /var/www/myapp/app
```

### Spiegazione

- **`__DIR__`**: Restituisce la directory del file corrente, che in questo caso è `/var/www/myapp/app/bootstrap`.
- **`dirname(__DIR__)`**: Restituisce la directory padre di `/var/www/myapp/app/bootstrap`, che è `/var/www/myapp/app`.

### Ulteriori Esempi

#### Esempio 1: `index.php`

```php
// File: /var/www/myapp/index.php

echo "__DIR__ output: " . __DIR__ . "\n";
echo "dirname(__DIR__) output: " . dirname(__DIR__) . "\n";
```

Quando esegui `index.php`, ottieni:

```
__DIR__ output: /var/www/myapp
dirname(__DIR__) output: /var/www
```

#### Esempio 2: Un file più profondo nella struttura

Immaginiamo di avere un altro file `deep.php` nella struttura seguente:

```
/var/www/myapp
└── app
    └── bootstrap
        └── deep
            └── deep.php
```

```php
// File: /var/www/myapp/app/bootstrap/deep/deep.php

echo "__DIR__ output: " . __DIR__ . "\n";
echo "dirname(__DIR__) output: " . dirname(__DIR__) . "\n";
echo "dirname(dirname(__DIR__)) output: " . dirname(dirname(__DIR__)) . "\n";
```

Quando esegui `deep.php`, ottieni:

```
__DIR__ output: /var/www/myapp/app/bootstrap/deep
dirname(__DIR__) output: /var/www/myapp/app/bootstrap
dirname(dirname(__DIR__)) output: /var/www/myapp/app
```

### Conclusione

- **`__DIR__`**: Restituisce la directory del file corrente.
- **`dirname(__DIR__)`**: Restituisce la directory padre del file corrente.

In breve, `dirname()` è utile per navigare verso l'alto nella struttura delle directory, consentendo di risolvere percorsi relativi in modo efficace.
