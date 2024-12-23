# ✅ Checklist per l’Implementazione del Modulo Ruoli, Permessi e Wallet

### 🔹 **1. Definizione dei Ruoli e Permessi**

1. **Definire i ruoli principali**:  
   - Superadmin  
   - Creator  
   - Admin  
   - Editor  
   - Guest  

2. **Definire i permessi per ciascun ruolo**:
   - Ambito Team  
   - Ambito Collection  
   - Ambito EGI  
   - Gestione dei Wallet  
   - Adesione alle Drop  

3. **Creare i Seeder per ruoli e permessi**.

4. **Configurare i Middleware per proteggere le rotte in base ai permessi**.

---

### 🔹 **2. Creazione delle Policy**

5. **Creare le policy per**:
   - **Biografie**: solo il proprietario può modificarle.  
   - **Wallet**: solo il proprietario può modificarli.  
   - **EGI e Collection**: in base ai permessi definiti.

---

### 🔹 **3. Database e Migrazioni**

6. **Creare le tabelle necessarie**:
   - `roles` e `permissions` (se non esistono già).  
   - `team_wallet` per i wallet associati ai team.  
   - `wallet_change_approvals` per tracciare le richieste di modifica dei wallet.

7. **Aggiungere campi alla tabella `team_wallet`**:
   - `approval` (`string`) – Stato della richiesta (`pending`, `approved`).  
   - `previous_values` (`json`) – Backup dei vecchi valori.

---

### 🔹 **4. Dashboard e UI**

8. **Creare la Dashboard per i Creator** con le seguenti sezioni:
   - **Gestione Team**:  
     - Elenco dei membri del team.  
     - Invio e gestione degli inviti.  
   - **Gestione Collection**:  
     - CRUD delle collection.  
   - **Gestione Wallet**:  
     - Elenco dei wallet associati al team.  
     - Tabella delle richieste di modifica ai wallet in attesa di approvazione.  
   - **Gestione Drop**:  
     - Sezione per l’adesione alle drop.

9. **Creare Componenti UI** per:
   - Visualizzare lo stato delle approvazioni dei wallet (es. badge colorati: `pending`, `approved`).  
   - Modificare i wallet con form dedicati.  
   - Notificare le modifiche in attesa di approvazione.

---

### 🔹 **5. Logica di Invito ai Team**

10. **Implementare la logica per invitare nuovi utenti**:
    - Invio dell’invito via email con il ruolo assegnato.  
    - Gestione delle risposte (`accetta invito`, `rifiuta invito`).  
    - Registrazione del nuovo utente nel team con il ruolo specificato.

11. **Creare una sezione nella Dashboard del Creator** per:
    - Visualizzare gli inviti in sospeso.  
    - Gestire manualmente gli inviti non accettati.

---

### 🔹 **6. Logica di Approvazione dei Wallet**

12. **Implementare la creazione e modifica dei wallet** con approvazione:
    - Creazione del wallet da parte del membro del team.  
    - Richiesta di approvazione inviata al Creator.

13. **Implementare la gestione delle richieste di approvazione**:
    - Sezioni per approvare o rifiutare le modifiche.  
    - Ripristino dei vecchi valori in caso di rifiuto.

14. **Aggiungere notifiche visive** per i wallet in attesa di approvazione.

---

### 🔹 **7. Sicurezza e Policy di Accesso**

15. **Configurare Middleware e Policy** per garantire:
    - Solo gli utenti autorizzati possono modificare i wallet.  
    - Solo il proprietario può modificare la propria biografia.  
    - Accesso ai dati delle collection ed EGI in base ai permessi.

---

### 🔹 **8. Test e Validazione**

16. **Scrivere Test Automatizzati** per:
    - Verificare i permessi di ciascun ruolo.  
    - Testare il flusso di inviti ai team.  
    - Verificare la logica di approvazione dei wallet.  
    - Assicurarsi che le policy di accesso funzionino correttamente.

17. **Test Manuali** per validare l’interfaccia utente e l’esperienza d’uso:
    - Dashboard del Creator.  
    - Notifiche visive per le approvazioni.  
    - Modifica e approvazione dei wallet.

---

### 🔹 **9. Documentazione**

18. **Documentare**:
    - Ruoli e permessi.  
    - Flussi di approvazione dei wallet.  
    - Procedure di invito ai team.  
    - Policy di sicurezza.

---

## 🚀 **Pronti per lo Sviluppo!**

Questa checklist ti guiderà nell’implementazione completa del modulo **ruoli e permessi**, inclusa la gestione dei wallet e l’integrazione con la dashboard e la UI. Se hai bisogno di ulteriori dettagli o chiarimenti su uno specifico punto, sono qui per aiutarti! 😊