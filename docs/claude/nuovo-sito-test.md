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
- **Due punti nelle description = pagina 404 senza errori.** `pubblica_chi_siamo.py` scrive `description:` nel front matter senza virgolette: un ':' nel testo rompe lo YAML e Jekyll non genera `/chi-siamo/`. Nel pacchetto niente ':' in `chi_siamo.pagina.description` (e meglio anche in `seo_description`).
- **`servizi_gruppi.yml` obbligatorio:** i servizi con un `gruppo` non elencato in `_data/servizi_gruppi.yml` finiscono in "Altri servizi". Il pacchetto ha la chiave `gruppi` (lista, nell'ordine di visualizzazione) e lo script riscrive il file.
- **Slug con `_`** (`1_project`, `announcement_1`, `the_godfather`): il motore li elenca, ma la regex iniziale di `popola_sito.py` non li vedeva (corretto: `[a-z0-9_-]`). `_teachings` e `_books` non hanno modulo: si svuotano con `svuota_cartelle`.

## Chiavi del pacchetto (tutte opzionali)

`home` (come `testi/home_*.json`, puo' contenere piu' pagine: home, contatti...), `gruppi`, `menu` ({"agenzia": "Birrificio"}, rinomina la voce di menu), `chi_siamo` (intero contenuto di `chi_siamo.json`, poi `pubblica_chi_siamo.py`), `config_descrizione` (la riga sotto `description: >` di `_config.yml`), `svuota_cartelle`, `servizi`, `post`, `progetti`. Pulizia ereditata: servizi, post, progetti e news (`--no-pulisci` per saltarla).

## Verifica fatta il 06/10/2026 (dopo la seconda esecuzione)

`/`, `/chi-siamo/`, `/servizi/`, `/contatti/` rispondono 200 senza "Web Agency"/"a Roma"; `/projects/1_project/` e `/teachings/...` sono 404 (demo rimossi). Restano le immagini del modello e il footer 'Powered by Jekyll'.
