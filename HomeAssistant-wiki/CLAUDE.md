# CLAUDE.md — Home Assistant Smart Home Wiki

Tento súbor je **schéma** pre AI agenta (Claude Code / Codex / Cursor).
Hovorí, ako je wiki štruktúrovaná a aké workflows sledovať.
Vznikol podľa vzoru Andreja Karpathyho „LLM Wiki" (apríl 2026).

---

## 1. Čo je tento repozitár

Centrálny repozitár pre celý smart home systém (Rastislav).
Obsahuje:
- **`raw/`** — nemenné zdrojové súbory (manuály, logy, zálohy konfigov). AI z nich číta, nikdy ich neupravuje.
- **`wiki/`** — znalostná báza, ktorú vlastní a udržiava AI. Človek ju číta, AI ju píše.
- **`projekty/`** — pôvodná štruktúra projektov (kotolna, solar, kamery, nspanel).
- **`home-assistant/`** — živá HA konfigurácia (bez hesiel).

---

## 2. Prehľad systému

Home Assistant OS beží na Raspberry Pi. Vzdialený prístup cez **Tailscale**.

| Oblasť | Hardware | Stav |
|--------|----------|------|
| Kúrenie | TECH EU-i-2 Plus OT + EU-WiFi-RS | V prevádzke |
| Solár | Victron MPPT (2×), batéria, Modbus (Waveshare) | V prevádzke |
| Kamery | Frigate, Hikvision DVR, go2rtc | V prevádzke |
| Izbové ovládanie | Sonoff NSPanel EU | Plánované |
| Sieť | Tailscale | V prevádzke |

---

## 3. Konvencie

- **Entitné prefixy:** `eu_i_2_plus_ot_*` (kúrenie), `victron_*` (solár)
- **Snímače kotolne:** všetky typ **NTC 1 kΩ** (NIE 10 kΩ — častá chyba!)
- **Dashboard:** `VELIN` (hlavný), tab `Kotolňa` pridaný
- **Jazyk:** slovenčina pre všetku dokumentáciu a komunikáciu
- **Pracovné nástroje:** MacBook M4, VS Code, Obsidian, GitHub, Google Drive

---

## 4. Projekty (odkazy do wiki)

| Projekt | Wiki | Popis |
|---------|------|-------|
| Kotolňa | `wiki/kotolna/` | TECH EU-i-2 Plus OT, Viadrus U22, akum. nádrž |
| Solár | `wiki/solar/` | Victron MPPT 2×, batéria, Modbus/Waveshare |
| Kamery | `wiki/kamery/` | Frigate, Hikvision DVR, go2rtc |
| NSPanel | `wiki/nspanel/` | Sonoff NSPanel EU (plánované, ESPHome) |

---

## 5. Workflow — INGEST novej zmeny

Keď príde nová informácia (zmena konfigu, nový log, vyriešený problém):

1. Ulož zdroj do `raw/` (logy → `raw/logy/`, konfigy → `raw/konfigy/`).
2. Aktualizuj relevantnú stránku vo `wiki/`.
3. Pridaj riadok do `wiki/log.md` s prefixom:
   `## [RRRR-MM-DD] typ | Popis`
   (typy: `install`, `config`, `fix`, `todo`, `query`, `lint`)
4. Skontroluj, či treba aktualizovať `wiki/index.md`.
5. Over cross-referencie — odkazuje nová stránka na súvisiace?

## 6. Workflow — QUERY (dopyt)

1. Najprv prečítaj `wiki/index.md` → nájdi relevantné stránky.
2. Načítaj len tie stránky, nie celé repo.
3. Odpovedz so syntézou + odkazmi na zdroje.
4. **Dobré odpovede zapíš späť do wiki** ako nové stránky (analýzy, porovnania) — nech sa nestratia v chate.

## 7. Workflow — LINT (kontrola zdravia)

Pravidelne over wiki:
- Kontradikcie medzi stránkami
- Zastarané tvrdenia (novší zdroj ich prekonal)
- Osirelé stránky bez odkazov
- Dôležité koncepty bez vlastnej stránky
- Chýbajúce cross-referencie

---

## 8. ⚠️ KRITICKÉ BEZPEČNOSTNÉ POZNÁMKY

Tieto sa NIKDY nesmú stratiť pri žiadnej úprave wiki:

1. **Viadrus U22 NEMÁ havarijný chladič** — povinné podľa STN EN 303-5. Musí doriešiť kúrenár. NEDORIEŠENÉ.
2. **Snímače sú NTC 1 kΩ** — nie 10 kΩ. Nastavenie v menu regulátora to musí odrážať.
3. **Čerpadlá sú prehodené:** čerpadlo kotla → Valve pump 2, čerpadlo radiátorov → Valve pump 1.
4. **Servo ESBE:** Hnedá=Open, Čierna=Close, Modrá=N (boli pôvodne zapojené naopak — opravené).

---

## 9. Indexovacie a logovacie súbory

- **`wiki/index.md`** — obsahový katalóg. Každá stránka s odkazom a jednoriadkovým popisom. AI ho aktualizuje pri každom ingeste.
- **`wiki/log.md`** — chronologický append-only denník. Greppovateľný:
  `grep "^## \[" wiki/log.md | tail -5` → posledných 5 záznamov.
