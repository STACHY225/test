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

Przy spokojnej służbie bez interakcji z graczami (np. LSPD bez mandatów) zarobek wynosi ok. **85$/h** — poniżej side joba. Wyższe widełki frakcji i crime wymagają aktywnej gry, współpracy z innymi graczami oraz — w przypadku crime — ponoszenia ryzyka i cooldownów. Firma prywatna stanowi przejście między side jobem a frakcją.

Kotwica side job (500$/h vs 550–600$/h) — patrz sekcja **Status decyzji**.

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
| LSPD / LSSD / EMS | 20–22$ | 80–88$ | on duty |
| DOJ | 25–30$ | 100–120$ | on duty |

**Zasady:** wypłata co 15 min na konto gracza; DOJ — wyższa minutówka ze względu na brak powtarzalnych kursów i usług jak w EMS.

---

## Frakcje publiczne

### Model wypłat

| Źródło | Kiedy | Rola |
|---|---|---|
| Paycheck | co 15 min, on duty | podłoga 80–120$/h |
| Zarobek aktywny | natychmiast po akcji | 0–1200$/h zależnie od aktywności |
| Premia tygodniowa | bossmenu, raz/tydz. | dodatek; +50–120$/h średnio przy 36 h/tydz. |

### Zarobek aktywny — widełki

| Poziom | Dodatkowy $/h | Kontekst |
|---|---:|---|
| Spokojna służba | 0–200 | patrol, brak interakcji |
| Normalna służba | 200–600 | mandaty, revive, drobne sprawy |
| Wysoka aktywność | 600–1200 | eventy, dużo interakcji, poważne sprawy |

**Przykłady (1 h):**

| Scenariusz | Paycheck | Aktywność | Razem |
|---|---:|---:|---:|
| LSPD, zero mandatów | ~85$ | 0$ | ~85$/h |
| LSPD, 4 mandaty × ~120$ (25%) | ~85$ | ~120$ | ~205$/h |
| EMS, 6× revive (~80$ udziału) | ~85$ | ~480$ | ~565$/h |
| DOJ, spokojna służba | ~110$ | 0$ | ~110$/h |
| DOJ, 2 sprawy × ~400$ (25%) | ~110$ | ~200$ | ~310$/h |

### LSPD / LSSD

Paycheck + mandaty/faktury (25% kwoty) + okazjonalne konwoje i sprawy.

### EMS

Paycheck + revive/leczenie (25%) + **kursy do lokalnych medyków** (główne, powtarzalne źródło).

### DOJ

| Element | Założenie |
|---|---|
| Paycheck gracza | 100–120$/h — wyższy niż LSPD/EMS |
| Zarobek aktywny | wyroki, ugody, licencje — nieregularny |
| Konto DOJ (society) | 10% od podatków (mandaty, sprzedaż firm, salon pojazdów, usługi EMS itd.) |
| Premia tygodniowa | jak pozostałe frakcje |

Środki society DOJ finansują frakcję i premie — nie trafiają bezpośrednio do gracza.

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

| Item | Skup docelowy | Źródło |
|---|---:|---|
| silver_ring, earrings, bracelet, wallet | 12–30$ | NPC |
| watch | 30–55$ | NPC / lombard |
| gold_ring, gold_chain, necklace_* | 45–90$ | jubiler |
| earrings_diamond, necklace_emerald | 80–150$ | jubiler |
| diamond | 100–160$ | jubiler (rzadki) |
| designer_bag | 70–120$ | jubiler / lombard |
| phone, smartwatch (dark_lombard) | 35–70$ | NPC |

Dark lombard (Lester): mnożnik **×0.95**.

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

## Mieszkania (qs-housing)

Osobna iteracja po głównym przeskalowaniu. Raty i czynsze muszą być spójne z kotwicą 500$/h.

---

## Kolejność wdrożenia

1. Paycheck + zasiłek + warunek on duty  
2. Prace dorywcze (KQ — priorytet)  
3. Mandaty MDT + usługi EMS/DOJ  
4. Premie tygodniowe (bossmenu — implementacja tierów)  
5. Napady aktywne + narzędzia crime  
6. Pralnia  
7. Narkotyki + lombard  
8. Sklepy, usługi, bronie  
9. Pojazdy  
10. Napady planowane (boosting, tracker, jubiler, lombard, jacht, humane, cayo)  
11. qs-housing  

---

## Zakres i ograniczenia dokumentu

| W zakresie v2 | Poza zakresem / osobna iteracja |
|---|---|
| Paycheck, side joby, frakcje, firmy, crime | qs-housing (szczegóły rat i czynszów) |
| 14 napadów (7 aktywnych + 7 planowanych) | Mandaty MDT per wykroczenie (tabela w wdrożeniu) |
| Narkotyki (17 itemów), lombard, pralnia | Paliwo (`ox_fuel`) — do ustalenia |
| Pojazdy, sklepy, usługi, bronie | Zadania Fernando, hunter job — niski priorytet |
| | FIB — paycheck jak LSPD; brak osobnych widełek |

**Napad truckera** w komentarzu `rage_heists/Config.lua` nie wchodzi w progresję v2 — do ewentualnego dodania osobno.

---

## Status decyzji

| Temat | Status |
|---|---|
| Zasiłek 60$/h | Ustalone |
| Model wypłat: aktywny + premia tygodniowa | Ustalone |
| Skala typowy / aktywna sesja | Ustalone |
| DOJ: wyższa minutówka + podatki society | Ustalone |
| Mandaty i usługi — widełki ogólne | Ustalone |
| Premie tygodniowe (~36 h/tydz., max 10k) | Ustalone |
| Progresja 14 napadów (pełne karty) | Ustalone |
| Narkotyki — tabela per item | Ustalone |
| Narzędzia napadów | Ustalone |
| Pojazdy — segmenty i limity | Ustalone |
| Side job 500 vs 550–600$/h (kotwica) | Do decyzji |
| Cooldowny ATM (karta/hack) | Do decyzji |
| Mandaty MDT per wykroczenie | Wdrożenie |
| qs-housing | Osobna iteracja |
| Napady planowane (7 szt.) | Implementacja |

---

## Podsumowanie

Dokument jest **spójny i kompletny na etapie planowania**. Kluczowe założenia:

- **Kotwica side job ~500$/h** + zasiłek 60$/h daje sensowny progres (pierwsze auto w kilka godzin).
- **Luka zarobków** jest akceptowalna przy rozróżnieniu typowy / aktywna sesja — spokojna służba LSPD nie dominuje nad side jobem.
- **14 napadów** ma pełne karty: gracze, łup, mnożniki, policja, cooldown, wymagania.
- **7 napadów planowanych** wymaga implementacji — zespół powinien traktować je jako roadmap, nie obecny stan serwera.
- **Wdrożenie musi być globalne** — szczególnie KQ Deliveries/Powerwashing, bez tego plan traci sens.

**Warunki poprawnego balansu po wdrożeniu:** synchronizacja wszystkich configów, górne widełki jako sufit sesji (nie stała stawka), testy po deployu.
