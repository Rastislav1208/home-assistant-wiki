# ❄️ Ochrana studne pred zamrznutím — Dokumentácia

> **Autor:** Aurora (AI asistent) + Rastislav
> **Projekt:** Automatická ochrana studňového potrubia pred zamrznutím
> **Posledná aktualizácia:** 26. 7. 2026

---

## 📋 Základné informácie

| Vlastnosť | Hodnota |
|---|---|
| **Hĺbka studne** | 20 m |
| **Potrubie — materiál** | Meď, ⌀ 22 mm (vonkajší), vnútorný ~20 mm |
| **Potrubie — vodorovná časť** | ~30 m, uložené 50 cm pod povrchom |
| **Kritický úsek** | ~2 m nad povrchom, izolovaný + nerezová chránička |
| **Ovládanie čerpadla** | Sonoff zásuvka „Šteker 3" (priame napájanie) |
| **Tlaková nádoba / hydrofor** | ❌ Nie je — čerpadlo beží presne počas zapnutia štekra |
| **Odber počas cyklu** | Pootvorený kohútik / záhradná hadica |

---

## 🧮 Výpočet objemu potrubia

| Úsek | Výpočet | Objem |
|---|---|---|
| Kritický úsek (2 m) | π × (0,01 m)² × 2 m | **0,63 l** |
| Celé potrubie (30 m vodorovne + 20 m zvisle = 50 m) | π × (0,01 m)² × 50 m | **15,7 l** |

> 💡 **Záver:** pri bežnom prietoku studňového čerpadla (20–50 l/min) sa celý objem
> potrubia vymení za **20–45 sekúnd**. Zvolené **2 minúty** behu = bezpečná rezerva
> + čas na prehriatie medi teplom spodnej vody (typicky 8–10 °C aj v zime).

---

## 🎛️ Entity v HA

| Entita | Typ | Funkcia |
|---|---|---|
| `sensor.eu_i_2_plus_ot_vonkajsia_teplota_teplota` | sensor | Vonkajšia teplota (Tech Controllers EU-i-2 Plus OT) |
| `switch.sonoff_1002a54f5d_1` | switch | Šteker 3 — napájanie studňového čerpadla |
| `input_datetime.cerpadlo_posledne_spustenie` | input_datetime | Čas posledného ochranného cyklu |
| `automation.studna_ochrana_pred_zamrznutim` | automation | Hlavná 3-vrstvová automatizácia |
| `automation.studna_poistka_proti_zaseknutemu_cerpadlu` | automation | Bezpečnostná poistka |

---

## 🌡️ Logika ochrany — 3 vrstvy + poistka

| Vrstva | Prah teploty | Interval | Beh | Notifikácia |
|---|---|---|---|---|
| **3** (priorita) | ≤ −18 °C | každú 1 h | 2 min | ✅ Áno |
| **2** | ≤ −15 °C a > −18 °C | každé 2 h | 2 min | ❌ |
| **1** | ≤ −10 °C a > −15 °C | každé 4 h | 2 min | ❌ |
| **Poistka** | — | trigger `for: 5 min` | vypne | ✅ Áno |

### Prečo takto

- **`choose` s poradím 3 → 2 → 1** — spustí sa len **prvý** vyhovujúci blok, takže
  pri extrémnom mraze sa vždy uplatní najprísnejšia vrstva (rovnaký vzor ako pri
  postupnom spínaní žhavičov v solárnej automatizácii).
- **`input_datetime` namiesto pevných časových slotov** — odolné voči reštartu HA,
  výpadku siete aj posunom v čase kontroly. Automatizácia sa každých 15 min pýta
  „koľko minút ubehlo od posledného behu" a podľa teploty rozhodne.
- **Spoločný helper pre všetky 3 vrstvy** — zabraňuje duplicitnému spusteniu
  z rôznych vrstiev.
- **Poistka ako samostatná automatizácia** — čistá state-based logika
  (`for: 00:05:00`), funguje aj keby hlavná automatizácia úplne zlyhala.

---

## ⚙️ Automatizácia 1 — Hlavná ochrana (YAML)

