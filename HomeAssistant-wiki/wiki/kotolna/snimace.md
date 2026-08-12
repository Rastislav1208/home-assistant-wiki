# Kotolňa — Snímače

⚠️ **Všetky snímače sú typ NTC 1 kΩ** (NIE 10 kΩ). Toto musí zodpovedať
nastaveniu typu snímača v menu regulátora.

Súvisí s: [[system]] · [[problemy]]

---

## Zapojené snímače

| Svorka regulátora | Typ | Umiestnenie | Stav |
|-------------------|-----|-------------|------|
| CH sensor | NTC 1 kΩ | Výstup kotla Viadrus U22 | Zapojený |
| DHW sensor | NTC 1 kΩ | Horný výstup akum. nádrže (TÚV) | Zapojený |
| Return sensor | NTC 1 kΩ | Spiatočka (za bajonetem ESBE 60 °C) | Zapojený |
| S1 sensor | NTC 1 kΩ | Stred nádrže (povrchový) | Zapojený |
| S2 sensor | NTC 1 kΩ | Dolná oblasť nádrže | Zapojený |
| Valve sensor 1 | NTC 1 kΩ | Výstup trojcestného ventilu | Zapojený |
| External sensor | NTC 1 kΩ (291p) | Vonkajšia (severná) stena | ⚠️ Len vo svorkovnici, NEnamontovaný |

---

## Poznámky k zapojeniu

- Snímače sú **polárne nezávislé** — nezáleží na poradí dvoch vodičov.
- External 291p je zatiaľ len zapojený do svorkovnice, treba ho fyzicky
  namontovať na severnú stenu domu — viď [[todo]].

---

## Charakteristika NTC 1 kΩ

NTC = Negative Temperature Coefficient — odpor klesá s rastúcou teplotou.
Pri 1 kΩ type je menovitý odpor 1000 Ω pri 25 °C. Ak by bol v regulátore
nastavený typ 10 kΩ, namerané teploty by boli úplne nesprávne.
