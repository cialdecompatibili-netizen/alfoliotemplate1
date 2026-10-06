# Workflow: sito di test automatico (clone + contenuti) in 2 comandi

Provato il 06/10/2026 con `testbirrificio` (tema birrificio artigianale, scelto a caso).
Sito: https://cialdecompatibili-netizen.github.io/testbirrificio/ - repo `cialdecompatibili-netizen/testbirrificio`.

## Comandi (dalla cartella `alfoliotemplate1`, output di poche righe)

1. `python nuovo_sito.py <nome-repo> [--dry-run]` - crea repo pubblica, copia (senza `.git`, `_site`, `node_modules`,
   `.env`, `_cestino`), riscrive le 4 righe iniziali di `CLAUDE.md` del clone, git init + push, attende 'Deploy site',
   abilita Pages (`gh-pages`), verifica l'HTML online (percorsi `/<repo>/assets/` e 0 occorrenze del nome di partenza).
   Se la cartella di destinazione esiste, non la tocca e continua da li. Durata: circa 3-4 minuti.
2. `python popola_sito.py <nome-repo> <pacchetto> [--dry-run] [--no-pulisci] [--no-push]` - legge
   `automazioni/testi/pacchetto_<pacchetto>.json` e: sposta in `_cestino/` servizi, post e progetti ereditati (69 servizi,
   36 post, 4 progetti), riscrive i testi della home (`pagine testi`), crea servizi, articoli e progetti nuovi, un solo
   commit + push. Idempotente (salta cio' che esiste gia'). Durata: circa 3 minuti (un processo Python per operazione).

Per un tema nuovo basta copiare `pacchetto_birrificio.json` in `pacchetto_<tema>.json` e cambiare i testi
(le chiavi `inizia_con` della home restano uguali: partono dal testo del modello, "## Web Agency" ecc.).

## Verificato online (dopo ~2 minuti dal push)

`/`, `/servizi/`, `/servizi/visite-guidate/`, `/blog/` rispondono 200 e non contengono piu' "Web Agency"/"web marketing".

## Punti critici

- Il clone eredita `nuovo_sito.py`, `popola_sito.py` e i pacchetti: vanno committati nel modello PRIMA di clonare.
- Prima di modificare il modello: checkpoint `checkpoint-AAAA-MM-GG` (fatto a mano il 06/10/2026: i moduli lo creano da soli solo sul sito che scrivono).
- Non toccano `url`/`baseurl`/`repos.json` (automatici). Il titolo del sito e' automatico dal nome repo.
- Da fare/guardare ancora: pagine `chi-siamo` e altre `_pages` conservano i testi del modello (`pagine testi` va esteso nel pacchetto con altre pagine), immagini e foto sono quelle del modello.
