# Coda contenuti

Ogni sottocartella qui (tranne `_config/`) è un item di contenuto in lavorazione o completato, prodotto dalla pipeline di agenti descritta in `.claude/skills/content-pipeline/` e governato via `.claude/skills/content-admin/`.

## Struttura di un item

```
content-queue/2026-07-26-esempio-slug/
├── status.json          # stato macchina-leggibile: stadio corrente, cronologia, blocchi admin
├── brief.md              # output di content-strategist
├── draft.md              # output di content-copywriter
├── visual-brief.md       # output di content-art-director
├── qa-report.md          # output di content-qa (verdetto + problemi)
└── final/
    └── README.md         # output di content-producer: pacchetto pronto alla pubblicazione
```

Non tutti i file esistono finché il relativo stadio non è stato eseguito.

## Come si usa

- **Per avviare un nuovo contenuto**: invoca la skill `content-pipeline` con l'idea o il brief di partenza. Crea automaticamente una nuova cartella qui.
- **Per vedere lo stato di tutto ciò che è in corso, bloccare, sbloccare, riassegnare o correggere a mano un item**: invoca la skill `content-admin`.
- **Per cambiare le regole valide su tutti gli item futuri** (tono, palette, servizi, vincoli): modifica `_config/brand-guidelines.md` direttamente. Non serve nessuna skill per questo, è un file di configurazione letto da tutti gli agenti.

## Perché file e non un database

Il sistema gira dentro Claude Code, senza un server persistente: lo stato deve essere leggibile e modificabile sia dagli agenti che dall'admin come testo semplice, e deve sopravvivere tra una sessione e l'altra. `status.json` è la fonte di verità sullo stato; i file `.md` sono sia output di lavoro sia, cumulativamente, la cronologia di cosa ha prodotto ogni stadio.
