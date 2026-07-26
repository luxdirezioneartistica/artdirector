---
name: content-admin
description: Cruscotto e controllo amministratore sulla pipeline contenuti LUX — mostra lo stato di tutti gli item in corso e permette di bloccare, sbloccare, riassegnare a uno stadio precedente, modificare un output o approvare formalmente un item. Usa questa skill quando l'utente chiede lo stato della pipeline/coda contenuti, o vuole bloccare/sbloccare/riassegnare/modificare/approvare un item specifico. Non usarla per avviare un nuovo contenuto da zero: per quello usa content-pipeline.
---

# Cruscotto admin — pipeline contenuti LUX

L'utente è l'amministratore del sistema di agenti di produzione contenuti. Gli agenti (`content-strategist`, `content-copywriter`, `content-art-director`, `content-qa`, `content-producer`, coordinati dalla skill `content-pipeline`) lavorano in autonomia stadio per stadio, ma l'admin deve poter vedere tutto in ogni momento e intervenire su qualunque item, anche a metà lavorazione. Questa skill è quell'interfaccia.

Ogni item vive in `content-queue/<id>/status.json`. Non esiste un processo in background: lo stato riflette sempre l'ultima esecuzione della pipeline, e le tue azioni come admin vengono lette e rispettate dalla pipeline la prossima volta che gira su quell'item.

## Comando: stato (default se l'utente non specifica un'azione)

1. Elenca tutte le cartelle sotto `content-queue/` (esclusa `_config`) e leggi ogni `status.json`.
2. Presenta una tabella sintetica: id, idea (troncata), stadio corrente, stato dello stadio corrente, cicli di QA, ed eventuale blocco/nota admin attiva.
3. Segnala in evidenza gli item bloccati o fermi da fasi di QA ripetute (`qa_revisions >= 2`), perché sono quelli che richiedono davvero attenzione umana.

## Comando: blocca `<id>` [motivo]

Imposta in `status.json` di quell'item `admin.action = "block"` e `admin.note` col motivo fornito (o "bloccato manualmente dall'admin" se non specificato). Aggiorna `updated_at`. Conferma all'utente che la pipeline si fermerà prima del prossimo stadio non ancora eseguito.

## Comando: sblocca `<id>`

Imposta `admin.action = null` e `admin.note = null`. Non fa ripartire da sola la pipeline: informa l'utente che può rilanciarla con `content-pipeline` se vuole procedere.

## Comando: riassegna `<id>` `<stadio>`

Stadi validi: `strategia`, `copy`, `direzione-visiva`, `qa`, `produzione`. Rimetti a `"pending"` lo stadio indicato e tutti quelli successivi nell'array `stages` (cancellando i loro `started_at`/`ended_at` ma non i file di output già scritti su disco, che restano come cronologia), imposta `current_stage` su quello stadio, e `admin.action = null` se era impostato. Utile quando l'admin vuole far ripartire un item da un punto diverso da dove si era fermato il ciclo automatico di QA.

## Comando: modifica `<id>` `<file>`

L'admin vuole intervenire direttamente su un output (es. correggere a mano `draft.md` invece di far ripetere lo stadio a un agente). Leggi il file indicato, mostralo, applica le modifiche che l'utente richiede con Edit, e annota in `status.json` un campo `"human_edited": ["<file>", ...]` sull'item (accumulando, non sovrascrivendo le voci precedenti) così resta traccia che quel file non è (più, o non solo) output automatico.

## Comando: approva `<id>`

Imposta `admin.approved = true` con un timestamp. È un timbro di approvazione formale dell'admin sul pacchetto finale (utile come registro), indipendente dal fatto che la pipeline giri in autonomia senza aspettare gate di approvazione per procedere da uno stadio al successivo.

## Regole

- Non eseguire mai tu stesso uno stadio della pipeline da questa skill (non scrivere `brief.md`, `draft.md`, ecc.): per far avanzare un item usa/richiama `content-pipeline`, questa skill governa lo stato, non produce contenuto.
- Ogni azione admin va sempre scritta su `status.json`, mai tenuta solo a voce nella conversazione: è quello il registro che la pipeline e le sessioni future leggono.
- Se l'utente chiede un'azione su un id che non esiste, elenca gli id disponibili invece di procedere alla cieca.
