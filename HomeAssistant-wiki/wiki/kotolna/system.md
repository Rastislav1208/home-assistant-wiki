# Kotolňa — Systém

Architektúra kúrenia: kotol, regulátor, akumulačná nádrž, trojcestný ventil.

Súvisí s: [[snimace]] · [[problemy]] · [[todo]]

---

## Komponenty

| Komponent | Model | Detail |
|-----------|-------|--------|
| Kotol | Viadrus U22 | drevo, prevádzková teplota ~80 °C |
| Regulátor | TECH EU-i-2 Plus OT | FW 1.6.9G, ID 3331141379 |
| Internetový modul | TECH EU-WiFi-RS | IP 192.168.1.74, MAC 90:E5:B1:87:12:08 |
| Trojcestný ventil | + servo ESBE ARA600 | bajonet ESBE 60 °C ochrana spiatočky |
| Akumulačná nádrž | povrchové snímače | horný/stredný/dolný výstup |
| HA integrácia | Tech Controllers v2.8.0 | cez cloud emodul.eu |

---

## Schéma toku tepla

```
KOTOL VIADRUS U22
    │ [CH sensor]
    ├── bajonet ESBE 60 °C (ochrana spiatočky)
    │
[Čerpadlo 1 = Valve pump 2] → kotol → nádrž
    │
AKUMULAČNÁ NÁDRŽ
    ├── horný výstup [DHW sensor]
    ├── stred       [S1 sensor]
    └── dolný       [S2 sensor]
    │
[Čerpadlo 2 = Valve pump 1] → nádrž → radiátory
    │
[Trojcestný ventil + ESBE] → [Valve sensor 1]
    │
RADIÁTORY (liatinové, 60 °C)
```

---

## Nastavenia regulátora

| Parameter | Hodnota |
|-----------|---------|
| Prevádzkový režim | Paralelné čerpadlá |
| CH sensor — spínanie čerpadla 1 | 42 °C |
| CH sensor — vypínanie | 40 °C |
| Zadaná teplota ventilu (Okruh 1) | 60 °C |
| Typ ventila | ÚK |
| Ochrana kotla | Zapnutá |

---

## Sieťová konfigurácia EU-WiFi-RS

| Položka | Hodnota |
|---------|---------|
| IP adresa | 192.168.1.74 |
| Maska podsiete | 255.255.255.0 |
| Brána | 192.168.1.1 |
| DNS | 192.168.1.1 |
| MAC | 90:E5:B1:87:12:08 |

---

## HA Dashboard "Kotolňa"

Súbor: `projekty/kotolna/kotolna-dashboard.yaml`. Sekcie:
1. Stav systému (režim, antistop, letný režim, ochrana kotla/spiatočky)
2. Teploty — snímače (viď [[snimace]])
3. Ventil 1 — trojcestný (nastavenia, kalibrácia, hysterézia)
4. Okruhy 1 / 2 / 3
5. TÚV — teplá úžitková voda
6. Alarmy a výstupy
7. Kaskáda kotlov
