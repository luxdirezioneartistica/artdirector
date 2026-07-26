# artdirector

Repo di LUX, agenzia di direzione artistica. Contiene due cose distinte:

1. **`index.html`** — landing page statica del sito LUX. Singolo file, nessuna build.
2. **Sistema di agenti per la produzione contenuti**, governato dall'admin (l'utente), che porta un'idea dalla ideazione alla produzione finale pronta alla pubblicazione.

## Il sistema di agenti

- `.claude/agents/content-strategist.md`, `content-copywriter.md`, `content-art-director.md`, `content-qa.md`, `content-producer.md` — i cinque stadi della pipeline, ognuno un subagent con un ruolo unico.
- `.claude/skills/content-pipeline/` — orchestratore: fa girare un'idea attraverso tutti e cinque gli stadi in sequenza, in autonomia, tracciando tutto su file.
- `.claude/skills/content-admin/` — cruscotto admin: stato di tutti gli item in corso, blocco/sblocco, riassegnazione a uno stadio precedente, modifica manuale di un output, approvazione formale.
- `content-queue/` — stato e output di ogni item (uno per idea/contenuto), più `_config/brand-guidelines.md` con tono di voce, palette, servizi e vincoli che tutti gli agenti rispettano.

Per avviare un contenuto: skill `content-pipeline`. Per controllare o intervenire su quelli in corso: skill `content-admin`. Dettagli su formato di `status.json` e struttura cartelle in `content-queue/README.md`.

Il modello di governance è "supervisione totale + intervento": la pipeline non si ferma ad aspettare un'approvazione a ogni stadio, ma ogni passo è tracciato e l'admin può bloccare, riassegnare o correggere qualunque item in qualunque momento.
