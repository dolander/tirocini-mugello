# Dashboard Studenti Infermieristica e OSS — Zona Mugello

Pannello web di consultazione per la gestione dei tirocini degli studenti presso la Zona Mugello dell'Azienda USL Toscana Centro.

---

## Accesso

| Rete | URL | Note |
|---|---|---|
| **Rete aziendale USL** | https://dolander.github.io/tirocini-mugello/ | Funziona con Fortinet |
| **Fuori rete aziendale** | https://turni-studenti-mugello.netlify.app | Bloccato da Fortinet |

**Password**: dolander

**Foglio Google (sorgente dati)**: https://docs.google.com/spreadsheets/d/1aLhvV7ZdcjMiyVHRCgbp4JayLfKC_1e0AHRbqasi41s/edit

---

## Architettura del sistema

```
Google Sheets  →  Google Apps Script (proxy CORS)  →  Dashboard HTML
   (dati)         script.google.com/macros/s/...       (visualizzazione)
```

- **Google Sheets**: fonte unica dei dati, si modifica da qui (app mobile o browser).
- **Google Apps Script**: proxy che scarica il CSV lato server e lo restituisce con header CORS corretti, così il browser può leggerlo senza blocchi di sicurezza. Script: "Proxy CSV Tirocini" sull'account enrico.dolabelli@gmail.com.
- **GitHub Pages** (`dolander/tirocini-mugello`, pubblico): deploy per la rete aziendale.
- **Netlify** (`dolander/dashboard-studenti`, privato): deploy alternativo per uso fuori rete aziendale. Auto-deploy ad ogni push su main.

**Importante**: quando si modifica `index.html`, va aggiornato su **entrambi** i repository.

---

## Come modificare i dati

Tutto si modifica **nel foglio Google**, non nella dashboard (che è sola lettura).

**Da telefono/tablet**: app Google Sheets → file "Dashboard Studenti Infermieristica e OSS".
**Da PC**: aprire il link sopra oppure usare il pulsante "📝 Apri Google Sheet" nella dashboard.

La dashboard si aggiorna automaticamente ogni 60 secondi. In alternativa, usare il pulsante "↻ Aggiorna".

### Struttura colonne del foglio (tab per anno)

| Colonna | Valori ammessi | Note |
|---|---|---|
| ID | numero intero | univoco per riga, non ripetere |
| NOME | testo | |
| COGNOME | testo | |
| CORSO | INF / OSS | dropdown |
| DATA INIZIO | GG/MM/AAAA | |
| DATA FINE | GG/MM/AAAA | |
| DATA FINE proroga | GG/MM/AAAA | lasciare vuoto se non c'è proroga |
| ANNO CORSO | 1 / 2 / 3 | anno del corso di studi |
| Tipo di orario | H 12 / H 24 | dropdown |
| SETTING | testo | nome del reparto |
| Orario | SI / NO | segue orario standard del reparto |
| TUTOR | testo | cognome del tutor |
| Accoglienza | SI / NO | |
| Note | testo | scrivere "VALUTAZIONE SI" per la stellina ★ |
| STATO | ATTIVO / SOSPESO | lasciare vuoto = ATTIVO |

### Studenti con più tirocini
Uno stesso studente può avere più righe (reparti o periodi diversi). Assegnare sempre un **ID diverso** a ogni riga.

---

## Come aggiungere un nuovo anno (es. 2027)

1. Nel foglio Google: clic destro sul tab "2026" → **Duplica** → rinomina in "2027" → svuota le righe dati.
2. Pubblica il tab: **File → Condividi → Pubblica sul web** → tab "2027" → CSV → copia il link.
3. Nello script Google Apps Script ("Proxy CSV Tirocini" su script.google.com): aggiungi `"2027": "IL_LINK_CSV"` nel dizionario `SHEET_YEARS`, poi fai una nuova versione del deployment.
4. In `index.html` aggiungi `{ anno: 2027 }` nell'array `SHEET_YEARS`.
5. Pusha su entrambi i repository.

---

## Protezione accesso

Password lato client con hash SHA-256. Sufficiente per scoraggiare accesso casuale, non è sicurezza enterprise. Per una protezione robusta servirebbe Netlify Pro (~19€/mese).

**Per cambiare la password**: chiedere a Claude.

---

## Funzionalità della dashboard

| Funzione | Comportamento default |
|---|---|
| **Selettore anno** | Anno corrente |
| **Solo in corso** | ✅ Attivo (nasconde conclusi e futuri) |
| **Filtra per periodo** | Disattivo — inserire date dal/al per filtrare e stampare un gruppo specifico |
| **Solo dati mancanti** | Disattivo |
| **Mostra sospesi** | Disattivo |
| **Calendario** | Vista barre temporali dinamica (range calcolato sui dati reali) |
| **Elenco** | Vista tabella filtrabile |
| **Stampa / Esporta PDF** | Stampa la vista attuale con i filtri attivi |
| **Apri Google Sheet** | Apre il foglio in una nuova scheda |

### Lettura visiva delle barre (calendario)

| Elemento | Significato |
|---|---|
| Barra **verde** | Turno H12 |
| Barra **rossa** | Turno H24 |
| Barra **semi-trasparente, righe diagonali** | Concluso |
| Barra **semi-trasparente, righe oblique chiare** | Non ancora iniziato |
| Barra **grigia** | Sospeso |
| **Bordo tratteggiato arancione** | Dati mancanti |
| **★ stellina dorata** | Valutazione consegnata |
| Badge **INF** azzurro / **OSS** ambra | Tipo di corso |

---

## Struttura repository

```
dolander/tirocini-mugello  (pubblico — GitHub Pages, rete aziendale)
└── index.html

dolander/dashboard-studenti  (privato — Netlify, uso fuori rete)
├── index.html               ← stesso file del repo pubblico
├── netlify/functions/
│   └── sheet-csv.mts        ← proxy obsoleto (ora si usa Apps Script)
└── README.md
```

---

## Note tecniche (lezioni apprese)

- **Fortinet USL**: blocca `netlify.app` con `NET::ERR_CERT_AUTHORITY_INVALID`, non mostra "Procedi comunque". Soluzione: GitHub Pages su dominio `github.io`.
- **CORS**: il browser non può fare fetch diretto dei CSV Google. Soluzione attuale: Google Apps Script come proxy lato server.
- **GitHub Pages gratuito**: funziona solo su repository pubblici.
- **Netlify password** (`_headers Basic-Auth`): richiede piano Pro, non funziona sul piano gratuito.
- **Calendario dinamico**: il range Gen-Ago era hardcoded e non mostrava tirocini autunnali. Ora calcolato dinamicamente dai dati reali con margine di 3 settimane.
- **Deploy su due repo**: ogni modifica a index.html va pushata su entrambi dolander/tirocini-mugello e dolander/dashboard-studenti.

---

## Contatti tecnici

Per modifiche al codice, nuovi anni, aggiornamento password o problemi: chiedere a **Claude** nella stessa chat di progetto.
