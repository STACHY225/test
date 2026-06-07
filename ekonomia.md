# RageCity — docelowa ekonomia serwera (v2)

Dokument planistyczny do analizy zespołowej. Opisuje **docelowy stan** ekonomii po przeskalowaniu — bez wdrożenia w kodzie.

| Dokument | Przeznaczenie |
|---|---|
| **Ten plik** | Wizja, balans, widełki, decyzje do zatwierdzenia |
| [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) | Checklista zmian per skrypt (dla devów) |
| `docs/ekonomia-*-discord.txt` | Stan obecny (snapshoty z configów) |

---

## Założenia ogólne

Ekonomia RageCity ma być wolniejsza, czytelna i oparta o progres. Praca dorywcza jest punktem odniesienia. Frakcje, firmy i crime oferują wyższy potencjał zarobku kosztem odpowiedzialności, ryzyka, wymagań organizacyjnych lub zależności od innych graczy.

Endgame (pojazdy 600k–1.2 mln) wymaga **tygodni lub miesięcy** aktywnej gry — nie kilkunastu dni.

**Sesja referencyjna:** 6 h/dzień, 6 dni/tydzień ≈ **36 h/tydzień**.

### Skala zarobków

Wartości w tabeli mają dwa poziomy: **typowy** (przeciętna sesja) i **aktywna sesja** (górna granica przy wysokiej aktywności). Cele górne **nie są stałą stawką godzinową**.

| Obszar | Typowy $/h | Aktywna sesja $/h |
|---|---:|---:|
| Zasiłek / przeżycie | 60 | 60 |
| Praca dorywcza (zasiłek + aktywność) | 450–550 | 600–750 |
| Pracownik firmy prywatnej | 700–1000 | 1200–1500 |
| LSPD / LSSD / EMS | 500–900 | 1400–2000 |
| DOJ | 400–700 | 1100–1600 |
| Crime (po praniu, z cooldownami) | 700–1200 | 1800–2800 |
| Właściciel aktywnej firmy | 900–1400 | 1800–2800 |

### Hierarchia względem side joba

Side job (~500$/h) jest kotwicą ekonomii. Pozostałe ścieżki odnoszą się do niej przez dwa poziomy: **typowy** (przeciętna sesja) i **aktywna sesja** (górna granica).

| Ścieżka | Typowy $/h | × side job | Aktywna sesja $/h | × side job |
|---|---:|---:|---:|---:|
| Side job | 450–550 | 1.0× | 600–750 | 1.0× |
| Firma (pracownik) | 700–1000 | 1.4–2.0× | 1200–1500 | 2.0–2.5× |
| Frakcja (LSPD / LSSD / EMS) | 500–900 | 1.0–1.8× | 1400–2000 | 2.3–3.3× |
| DOJ | 400–700 | 0.8–1.4× | 1100–1600 | 2.2–3.2× |
| Crime | 700–1200 | 1.4–2.4× | 1800–2800 | 3.0–4.7× |

Kotwica side job (500$/h vs 550–600$/h) — **Status decyzji**.

### Tempo progresu (orientacyjne)

| Cel | @ 500$/h (side job) | @ 800$/h (typ. frakcja) | @ 1200$/h (typ. crime) |
|---|---:|---:|---:|
| Pierwsze auto (1.5k) | 3 h | 2 h | 1 h |
| Sensowne auto (10k) | 20 h | 13 h | 8 h |
| Dobre auto (75k) | 150 h | 94 h | 63 h |
| Endgame auto (1.2 mln) | 2400 h | 1500 h | 1000 h |

Przy 6 h/dzień i endgame auto: **~67 tygodni** (side job), **~42 tygodnie** (typ. frakcja), **~28 tygodni** (typ. crime).

---

## Paycheck i zasiłek

**Stan obecny:** `[core]/es_extended/server/paycheck.lua` wypłaca 125$/15 min cywilom i 250$/15 min frakcjom — bez warunku on duty.

| Grupa | Co 15 min | $/h | Warunek |
|---|---:|---:|---|
| Bezrobotny (zasiłek) | 15$ | 60$ | brak aktywnej pracy / off-duty |
| Praca dorywcza | 15$ | 60$ | on duty; reszta z aktywności jobu |
| Firma prywatna `c_*` | 10–15$ | 40–60$ | on duty |
| LSPD / LSSD / FIB | 105–112$ | 420–448$ | on duty |
| EMS | 72–78$ | 288–312$ | on duty |
| DOJ | 118–125$ | 472–500$ | on duty |

Wypłata co 15 min, on duty. Czas AFK odejmowany od czasu służby.

---

## Frakcje publiczne

### Model wypłat

| Źródło | Interwał | Udział w dochodzie |
|---|---|---|
| Minutówka (paycheck) | co 15 min, on duty | główne — DOJ; bazowe — LSPD/FIB; uzupełniające — EMS |
| Mandaty / faktury / usługi | po akcji | uzupełnienie — LSPD/FIB; główne — EMS |
| Premia tygodniowa | bossmenu, 1×/tydz. | dodatek |

| Frakcja | Minutówka $/h | Interakcje | Minutówka w typ. dochodzie (~10 online) |
|---|---:|---|---:|
| DOJ | 472–500 | rzadkie (licencje, sprawy) | ~85–95% |
| LSPD / LSSD / FIB | 420–448 | opcjonalne (mandaty) | ~75–90% |
| EMS | 288–312 | częste (revive) + kursy NPC | ~50–60% |

| Frakcja | Typowy razem $/h (~10 online) | Typowy razem $/h (~50 online) |
|---|---:|---:|
| DOJ | 520–580 | 620–780 |
| LSPD / LSSD / FIB | 460–520 | 640–880 |
| EMS | 500–580 | 900–1400 |

### Scenariusze (1 h)

Średni mandat 100$, udział funkcjonariusza 25% → 25$/mandat.