```yaml
alias: "Studňa - ochrana pred zamrznutím"
description: "Periodické prepláchnutie kritického úseku potrubia pri mraze (3 stupne)"
mode: single
triggers:
  - trigger: time_pattern
    minutes: "/15"
conditions: []
actions:
  - variables:
      teplota: "{{ states('sensor.eu_i_2_plus_ot_vonkajsia_teplota_teplota') | float(99) }}"
      posledne: "{{ states('input_datetime.cerpadlo_posledne_spustenie') }}"
      minuty_od_posledneho: >
        {% if posledne not in ['unknown', 'unavailable', None, ''] %}
          {{ ((as_timestamp(now()) - as_timestamp(strptime(posledne, '%Y-%m-%d %H:%M:%S'))) / 60) | round(0) }}
        {% else %}
          9999
        {% endif %}
  - choose:
      # VRSTVA 3 — extrémny mráz, najvyššia priorita
      - conditions:
          - condition: template
            value_template: "{{ teplota <= -18 and minuty_od_posledneho >= 60 }}"
        sequence:
          - action: switch.turn_on
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - delay: "00:02:00"
          - action: switch.turn_off
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - action: input_datetime.set_datetime
            target: {entity_id: input_datetime.cerpadlo_posledne_spustenie}
            data:
              datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
          - action: notify.mobile_app_norris
            data:
              title: "❄️ Extrémny mráz"
              message: >
                Vonkajšia teplota {{ teplota }}°C. Studničné čerpadlo
                spustené na 2 min (ochranný cyklus, interval 1h).
      # VRSTVA 2 — silný mráz
      - conditions:
          - condition: template
            value_template: "{{ teplota <= -15 and teplota > -18 and minuty_od_posledneho >= 120 }}"
        sequence:
          - action: switch.turn_on
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - delay: "00:02:00"
          - action: switch.turn_off
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - action: input_datetime.set_datetime
            target: {entity_id: input_datetime.cerpadlo_posledne_spustenie}
            data:
              datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
      # VRSTVA 1 — mráz
      - conditions:
          - condition: template
            value_template: "{{ teplota <= -10 and teplota > -15 and minuty_od_posledneho >= 240 }}"
        sequence:
          - action: switch.turn_on
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - delay: "00:02:00"
          - action: switch.turn_off
            target: {entity_id: switch.sonoff_1002a54f5d_1}
          - action: input_datetime.set_datetime
            target: {entity_id: input_datetime.cerpadlo_posledne_spustenie}
            data:
              datetime: "{{ now().strftime('%Y-%m-%d %H:%M:%S') }}"
```

---

## 🛡️ Automatizácia 2 — Bezpečnostná poistka (YAML)

```yaml
alias: "Studňa - poistka proti zaseknutému čerpadlu"
description: "Núdzové vypnutie ak čerpadlo beží nezmyselne dlho"
mode: single
triggers:
  - trigger: state
    entity_id: switch.sonoff_1002a54f5d_1
    to: "on"
    for: "00:05:00"
conditions: []
actions:
  - action: switch.turn_off
    target: {entity_id: switch.sonoff_1002a54f5d_1}
  - action: notify.mobile_app_norris
    data:
      title: "⚠️ Poistka studňa"
      message: "Čerpadlo bežalo viac ako 5 minút — automaticky vypnuté poistkou. Skontroluj automatizáciu/skript."
```

---

## 🔧 Helper v `configuration.yaml`

```yaml
input_datetime:
  cerpadlo_posledne_spustenie:
    name: "Čerpadlo studňa - posledné spustenie (mráz)"
    has_date: true
    has_time: true
```

> ⚠️ Po pridaní nutný **plný reštart HA** — rýchle načítanie helper neregistruje.

---

## 🖥️ Dashboard (VELIN) — sekcia STUDŇA

> ⚠️ **Nepoužívať `type: grid` kartu ako obal!** Karty pridávať **priamo do sekcie**,
> veľkosť riadiť cez `grid_options` na každej karte zvlášť. (Viď know-how nižšie.)

### Karta 1 — Budík vonkajšej teploty (celá šírka)

