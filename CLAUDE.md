# CLAUDE.md — Equilibrio dei corpi (Statica)

Guida per assistenti AI che lavorano su questo repository. Leggila prima di
modificare il codice.

## Cos'è

Web app didattica **in italiano** sulla **statica** (equilibrio dei corpi)
per studenti di **liceo**. Tutto in un **unico file** `index.html`
(~2400 righe): **nessuna installazione, nessuna dipendenza, nessun build**.
Si apre facendo doppio clic sul file in un browser moderno.

Il pubblico è composto da studenti: mantieni un tono **chiaro e didattico** e
tutti i contenuti **in italiano**.

## File nel repository

- `index.html` — l'intera applicazione (HTML + CSS + JS in-line). È l'unico
  file del progetto.

## Come eseguire e testare

Non esiste una suite di test automatica. Il testing è **manuale nel browser**:

```bash
xdg-open index.html            # apri direttamente il file
# oppure
python3 -m http.server 8000    # poi apri http://localhost:8000
```

Dopo ogni modifica, verifica a mano:
- la navigazione tra le **scene** (barra `nav.main`) e le **sotto-schede**
  (teoria / sim / eserc / quiz);
- i **diagrammi SVG** interattivi e il **trascinamento** dei punti;
- i **cursori** (slider) e i valori mostrati;
- gli **esercizi** con suggerimenti progressivi e i **quiz**;
- la **stampa** (pulsante stampa → anteprima, regole `@media print`);
- il layout **responsive**.

## Deploy

Pubblicato tramite **GitHub Pages dalla radice** del repository. Nessun passo
di build.

## Architettura

Il JavaScript è in un unico `<script>` (da riga ~810). A differenza di un'app
data-driven, qui **i contenuti sono scritti direttamente nel markup**: le
scene sono blocchi HTML, e ogni scena ha una funzione JS che ne disegna il
diagramma.

### Scene (contenuto nel markup)

Ci sono 9 `<section class="scene">`, ognuna con `id` e `data-title`:

`intro` · `funi` · `orizz` · `inclin` · `molla` · `momento` · `leva` ·
`trave` · `rigido`

Per aggiungere/modificare un argomento si edita il blocco `<section>`
corrispondente (non un array di dati). Ogni scena contiene fino a 4
sotto-schede tramite `nav.subnav` + `data-sub` (`teoria`, `sim`, `eserc`,
`quiz`); i pannelli sono `.subscene[data-sub=...]`.

### Diagrammi (SVG, non Canvas)

Le simulazioni usano **SVG** disegnato via JS (non `<canvas>`). Ogni scena ha
un `<svg id="svgXxx" viewBox=...>` e una funzione di rendering dedicata:

| Scena | Funzione | Riga |
|---|---|---|
| Corpo appeso a funi | `renderFuni()` | :882 |
| Piano orizzontale | `renderOrizz()` | :924 |
| Piano inclinato | `renderInclin()` | :964 |
| Forze elastiche (Hooke) | `renderMolla()` | :1045 |
| Momento di una forza | `renderMomento()` | :1161 |
| Leva | `renderLeva()` | :1208 |
| Trave su due appoggi | `renderTrave()` | :1266 |
| Corpo rigido | `renderRigido()` | :1342 |

Utility di disegno/interazione condivise:
- `arrow(...)` (:817) — disegna un vettore/freccia; `hatch(...)` (:835) —
  tratteggio (superfici/vincoli); `arc(...)` (:910) — archi (angoli/momenti).
- Trascinamento: `makeDrag(...)` (:1421), `svgPt(...)` (:1407),
  `ratioFromEvent(...)` (:1319), con `clamp`/`setSlider`.
- `bindRange(id, valId, fmt)` (:872) — collega uno slider al testo del valore
  e chiama il render su `input`.

### Navigazione

- `showTab(name)` (:850) attiva la scena e il pulsante corrispondente e
  scrolla in alto; le `.home-card` della intro usano `data-go` per navigare.
- Le sotto-schede sono gestite genericamente iterando su `nav.subnav`.

### Dati di esercizi e quiz

- `const ESERCIZI` (:1604) — esercizi (con suggerimenti progressivi);
  `renderEsercizi()` (:2017) li mostra.
- `const QUIZZES` (:2074) — quiz per scena; `renderQuiz`/`answerQuiz`/
  `nextQuiz`/`restartQuiz` (:2313+) ne gestiscono il flusso.

### Colori dei vettori (CSS custom properties)

Le frecce sono **codificate per colore** tramite variabili su `:root`:
`--vT` tensione · `--vP` peso · `--vN` normale · `--vF` attrito ·
`--vFa` · `--vFe` · `--vM` momento. Usa sempre queste variabili invece di
colori hardcoded, così i diagrammi restano coerenti.

## Convenzioni

- **Lingua**: italiano ovunque — testo UI, commenti, stringhe. I nuovi
  contenuti devono essere in italiano.
- **Zero dipendenze**: niente npm, bundler, framework o CDN. Non introdurre un
  passo di build né dipendenze esterne senza richiesta esplicita.
- **Un solo file**: HTML, CSS e JS restano dentro `index.html`.
- **SVG, non Canvas**: i diagrammi si disegnano con nodi SVG; riusa `arrow`,
  `hatch`, `arc` e i colori `--vX` esistenti.
- **Stampa**: mantieni funzionanti le regole `@media print` (:220) e il
  pulsante di stampa.
- **Nessuna rete/stato persistente**: l'app non fa chiamate di rete e non usa
  `localStorage`. Non aggiungere tracciamento.
- **Matematica/fisica**: caratteri Unicode (Σ, ΣF, ΣM, °, pedici) e le classi
  di formattazione già presenti.

## Note per le modifiche

- Il file è grande: raggiungi con la ricerca la `<section>` della scena o la
  funzione `renderXxx` prima di modificare.
- Aggiungere una scena = nuovo blocco `<section class="scene">` con
  sotto-schede + eventuale `renderXxx()` per il diagramma SVG, più un pulsante
  in `nav.main` e (se serve) una `.home-card` nella intro.
- Dopo modifiche ai diagrammi, verifica nel browser il trascinamento e che il
  render venga richiamato sugli `input` degli slider.
