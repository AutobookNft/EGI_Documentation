# HTMLPurifier

La libreria HTMLPurifier è utilizzata per ripulire il codice HTML da contenuti indesiderati, migliorando la sicurezza e l'affidabilità del codice. Ecco una spiegazione delle configurazioni che hai menzionato:

HTMLPurifier protegge da diversi tipi di attacchi legati all'inserimento di codice HTML non sicuro. Ecco i principali:

1. **Cross-Site Scripting (XSS)**: Previene l'inserimento di script maligni che possono essere eseguiti nel browser dell'utente.
2. **Injection di contenuti non sicuri**: Filtra tag e attributi HTML che possono essere utilizzati per iniettare contenuti dannosi.
3. **URL Maligni**: Verifica e pulisce i link per impedire l'uso di schemi URL pericolosi.
4. **Attributi CSS dannosi**: Rimuove o filtra le proprietà CSS che possono essere utilizzate per attacchi stilistici o di layout.

Queste protezioni aiutano a mantenere il contenuto HTML sicuro e conforme agli standard.

1. **Creazione di una configurazione di default**:
   ```php
   $config = HTMLPurifier_Config::createDefault();
   ```
   Questo crea una configurazione di base per HTMLPurifier con le impostazioni predefinite.

2. **Impostazione di HTML.Allowed**:
   ```php
   $config->set('HTML.Allowed', 'p,b,a[href],i');
   ```
   Questo permette solo alcuni tag HTML specifici e gli attributi associati:
   - `p`: Paragrafi
   - `b`: Testo in grassetto
   - `a[href]`: Link con l'attributo `href`
   - `i`: Testo in corsivo

### Altre Configurazioni Comuni di HTMLPurifier

Ecco alcune altre configurazioni utili:

1. **HTML.Forbidden**: Permette di specificare quali tag HTML devono essere rimossi, anche se sono presenti in `HTML.Allowed`.
   ```php
   $config->set('HTML.Forbidden', 'img,script');
   ```

2. **URI.AllowedSchemes**: Specifica quali schemi URI sono permessi (ad esempio, `http`, `https`).
   ```php
   $config->set('URI.AllowedSchemes', ['http' => true, 'https' => true]);
   ```

3. **Attr.EnableID**: Permette l'uso degli attributi `id`.
   ```php
   $config->set('Attr.EnableID', true);
   ```

4. **CSS.AllowedProperties**: Permette di specificare quali proprietà CSS sono consentite.
   ```php
   $config->set('CSS.AllowedProperties', ['color', 'background-color', 'font']);
   ```

### Esempio Completo

Ecco un esempio più completo di configurazione:

```php
$config = HTMLPurifier_Config::createDefault();
$config->set('HTML.Allowed', 'p,b,a[href],i');
$config->set('URI.AllowedSchemes', ['http' => true, 'https' => true]);
$config->set('Attr.EnableID', true);
$config->set('CSS.AllowedProperties', ['color', 'background-color', 'font']);
```

Questa configurazione assicura che solo certi tag HTML, attributi e proprietà CSS siano permessi, migliorando la sicurezza e la pulizia del contenuto HTML.