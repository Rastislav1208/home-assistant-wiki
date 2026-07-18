# Kotolňa — Snímače

⚠️ **Všetky snímače sú typ NTC 1 kΩ** (NIE 10 kΩ). Toto musí zodpovedať
nastaveniu typu snímača v menu regulátora.

Súvisí s: [[system]] · [[problemy]]

---

## Zapojené snímače (overené 18.7.2026)

| Svorka regulátora | Umiestnenie | Názov v HA | Stav |
|-------------------|-------------|------------|------|
| CH sensor | Výstup kotla Viadrus U22 | Teplota kotol | ✅ |
| **S3 sensor** | **TÚV výmenník v akumulačke** | **TÚV** | ✅ |
| S1 sensor | Stred akumulačky | AKU Stred | ✅ |
| S2 sensor | Spiatočka radiátorov (informatívne) | Spiatočka | ✅ |
| Valve sensor 1 | Výstup trojcestného ventilu | Radiátory | ✅ |
| External sensor | Vonkajšia (severná) stena | Vonkajšia | ✅ |
| DHW sensor | — | — | ❌ Nepripojený (úmyselne) |
| Return sensor | — | — | ❌ Nepripojený (viď nižšie) |
| S4, Valve sensor 2 | — | — | ❌ Nepoužité |

> Nepripojené vstupy zobrazuje regulátor ako **−203,0 °C**.
> Sivý text na displeji = funkcia je v regulátore vypnutá.

---

## ⚠️ Wiring gotcha — TÚV snímač na S3, nie na DHW

**Problém:** Snímač TÚV bol pôvodne na svorke `DHW sensor`. Regulátor
meral správne (70 °C na displeji), ale hodnotu **neposielal do HA**
(`unavailable`), pretože **okruh TÚV je v regulátore vypnutý**.

**Prečo je okruh vypnutý:** TÚV výmenník v akumulačke nepoužíva žiadne
čerpadlo ani ventil — voda je z vonku pod tlakom. Netreba nič regulovať,
len merať.

**Riešenie (18.7.2026):** Snímač prepojený zo svorky `DHW sensor` na
`S3 sensor`. Prídavné snímače sú čisto meracie, nemajú naviazanú riadiacu
logiku → **riadenie ostalo nezmenené**, len pribudla hodnota v HA.

> 💡 Zovšeobecnenie: ak chceš v HA vidieť hodnotu, ktorú regulátor meria,
> ale neposiela, presuň snímač na voľný **prídavný** vstup (S1–S4).

---

## ⚠️ Ochrana spiatočky je mechanická

Svorka `Return sensor` je **nepripojená** (−203 °C). Elektronická ochrana
spiatočky teda nefunguje — rieši ju **mechanicky bajonet ESBE 60 °C**.

Pozor: entita „Spiatočka" na dashboarde je **S2 = spiatočka radiátorov**,
nie spiatočka kotla. Je len informatívna.

---

## Charakteristika NTC 1 kΩ

NTC = Negative Temperature Coefficient — odpor klesá s rastúcou teplotou.
Pri 1 kΩ type je menovitý odpor 1000 Ω pri 25 °C. Ak by bol v regulátore
nastavený typ 10 kΩ, namerané teploty by boli úplne nesprávne.
