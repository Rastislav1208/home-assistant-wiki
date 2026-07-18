# Kotolňa — Problémy a riešenia

Záznam známych problémov nájdených pri inštalácii a ich riešení.
Slúži ako referencia, aby sa rovnaká chyba neopakovala.

Súvisí s: [[system]] · [[snimace]] · [[todo]]

---

## VYRIEŠENÉ

### Servo ESBE — prehodená polarita vodičov
- **Problém:** Hnedá a čierna boli pôvodne zapojené naopak → ventil sa otáčal nesprávnym smerom.
- **Riešenie:** Správne zapojenie na svorky Valve 1:
  - Hnedá → **Open**
  - Čierna → **Close**
  - Modrá → **N**
- **Overenie:** Po spustení sleduj smer otáčania; ak zaviera keď má otvárať, vymeň hnedú ↔ čiernu.

### Čerpadlá — prehodené priradenie
- **Problém:** Logické pomenovanie svoriek nezodpovedá fyzickému zapojeniu.
- **Stav (správny):**
  - **Valve pump 2** → čerpadlo kotol → nádrž (primárny okruh)
  - **Valve pump 1** → čerpadlo nádrž → radiátory (sekundárny okruh)
- **Poznámka:** Čerpadlá max. 80 W, zapojené priamo bez relé.

### Typ snímača NTC
- **Problém:** Riziko nastavenia 10 kΩ namiesto skutočných 1 kΩ.
- **Riešenie:** V menu regulátora nastavený typ **NTC 1 kΩ**. Viď [[snimace]].

---

## OTVORENÉ / KRITICKÉ

### ⚠️ Havarijný chladič chýba
- **Problém:** Viadrus U22 nemá nainštalovaný havarijný chladič.
- **Závažnosť:** KRITICKÁ — povinné podľa **STN EN 303-5**.
- **Riešenie:** Kúrenár musí doinštalovať termostatický havarijný ventil (~95 °C) + poistný ventil na nádrži. Viadrus U22 má na to pripravené vstupy.
- **Stav:** NEDORIEŠENÉ — viď [[todo]].
