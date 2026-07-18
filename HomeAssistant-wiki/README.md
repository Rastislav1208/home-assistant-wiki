# Smart Home Wiki — návod na použitie

Toto je **LLM Wiki** pre smart home systém, postavená podľa vzoru
Andreja Karpathyho (apríl 2026). Princíp: namiesto toho aby AI zakaždým
čítala celé repo, udržiava perzistentnú, prepojenú znalostnú bázu, ktorá
sa kumulatívne obohacuje.

> Obsidian je IDE · LLM je programátor · wiki je kódová báza

---

## Štruktúra

```
HomeAssistant/
├── CLAUDE.md          ← schéma pre AI (číta sa ako prvé)
├── raw/               ← nemenné zdroje (AI číta, nikdy neupravuje)
│   ├── manualy/       (PDF manuály TECH, ESBE...)
│   ├── logy/          (HA logy, chybové hlásenia)
│   └── konfigy/       (zálohy YAML konfigurácií)
└── wiki/              ← znalostná báza (AI vlastní a udržiava)
    ├── index.md       (katalóg — čítaj ako prvé pri dopyte)
    ├── log.md         (chronologický append-only denník)
    └── kotolna/
        ├── system.md  (architektúra)
        ├── snimace.md (NTC 1 kΩ snímače)
        ├── problemy.md(vyriešené + otvorené)
        └── todo.md    (úlohy)
```

---

## Ako to používať

### 1. Pridanie do existujúceho repo
Skopíruj `CLAUDE.md`, `raw/` a `wiki/` do koreňa repozitára `HomeAssistant`.

### 2. Otvor v Obsidian (MacBook M4)
`Open folder as vault` → vyber priečinok `wiki/`. Graph view ti ukáže
prepojenia medzi stránkami ([[wikilinks]] fungujú natívne).

### 3. Práca s Claude Code
```bash
cd ~/HomeAssistant
# Claude Code prečíta CLAUDE.md automaticky a vie ako udržiavať wiki
```

### 4. Voliteľne — graphify knowledge graph
```bash
pip install graphifyy
graphify install
# v Claude Code:  /graphify
```

---

## Tri operácie (z Karpathy vzoru)

| Operácia | Čo robí |
|----------|---------|
| **Ingest** | Nový zdroj → `raw/`, aktualizuj `wiki/`, zapíš do `log.md` |
| **Query** | Čítaj `index.md` → relevantné stránky → odpoveď so syntézou |
| **Lint** | Pravidelná kontrola: kontradikcie, zastarané, osirelé stránky |

---

## Ďalšie kroky

- Ingestovať projekty solár / kamery / nspanel (zatiaľ len kostra v index.md)
- Skopírovať PDF manuály do `raw/manualy/`
- Pridať zálohy YAML konfigov do `raw/konfigy/`
