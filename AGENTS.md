# AGENTS.md

## Cosa è
App a **file singolo**: `Turni Swisscom.html` (HTML + CSS + JS inline, zero dipendenze, zero build). Mostra i turni di lavoro importati da un calendario Genesys (`.ics`) o da testo incollato, con vista settimanale, export WhatsApp e PDF.

- Nessun package manager, nessun test, nessuna CI. La verifica è manuale: apri il file nel browser, trascina un `.ics`, controlla la console.
- Repository git: remote `origin` = `https://github.com/lorissalerno/turni.git`, branch `main` (`gh` non è installato).
- UI e messaggi in italiano: mantieni la lingua.

## Struttura del file (in ordine)
1. `<style>`: variabili tema in `html[data-theme="white"|"dark"]`, poi stili screen, poi blocco `@media print` per il PDF.
2. Markup: due contenitori separati — `.wrap.screen` (UI) e `#print` (solo stampa, generato al volo).
3. `<script>`: utility date → parser → storage → merge → link/sync → render → WhatsApp → PDF → `init()`.

## Regole non ovvie

### Date
- Le chiavi giorno sono stringhe `YYYY-MM-DD`. Tutta la matematica sui giorni usa helper UTC (`kParse`, `kAdd`, `dow`, `monday`, `kOf`). **Non usare `getDate()`/`setDate()` locali** per i calcoli; l'unico uso locale voluto è `kDate(new Date())` in `oggiK()` per "oggi".
- Gli orari degli eventi ICS sono convertiti in `Europe/Zurich` con `Intl` (`zurich()`). I `DTSTART` senza `Z`/`TZID` vengono corretti con l'offset Zurigo. Non sostituire con `new Date(...)` locale.
- Un evento che scavalca la mezzanotte viene troncato a `24*60` sul giorno di inizio.

### localStorage (persistenza)
- Chiavi: `turni_v7` (turni), `turni_meta7` (last sync), `turni_tema`, `turni_link`.
- Se cambi la forma dei dati salvati, **incrementa il suffisso `v7`** in `LS`/`LS_META`: i dati vecchi restano orfani ma non rompono il rendering.
- `LS_LINK` contiene l'URL personale del calendario: resta solo nel browser, non va mai scritto nel file né committato. Il campo è `type="password"` di proposito.

### Merge / import
- `merge()` sovrascrive anche i giorni passati e **cancella** i giorni salvati nell'intervallo importato che non compaiono nel nuovo file (diventano "liberi"). Non "aggiustarlo" in modo silenzioso: è il comportamento atteso.
- Due parser distinti: `parseICS` e `parseTXT`, scelti da `parseAuto`. Se cambi la classificazione (`isPause`/`isTrain`/`isVac`), ricorda che l'ordine conta: `cls()` valuta pausa prima di training, quindi "Lunch" dentro un training diventa pausa.
- Testo proveniente da ICS/TXT è **non fidato**: passalo sempre da `esc()` prima di inserirlo in `innerHTML`.

### Rendering
- Il render è a stringhe HTML con `innerHTML`; nessun framework, nessun DOM virtuale. `trackHTML()` è condiviso tra vista schermo (`track`) e stampa (`ptr`): se cambi la geometria delle barre, verifica entrambi.
- `fitLabels()` nasconde le etichette troppo strette con la classe `.tiny`; va richiamata dopo ogni render (già fatto via `requestAnimationFrame`).
- Il PDF è una vista separata: `pdf()` costruisce `#print` con classi proprie (`pg`, `pgrid`, `pscale`, `prow`, `ptr`) e colori hardcoded chiari. Modifiche al tema scuro non devono toccare la stampa.

### Firma
- `COPY` in fondo allo script (© Loris Salerno / reparto) finisce sia nel footer a schermo sia nel PDF. Non rimuoverla.

## Convenzioni di stile
- JS: funzioni corte, costanti in maiuscolo, nomi in italiano (`scala`, `sett`, `aperto`) — segui il file, non introdurre naming inglese misto.
- Niente librerie esterne né `<script src>`: l'app deve funzionare aprendo il file offline.
- Il file usa righe lunghe e dense: mantieni lo stile compatto, non riformattare tutto.
