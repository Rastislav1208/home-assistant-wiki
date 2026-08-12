# Log — Smart Home Wiki

Chronologický append-only denník. Každý záznam má prefix:
`## [RRRR-MM-DD] typ | Popis`

Typy: `install` `config` `fix` `todo` `query` `lint`

Greppovateľné: `grep "^## \[" wiki/log.md | tail -5`

---

## [2026-06-06] install | EU-i-2 Plus OT prvé spustenie, firmware 1.6.9G overený
## [2026-06-06] install | EU-WiFi-RS nakonfigurovaný — IP 192.168.1.74, GW/DNS 192.168.1.1
## [2026-06-06] config | Všetkých 7 snímačov NTC 1 kΩ zapojených a overených (CH, DHW, Return, External, S1, S2, Valve 1)
## [2026-06-06] fix | Servo ESBE — hnedá↔čierna boli prehodené, opravené (Open/Close)
## [2026-06-06] fix | Čerpadlá — Valve pump 1↔2 prehodené oproti logickému poradiu, zdokumentované
## [2026-06-06] config | emodul.eu registrácia, Tech Controllers integrácia v2.8.0 cez HACS
## [2026-06-06] config | Dashboard "Kotolňa" pridaný do VELIN, systém aktívne reguluje ventil
## [2026-06-06] todo | Havarijný chladič na Viadrus U22 — CHÝBA, povinné STN EN 303-5, treba kúrenára
## [2026-06-08] config | Vytvorená LLM Wiki štruktúra (raw/ + wiki/ + CLAUDE.md) podľa Karpathy vzoru
## [2026-06-28] config | Pridaný nástroj wiki/nastroje/prompt-inzinier.md (upravený "Prompt Inžinier" pre Claude + HA doménu, slovenčina)