| Online | Frakcja | Minutówka | Interakcje | Razem $/h |
|---:|---|---:|---:|---:|
| ~10 | LSPD, 1–2 mandaty | 440 | 25–50 | 465–490 |
| ~10 | LSPD, 0 mandatów | 440 | 0 | 440 |
| ~10 | EMS, 2 kursy + 1 revive | 300 | 200–280 | 500–580 |
| ~10 | DOJ, 1 sprawa | 480 | 50–125 | 530–605 |
| ~10 | DOJ, 0 spraw | 480 | 0 | 480 |
| ~25 | LSPD, ~4 mandaty | 440 | ~100 | ~540 |
| ~50 | LSPD, ~8 mandatów | 440 | 200–400 | 640–840 |
| ~50 | EMS, aktywna sesja | 300 | 600–1100 | 900–1400 |
| ~50 | DOJ, ~4 sprawy | 480 | 200–500 | 680–980 |

| Scenariusz | Minutówka | Interakcje | Razem $/h |
|---|---:|---:|---:|
| LSPD, 4 mandaty | 440 | ~100 | ~540 |
| FIB, 2 mandaty | 440 | ~50 | ~490 |
| EMS, 6× revive (~80$ udziału) | 300 | ~480 | ~780 |
| EMS, 3 kursy NPC (~80$ udziału) | 300 | ~240 | ~540 |
| DOJ, 2 sprawy × ~100$ (25%) | 480 | ~200 | ~680 |

### LSPD / LSSD / FIB

| Element | Wartość |
|---|---|
| Minutówka | 420–448$/h |
| Mandaty / faktury | 25% kwoty |
| Konwoje, sprawy | okazjonalnie |

### EMS

| Element | Wartość |
|---|---|
| Minutówka | 288–312$/h |
| Revive / leczenie | 25% kwoty |
| Kurs EMS → lokalny medyk | 200–500$ / kurs, 2–4/h solo |

### DOJ

| Element | Wartość |
|---|---|
| Minutówka | 472–500$/h (~85–95% typ. dochodu) |
| Zarobek z akcji | licencje, ugody, wyroki — rzadkie |
| Konto DOJ (society) | 10% od podatków |
| Premia tygodniowa | jak pozostałe frakcje |

### Mandaty (docelowe kwoty)

| Kategoria | Kwota | Udział LSPD/LSSD (25%) |
|---|---:|---:|
| Drobne wykroczenie | 30–80$ | 8–20$ |
| Standardowe wykroczenie | 80–200$ | 20–50$ |
| Poważne wykroczenie | 200–500$ | 50–125$ |
| Przestępstwo | 500–1200$ | 125–300$ |
| Ciężkie przestępstwo | 1200–3000$ | 300–750$ |

Szczegółowa tabela per wykroczenie (MDT) — w pliku wdrożeniowym.

### Usługi EMS / DOJ

| Usługa | Cena klienta | Udział pracownika (~25%) |
|---|---:|---:|
| Revive | 200–500$ | 50–125$ |
| Leczenie / transport | 150–400$ | 37–100$ |
| Kurs EMS → lokalny medyk | 200–500$ | 50–125$ |
| Licencja / dokument DOJ | 150–500$ | 37–125$ |
| Ugoda / sprawa DOJ | 300–1000$ | 75–250$ |
| Wyrok / większa sprawa | 800–2500$ | 200–625$ |

### Premie tygodniowe (bossmenu)

Założenie: ~36 h/tydzień. Premia jako **dodatek** do paychecka i zarobku aktywnego.

| Godziny/tydz. | Premia | ≈ $/h (przy 36 h) |
|---:|---:|---:|
| 5–15 | 500–1500$ | 14–42$ |
| 20–30 | 1500–2500$ | 50–69$ |
| 30–40 | 2500–4000$ | 69–111$ |
| 40–50 | 4000–6000$ | 80–120$ |
| Lider / odpowiedzialność | 6000–10000$ | 120–200$ |
| **Limit** | **max 10 000$/os./tydz.** | — |

---

## Firmy prywatne

| Rola | Typowy $/h | Aktywna sesja $/h |
|---|---:|---:|
| Pracownik | 700–1000 | 1200–1500 |
| Manager | 900–1300 | 1500–2000 |
| Właściciel | 900–1400 | 1800–2800 |

Właściciel dobrze prowadzonej firmy może przewyższyć frakcjonariusza — przy kosztach operacyjnych, personele i aktywnej sprzedaży. DOJ otrzymuje 10% od obrotu firmy.

---

## Skala cen pojazdów

| Poziom | Cena | @ 500$/h |
|---|---:|---:|
| Najtańszy pojazd | 1000–1500$ | 2–3 h |
| Pierwszy sensowny | 5000–10000$ | 10–20 h |
| Dobry | 25000–75000$ | 50–150 h |
| Bardzo dobry | 150000–400000$ | 300–800 h |
| Endgame auto | 600000–1200000$ | 1200–2400 h |

| Typ | Zakres cen |
|---|---|
| Auta osobowe | max **1.2 mln** |
| Motocykle | max **800k** |
| Skutery wodne | ~**1 mln** |
| Łodzie | **1–2.5 mln** |
| Helikoptery | **2–4 mln** (Zakup firmowy/grupowy) |
| Samoloty | **6–8 mln** |

```lua
types = {
    automobile = { minPrice = 1000,    maxPrice = 1200000, roundTo = 500   },
    bike       = { minPrice = 1500,    maxPrice = 800000,  roundTo = 500   },
    bicycle    = { minPrice = 100,     maxPrice = 1000,    roundTo = 10    },
    boat       = { minPrice = 1000000, maxPrice = 2500000, roundTo = 10000 },
    submarine  = { minPrice = 500000,  maxPrice = 1500000, roundTo = 10000 },
    heli       = { minPrice = 2000000, maxPrice = 4000000, roundTo = 10000 },
    plane      = { minPrice = 6000000, maxPrice = 8000000, roundTo = 10000 },
    trailer    = { minPrice = 2500,    maxPrice = 100000,  roundTo = 500   },
    train      = { minPrice = 0,       maxPrice = 0,       roundTo = 1     },
}
```

