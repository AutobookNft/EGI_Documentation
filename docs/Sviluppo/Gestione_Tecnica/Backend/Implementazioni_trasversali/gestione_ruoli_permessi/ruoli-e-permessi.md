# Elenco dei Ruoli e dei Permessi della piattaforma

### 1. Permessi di Base per la Gestione degli Utenti
- **View Users**: Visualizzare l'elenco degli utenti.
- **Create Users**: Creare nuovi utenti.
- **Edit Users**: Modificare gli utenti esistenti.
- **Delete Users**: Eliminare gli utenti.

### 2. Permessi per la Gestione dei Ruoli e dei Permessi
- **View Roles**: Visualizzare l'elenco dei ruoli.
- **Create Roles**: Creare nuovi ruoli.
- **Edit Roles**: Modificare i ruoli esistenti.
- **Delete Roles**: Eliminare i ruoli.
- **View Permissions**: Visualizzare l'elenco dei permessi.
- **Create Permissions**: Creare nuovi permessi.
- **Edit Permissions**: Modificare i permessi esistenti.
- **Delete Permissions**: Eliminare i permessi.

### 3. Permessi per la Gestione degli EGI
- **View EGI**: Visualizzare gli EGI.
- **Create EGI**: Creare nuovi EGI.
- **Edit EGI**: Modificare gli EGI esistenti.
- **Delete EGI**: Eliminare gli EGI.

### 4. Permessi per la Gestione delle Collezioni e Categorie
- **View Collections**: Visualizzare le collezioni.
- **Create Collections**: Creare nuove collezioni.
- **Edit Collections**: Modificare le collezioni esistenti.
- **Delete Collections**: Eliminare le collezioni.
- **View Categories**: Visualizzare le categorie.
- **Create Categories**: Creare nuove categorie.
- **Edit Categories**: Modificare le categorie esistenti.
- **Delete Categories**: Eliminare le categorie.

### 5. Permessi per la Gestione del Marketplace
- **View Marketplace**: Visualizzare il marketplace.
- **Manage Listings**: Gestire le inserzioni (creare, modificare, eliminare).

### 6. Permessi Specifici per Tipologia di Utente
- **Creator Permissions**:
  - **View Own EGI**: Visualizzare i propri EGI.
  - **Create Own EGI**: Creare nuovi EGI.
  - **Edit Own EGI**: Modificare i propri EGI.
  - **Delete Own EGI**: Eliminare i propri EGI.

- **Company Permissions**:
  - **View All EGI**: Visualizzare tutti gli EGI.
  - **Approve EGI**: Approvare gli EGI per il marketplace.

- **EPP (Environment Protection Program) Permissions**:
  - **View EPP Projects**: Visualizzare i progetti EPP.
  - **Create EPP Projects**: Creare nuovi progetti EPP.
  - **Edit EPP Projects**: Modificare i progetti EPP.
  - **Delete EPP Projects**: Eliminare i progetti EPP.

### Implementazione dei Permessi

Con `spatie/laravel-permission`, puoi creare questi permessi e assegnarli ai ruoli. Ecco un esempio di come puoi farlo:

```php
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

// Permessi di base per la gestione degli utenti
Permission::create(['name' => 'view users']);
Permission::create(['name' => 'create users']);
Permission::create(['name' => 'edit users']);
Permission::create(['name' => 'delete users']);

// Permessi per la gestione dei ruoli e dei permessi
Permission::create(['name' => 'view roles']);
Permission::create(['name' => 'create roles']);
Permission::create(['name' => 'edit roles']);
Permission::create(['name' => 'delete roles']);
Permission::create(['name' => 'view permissions']);
Permission::create(['name' => 'create permissions']);
Permission::create(['name' => 'edit permissions']);
Permission::create(['name' => 'delete permissions']);

// Permessi per la gestione degli EGI
Permission::create(['name' => 'view EGI']);
Permission::create(['name' => 'create EGI']);
Permission::create(['name' => 'edit EGI']);
Permission::create(['name' => 'delete EGI']);

// Permessi per la gestione delle collezioni e categorie
Permission::create(['name' => 'view collections']);
Permission::create(['name' => 'create collections']);
Permission::create(['name' => 'edit collections']);
Permission::create(['name' => 'delete collections']);
Permission::create(['name' => 'view categories']);
Permission::create(['name' => 'create categories']);
Permission::create(['name' => 'edit categories']);
Permission::create(['name' => 'delete categories']);

// Permessi per la gestione del marketplace
Permission::create(['name' => 'view marketplace']);
Permission::create(['name' => 'manage listings']);

// Permessi specifici per tipologia di utente
// Permessi per Creator
Permission::create(['name' => 'view own EGI']);
Permission::create(['name' => 'create own EGI']);
Permission::create(['name' => 'edit own EGI']);
Permission::create(['name' => 'delete own EGI']);

// Permessi per Company
Permission::create(['name' => 'view all EGI']);
Permission::create(['name' => 'approve EGI']);

// Permessi per EPP
Permission::create(['name' => 'view EPP projects']);
Permission::create(['name' => 'create EPP projects']);
Permission::create(['name' => 'edit EPP projects']);
Permission::create(['name' => 'delete EPP projects']);

// Creazione dei ruoli e assegnazione dei permessi
$adminRole = Role::create(['name' => 'admin']);
$adminRole->givePermissionTo(Permission::all());

$creatorRole = Role::create(['name' => 'creator']);
$creatorRole->givePermissionTo([
    'view own EGI',
    'create own EGI',
    'edit own EGI',
    'delete own EGI'
]);

$companyRole = Role::create(['name' => 'company']);
$companyRole->givePermissionTo([
    'view all EGI',
    'approve EGI'
]);

$eppRole = Role::create(['name' => 'EPP']);
$eppRole->givePermissionTo([
    'view EPP projects',
    'create EPP projects',
    'edit EPP projects',
    'delete EPP projects'
]);
```

### Assegnazione dei Ruoli agli Utenti

Puoi assegnare i ruoli agli utenti utilizzando il seguente codice:

```php
$user = User::find(1);
$user->assignRole('admin');

$user2 = User::find(2);
$user2->assignRole('creator');
```

### Conclusione

Questa configurazione dei permessi e dei ruoli copre le principali funzionalità della tua applicazione, garantendo che ogni tipo di utente abbia accesso solo alle operazioni a cui è autorizzato. Puoi adattare questi permessi alle tue esigenze specifiche e gestire tutto attraverso l'interfaccia amministrativa di Filament.
