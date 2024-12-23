# Logica base

### 📊 **Riepilogo della Logica `Collection`/`Team` dell'Applicazione**

L'applicazione è organizzata per gestire in modo flessibile le relazioni tra **creator**, **team** e **collection**. Ecco una panoramica dettagliata della logica e delle funzionalità implementate:

---

## 🧑‍💼 **Utenti e Team**

### 1. **Creazione Automatica del Team Personale**

- Quando un nuovo utente (`Creator`) viene registrato:
  - Viene creato automaticamente un **team personale** associato all'utente.
  - L'utente viene aggiunto come **proprietario** del team nella tabella `team_user` con il ruolo di `creator`.

**Codice Rilevante:**

```php
protected function createTeam(User $user): Team
{
    $team = $user->ownedTeams()->save(Team::forceCreate([
        'user_id' => $user->id,
        'name' => explode(' ', $user->name, 2)[0] . "'s Team",
        'personal_team' => true,
    ]));

    // Aggiunta dell'owner alla tabella pivot team_user
    $team->users()->attach($user->id, ['role' => 'owner']);

    return $team;
}
```

---

## 🗂️ **Collection**

### 2. **Creazione Automatica della Collection**

- Quando un team personale viene creato, viene automaticamente creata una **collection predefinita** associata al team e al creator.
- La collection viene collegata al team tramite la tabella pivot `team_collection`.

**Codice Rilevante:**

```php
protected function createDefaultCollection(Team $team, User $user): void
{
    tap(Collection::create([
        'user_id' => $user->id,
        'collection_name' => explode(' ', $user->name, 2)[0] . "'s collection",
        'description' => 'Questa è la collection predefinita del team.',
        'creator_id' => $user->id,
        'type' => __('collection.type_image') ?? 'image',
        'position' => 1,
        'EGI_number' => 1,
        'floor_price' => 0.0,
        'show' => true,
    ]), function (Collection $collection) use ($team) {
        // Associa la collection al team nella tabella pivot team_collection
        $team->collections()->attach($collection->id);
    });
}
```

---

## 🔗 **Relazioni tra Modelli**

### 3. **Modello `Team`**

Relazione con le **collection** e gli **utenti**:

```php
// App\Models\Team.php

public function collections()
{
    return $this->belongsToMany(Collection::class, 'team_collection');
}

public function users()
{
    return $this->belongsToMany(User::class, 'team_user')->withPivot('role');
}
```

### 4. **Modello `Collection`**

Relazione con i **team** e il **creator**:

```php
// App\Models\Collection.php

public function teams()
{
    return $this->belongsToMany(Team::class, 'team_collection');
}

public function creator()
{
    return $this->belongsTo(User::class, 'creator_id');
}
```

### 5. **Modello `User`**

Relazione con i **team posseduti** e i **team di cui fa parte**:

```php
// App\Models\User.php

public function ownedTeams()
{
    return $this->hasMany(Team::class, 'user_id');
}

public function teams()
{
    return $this->belongsToMany(Team::class, 'team_user')->withPivot('role');
}
```

---

## 🗃️ **Struttura delle Tabelle**

### 6. **Tabella `teams`**

| Colonna        | Tipo          | Descrizione                          |
|----------------|---------------|--------------------------------------|
| `id`          | `bigIncrements` | ID del team                         |
| `user_id`     | `foreignId`     | ID del proprietario del team        |
| `name`        | `string`        | Nome del team                       |
| `personal_team` | `boolean`     | Indica se è un team personale       |

### 7. **Tabella `collections`**

| Colonna           | Tipo            | Descrizione                              |
|-------------------|-----------------|------------------------------------------|
| `id`              | `bigIncrements` | ID della collection                      |
| `user_id`         | `foreignId`     | ID del creator                           |
| `creator_id`      | `foreignId`     | ID del creator                           |
| `collection_name` | `string`        | Nome della collection                    |
| `type`            | `string`        | Tipo della collection                    |
| `floor_price`     | `float`         | Prezzo minimo                            |
| `show`            | `boolean`       | Visibilità della collection              |

### 8. **Tabella `team_user` (Tabella Pivot)**

| Colonna     | Tipo        | Descrizione                      |
|-------------|-------------|----------------------------------|
| `team_id`  | `foreignId` | ID del team                      |
| `user_id`  | `foreignId` | ID dell’utente                   |
| `role`     | `string`    | Ruolo dell’utente nel team       |

### 9. **Tabella `team_collection` (Tabella Pivot)**

| Colonna          | Tipo        | Descrizione                  |
|------------------|-------------|------------------------------|
| `team_id`       | `foreignId` | ID del team                  |
| `collection_id` | `foreignId` | ID della collection          |

---

## 🔄 **Flusso di Creazione**

1. **Registrazione di un Nuovo Utente**:
   - Viene creato l'utente (`Creator`).
   - Viene creato un **team personale** per l'utente.
   - L'utente viene aggiunto come **owner** nella tabella `team_user`.
   - Viene creata una **collection predefinita** associata al team personale.

2. **Gestione delle Collection**:
   - Ogni collection può essere associata a uno o più team tramite la tabella pivot `team_collection`.
   - Un team può avere più collection.

3. **Ruoli e Permessi**:
   - I ruoli (`superadmin`, `admin`, `creator`) sono gestiti tramite la libreria **Spatie**.
   - I permessi sono assegnati ai ruoli e agli utenti durante il seeding del database.

---

## ✅ **Riepilogo**

- **Creator**: Utente che crea e possiede le collection.
- **Team**: Gruppo di lavoro a cui possono essere assegnate una o più collection.
- **Collection**: Può appartenere a uno o più team e può essere gestita dai membri del team.
- **Associazione**: Le relazioni sono mantenute tramite le tabelle pivot `team_user` e `team_collection`.
- **Ruoli e Permessi**: Gestiti con **Spatie** per garantire flessibilità e sicurezza.

Se hai bisogno di ulteriori chiarimenti o modifiche, sono qui per aiutarti! 😊🚀