---

## Prace dorywcze

| Praca | Aktywność | + zasiłek 60$ | Razem |
|---|---:|---:|---:|
| Beach vendor | 290–440$ | 60$ | 350–500$ |
| Tailor | 390–490$ | 60$ | 450–550$ |
| Slaughterer | 390–540$ | 60$ | 450–600$ |
| Lumberjack | 440–590$ | 60$ | 500–650$ |
| Miner | 440–640$ | 60$ | 500–700$ |
| Deliveries (KQ) | 390–590$ | 60$ | 450–650$ |
| Powerwashing (KQ) | 440–690$ | 60$ | 500–750$ |
| Trucker | 540–790$ | 60$ | 600–850$ |

**Uwaga wdrożeniowa:** KQ Deliveries i Powerwashing — obecnie ~20k–30k+/h; wymagają ~40× obniżki, inaczej kotwica 500$/h nie będzie respektowana.

---

## Crime

Typowy: **700–1200$/h** | Aktywna sesja: **1800–2800$/h** (po praniu, z cooldownami).

Crime opiera się na **miksie aktywności** (NPC, ATM, kasetki, boosting, narkotyki), nie na powtarzaniu jednego napadu.

| Element | Założenie |
|---|---|
| Koszt wejścia | narzędzia dostępne po kilku h side joba |
| Ryzyko | policja, więzienie, utrata itemów |
| Cooldowny | ograniczenie farmienia |
| Biżuteria | sprzedaż w lombardzie / dark_lombardzie |
| Brudna gotówka | zysk po praniu (strata 20–45%) |

---

## Napady — progresja

### Kolejność (od najłatwiejszego)

NPC → Bankomat (karta) → Boosting → Bankomat (hack) → Kasetka → Tracker → Sejf sklepu → Jubiler → Fleeca → Lombard → Jacht → Pacific → Humane Labs → Cayo Perico

### Tabela zbiorcza

| # | Napad | Min os. | Max os. | Łup min | Łup max | Typ | Status |
|---:|---|---:|---:|---:|---:|---|---|
| 1 | NPC | 1 | 1 | 25$ | 150$ | gotówka + biżuteria | Aktywny |
| 2 | Bankomat — karta | 1 | 1 | 100$ | 350$ | gotówka | Aktywny |
| 3 | Boosting | 1 | 1 | 350$ | 900$ | gotówka | Planowany |
| 4 | Bankomat — hack | 1 | 1 | 200$ | 700$ | black_money | Aktywny |
| 5 | Kasetka sklepowa | 1 | 2 | 250$ | 600$ | 50–70% dirty + gotówka | Aktywny |
| 6 | Tracker | 1 | 2 | 500$ | 1400$ | black_money | Planowany |
| 7 | Sejf sklepu | 1 | 3 | 800$ | 2200$ | black_money | Aktywny |
| 8 | Jubiler | 2 | 5 | 1200$ | 4500$ | biżuteria → lombard | Planowany |
| 9 | Bank Fleeca | 2 | 4 | 6000$ | 16000$ | black_money | Aktywny |
| 10 | Napad na lombard | 1 | 3 | 600$ | 1800$ | gotówka + biżuteria | Planowany |
| 11 | Jacht | 3 | 5 | 10000$ | 25000$ | black_money / loot | Planowany |
| 12 | Bank Pacific | 4 | 6 | 35000$ | 90000$ | black_money / loot | Aktywny |
| 13 | Humane Labs | 3 | 5 | 22000$ | 55000$ | black_money / loot | Planowany |
| 14 | Cayo Perico | 5 | 8 | 70000$ | 130000$ | black_money / loot | Planowany |

*Łup w biżuterii = łączna wartość skupu w lombardzie (nie gotówka bezpośrednio).*

### Mnożniki nagrody (łączny loot)

Mnożnik stosuje się do **łącznej** puli. Więcej osób ≠ proporcjonalnie więcej na osobę.

```lua
Config.AttackerMultipliers = {
    shop_cashregister = { [1] = 1.0,  [2] = 1.15 },
    boosting          = { [1] = 1.0 },
    tracker           = { [1] = 1.0,  [2] = 1.10 },
    shop_vault        = { [1] = 1.0,  [2] = 1.2,  [3] = 1.35 },
    jeweler           = { [2] = 1.0,  [3] = 1.15, [4] = 1.25, [5] = 1.35 },
    bank              = { [2] = 1.0,  [3] = 1.2,  [4] = 1.35 },
    pawnshop          = { [1] = 1.0,  [2] = 1.15, [3] = 1.25 },
    yacht             = { [3] = 1.0,  [4] = 1.15, [5] = 1.30 },
    bank_pacific      = { [4] = 1.0,  [5] = 1.2,  [6] = 1.35 },
    humane_labs       = { [3] = 1.0,  [4] = 1.15, [5] = 1.30 },
    cayo_perico       = { [5] = 1.0,  [6] = 1.15, [7] = 1.25, [8] = 1.35 },
}
```

---

### 1. Napad na NPC

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 1 (solo) |
| Mnożnik | brak |
| Łup gotówka | 25–150$ |
| Łup dodatkowy | Biżuteria niskiej jakości (12–55$/szt. skupu), bankcard, telefon (dark_lombard) |
| Typ nagrody | Gotówka + itemy |
| Policja min. | 1 |
| Cooldown | ~15 s między próbami |
| Czas akcji | ~30–60 s |
| Wymagania | Broń zwiększa szansę (80% vs 20% bez broni) |
| Sprzedaż lootu | Lombard / dark_lombard (Lester ×0.95) |

---

