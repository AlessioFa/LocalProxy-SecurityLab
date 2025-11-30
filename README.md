# LocalProxy-SecurityLab

**Obiettivo: Simulazione di un'infrastruttura di rete sicura sfruttando Squid e Nginx**

*Questo repository documenta il laboratorio pratico svolto durante i miei studi presso l'istituto **ITS-ICT Academy**, seguendo le direttive del docente di Sicurezza Informatica.*

*Il progetto è stato sviluppato per comprendere a fondo le dinamiche di sicurezza web, con l'obiettivo di configurare e mettere in sicurezza (hardening) due componenti critici di rete all'interno di un ambiente locale Ubuntu:*

1.  **Forward Proxy (Squid):** Configurato per filtrare il traffico in uscita, gestire Access Control Lists (ACL) e implementare l'autenticazione utente.
2.  **Reverse Proxy (Nginx):** Implementato per proteggere un backend server simulato, mascherando l'infrastruttura interna e mitigando attacchi comuni.
3. **apache2-utils-y:** Utility suite utilizzata per la generazione degli hash delle password (htpasswd) necessari per l'autenticazione in Squid.

### Cosa sto imparando
Attraverso questo progetto, sto mettendo in pratica concetti chiave della cybersecurity:
* Gestione delle ACL e filtraggio dei pacchetti.
* Mitigazione di attacchi DoS tramite Rate Limiting.
* Prevenzione dell'Header Injection e Information Disclosure.
* Analisi dei log per il monitoraggio del traffico.