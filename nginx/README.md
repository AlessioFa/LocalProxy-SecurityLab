Hai perfettamente ragione, è brutto perché **mancano gli "a capo" (invio)** tra le righe.

Il Markdown è molto schizzinoso: se scrivi `Testo.**Grassetto**` tutto attaccato, lui non capisce e ti mostra gli asterischi. Devi dare "aria" al testo.

Ecco la versione **corretta e pulita**. Ho aggiunto tutte le spaziature necessarie affinché GitHub lo trasformi in una pagina bella da vedere (con le linee, i titoli grandi e i grassetti veri).

Copia e incolla questo blocco intero nel tuo `README.md` della cartella `nginx/`:

````markdown
# Configurazione Reverse Proxy (Nginx)

In questa sezione del laboratorio ho configurato **Nginx** come *Reverse Proxy*.
L'obiettivo è proteggere un **Backend Server** simulato (che gira su una porta interna), nascondendolo dal traffico diretto e applicando filtri di sicurezza avanzati.

---

## 🏗 Architettura del Laboratorio

Lo scenario simulato sulla macchina locale (`localhost`) è il seguente:

1.  **Client (Browser):** Accede al sito pubblico `www.miositoesterno.com` (simulato via `/etc/hosts`).

2.  **Front-End (Nginx):** Ascolta sulla porta **80**, riceve la richiesta, applica i controlli di sicurezza e la gira al backend.

3.  **Back-End (Python):** Un semplice server HTTP in ascolto sulla porta **8080** che contiene i dati "segreti".

---

## 📝 Diario di Configurazione

### Fase A: Preparazione Backend e Installazione

Non avendo un secondo server fisico, ho simulato il Backend utilizzando il modulo `http.server` di Python.

**Comandi eseguiti:**

```bash
# Terminale 1: Avvio del Backend Simulato
# Crea una risposta finta per verificare l'accesso
mkdir backend_segreto
echo "<h1>Backend Raggiunto!</h1>" > index.html
python3 -m http.server 8080 --bind 127.0.0.1
````

Successivamente (in un secondo terminale) ho installato Nginx:

```bash
sudo apt update && sudo apt install nginx -y
```

### Fase B: Configurazione Reverse Proxy

Ho creato il file `/etc/nginx/sites-available/reverse-app.conf` per istruire Nginx a fare da "ponte".

**Punti chiave della configurazione:**

  * **`proxy_pass http://127.0.0.1:8080`**: Reindirizza il traffico verso il server Python.
  * **Header Forwarding**: Ho configurato Nginx per passare al backend gli header originali (`Host`, `X-Real-IP`), altrimenti il backend vedrebbe tutte le richieste provenire da Nginx stesso.

### Fase C: Hardening e Sicurezza (Rate Limiting)

Per proteggere il server da attacchi DoS (Denial of Service) e Fingerprinting, ho applicato le seguenti direttive di sicurezza nel file di configurazione:

```nginx
# 1. Definizione Zona Rate Limiting (fuori dal blocco server)
# Crea un buffer di 10MB ('one') e accetta max 1 richiesta al secondo per IP
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    listen 80;
    server_name [www.miositoesterno.com](https://www.miositoesterno.com);

    # 2. Anti-Fingerprinting
    # Nasconde la versione di Nginx dagli header di risposta
    server_tokens off;

    location / {
        # 3. Applicazione del Limite
        # Tollera brevi picchi (burst=5) poi blocca con errore 503
        limit_req zone=one burst=5 nodelay;
        
        proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
        
        # Propagazione degli header originali
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Fase D: Verifica Funzionale

1.  **Simulazione DNS:** Ho modificato il file `/etc/hosts` aggiungendo la riga `127.0.0.1 www.miositoesterno.com` per simulare un dominio reale.
2.  **Test Accesso:** Visitando `http://www.miositoesterno.com`, vedo correttamente la pagina servita da Python (Porta 8080).
3.  **Test DoS (Rate Limiting):** Aggiornando rapidamente la pagina (più di 5 volte al secondo), Nginx blocca le richieste restituendo l'errore **503 Service Temporarily Unavailable**.

<!-- end list -->

```
```