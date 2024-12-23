# Sticky Bit con Setgid


Lo **Sticky Bit** è una proprietà speciale nei permessi dei file e delle directory in sistemi operativi basati su Unix (incluso Linux) che controlla come i file all'interno di una directory possono essere modificati o eliminati.

### **Cosa fa lo Sticky Bit?**
Quando il Sticky Bit è impostato su una directory:
- **Solo il proprietario del file** (o l'utente root) può eliminarlo o modificarlo, anche se altri utenti hanno permessi di scrittura nella directory.

È comunemente usato in directory pubbliche o condivise (come `/tmp`) per prevenire che utenti non autorizzati cancellino o modifichino i file di altri.

---

### **Applicazioni pratiche dello Sticky Bit**
Nel contesto di Laravel (o altre applicazioni web):
- Può essere usato per mantenere la **coerenza dei gruppi** sui file creati all'interno di una directory. 
- In combinazione con il **Setgid** (che vedremo a breve), assicura che i file creati abbiano sempre il gruppo corretto.

Ad esempio, se imposti lo Sticky Bit su una directory come `storage`, i file creati al suo interno saranno protetti da cancellazioni accidentali da parte di utenti con accesso al gruppo.

---

### **Come impostare lo Sticky Bit**
Puoi impostarlo su una directory con il comando `chmod`:
```bash
sudo chmod +t <nome_della_directory>
```

Per verificare che sia impostato, usa:
```bash
ls -ld <nome_della_directory>
```

Se il Sticky Bit è attivo, vedrai una `t` nei permessi della directory, per esempio:
```
drwxrwxrwt  2 www-data www-data 4096 Nov 23 10:00 storage
```

---

### **Combina Sticky Bit con Setgid**
Il **Setgid** (Set Group ID) è un'altra proprietà speciale. Se impostato su una directory:
- Tutti i file e le directory creati al suo interno **ereditano il gruppo** della directory madre, anziché il gruppo dell'utente che li crea.

Per impostare sia il Setgid che lo Sticky Bit:
```bash
sudo chmod g+s,+t storage
```

Ora:
1. I file e le directory creati in `storage` erediteranno il gruppo del genitore (ad esempio, `www-data`).
2. Solo il proprietario dei file o root potrà eliminarli.

---

### **Esempio nel contesto Laravel**
1. Imposta permessi per la directory `storage`:
   ```bash
   sudo chmod -R 775 storage bootstrap/cache
   sudo chmod g+s,+t storage bootstrap/cache
   ```
2. Assegna il gruppo corretto:
   ```bash
   sudo chgrp -R www-data storage bootstrap/cache
   ```
3. Aggiungi l'utente corrente (ad esempio `fabio`) al gruppo `www-data`:
   ```bash
   sudo usermod -a -G www-data fabio
   ```

Con questa configurazione:
- Tutti i file creati in `storage` e `bootstrap/cache` apparterranno al gruppo `www-data`.
- Solo i proprietari o root potranno eliminarli.
- `fabio`, come membro del gruppo `www-data`, avrà permessi per leggere e scrivere sui file.

---

### **In sintesi**
- **Sticky Bit**: Protegge i file dalla cancellazione da parte di altri utenti.
- **Setgid**: Garantisce che tutti i file creati in una directory condividano lo stesso gruppo.

Usare entrambe le proprietà nelle directory chiave del tuo progetto (come `storage`) ti aiuta a gestire meglio i permessi e prevenire problemi di accesso e modifica dei file!