### 2. Bankomat — karta

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 1 (solo) |
| Mnożnik | brak |
| Łup | 100–350$ gotówki |
| Typ nagrody | Czysta gotówka |
| Policja min. | 0 |
| Cooldown | Do ustalenia (~5 min per bankomat) |
| Wymagania | Karta bankowa (loot z NPC, metadata illegal) |
| Uwagi | Entry-level crime; bez narzędzi hackingowych |

---

### 3. Boosting

| Parametr | Wartość |
|---|---|
| Status | Planowany (implementacja w `rage_heists` lub osobny resource) |
| Gracze | 1 (solo) |
| Mnożnik | ×1.0 |
| Łup | 350–900$ gotówki |
| Typ nagrody | Czysta gotówka |
| Policja min. | 1 |
| Cooldown | ~15 min |
| Czas akcji | ~10–20 min (szukanie auta + dostawa) |
| Wymagania | Brak thermite/hack; opcjonalnie lockpick |
| Mechanika | Otrzymaj wskazane auto w mieście → dostarcz do punktu |

---

### 4. Bankomat — hack

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 1 (solo) |
| Mnożnik | brak |
| Łup | 200–700$ black_money (łącznie ze stosów) |
| Typ nagrody | black_money |
| Policja min. | 1 (promień ~2500 m) |
| Cooldown | Do ustalenia (~8–10 min) |
| Wymagania | Hackingdevice (400–800$) |
| Uwagi | Wyższe ryzyko niż karta; wymaga prania |

---

### 5. Kasetka sklepowa

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 1–2 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.15 |
| Łup łączny | 250–600$ |
| Podział | 50–70% black_money, reszta gotówka |
| Policja min. | 2 (promień ~3000 m) |
| Cooldown | ~8 min (480 s) |
| Czas akcji | do ~330 s |
| Wymagania | Lockpick (80–200$) |
| Lokalizacje | Sklepy Market + Liquor (`rage_market`) |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 250–600$ | 250–600$ |
| 2 | ×1.15 | 287–690$ | 143–345$ |

---

### 6. Tracker

| Parametr | Wartość |
|---|---|
| Status | Planowany — istnieje `[rage]/BabiczTracker` (wymaga przeskalowania i integracji z progresją) |
| Gracze | 1–2 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.10 |
| Łup | 500–1400$ black_money |
| Typ nagrody | black_money |
| Policja min. | 1 |
| Cooldown | ~20 min |
| Czas akcji | ~15–25 min |
| Wymagania | Opłata startowa ~200–400$; brak thermite |
| Mechanika | Weź auto z nadajnikiem → ucieczka przed LSPD przez określony czas → punkt zrzutu |
| Uwagi | Obecny config: nagrody 500–5000$, opłata startowa 500$ — do przeskalowania |

**Loot na osobę (docelowo):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 500–1400$ | 500–1400$ |
| 2 | ×1.10 | 550–1540$ | 275–770$ |

---

### 7. Sejf sklepu

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 1–3 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.2 · 3 os. ×1.35 |
| Łup łączny | 800–2200$ black_money |
| Policja min. | 2 (promień ~7500 m) |
| Cooldown | ~10 min (600 s) globalny |
| Czas akcji | 180–240 s |
| Wymagania | Lockpick / drill (do ustalenia w implementacji) |
| Uwagi | Wyraźnie wyższy próg niż kasetka; bonus lokacji Sandy do usunięcia lub przeskalowania |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 800–2200$ | 800–2200$ |
| 2 | ×1.2 | 960–2640$ | 480–1320$ |
| 3 | ×1.35 | 1080–2970$ | 360–990$ |

---

### 8. Jubiler

| Parametr | Wartość |
|---|---|
| Status | Planowany (`rage_heists` — mnożnik w Config, brak aktywnego heistu) |
| Gracze | 2–5 |
| Mnożnik | 2 os. ×1.0 · 3 os. ×1.15 · 4 os. ×1.25 · 5 os. ×1.35 |
| Łup | Biżuteria o łącznej wartości skupu **1200–4500$** |
| Typ nagrody | Itemy → sprzedaż w lombardzie / dark_lombardzie |
| Policja min. | 3 |
| Cooldown | ~30 min |
| Czas akcji | ~5–10 min |
| Wymagania | Broń, opcjonalnie hackingdevice |
| Loot | Pierścionki, łańcuszki, kolczyki, diamenty (patrz tabela biżuterii) |

**Loot na osobę (przykład, baza 3000$):**

| Os. | Mnożnik | Wartość łączna | Na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 1200–4500$ | 600–2250$ |
| 3 | ×1.15 | 1380–5175$ | 460–1725$ |
| 5 | ×1.35 | 1620–6075$ | 324–1215$ |

---

### 9. Bank Fleeca

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 2–4 (min. 2 w promieniu 15 m) |
| Mnożnik | 2 os. ×1.0 · 3 os. ×1.2 · 4 os. ×1.35 |
| Łup łączny | 6000–16000$ black_money |
| Policja min. | 4 (promień ~9500 m) |
| Cooldown | ~15 min per lokalizacja |
| Czas akcji | ~360 s |
| Wymagania | Hackingdevice, wiertło |
| Lokalizacje | 5 banków Fleeca |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 6000–16000$ | 3000–8000$ |
| 3 | ×1.2 | 7200–19200$ | 2400–6400$ |
| 4 | ×1.35 | 8100–21600$ | 2025–5400$ |

*Po praniu (25% straty): ok. 2250–6000$ czystego na osobę przy 2 napastnikach.*

---

### 10. Napad na lombard

| Parametr | Wartość |
|---|---|
| Status | Planowany |
| Gracze | 1–3 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.15 · 3 os. ×1.25 |
| Łup gotówka | 600–1800$ |
| Łup biżuteria | Itemy o wartości skupu **800–2500$** |
| Typ nagrody | Gotówka + biżuteria → część itemów tylko w dark_lombardzie |
| Policja min. | 2 |
| Cooldown | ~20 min |
| Czas akcji | ~3–6 min |
| Wymagania | Broń, lockpick |
| Uwagi | Oddzielny od „skupu” w lombardzie — to napad na lokal |

