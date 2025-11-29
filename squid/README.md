# Configurazione Forward Proxy (Squid)

In questa sezione del laboratorio ho configurato **Squid** come *Forward Proxy* sulla mia macchina locale (`localhost:3128`).

L'obiettivo è intercettare e filtrare il traffico web in uscita dal mio computer, simulando uno scenario aziendale in cui viene controllato l'accesso a Internet dei dipendenti.

---

## 📂 Contenuto della Cartella

Questa directory contiene i file di configurazione reali utilizzati nel laboratorio:

- **`squid.conf`**  
  Il file di configurazione principale del servizio (una copia di quello presente in `/etc/squid/`).

- **`siti_vietati.txt`**  
  La lista personalizzata (blacklist) dei domini bloccati.

---

## 📝 Diario di Configurazione

Di seguito documento i passaggi tecnici eseguiti per l'implementazione del proxy, divisi per fasi operative.

---

## Fase A: Installazione e Setup Iniziale

Ho installato il pacchetto **Squid** dai repository ufficiali di Ubuntu e ho creato un backup della configurazione di default per poterla ripristinare in caso di errori critici.

### Comandi eseguiti


sudo apt update && sudo apt install squid -y
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.orig

## Fase B: Configurazione ACL (Blacklist)

In questa fase ho modificato il file di configurazione per attivare i blocchi.

Verifica Porta: Mi sono accertato che Squid fosse in ascolto sulla porta default 3128.

Creazione Lista: Sono entrato nella directory /etc/squid/ e ho creato il file siti_vietati.txt contenente i domini da bloccare (es. .facebook.com).

Modifica Configurazione: Nel file /etc/squid/squid.conf ho inserito i seguenti comandi per attivare l'AC:

# Definizione ACL (Access Control List)
acl siti_vietati dstdomain "/etc/squid/siti_vietati.txt"

# Applicazione del blocco (Deny)
http_access deny siti_vietati

In questo modo Squid utilizza l'ACL "siti_vietati.txt" per bloccare specifici domini e i loro sottodomini.


## Fase C: Configurazione Autenticazione

Per proteggere l'accesso al proxy, ho implementato un sistema di autenticazione obbligatoria.

* **Tool:** Ho installato apache2-utils per generare le password.
* **Creazione Utente:** Ho generato l'utente utente_proxy nel file /etc/squid/passwd.
* **Configurazione:** Ho aggiunto queste direttive in squid.conf e disabilitato l'accesso libero da localhost:

    # Configurazione Basic Authentication
      auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
      auth_param basic realm "Accesso Riservato Laboratorio"
      acl utenti_autenticati proxy_auth REQUIRED

    # Abilitazione accesso solo agli utenti autenticati
    # (Nota: ho commentato la riga 'http_access allow localhost')
      http_access allow utenti_autenticati


**Configurazione Client (Firefox)**
Per verificare il funzionamento, ho configurato il mio browser (**Firefox**) per passare attraverso il proxy.

* **Impostazioni Rete**: Sono entrato nelle impostazioni di rete del browser.
* **Configurazione Manuale**: Ho inserito il mio indirizzo di loopback `127.0.0.1` alla porta `3128`.
* **HTTPS**: Ho abilitato l'opzione per utilizzare questo proxy anche per le connessioni sicure (HTTPS).

**Risultato**: Tentando di accedere ai siti inseriti nella blacklist, il browser visualizza correttamente la pagina di errore "Access Denied" generata da Squid.




















## Dettagli Tecnici: La Blacklist

### File: `siti_vietati.txt`
Il file contiene la lista dei domini bloccati.
Ho inserito un commento iniziale con `#` per descrivere il contenuto, ma il resto del file è letto direttamente da Squid.

**Sintassi utilizzata:**
Utilizzo il punto `.` davanti ai domini (es. `.facebook.com`).
Questa è una sintassi specifica di Squid (`dstdomain`) che istruisce il proxy a bloccare il dominio principale e tutti i suoi sottodomini.

**Esempio:**
* Inserendo: `.facebook.com`
* Vengono bloccati: `facebook.com`, `www.facebook.com`, `m.facebook.com`, `login.facebook.com`, ecc.

**Domini attualmente bloccati nel laboratorio:**
* `.facebook.com`
* `.bing.com`
* `.gazzetta.it`