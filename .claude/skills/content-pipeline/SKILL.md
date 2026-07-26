---
name: content-pipeline
description: Esegue la pipeline completa di produzione contenuti LUX, da un'idea grezza al pacchetto pronto alla pubblicazione, delegando in sequenza a content-strategist, content-copywriter, content-art-director, content-qa e content-producer. Usala quando l'utente porta un'idea o un brief cliente da trasformare in contenuto, o chiede di "avviare"/"far partire" un contenuto, un post, un reel o una campagna. Non usarla per interventi di supervisione su item già in corso (blocco, riassegnazione, stato): per quello usa content-admin.
---

# Pipeline contenuti LUX

Questa skill orchestra i cinque agenti in `.claude/agents/content-*.md` su un singolo item di contenuto, dall'idea al pacchetto finale, lasciando una traccia completa e ispezionabile in `content-queue/`. Il flusso gira in autonomia (non si ferma ad aspettare un'approvazione a ogni stadio), ma ogni passo è tracciato: l'utente, come admin, può in ogni momento controllare lo stato con la skill `content-admin` e intervenire — bloccare, riassegnare, modificare — su qualunque item, anche mentre è in corso.

## Passo 0 — crea l'item

1. Se `content-queue/_config/brand-guidelines.md` non esiste ancora, creala con dei default ragionevoli desunti dal brand (vedi il file se presente in un altro item, o chiedi all'utente prima di inventare vincoli). Non bloccare la pipeline per questo: procedi comunque segnalando che i default non sono stati validati dall'admin.
2. Genera uno slug breve dall'idea (kebab-case, poche parole) e usa la data odierna per creare `content-queue/AAAA-MM-GG-slug/`.
3. Scrivi `status.json` nella cartella dell'item con questa forma:

```json
{
  "id": "AAAA-MM-GG-slug",
  "idea": "testo originale fornito dall'utente",
  "created_at": "ISO8601",
  "updated_at": "ISO8601",
  "current_stage": "strategia",
  "stages": [
    {"name": "strategia", "agent": "content-strategist", "status": "pending", "output": "brief.md"},
    {"name": "copy", "agent": "content-copywriter", "status": "pending", "output": "draft.md"},
    {"name": "direzione-visiva", "agent": "content-art-director", "status": "pending", "output": "visual-brief.md"},
    {"name": "qa", "agent": "content-qa", "status": "pending", "output": "qa-report.md"},
    {"name": "produzione", "agent": "content-producer", "status": "pending", "output": "final/README.md"}
  ],
  "qa_revisions": 0,
  "admin": {"action": null, "note": null, "approved": false}
}
```

## Passo 1 — esegui gli stadi in sequenza

Per ogni stadio in ordine (strategia → copy → direzione-visiva → qa → produzione):

1. **Prima di iniziare lo stadio, rileggi `status.json`.** Se `admin.action` è `"block"`, fermati subito e riporta all'utente che l'item è bloccato dall'admin con la nota presente, senza eseguire lo stadio. Se `admin.action` è `"reassign"` verso uno stadio diverso da quello corrente, riparti da lì invece che dal punto in cui ti eri fermato.
2. Aggiorna lo stadio a `"status": "in_progress"` e `started_at`.
3. Invoca l'agente indicato (via Task/Agent tool) passandogli solo il percorso della cartella dell'item — l'agente legge da sé i file di cui ha bisogno.
4. Al termine, verifica che il file di output atteso esista. Aggiorna lo stadio a `"status": "done"`, `ended_at`, e `updated_at` e `current_stage` sull'intero status.json.
5. **Dopo lo stadio QA**: se il verdetto è "DA CORREGGERE", incrementa `qa_revisions`, riporta indietro `current_stage` allo stadio indicato nel `qa-report.md` (rimettendolo a `pending` insieme a tutti gli stadi successivi), e ripeti da lì. **Massimo due cicli di correzione automatici** (`qa_revisions <= 2`): al terzo esito negativo consecutivo, fermati, imposta `admin.action` a `"block"` con nota "QA ha respinto l'item 3 volte, serve intervento umano" e riporta il problema all'utente invece di continuare a ciclare.

## Passo 2 — chiudi

Quando lo stadio di produzione è `done`, riporta all'utente in breve: cosa è stato prodotto, dove si trova (`content-queue/<id>/final/README.md`), quanti cicli di QA sono serviti, ed eventuali note lasciate dagli agenti (es. asset Canva non generato, informazioni mancanti segnalate dal copywriter). Non riscrivere tu il contenuto degli output: rimanda ai file.

## Regole generali

- Non saltare stadi e non eseguirne più di uno in parallelo sullo stesso item: ogni stadio dipende dall'output di quello precedente.
- Se l'utente fornisce già un brief pronto invece di un'idea grezza, salta comunque `content-strategist` solo se esplicitamente richiesto; altrimenti fallo comunque passare per lo stadio di strategia, che può limitarsi a validare e formalizzare un brief già buono.
- Un item alla volta per invocazione della skill, a meno che l'utente non chieda esplicitamente di lanciarne più di uno in sequenza.
- Non modificare mai `status.json` di un item mentre `admin.action` è `"block"`, tranne per riflettere uno sblocco esplicito dell'admin.