**Loot na osobę (gotówka, baza 1200$):**

| Os. | Mnożnik | Gotówka łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 600–1800$ | 600–1800$ |
| 2 | ×1.15 | 690–2070$ | 345–1035$ |
| 3 | ×1.25 | 750–2250$ | 250–750$ |

---

### 11. Napad na jacht

| Parametr | Wartość |
|---|---|
| Status | Planowany (`rage_heists` — mnożnik w Config) |
| Gracze | 3–5 |
| Mnożnik | 3 os. ×1.0 · 4 os. ×1.15 · 5 os. ×1.30 |
| Łup łączny | 10000–25000$ black_money (+ opcjonalny loot) |
| Policja min. | 4–5 |
| Cooldown | ~45–60 min |
| Czas akcji | ~15–25 min |
| Wymagania | Broń, łódź/transport, thermite opcjonalnie |
| Uwagi | Poziom między Fleeca a Pacific; wymaga logistyki morskiej |

**Loot na osobę (baza 17500$):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 3 | ×1.0 | 10000–25000$ | 3333–8333$ |
| 4 | ×1.15 | 11500–28750$ | 2875–7187$ |
| 5 | ×1.30 | 13000–32500$ | 2600–6500$ |

---

### 12. Bank Pacific

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists` — custom implementacja) |
| Gracze | 4–6 |
| Mnożnik | 4 os. ×1.0 · 5 os. ×1.2 · 6 os. ×1.35 |
| Łup łączny | 35000–90000$ (black_money + itemy do sprzedaży) |
| Policja min. | 5 |
| Cooldown | ~60–90 min (event serwera) |
| Czas akcji | ~20–40 min |
| Wymagania | Hackingdevice, hackingtablet, thermite, broń |
| Uwagi | Obecnie minGroup=1 w config — docelowo min. 4 os.; loot w server flow |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 4 | ×1.0 | 35000–90000$ | 8750–22500$ |
| 5 | ×1.2 | 42000–108000$ | 8400–21600$ |
| 6 | ×1.35 | 47250–121500$ | 7875–20250$ |

---

### 13. Humane Labs

| Parametr | Wartość |
|---|---|
| Status | Planowany (`rage_heists` — mnożnik w Config) |
| Gracze | 3–5 |
| Mnożnik | 3 os. ×1.0 · 4 os. ×1.15 · 5 os. ×1.30 |
| Łup łączny | 22000–55000$ black_money / loot |
| Policja min. | 5 |
| Cooldown | ~60 min |
| Czas akcji | ~20–30 min |
| Wymagania | Hackingdevice, hackingtablet, thermite, broń ciężka |
| Uwagi | Poziom między Pacific a Cayo |

**Loot na osobę (baza 38500$):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 3 | ×1.0 | 22000–55000$ | 7333–18333$ |
| 4 | ×1.15 | 25300–63250$ | 6325–15812$ |
| 5 | ×1.30 | 28600–71500$ | 5720–14300$ |

---

### 14. Cayo Perico (endgame)

| Parametr | Wartość |
|---|---|
| Status | Planowany |
| Gracze | 5–8 |
| Mnożnik | 5 os. ×1.0 · 6 os. ×1.15 · 7 os. ×1.25 · 8 os. ×1.35 |
| Łup łączny | 70000–130000$ black_money / loot |
| Policja min. | 6+ |
| Cooldown | ~2–4 h (event serwera) |
| Czas akcji | ~30–60 min |
| Wymagania | Pełen zestaw: hackingdevice, hackingtablet, thermite ×N, broń ciężka, transport |
| Uwagi | Najwyższy tier crime; wymaga organizacji i przygotowania |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę (brutto) |
|---:|---:|---:|---:|
| 5 | ×1.0 | 70000–130000$ | 14000–26000$ |
| 6 | ×1.15 | 80500–149500$ | 13416–24916$ |
| 7 | ×1.25 | 87500–162500$ | 12500–23214$ |
| 8 | ×1.35 | 94500–175500$ | 11812–21937$ |

*Po praniu (25% straty): ok. 8900–19500$ czystego na osobę przy 6 napastnikach.*

---

## Biżuteria i lombard

Dark lombard (Lester): mnożnik **×0.95**. Telefony i elektronika z NPC → `dark_lombard`.

| Item | Obecnie | Docelowo |
|---|---:|---:|
| silver_ring | 180–280$ | 15–25$ |
| earrings | 160–260$ | 12–20$ |
| bracelet | 220–340$ | 18–30$ |
| wallet | 240–360$ | 15–25$ |
| powerbank | 210–330$ | 12–22$ |
| headphones | 260–420$ | 18–30$ |
| watch | 450–800$ | 30–55$ |
| earrings_silver | 260–420$ | 20–35$ |
| bracelet_silver | 320–520$ | 22–38$ |
| gold_ring | 650–1050$ | 45–75$ |
| earrings_gold | 420–680$ | 40–70$ |
| necklace_silver | 500–820$ | 35–60$ |
| gold_chain | 800–1250$ | 55–90$ |
| watch_silver | 650–1050$ | 40–65$ |
| watch_gold | 950–1500$ | 55–85$ |
| smartwatch | 550–900$ | 35–60$ |
| vintage_camera | 700–1150$ | 45–75$ |
| designer_bag | 900–1400$ | 70–120$ |
| phone | 750–1250$ | 40–70$ (dark) |
| sapphire_ring | 850–1350$ | 55–90$ |
| ruby_ring | 880–1400$ | 55–90$ |
| emerald_ring | 900–1450$ | 55–90$ |
| thornburn_turquoise_ring | 980–1550$ | 60–95$ |
| earrings_ruby | 760–1220$ | 50–80$ |
| earrings_emerald | 820–1300$ | 55–85$ |
| earrings_turquoise | 700–1120$ | 45–72$ |
| earrings_diamond | 980–1600$ | 80–130$ |
| bracelet_gold | 520–860$ | 40–70$ |
| necklace_gold | 760–1220$ | 55–90$ |
| necklace_emerald | 1000–1620$ | 90–150$ |
| necklace_sapphire | 960–1540$ | 85–140$ |
| necklace_ruby | 980–1580$ | 88–145$ |
| diamond | 155–160$ | 100–160$ |

---

## Narzędzia do napadów

| Item | Docelowo | Obecnie | Użycie |
|---|---:|---:|---|
| Lockpick | 80–200$ | 1200$ | kasetka, NPC |
| Hackingdevice | 400–800$ | 4000–5000$ | ATM hack, Fleeca |
| Hackingtablet | 800–1500$ | 8000–10000$ | Pacific, Humane |
| Thermite | 400–600$ | 2000$ | Pacific, Humane, Cayo |

Progresja: ~3 h side joba (1500$) → lockpick + hackingdevice → pierwsze napady.

---

## Pralnia pieniędzy

| Typ | Strata | Uwagi |
|---|---:|---|
| Automatyczna | 35–45% | 25–75$/s (obecnie 175$/s) |
| Ręczna trasa | 20–30% | 3000–7500$/punkt |

---

## Narkotyki

Docelowy zarobek: **800–1500$/h** typowo | **1500–2200$/h** aktywna sesja.

| Item | Ilość | Szansa | Obecnie | Docelowo |
|---|---:|---:|---:|---:|
| joint | 1–2 | 70% | 75–150$ | 25–45$ |
| magic_mushrooms_dry | 1–8 | 65% | 87–147$ | 20–40$ |
| weed_packed | 1–3 | 65% | 243–303$ | 55–85$ |
| kq_weed_joint_og_kush | 1–4 | 67% | 175–235$ | 40–65$ |
| kq_weed_joint_purple_haze | 1–4 | 67% | 240–300$ | 55–80$ |
| kq_weed_joint_white_widow | 1–4 | 67% | 250–330$ | 60–85$ |
| kq_weed_joint_blue_dream | 1–4 | 67% | 280–360$ | 65–95$ |
| heroin_opium | 1–4 | 60% | 339–395$ | 70–100$ |
| coke_packed | 1–4 | 55% | 431–541$ | 90–130$ |
| meth_packed | 1–5 | 50% | 455–570$ | 95–140$ |
| heroin_packed | 1–3 | 55% | 545–605$ | 100–145$ |
| kq_weed_bag_og_kush | 1–3 | 65% | 370–480$ | 80–120$ |
| kq_weed_bag_purple_haze | 1–3 | 65% | 480–580$ | 100–150$ |
| kq_weed_bag_white_widow | 1–4 | 65% | 500–640$ | 110–160$ |
| kq_weed_bag_blue_dream | 1–3 | 65% | 560–700$ | 120–175$ |
| coke | 1–2 | 60% | 750–950$ | 130–190$ |
| coke_bag | 1–3 | 65% | 850–1050$ | 150–220$ |

---

## Koszty życia i usługi

| Pozycja | Docelowo | Obecnie |
|---|---:|---:|
| Woda | 5–10$ | 20$ |
| Proste jedzenie | 8–20$ | 36$ |
| Lepsze jedzenie / restauracja | 25–80$ | — |
| Telefon | 500–1000$ | 1250–1500$ |
| Radio | 300–700$ | 850$ |
| Lockpick | 80–200$ | 1200$ |
| Fixkit | 250–600$ | 5000$ |
| Odholowanie | 300–700$ | 800$ |
| Impound specjalny | 1500–3000$ | 5000$ |
| Lokalny medyk | 500–1500$ | 50–10000$ |
| Lokalny mechanik | 500–1500$ | 1000–5000$ |
| Naprawa z garażu | 500–1500$ | 1500$ |
| Odsprzedaż pojazdu | 65% ceny zakupu | 65% |
| Startowy bank | 2500$ | 2500$ |

---

## Bronie

| Item | Docelowo |
|---|---:|
| Broń biała | 500–2500$ |
| Słaby pistolet | 8000–15000$ |
| Dobry pistolet | 15000–30000$ |
| SMG | 50000–120000$ |
| Karabin | 120000–300000$ |

---

## Mieszkania (`qs-housing`)

Ceny domów są **per lokalizacja** (`Config.Houses` + baza danych). Poniżej widełki docelowe segmentów — do zastosowania przy audycie DB i tworzeniu nowych nieruchomości.

### Segmenty cenowe

| Segment | Cena | Czas @ 500$/h | Uwagi |
|---|---:|---:|---|
| Garaż / komórka / najtańsze | 15000–25000$ | 30–50 h | entry housing |
| Mieszkanie starter | 25000–50000$ | 50–100 h | pierwsza nieruchomość |
| Mieszkanie standard | 50000–100000$ | 100–200 h | porównywalne z dobrym autem |
| Dom średni | 100000–200000$ | 200–400 h | |
| Dom duży | 200000–400000$ | 400–800 h | |
| Premium / willa | 400000–650000$ | 800–1300 h | poniżej endgame auta (1.2 mln) |
| Max (rzadkie lokacje) | 650000–800000$ | 1300–1600 h | hard cap nieruchomości |

**Zasada:** endgame dom **nie powinien przewyższać** endgame auta osobowego (1.2 mln).

### Opłaty przy zakupie

| Opłata | % | Przykład @ 100k$ |
|---|---:|---:|
| Bank | 10% | 10000$ |
| Pośrednik (broker) | 5% | 5000$ |
| Podatki | 5% | 5000$ |
| **Razem** | **~20%** | **~20000$** |

### Hipoteka i czynsz

| Parametr | Obecnie | Docelowo |
|---|---|---|
| Hipoteka włączona | tak | tak |
| `CreditEq` (spłata %) | 30% co 5 min | **5–8% co 30–60 min** |
| Czynsz (wynajem) | co 5 min / miesięcznie | **0,3–0,8% wartości / tydzień gry** |
| Max domów na gracza | 5 | 5 |

Obecny model (30% co 5 min) jest zbyt agresywny po deflacji — wymaga zmiany w config.

### Rachunki (bills)

| Typ | Obecnie | Docelowo |
|---|---|---|
| Woda | 30–150$ | 15–40$ |
| Internet | 80–300$ | 25–80$ |
| Prąd | 50$/kWh | 12–20$/kWh |
| Interwał naliczania | co 1 h | co 1 h |

### Upgrade'y posesji

| Upgrade | Obecnie | Docelowo |
|---|---:|---:|
| Alarm | 10000$ | 2500–4000$ |
| Kamery | 35000$ | 8000–12000$ |
| Czujnik ruchu | 45000$ | 10000–15000$ |
| Sejf | 50000$ | 12000–18000$ |
| Rozbudowa mebli | 60000$ | 15000–22000$ |
| Dzwonek | 15000$ | 3500–5500$ |
| Asystent Aura | 30000$ | 7000–11000$ |
| Dekoracje ścienne | 25000$ | 6000–9000$ |

### Meble i klucze

| Element | Obecnie | Docelowo |
|---|---:|---:|
| Meble (IKEA) | 61–2500$+ | 30–800$ (większość 50–400$) |
| Klucz do domu (meta key) | 500$ | 150–300$ |
| Prowizja od sprzedaży mebli | 30% | 30% (bez zmian) |

---

## Paliwo i utrzymanie pojazdu

| Pozycja | Obecnie | Docelowo |
|---|---:|---:|
| Tankowanie (`priceTick`) | 8$/tick | 2–4$/tick |
| Kanister | 1500$ | 400–700$ |
| Uzupełnienie kanistra | 1000$ | 250–500$ |
| Kanister eco | 800$ | 200–400$ |
| Myjnia (brud × stawka) | brud × 12$ | brud × 4–8$ |
| Tablica rejestracyjna (custom) | 750000$ | 15000–40000$ |
| Jazda testowa (salon) | 80$ | 50–100$ |
| Współwłaściciel garażu | 5000$ | 1500–3000$ |

---

## Wygląd postaci

| Usługa | Obecnie | Docelowo |
|---|---:|---:|
| Sklep odzieżowy | 200$ | 80–150$ |
| Fryzjer | 180$ | 60–120$ |
| Tatuaż | 750$ | 250–500$ |
| Maski | 50$ | 25–50$ |

Źródło: `[rage]/rage_multicharacter/Config.lua`

---

## Siłownia

| Karnet | Obecnie | Docelowo |
|---|---:|---:|
| 7 dni | 500$ | 150–250$ |
| 14 dni | 900$ | 250–400$ |
| 30 dni | 1500$ | 400–700$ |
| 90 dni | 4000$ | 1000–1800$ |

Źródło: `[rage]/RageCity/Config.lua` → `Config.Gym`

---

## Ubezpieczenie zdrowotne (EMS)

| Karnet | Obecnie | Docelowo |
|---|---:|---:|
| 7 dni | 3500$ | 800–1200$ |
| 14 dni | 6000$ | 1400–2000$ |
| 30 dni | 12000$ | 2500–3500$ |
| Zniżka na usługi medyczne | −50% | −50% (bez zmian) |

| Opłata | Obecnie | Docelowo |
|---|---:|---:|
| Respawn (brak EMS) | 500$ | 150–300$ |
| Respawn (EMS online) | 5000$ | 800–1500$ |

---

## FIB

FIB korzysta z tego samego modelu co LSPD/LSSD:

| Element | Wartość |
|---|---|
| Paycheck | 250$/15 min → LSPD/FIB **105–112$/15 min**; EMS **72–78$/15 min**; DOJ **118–125$/15 min** |
| Udział z mandatów | **25%** kwoty (jak LSPD) |
| Mnożnik frakcji | 2.5× |
| Katalog mandatów | ten sam co LSPD (`policeFines`) |
| Premie tygodniowe | jak pozostałe frakcje |

Typowy / aktywna sesja: jak LSPD/LSSD (500–900 / 1400–2000$/h).

---

## Frakcja mechanik

### Lokalny mechanik (NPC)

| Warunek | Obecnie | Docelowo |
|---|---:|---:|
| Brak mechaników online | 1000$ | 500–800$ |
| Mechanicy online | 5000$ | 1000–1500$ |

### Tuning (frakcja)

| Element | Wartość |
|---|---|
| Split | firma 35% · DOJ 10% · zwrot klientowi 10% |
| Ceny modów | % ceny pojazdu — przeskalować po `vehicles.json` |

Źródło: `BabiczTuningMenu_sv.lua`, `BabiczMechanic_shared.lua`

---

## Prace dorywcze — uzupełnienie

### Kaucje pojazdów / strojów

| Praca | Kaucja pojazd | Kaucja strój |
|---|---:|---:|
| Górnik | 1500$ → **400–600$** | 500$ → **100–200$** |
| Drwal | 2500$ → **600–900$** | 500$ → **100–200$** |
| Krawiec | 1500$ → **400–600$** | 500$ → **100–200$** |
| Rzeźnik | 500$ → **150–250$** | 500$ → **100–200$** |
| Myśliwy | 2500$ → **600–900$** | — |
| Śmieciarz | 1500$ → **400–600$** | 500$ → **100–200$** |

### Myśliwy (hunter)

| Item | Obecnie | Docelowo |
|---|---:|---:|
| meat_boar | 128$ | 45–55$ |
| meat_deer | 183$ | 65–75$ |
| skin_boar | 85$ | 30–38$ |
| skin_deer | 116$ | 40–48$ |
| boar_tusks | 228$ | 80–95$ |
| deer_horns | 300$ | 105–120$ |

Docelowy zarobek: **450–600$/h** (+ zasiłek 60$/h).

### Śmieciarz

Obecnie brak pętli wypłaty w configu — **do implementacji** (docelowo ~450–550$/h) lub wyłączenia z job center.

---

## Sprzedaż narkotyków — parametry

| Parametr | Obecnie | Docelowo |
|---|---|---|
| Min. policjantów online | 2 | 2 |
| Szansa wezwania policji | 65% | 65% |
| Strefa sprzedaży | 1 (duży promień) | bez zmian |
| Blacklisted jobs | police, doj, fib | bez zmian |

Ceny per item — tabela w sekcji **Narkotyki**.

---

## Zadania Fernando (Moris)

Tutorial / progresja crime. Nie są głównym źródłem dochodu po wdrożeniu v2.

| Zadanie | Obecnie | Docelowo |
|---|---|---|
| 1 — paczki | 250–750$ dirty / paczka | 80–200$ dirty |
| 2 — pralnia (wypłata) | 8000–12000$ czyste + 25000$ dirty do prania | 1500–2500$ czyste + 4000–6000$ dirty |
| 3 — sprzedaż weed_packed | 5000–8000$ dirty + sprzedaż | 1200–2000$ dirty + sprzedaż |
| 4 — poszukiwacz | 7000–10000$ | 1500–2500$ |
| Cooldown zad. 2 | 24 h | 24 h |

---

## Mandaty MDT — skala docelowa

Pełna tabela per wykroczenie w [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md). Zasady ogólne:

| Kategoria | Obecnie (przykłady) | Docelowo |
|---|---:|---:|
| Drobne wykroczenia | 1000–2000$ | 30–150$ |
| Średnie | 3000–10000$ | 150–800$ |
| Ciężkie | 25000–50000$ | 800–3000$ |
| Najcięższe (zabójstwo itd.) | 50000$+ | 3000–8000$ |

Udział wystawiającego: LSPD/FIB/EMS **25%**, firmy mechaniczne **40%**.

---

## Kolejność wdrożenia

1. Paycheck + zasiłek + warunek on duty  
2. Prace dorywcze (KQ — priorytet) + hunter  
3. Mandaty MDT + usługi EMS/DOJ + FIB  
4. Premie tygodniowe (bossmenu)  
5. Napady aktywne + narzędzia crime  
6. Pralnia + lombard (pełna tabela) + narkotyki  
7. Sklepy, usługi, bronie, paliwo, wygląd, siłownia  
8. Pojazdy (`vehicles.json` + `VehiclePriceConfig`)  
9. Mechanik (lokalny + tuning)  
10. Napady planowane + Tracker + Fernando  
11. qs-housing (DB + config)  
12. Job center UI  

---

## Zakres dokumentu

| Moduł | Status w v2 |
|---|---|
| Paycheck, side joby, frakcje, firmy, crime | Opisane |
| 14 napadów | Opisane (7 aktywnych + 7 planowanych) |
| Narkotyki, lombard, pralnia | Opisane |
| Pojazdy, sklepy, bronie | Opisane |
| Mieszkania (segmenty, opłaty, bills) | Opisane |
| Paliwo, wygląd, siłownia, ubezpieczenie | Opisane |
| FIB, mechanik, hunter, Fernando | Opisane |
| Mandaty MDT per wykroczenie | Wdrożenie (szczegóły w `ekonomia-wdrozenie.md`) |
| Śmieciarz | Do implementacji |
| Napad truckera | Poza progresją v2 |
| KQ weed (produkcja) | Produkcja pozostaje; ceny sprzedaży w tabeli narkotyków |

**Napad truckera** w komentarzu `rage_heists/Config.lua` nie wchodzi w progresję v2 — do ewentualnego dodania osobno.

---

## Status decyzji

| Temat | Status |
|---|---|
| Zasiłek 60$/h | Ustalone |
| Model wypłat: minutówka + interakcje + premia tygodniowa | Ustalone |
| Minutówka: DOJ 472–500; LSPD/FIB 420–448; EMS 288–312 $/h | Ustalone |
| Miękki limit mandatów → society | Do ustalenia |
| Skala typowy / aktywna sesja | Ustalone |
| DOJ: wyższa minutówka + podatki society | Ustalone |
| Mandaty i usługi — widełki ogólne | Ustalone |
| Premie tygodniowe (~36 h/tydz., max 10k) | Ustalone |
| Progresja 14 napadów (pełne karty) | Ustalone |
| Narkotyki — tabela per item | Ustalone |
| Lombard — pełna tabela | Ustalone |
| Narzędzia napadów | Ustalone |
| Pojazdy — segmenty i limity | Ustalone |
| Mieszkania — segmenty cenowe | Ustalone |
| Hipoteka / czynsz — rebalance | Do ustalenia (5–8% / 30–60 min) |
| Side job 500 vs 550–600$/h (kotwica) | Do ustalenia |
| Cooldowny ATM (karta/hack) | Do ustalenia |
| Mandaty MDT per wykroczenie | Wdrożenie |
| Śmieciarz | Do implementacji |
| Napady planowane (7 szt.) | Implementacja |

---

## Podsumowanie

Dokument obejmuje **pełny zakres ekonomii serwera**: zarobki, crime, frakcje, koszty życia, nieruchomości, utrzymanie pojazdów i moduły pomocnicze. Kluczowe założenia:

- Kotwica side job ~500$/h; zasiłek 60$/h.
- Minutówka: DOJ 472–500$/h; LSPD/FIB 420–448$/h; EMS 288–312$/h.
- 14 napadów — pełne karty w dokumencie.
- Endgame auto max 1.2 mln; endgame dom max ~800k.
- 7 napadów planowanych — implementacja.

[`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md)