```yaml
type: custom:modern-circular-gauge
entity: sensor.eu_i_2_plus_ot_vonkajsia_teplota_teplota
name: Vonkajšia teplota
min: -25
max: 25
smooth_segments: true
segments:
  - from: -25
    to: -18
    color: "#d32f2f"
  - from: -18
    to: -10
    color: "#f9a825"
  - from: -10
    to: 25
    color: "#43a047"
grid_options:
  columns: full
  rows: 4
```

> 💡 Farebné zóny gaugu **presne kopírujú prahy automatizácie**
> (červená = vrstva 3, žltá = vrstva 2/1, zelená = bez zásahu).

### Karty 2–5 — Ovládacie dlaždice (po dvoch vedľa seba)

```yaml
type: tile
entity: switch.sonoff_1002a54f5d_1
name: Čerpadlo (Šteker 3)
icon: mdi:pump
features:
  - type: toggle
grid_options:
  columns: 6
```

```yaml
type: tile
entity: input_datetime.cerpadlo_posledne_spustenie
name: Posledné spustenie
icon: mdi:clock-outline
grid_options:
  columns: 6
```

```yaml
type: tile
entity: automation.studna_ochrana_pred_zamrznutim
name: Ochrana - 3 vrstvy
icon: mdi:shield-check
features:
  - type: toggle
grid_options:
  columns: 6
```

```yaml
type: tile
entity: automation.studna_poistka_proti_zaseknutemu_cerpadlu
name: Poistka
icon: mdi:shield-alert
features:
  - type: toggle
grid_options:
  columns: 6
```

---

## 🎓 Know-how z tejto session

### 1. Chyba `TypeError: can't subtract offset-naive and offset-aware datetimes`

**Príčina:** `now()` v HA vracia dátum **s** časovou zónou (offset-aware), zatiaľ čo
`strptime()` vytvorí **naivný** dátum (bez časovej zóny). Python ich odčítať nevie.

```jinja2
{# ❌ NEFUNGUJE #}
{{ ((now() - strptime(posledne, '%Y-%m-%d %H:%M:%S')).total_seconds() / 60) | round(0) }}

{# ✅ SPRÁVNE — as_timestamp() prevedie oba na epoch sekundy #}
{{ ((as_timestamp(now()) - as_timestamp(strptime(posledne, '%Y-%m-%d %H:%M:%S'))) / 60) | round(0) }}
```

> 💡 Použiteľné pri **akejkoľvek** automatizácii, ktorá počíta čas od poslednej udalosti.

### 2. `input_datetime` sa po vytvorení nastaví na polnoc, nie `unknown`

Pri návrhu šablóny **nepredpokladať `unknown`** — helper má hneď po vytvorení hodnotu
`YYYY-MM-DD 00:00:00`. Vetva `{% else %}` sa teda nemusí nikdy vykonať.

### 3. Sekciový dashboard ≠ `type: grid` karta ⭐

| Prístup | Podpora veľkostí | Odporúčanie |
|---|---|---|
| Karty **priamo v sekcii** + `grid_options` na každej | ✅ Plná | ⭐ **Používať** |
| `type: grid` karta ako obal, karty vnútri | ❌ `column_span`/`row_span`/`grid_options` nefungujú | ❌ Nepoužívať |

**Symptómy zlého prístupu (`type: grid` obal):**
- `heading` karta sa nezobrazí ako nadpis, ale ako obyčajná dlaždica v mriežke
- `column_span`, `row_span`, `grid_options` na vnorených kartách sa ignorujú
- `square: true` navyše prehodí poradie na stĺpcové (masonry) — dlaždice sa pomiešajú
- HA sám varuje: *„Táto karta ešte plne nepodporuje zmenu veľkosti"*

**Riešenie:** názov riešiť ako **názov sekcie** (⋮ → Upraviť sekciu), karty pridávať
jednotlivo cez **+ Pridať kartu** priamo do sekcie.

### 4. `grid_options.columns` používa 12-stĺpcovú mriežku

| Hodnota | Výsledok |
|---|---|
| `columns: full` | Celá šírka sekcie |
| `columns: 6` | Polovica šírky (2 karty vedľa seba) |
| `columns: 4` | Tretina šírky (3 karty vedľa seba) |
| `rows: N` | Výška karty v jednotkách mriežky |

