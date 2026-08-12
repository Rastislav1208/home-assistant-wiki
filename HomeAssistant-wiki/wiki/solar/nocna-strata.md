# Solár — Výpočet nočnej straty akumulačky

Meranie, koľko tepla akumulačná nádrž stratí cez noc do kotolne.

Súvisí s: [[../kotolna/snimace]] · [[../index]]

---

## Princíp

```
Strata [kWh] = (T_večer − T_ráno) × konštanta
konštanta = objem × 4180 / 3 600 000 = 0.987
```

- **T_večer**: teplota S1 uložená o 22:00 → `input_number.aku_teplota_vecer`
- **T_ráno**: teplota S1 uložená o 6:00 → `input_number.aku_teplota_rano`
- **0.987** = 850 l × 4180 J/kg·K / 3 600 000 J/kWh (0,987 kWh na 1 °C poklesu)

---

## Objem: prečo 850 l a nie 450 l

Nádrž má 850 l. Ohrievacie telesá hrejú len vrch + stred (~450 l).

**Vo vzorci je správne 850 l**, nie 450:
- Nočná strata = koľko tepla unikne z **celej** nádrže do kotolne
- Aj neohrievaná spodná voda má v sebe teplo a to tiež uniká
- 450 l je otázka **vstupu** energie (koľko solár dodá), nie **straty**

---

## ⚠️ Kľúčová oprava (12.8.2026) — uzamknutie rannej hodnoty

**Pôvodný problém:** senzor počítal `T_ráno` ako **live** hodnotu S1
(aktuálna teplota, mení sa každú sekundu). Dôsledok:
- Ráno (nádrž najstudenšia) → strata správna
- Cez deň (solár hreje) → strata klesá
- Popoludní mohla ísť do mínusu

Číslo teda platilo len ráno, kým slnko nezačalo hriať.

**Riešenie:** pridaný `input_number.aku_teplota_rano`, ktorý sa uloží
o 6:00 a už sa nemení. Strata = večer − uzamknuté_ráno je **fixná celý deň**.

---

## Komponenty v HA

| Prvok | Kde | Funkcia |
|-------|-----|---------|
| `input_number.aku_teplota_vecer` | Helper | Uloží S1 o 22:00 |
| `input_number.aku_teplota_rano` | configuration.yaml | Uloží S1 o 6:00 |
| Automatizácia "Aku - ulož večernú teplotu" | automations.yaml | Trigger 22:00 |
| Automatizácia "Aku - ulož rannú teplotu" | automations.yaml | Trigger 6:00 |
| `sensor.aku_nocna_strata` | configuration.yaml | Výpočet kWh |

---

## Dashboard

ApexCharts karta "Nočná strata vs teploty":
- Oranžové stĺpce = strata (ľavá os, kWh)
- Modrá čiara = večerná teplota (pravá os, °C)
- Červená čiara = ranná teplota (pravá os, °C)
- Rozsah teplôt 30–100 °C (kvôli zimnému kúreniu pecou)

Vyžaduje HACS **apexcharts-card** (vstavaný statistics-graph nevie
dve osi Y).

---

## Overená úmernosť

Teplejšia nádrž = väčšia strata (väčší rozdiel voči kotolni → rýchlejšie
chladnutie). Empiricky potvrdené: pri 63–65 °C strata 2,0–2,9 kWh,
pri 42–48 °C strata 1,0–1,2 kWh. Výpočet je fyzikálne konzistentný.