### 5. GARNI 1085 Arcus — NEintegrovateľná lokálne (slepá ulička)

| Zistenie | Detail |
|---|---|
| **Appka ProWeatherLive** | Len cloudový dashboard/viewer — **nemá** pole na vlastný server |
| **Voľby „Meteo server"** | PWS, WeatherCloud, Wunderground, AWEKAS — **všetko cloud**, len ID + Key, žiadna URL |
| **Konzola stanice** | Menu neobsahuje sieťové nastavenia / Custom Server |
| **Záver** | Bez poľa na hostname sa dáta lokálne presmerovať nedajú |

**Možné budúce cesty (neotestované):**
- Appka **WSLink / WS View / EasyWeather** (ak by bola pre tento model dostupná) —
  tá u príbuzných Ecowitt/Fine Offset staníc pole „Custom Server" má
- HACS komponenta [`SWS-12500-custom-component`](https://github.com/schizza/SWS-12500-custom-component)
  (Sencor, Garni, Bresser, Ecowitt) — podporuje protokoly WSLink aj PWS/WU
- Diagnostický nástroj [`test-station.schizza.cz`](https://test-station.schizza.cz) —
  na 30 min vytvorí testovaciu subdoménu a ukáže, aký protokol stanica reálne posiela
  (⚠️ vyžaduje pole na hostname v stanici — čo práve chýba)

> 💡 **Ponaučenie:** Pred integráciou nového zariadenia najprv overiť, či appka/konzola
> vôbec ponúka pole na **vlastný server (hostname/IP)**. Bez neho je lokálna
> integrácia nemožná bez ohľadu na to, aký hardvér je vnútri.

---

## 🧪 Postup testovania (bez čakania na zimu)

| Krok | Akcia | Očakávaný výsledok |
|---|---|---|
| 1 | Vývojárske nástroje → Šablóna: overiť výpočet `minuty_od_posledneho` | Kladné číslo bez chyby |
| 2 | Automatizácia → ⋮ → Spustiť | Trasovanie bez červenej chyby, žiadna vrstva sa nespustí |
| 3 | Fyzicky pootvoriť kohútik/hadicu | Voda pripravená tiecť |
| 4 | Dočasne zmeniť vrstvu 3 na `teplota <= 100` | — |
| 5 | Spustiť automatizáciu | Čerpadlo ON → 2 min → OFF, príde notifikácia |
| 6 | **Vrátiť podmienku späť na `teplota <= -18`** ⚠️ | Produkčný stav |

---

## ✅ Stav projektu

- [x] Analýza rizika (kritický úsek 2 m nad zemou)
- [x] Výpočet objemu potrubia a času prepláchnutia
- [x] Helper `input_datetime.cerpadlo_posledne_spustenie`
- [x] Automatizácia — 3 vrstvy (−10 / −15 / −18 °C)
- [x] Bezpečnostná poistka (5 min timeout)
- [x] Oprava chyby `offset-naive vs offset-aware`
- [x] Reálny test čerpadla (2 min beh, voda tiekla)
- [x] Vrátenie produkčných prahov
- [x] Dashboard sekcia STUDŇA (VELIN)
- [ ] *(voliteľné)* Denná súhrnná notifikácia o počte cyklov
- [ ] *(voliteľné)* Overenie funkčnosti pri prvom reálnom mraze (zima 2026/27)
- [ ] *(odložené)* Integrácia GARNI 1085 Arcus — viď know-how bod 5

---

## 🔮 Možné budúce vylepšenia

| Nápad | Prínos | Náročnosť |
|---|---|---|
| Denná súhrnná notifikácia o 8:00 | Prehľad, koľko cyklov prebehlo v noci | Nízka |
| Meranie spotreby čerpadla (Sonoff POWR) | Overenie, že čerpadlo naozaj beží (nie len relé) | Stredná |
| Senzor teploty priamo na kritickom úseku | Presnejšie riadenie než podľa vzduchu | Stredná (hardvér) |
| History graf teploty + cyklov na dashboarde | Vizuálna kontrola zásahov cez zimu | Nízka |

---

*Dokument: Aurora & Rastislav | Aktualizovať po každej významnej session*