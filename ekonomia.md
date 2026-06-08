# RageCity — docelowa ekonomia serwera (v2)

Dokument planistyczny do analizy zespołowej. Opisuje **docelowy stan** ekonomii po przeskalowaniu — bez wdrożenia w kodzie.

| Dokument | Przeznaczenie |
|---|---|
| **Ten plik** | Wizja, balans, widełki — **pełny zakres ekonomii serwera** |
| [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) | Checklista zmian per skrypt + tabele MDT |
| `docs/ekonomia-*-discord.txt` | Snapshoty stanu obecnego (configy) |
| `docs/_gen_mdt_fines_table.py` | Generator tabel MDT (police / EMS / mechanik) |

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
| LSPD / LSSD / EMS | 650–900 | 1400–2000 |
| DOJ | 580–750 | 1100–1600 |
| Crime (po praniu, z cooldownami) | 700–1200 | 1800–2800 |
| Właściciel aktywnej firmy | 900–1400 | 1800–2800 |

### Hierarchia względem side joba

Side job (~500$/h) jest kotwicą ekonomii. Pozostałe ścieżki odnoszą się do niej przez dwa poziomy: **typowy** (przeciętna sesja) i **aktywna sesja** (górna granica).

| Ścieżka | Typowy $/h | × side job | Aktywna sesja $/h | × side job |
|---|---:|---:|---:|---:|
| Side job | 450–550 | 1.0× | 600–750 | 1.0× |
| Firma (pracownik) | 700–1000 | 1.4–2.0× | 1200–1500 | 2.0–2.5× |
| Frakcja (LSPD / LSSD / EMS) | 650–900 | 1.3–1.8× | 1400–2000 | 2.3–3.3× |
| DOJ | 580–750 | 1.2–1.5× | 1100–1600 | 2.2–3.2× |
| Crime | 700–1200 | 1.4–2.4× | 1800–2800 | 3.0–4.7× |

Kotwica side job: **500–600$/h** (ustalone).

### Tempo progresu (orientacyjne)

| Cel | @ 500$/h (side job) | @ 800$/h (typ. frakcja) | @ 1200$/h (typ. crime) |
|---|---:|---:|---:|
| Pierwsze auto (1.5k) | 3 h | 2 h | 1 h |
| Sensowne auto (10k) | 20 h | 13 h | 8 h |
| Dobre auto (75k) | 150 h | 94 h | 63 h |
| Super auto (500k) | 1000 h | 667 h | 417 h |
| Perełka (1.2 mln) | 2400 h | 1600 h | 1000 h |

Przy 6 h/dzień, super auto 500k: **~28 tygodni** (side job), **~19 tygodni** (frakcja), **~12 tygodni** (crime).

---

## Paycheck i zasiłek

**Stan obecny:** `[core]/es_extended/server/paycheck.lua` wypłaca 125$/15 min cywilom i 250$/15 min frakcjom — bez warunku on duty.

| Grupa | Co 15 min | $/h | Warunek |
|---|---:|---:|---|
| Bezrobotny (zasiłek) | 15$ | 60$ | brak aktywnej pracy / off-duty |
| Praca dorywcza | 15$ | 60$ | on duty; reszta z aktywności jobu |
| Firma prywatna `c_*` | 10–15$ | 40–60$ | on duty |
| LSPD / LSSD / FIB | 168–175$ | 672–700$ | on duty |
| EMS | 115–125$ | 460–500$ | on duty |
| DOJ | 150–158$ | 600–632$ | on duty |

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
| LSPD / LSSD / FIB | 672–700 | opcjonalne (mandaty) | ~90–100% |
| EMS | 460–500 | revive + kursy NPC | ~60–70% |
| DOJ | 600–632 | rzadkie (licencje, sprawy) | ~85–95% |

| Frakcja | Spokojna zmiana (~10 online) | Aktywna sesja (~50 online) |
|---|---:|---:|
| LSPD / LSSD / FIB | 650–750 | 850–1100 |
| EMS | 650–800 | 1000–1500 |
| DOJ | 580–700 | 750–950 |

### Scenariusze (1 h)

Średni mandat 100$, udział funkcjonariusza 25% → 25$/mandat.

| Online | Frakcja | Minutówka | Interakcje | Razem $/h |
|---:|---|---:|---:|---:|
| ~10 | LSPD, 0 mandatów | 680 | 0 | 680 |
| ~10 | LSPD, 1–2 mandaty | 680 | 25–50 | 705–730 |
| ~10 | EMS, 3 kursy NPC | 480 | 240–375 | 720–855 |
| ~10 | EMS, 3 kursy + 1 revive | 480 | 320–455 | 800–935 |
| ~10 | DOJ, 0 spraw | 620 | 0 | 620 |
| ~10 | DOJ, 1 sprawa | 620 | 50–125 | 670–745 |
| ~25 | LSPD, ~4 mandaty | 680 | ~100 | ~780 |
| ~50 | LSPD, ~8 mandatów | 680 | 200–400 | 880–1080 |
| ~50 | EMS, aktywna sesja | 480 | 600–1100 | 1080–1580 |
| ~50 | DOJ, ~4 sprawy | 620 | 200–500 | 820–1120 |

| Scenariusz | Minutówka | Interakcje | Razem $/h |
|---|---:|---:|---:|
| LSPD, 4 mandaty | 680 | ~100 | ~780 |
| FIB, 2 mandaty | 680 | ~50 | ~730 |
| EMS, 6× revive (~80$ udziału) | 480 | ~480 | ~960 |
| EMS, 3 kursy NPC (~80$ udziału) | 480 | ~240 | ~720 |
| DOJ, 2 sprawy × ~100$ (25%) | 620 | ~200 | ~820 |

### LSPD / LSSD / FIB

| Element | Wartość |
|---|---|
| Minutówka | 672–700$/h |
| Mandaty / faktury | 25% kwoty |
| Konwoje, sprawy | okazjonalnie |

### EMS

| Element | Wartość |
|---|---|
| Minutówka | 460–500$/h |
| Revive / leczenie | 25% kwoty |
| Kurs EMS → lokalny medyk | 200–500$ / kurs, 2–4/h solo |
| `BabiczHospitality` (leczenie random pedów) | 100–500$ / pacjent (osobny resource; nie w `server.cfg`) | **scalić z widełkami kursu EMS** lub wyłączyć przy wdrożeniu |

### DOJ

| Element | Wartość |
|---|---|
| Minutówka | 600–632$/h (~85–95% typ. dochodu) |
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

Właściciel dobrze prowadzonej firmy może przewyższyć frakcjonariusza — **ale dopiero po pokryciu kosztów operacyjnych**. Firmy mają **świadomie wyższe koszty** niż side job: przy obecnych widełkach cen menu rozwinięta firma generuje duży obrót; bez tej barier właściciel szybko staje się wielomilionerem względem kotwicy 500$/h.

### Koszty operacyjne firm

| Składnik | Docelowo | Uwagi |
|---|---|---|
| Składniki (`company shop`) | **40–65% ceny menu** | meat, cheese, tortilla — drożej niż sklep 24/7 |
| Paycheck pracowników | **40–60$/h** | on duty, job `c_*` |
| Podatek DOJ | **10% obrotu** | przy każdej sprzedaży klientowi |
| Kursy / dostawy | czas + opłaty | 50% nagrody kursu wraca do firmy |
| Premie pracownikom | z bossmenu | obniża marżę netto właściciela |
| Stock / resupply | regularny | brak zapasów = brak sprzedaży |
| `depositProducts` | **100–110%** ceny menu → society | obecnie **150%** — zbyt hojne po obniżeniu menu |

**Przykład — cheeseburger 34$:** składniki ~18–22$ (55–65%) · DOJ 10% ≈ 3,4$ · **marża brutto ~8–12$** przed wypłatami i premiami.

DOJ otrzymuje 10% od obrotu firmy. Wysoki potencjał właściciela (1800–2800$/h aktywna sesja) wymaga **inwestycji, personelu i ciągłej aktywności** — nie pasywnego dochodu.

### Kursy firmowe (`BabiczCompanyCourses`)

Wspólny moduł: `rage_fractions/Fractions/company_functions/` — ten sam flow dla wszystkich firm z `C.Course` w configu.

| Firma (`job`) | Label | Pojazd | Nagroda / paczka (obecnie) |
|---|---|---|---|
| `c_burgershot` | Burger Shot | `nspeedo` | 400–800$ |
| `c_tequilala` | Tequi-la-la | `benson` | 400–800$ |
| `c_blazingtattoo` | Blazing Tattoo | `bison3` | 450–850$ |
| `c_fiverecords` | Five Records | `nspeedo` | 400–800$ |
| `c_quiettech` | Quiet Tech | `nspeedo` | 400–800$ |
| `c_blackrepair` | Ghost Grid | `nspeedo` | 400–800$ |
| `mechanic` | Los Santos Customs | `nspeedo` | 400–800$ |

**Flow:** garaż → port (dock, wspólne coords) → max **10 paczek** → rozładunek HQ · **7,5 s** na akcję · **brak cooldownu**.

**Podział nagrody (kod):**

| Odbiorca | Udział |
|---|---|
| Pracownik | **100%** `reward` (gotówka, `ox_inventory` money) |
| Society (bossmenu) | **`floor(reward × 0.5)`** |
| DOJ | **0%** |

**Efekt uboczny:** każda dostarczona paczka wywołuje `RefillJobShop` — uzupełnia stock sklepu klienta (Burger Shot / Tequi-la-la).

**Problem v2:** pełny kurs (10× ~600$) = **~6000$** pracownik + **~3000$** society ≈ **9000$** tworzone co ~20–30 min, bez limitu powtórzeń. Przy obecnym paychecku `c_*` (**250$/15 min = 1000$/h**) kursy i `depositProducts` **znacznie przekraczają** widełki v2 (pracownik 700–1000$/h).

**Docelowo (wszystkie firmy z kursem):**

| Parametr | Obecnie | Docelowo |
|---|---|---|
| Nagroda / paczka | 400–800$ | **80–150$** |
| Max paczek | 10 | 10 (bez zmian) |
| Split society | 50% | 50% (bez zmian) |
| Cooldown po pełnym kursie | brak | **30–45 min per gracz** |
| Tattoo kurs | 450–850$ | **90–160$** (ten sam tier) |

Przy 80–150$/paczka i ~6 paczek/h: **~480–900$/h** pracownik + **~240–450$/h** society — spójne z kotwicą side job + bonusem firmy.

### Sklepy firmowe (`BabiczCompanyShop`)

Dotyczy **sklepu klienta** (Burger Shot, Tequi-la-la) — nie sklepów ze składnikami.

| Zdarzenie | Pracownik | Society | DOJ |
|---|---|---|---|
| Klient kupuje z menu | 0$ auto | **ceil(90%)** | **floor(10%)** |
| `depositProducts` (odłożenie craftu) | 0$ | **ceil(cena menu × 1,5)** | 0% |
| Kupno składników (internal ox shop) | −cena z kieszeni | 0$ | 0$ (sink) |

**`depositProducts` (150%)** — obecnie **niewspomniane w v2**, a istotne: pracownik craftuje burgera ze składników (~23$), odkłada do sklepu → society dostaje **+72$** przy menu 48$. To **subsydia** zachęcająca do produkcji, ale po obniżeniu cen menu do v2 trzeba **obniżyć mnożnik** do **100–110%** (nie 150%).

**Składniki (internal shop, `Config.lua`):**

| Sklep | Przykłady cen | % ceny menu (cheeseburger 48$ → docel. 34$) |
|---|---|---|
| `c_burgershot_ingredients_shop` | meat 13$, cheese 5$, tortilla 5$ | ~23$ = **48%** menu (48$) · po v2 menu 34$ → **~68%** bez podwyżki składników |
| `c_tequilala_ingredients_shop` | meat 13$, sausage 10$, bread 5$ | podobnie |

**Wdrożenie v2 — składniki:** podnieść ceny składników tak, by **40–65% ceny menu docelowego** (np. cheeseburger 34$ → składniki **14–22$**).

**Menu klienta (obecnie vs docelowo):**

| Item | Obecnie | Docelowo v2 |
|---|---:|---:|
| cheeseburger | 48$ | 34$ |
| burgir (Tequi) | 48$ | 40$ |
| becon_burger | 56$ | 35$ |

Pracownik **nie dostaje automatycznego %** od sprzedaży klientowi — tylko kurs, paycheck i ewentualne premie bossmenu.

---

## Skala cen pojazdów

| Poziom | Cena | @ 500$/h |
|---|---:|---:|
| Najtańszy pojazd | 1000–1500$ | 2–3 h |
| Pierwszy sensowny | 5000–10000$ | 10–20 h |
| Dobry | 25000–75000$ | 50–150 h |
| Bardzo dobry | 150000–400000$ | 300–800 h |
| Sport / super (większość modeli) | 300000–700000$ | 600–1400 h |
| Perełki endgame | 900000–1200000$ | 1800–2400 h |

| Typ | Zakres cen |
|---|---|
| Auta osobowe | większość top: **300–700k**; perełki max **1.2 mln** |
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
| Cooldowny | **osobny cooldown per gracz na każdy typ napadu** — wymusza rotację, nie farmienie jednego źródła |
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
| 8 | Jubiler | 2 | 5 | 2800$ | 6500$ | biżuteria → lombard | Planowany |
| 9 | Bank Fleeca | 2 | 4 | 4500$ | 12000$ | black_money | Aktywny |
| 10 | Napad na lombard | 1 | 3 | 600$ | 1800$ | gotówka + biżuteria | Planowany |
| 11 | Jacht | 3 | 5 | 10000$ | 25000$ | black_money / loot | Planowany |
| 12 | Bank Pacific | 4 | 6 | 35000$ | 90000$ | black_money / loot | Aktywny |
| 13 | Humane Labs | 3 | 5 | 22000$ | 55000$ | black_money / loot | Planowany |
| 14 | Cayo Perico | 5 | 8 | 70000$ | 130000$ | black_money / loot | Planowany |

*Łup w biżuterii = łączna wartość skupu w lombardzie (nie gotówka bezpośrednio).*

### Cooldowny napadów (zasada globalna)

Każdy typ napadu ma **własny cooldown per gracz** (nie globalny serwerowy). Gracz nie może powtarzać tego samego napadu w pętli — musi rotować między aktywnościami crime.

| Tier | Napady | Cooldown per gracz |
|---|---|---|
| Szybkie (solo) | NPC, ATM karta, boosting | 2–15 min |
| Średnie | ATM hack, kasetka, tracker, sejf sklepu | 8–20 min |
| Grupowe | Jubiler, Fleeca, lombard | 30–60 min |
| Endgame | Jacht, Pacific, Humane Labs, Cayo | 60–120 min |

Szczegóły per napad — w kartach poniżej. Wdrożenie: `[rage]/rage_heists/Config.lua` (+ poszczególne heisty).

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
| Cooldown | **5 min per gracz per bankomat** |
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
| Cooldown | **10 min per gracz** |
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
| Łup | Biżuteria o łącznej wartości skupu **2800–6500$** |
| Typ nagrody | Itemy → sprzedaż w lombardzie / dark_lombardzie |
| Policja min. | 3 |
| Cooldown | ~30 min |
| Czas akcji | ~5–10 min |
| Wymagania | Broń, opcjonalnie hackingdevice |
| Loot | Pierścionki, łańcuszki, kolczyki, diamenty (patrz tabela biżuterii) |

**Loot na osobę (przykład, baza 3000$):**

| Os. | Mnożnik | Wartość łączna | Na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 2800–6500$ | 1400–3250$ |
| 3 | ×1.15 | 3220–7475$ | 1073–2492$ |
| 5 | ×1.35 | 3780–8775$ | 756–1755$ |

---

### 9. Bank Fleeca

| Parametr | Wartość |
|---|---|
| Status | Aktywny (`rage_heists`) |
| Gracze | 2–4 (min. 2 w promieniu 15 m) |
| Mnożnik | 2 os. ×1.0 · 3 os. ×1.2 · 4 os. ×1.35 |
| Łup łączny | 4500–12000$ black_money |
| Policja min. | 4 (promień ~9500 m) |
| Cooldown | ~15 min per lokalizacja |
| Czas akcji | ~360 s |
| Wymagania | Hackingdevice, wiertło |
| Lokalizacje | 5 banków Fleeca |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 4500–12000$ | 2250–6000$ |
| 3 | ×1.2 | 5400–14400$ | 1800–4800$ |
| 4 | ×1.35 | 6075–16200$ | 1519–4050$ |

*Po praniu (25% straty): ok. 1688–4500$ czystego na osobę przy 2 napastnikach.*

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
| Uwagi | Obecnie minGroup=1 w config — docelowo min. 4 os.; **uwaga:** w `BankPacific` server **brak wypłaty** money/black_money — karta v2 = docelowy stan po implementacji lootu |

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

### Model kosztów życia

Kotwica: **500–600$/h** (side job). Żywienie jest **obowiązkowym składnikiem** kosztów życia — ceny w sklepach wynikają z budżetu, nie odwrotnie.

| Składnik | % dochodu | @ 500$/h | @ 700$/h | @ 1000$/h |
|---|---:|---:|---:|---:|
| **Żywienie** (jedzenie + picie) | 8–10% | **40–50$** | **56–70$** | **80–100$** |
| Paliwo / pojazd | 4–6% | 20–30$ | 28–42$ | 40–60$ |
| Usługi bieżące (myjnia, drobne) | 2–4% | 10–20$ | 14–28$ | 20–40$ |
| Amortyzacja (telefon, wygląd) | 2–3% | 10–15$ | 14–21$ | 20–30$ |
| **Razem koszty życia** | **16–22%** | **80–115$** | **112–161$** | **160–230$** |

Po opłaceniu kosztów życia @ 500$/h zostaje **~385–420$/h** netto (side job).

### Metabolizm (`rage_hud/Config.lua`)

| Parametr | Wartość |
|---|---|
| Tick HUD | 300 ms |
| Spadek głód / tick | 0.0035 |
| Spadek pragnienie / tick | 0.006 |
| **Zużycie głód / h** | **~42 pkt** |
| **Zużycie pragnienie / h** | **~72 pkt** |

### Żywienie — budżet i profile

**Budżet bazowy @ 500$/h: 45$/h** (9% dochodu).

| Profil | Loadout / h | Głód | Pragnienie | Koszt / h |
|---|---|---:|---:|---:|
| **Minimum** (sklep 24/7) | 4× woda + 2× kanapka + 1× chips | ~49 | 72 | **45$** |
| Standard | 3× woda + 1× ecola + 2× kanapka + 1× hotdog | ~50 | 73 | **48$** |
| Restauracja | 1× posiłek BS / ~2,5 h + uzupełnienie napojami ze sklepu | ~42 | 72 | **55–65$** |

Restauracja = wybór gracza; **nie podnosi** obowiązkowego minimum kosztów życia.

**Stawki wynikające z profilu Minimum (45$/h):**

| Składnik | Obliczenie | Stawka |
|---|---|---:|
| Napoje sklep | 28$ / 72 pkt pragnienia | **~0,39 $/pkt** |
| Jedzenie sklep | 17$ / ~49 pkt głodu | **~0,35 $/pkt** |
| Restauracja | ×1,35–1,45 vs sklep | **~0,50–0,55 $/pkt** |

### Sklep 24/7 — ceny docelowe (`rage_market` → `Config.Food`)

| Item | Głód | Pragnienie | Obecnie | Docelowo |
|---|---:|---:|---:|---:|
| water | — | 18 | 20$ | **7$** |
| ecola | — | 18 | 22$ | **7$** |
| sprunk | — | 19 | 22$ | **7$** |
| pomarancz | 2 | 20 | — | **8$** |
| sandwich | 17 | — | 36$ | **7$** |
| bread | 3 | — | 7$ | **2$** |
| chips | 15 | — | 37$ | **3$** |
| donut | 15 | — | 34$ | **3$** |
| toast | 16 | — | — | **3$** |
| hotdog | 16 | — | 38$ | **6$** |
| burger | 22 | — | — | **8$** |
| pizza | 21 | — | — | **8$** |
| fries | 19 | — | — | **7$** |
| salad | 21 | 5 | — | **9$** |
| coffee | — | 15 | 90$ | **12$** |
| junkenergy | 5 | 12 | 120$ | **14$** |
| electrolytes | — | 14 | — | **6$** |
| apple | 6 | 1 | — | **3$** |
| banana | 8 | — | — | **3$** |

Automaty (`ox_inventory/data/shops.lua`): jak sklep 24/7.

Weryfikacja Minimum: 4×7 + 2×7 + 1×3 = **45$/h**.

### Burger Shot — ceny docelowe

| Item | Głód | Pragnienie | Obecnie | Docelowo |
|---|---:|---:|---:|---:|
| veegi_burger | 58 | — | 43$ | **32$** |
| cheeseburger | 61 | — | 48$ | **34$** |
| becon_burger | 63 | — | 56$ | **35$** |
| frytki | 54 | — | 38$ | **27$** |
| schakemalinowy | 34 | 30 | 46$ | **32$** |
| mint_lemonade | — | 34 | 34$ | **13$** |
| orange_juice | — | 32 | 32$ | **12$** |
| apple_juice | — | 31 | 31$ | **12$** |
| banana_smoothie | 7 | 33 | 39$ | **14$** |
| tropical_juice | — | 36 | 36$ | **14$** |

Posiłek BS (burger + frytki + napój): **~73$** · ~2,5 h głodu + ~1 h pragnienia → **~29$/h** sam posiłek + napoje ze sklepu.

### Tequi-la-la

Poza budżetem żywieniowym (alkohol / RP). Ceny ~1,2–1,5× Burger Shot za porównywalne staty.

| Item | Głód | Pragnienie | Obecnie | Docelowo |
|---|---:|---:|---:|---:|
| hot_cat | 31 | 34 | 43$ | **38$** |
| burgir | 33 | 32 | 48$ | **40$** |
| beer | — | 16 | 46$ | **10$** |
| woda_po_studencie | — | 28 | 34$ | **14$** |
| wino_z_kartonu | — | 27 | 36$ | **16$** |
| bum / woda_alko / rozwod | — | 32–40 | 36–56$ | **20–38$** |
| szampon | — | 30 | 56$ | **28$** |

### Alkohol — monopolowy (`rage_market` Liquor)

Poza kosztami życia (rozrywka).

| Item | Pragnienie | Obecnie | Docelowo |
|---|---:|---:|---:|
| beer | 16 | 45$ | **10$** |
| vine | 13 | 180$ | **22$** |
| champagne | 12 | 200$ | **30$** |
| rhum / vodka | 8–9 | 120–250$ | **45$** |
| whisky | 8 | 300$ | **55$** |

### Pozostałe składniki kosztów życia

| Pozycja | Docelowo | Obecnie | Uwagi |
|---|---:|---:|---|
| Paliwo (`priceTick`) | 2–4$/tick | 8$ | § Paliwo |
| Kanister / uzupełnienie | 400–700$ / 250–500$ | 1500/1000$ | § Paliwo |
| Myjnia | brud × 4–8$ | brud × 12$ | § Paliwo |
| Telefon | 500–1000$ | 1250–1500$ | amort. ~10–15$/h |
| Radio | 300–700$ | 850$ | jednorazowo |
| Wygląd (fryzjer / ubranie) | 60–150$ | 180–200$ | § Wygląd |
| Odholowanie | 300–700$ | 800$ | incydentalnie |
| Impound specjalny | 1500–3000$ | 5000$ | incydentalnie |
| Lokalny medyk | 500–1500$ | 50–10000$ | incydentalnie |
| Lokalny mechanik | 500–1500$ | 1000–5000$ | incydentalnie |
| Naprawa z garażu | 500–1500$ | 1500$ | incydentalnie |

### Narzędzia i jednorazówki (poza bieżącymi kosztami życia)

| Pozycja | Docelowo | Obecnie |
|---|---:|---:|
| Lockpick | 80–200$ | 1200$ |
| Fixkit | 250–600$ | 5000$ |
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
| Czynsz (wynajem) | co 5 min / miesięcznie | **0,6–1,6% wartości nieruchomości co 2 tygodnie** |
| Dochód wynajmującego | czynsz → konto bankowe właściciela (`qs-housing`) | **bez zmian mechanizmu**; po rescale czynszu — pasywny dochód, nie grind |
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
| Paycheck | **168–175$/15 min** (672–700$/h), on duty |
| Udział z mandatów | **25%** kwoty (jak LSPD) |
| Katalog mandatów | ten sam co LSPD (`policeFines`) |
| Premie tygodniowe | jak pozostałe frakcje |

Typowy / aktywna sesja: jak LSPD/LSSD (**650–750 / 850–1100$/h** spokojna · **1400–2000$/h** aktywna).

---

## Frakcja mechanik

Źródła: `Fractions/mechanic/`, `BabiczTuningMenu_sv.lua`, `BabiczMechanic_shared.lua`, `BabiczFlatbed_*`, `BabiczCompanyCourses` (kurs paczkowy LSC).

**Jobs:** `mechanic` (LSC), `c_blackrepair`, `c_quiettech` (dziedziczą tuning + kurs z mechanic, osobne society).

### Zarobki — widełki docelowe

| Rola | Typowy $/h | Aktywna sesja $/h | Składniki |
|---|---:|---:|---|
| Mechanik LSC (on duty) | 650–900 | 1200–1800 | minutówka + MDT usługi + tuning (10%) |
| Mechanik sub-shop | 700–950 | 1300–2000 | jak wyżej + 40% z mandatów MDT |
| Kurs paczkowy (obecny) | — | **2400–4800$** (za wysoko) | do obniżenia jak firmy |

Minutówka `mechanic` w v2: jak frakcja **672–700$/h** on duty (po zmianie `paycheck.lua` — obecnie **125$/15 min** cywilna stawka).

### Lokalny mechanik (NPC)

12 lokacji (`C.LocalMechanic.Locations`). Płatność **tylko od gracza** — pure sink, **0$** dla frakcji mechanik.

| Warunek | Obecnie | Docelowo |
|---|---:|---:|
| Brak mechaników online | 1000$ | 500–800$ |
| ≥1 mechanik online | 5000$ | 1000–1500$ |

Cel: NPC jako **fallback**, nie konkurencja dla mechaników graczy przy wysokiej stawce „mechanicy online”.

### Tuning — wzór ceny

```
cenaModu = GetModPrice(mod, poziom) + floor(cenaPojazduSklep × shopPricePercent)
```

- `cenaPojazduSklep` = `rage_vehicleshop:GetVehiclePrice(model)` (fallback 100 000$)
- **Performance** (silnik, hamulce, skrzynia, zawieszenie, pancerz, turbo): **+5%** ceny auta + flat
- **Domyślnie** (nieustawione): **+0,25%**
- **Kosmetyka / większość race-tech UI:** **0%**

**Przykład @ auto 100k$ — silnik poziom 4:** flat 5000$ + 5% = **10 000$** za jeden mod.

**Wysokie flaty (obecnie, do przeskalowania):**

| Mod | Obecnie |
|---|---:|
| cartech_nitro | 75 000$ |
| cartech_turbo | 100 000$ |
| rParachute | 1 200 000$ |
| rSusp / rTires / rAirBag / rArmour | 10–12k$ |

Po deflacji `vehicles.json`: obniżyć **flaty ×0,1–0,2** oraz **shopPricePercent** performance z **5% → 1,5–2,5%**.

### Tuning — podział przy fakturze (invoice → MDT)

| Odbiorca | Udział | Uwagi |
|---|---:|---|
| Society (firma) | **35%** | `DepositMoney(job, floor(total×0.35), …)` |
| DOJ | **10%** | 2. argument `DepositMoney` |
| Mechanik wystawiający | **10%** | `addAccountMoney` bank |
| Reszta | **~45%** | sink / flow MDT `Sentence` — nie wraca do klienta |

**Bez faktury:** mechanik płaci **gotówką z własnej kieszeni** — brak splitu, brak wpływu do society.

**Korekta dokumentacji:** „zwrot klientowi 10%” → **premia mechanika wystawiającego 10%**.

### MDT — usługi mechaniczne (`Config.Fines.mechanic`)

Osobny katalog (myjnia, blacharka, holowanie, palnik…). Split jak mandaty:

| Job | Udział wystawiającego | Obecnie |
|---|---:|---|
| `mechanic` (LSC) | `finePercentForPlayer` | **25%** |
| `c_blackrepair`, `c_quiettech` | | **40%** |

**Docelowo:** LSC **40%** (jak sub-shopy) · kwoty usług przeskalować (np. myjnia 400$ → **80–120$**, holowanie/km 400$ → **60–100$**, komplet naprawczy 1800$ → **400–700$**).

### Holowanie / laweta (stan obecny)

| System | Ekonomia |
|---|---|
| **Tow** (`towVehicle` w `BabiczMechanic_sv`) | 10 s → auto do impound · **0$** dla mechanika |
| **Flatbed** (`BabiczFlatbed_*`) | załaduj/rozładuj · **0$** · `flatbed3` · mechanic + police on duty |
| **MDT „holowanie za km”** | tylko jeśli mechanik **ręcznie wystawi fakturę** |

Odholowanie z `rage_garages` (odbiór auta): docelowo **300–700$** (osobna sekcja kosztów życia).

### Kurs paczkowy LSC (obecny)

Identyczny moduł co Burger Shot: port → LSC HQ · **400–800$/paczka** · 50% society · **brak cooldownu**.

**Docelowo:** **80–150$/paczka** + cooldown **30–45 min** (jak pozostałe firmy). Tematycznie słaby dla mechanika — docelowo zastąpić lub uzupełnić kursem lawety.

### Kurs lawetą — planowany (implementacja)

Nowy typ kursu w `BabiczCompanyCourses` lub osobny moduł — **priorytet po rescale tuningu**.

| Parametr | Propozycja |
|---|---|
| Pojazd | `flatbed3` (już w `C.Flatbed`) |
| Start | garaż LSC / sub-shop |
| Cel | jedna z **12 lokacji** `C.LocalMechanic` (losowa lub najbliższa) |
| Akcja | załaduj auto (NPC „awaria” lub pojazd gracza w misji) → dostarcz → rozładuj |
| Nagroda | **150–350$** gotówki + **50%** do society |
| Cooldown | **15–20 min** per gracz |
| Wymagania | on duty, `mechanic` / `c_*` z flatbed w garażu |
| Policja / ryzyko | opcjonalnie 0–1 PD online |

**Szacunek:** ~3–4 kursy/h → **450–1400$/h** + minutówka — spójne z frakcją, bez inflacji jak paczki 400–800$.

### Sklep frakcji (narzędzia)

| Item | Sklep frakcji | Market cywil | Docelowo v2 |
|---|---:|---:|---:|
| fixkit | 1500$ | 5000$ | 250–600$ / 400–800$ |
| carokit | 800$ | 1200$ | proporcjonalnie |
| blowtorch | 4500$ | — | 800–1500$ |

Mechanik na służbie: **fixkit / carokit nie zużywane** przy naprawie w terenie — przewaga frakcji, ale koszt stocku dla firmy.

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

---

## Zdrapki (`rage_scratchcard`)

Obecnie ceny 1000–10000$ i mnożniki wygranych (×10–×1000) generują **milionowe wygrane** — poza skalą v2. Zdrapki to rozrywka z **ujemnym lub zerowym EV**, nie źródło dochodu.

| Typ | Item | Cena sklep | Szansa | Wygrana | Max wygrana |
|---|---|---:|---:|---:|---:|
| BigWin | `scratchticketn` | **75$** | 12% | 400–800$ | ~800$ |
| Premium | `scratchticketp` | **200$** | 8% | 1000–2000$ | ~2000$ |
| Deluxe | `scratchticketd` | **500$** | 4% | 5000–8000$ | ~8000$ |

Parametry `price` / `multiplier` w `[rage]/rage_scratchcard/server.lua` — przeskalować razem z `[rage]/rage_market/Config.lua`.

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

**Miękki limit mandatów → society (odrzucony):** propozycja z fazy planowania — górny limit wpływu mandatów do konta frakcji przy dużej liczbie graczy online (ochrona przed inflacją). **Nie wdrażać** — wystarczy skala kwot v2 + udział 25%.

---

## Plażowy sprzedawca (`beach_vendor`)

**Nie jest w job center** — start u NPC na plaży (`rage_jobs`, `isCustom = true`).

| Parametr | Obecnie | Docelowo |
|---|---|---|
| Zarobek $/h (widełki v2) | 350–500$ + zasiłek | bez zmian |
| Zbieranie z budki | 25 s | bez zmian |
| Cooldown budki | 5 min | bez zmian |
| Bonus kursu (NPC klienci) | 40–80$/sprzedaż | 40–80$ (po rescale itemów) |

**Itemy (cena sprzedaży NPC):**

| Item | Cena obecnie | Docelowo |
|---|---:|---:|
| orzeszki_w_karmelu | 62–95$ | 25–40$ |
| prazona_kukurydza | 44–74$ | 18–30$ |
| lody_na_patyku | 170–220$ | 35–55$ |
| rogaliki_z_czekolada | 220–290$ | 45–70$ |

Źródło: `rage_jobs/Jobs/beach_vendor.lua`, `BabiczBeachVendor_sv.lua`.

---

## KQ weed — produkcja (`kq_weed`)

Ceny **sprzedaży** ulicznej — tabela Narkotyki. Poniżej **pętla produkcji** (obecny kod).

| Parametr | Obecnie (OG Kush) | Uwagi v2 |
|---|---|---|
| `baseGrowTime` | 20 min | po testach: ewentualnie +25–50% czasu |
| Yield buds | 6–8 | bez zmian na start; monitorować $/h vs crime |
| `wateringTime` | 9 min | — |
| `collectionLifespan` | 25 min | — |
| `budsPerBrick` | 20 | brick w tabeli narkotyków |
| Nawóz (dark shop) | 1200$ | po deflacji: **300–500$** |
| Szansa mutacji seed | 15% / 5% | bez zmian |

**Szacunek:** ~1 cykl ≈ 20–35 min → 6–8 budów; przy cenie buda 25–80$ (v2) produkcja **nie powinna** przewyższać crime 700–1200$/h bez ryzyka/policji. Korekta yields **po deployu** jeśli testy pokażą inflację.

---

## Kasyno (`pickle_casinos`)

**Krytyczne dla balansu** — obecnie poza skalą v2 (`MaxWager = 500 000$`, żetony do 1M).

| Parametr | Obecnie | Docelowo v2 |
|---|---|---|
| `Config.MaxWager` | 500 000$ | **5 000–15 000$** |
| `MaximumChipPurchaseAmount` | 1 000 000$ | **50 000–100 000$** |
| Podatek przy wypłacie (`tax`) | 10% | 10% (bez zmian) |
| `taxRecipients` | police 50% / ambulance 50% | bez zmian |
| VIP | 7 dni | bez zmian |
| Żetony (najwyższy nominał) | 1 000 000$ | **10 000–25 000$** max chip |

Gry: blackjack, ruletka, poker, sloty, koło fortuny, wyścigi. Ekonomia hazardu = **sink + rozrywka**, nie źródło dochodu — EV kasyna na korzyść banku (dom).

### Kręgle (`rtx_bowling`)

| Parametr | Obecnie | Docelowo |
|---|---|---|
| Wygrana | `addMoney` (kwota z gry) | **sink / rozrywka** — EV ujemne; bez zmian balansu grindu |
| Status resource | w bundle `[tebex]` | monitorować przy deployu |

---

## Dark Shop (`rage_market`)

Sklep crime — gotówka lub `black_money`. Lokalizacja aktywna w configu (góra mapy).

| Kategoria | Przykłady (obecnie) | Docelowo v2 |
|---|---|---|
| Klucze | black market 100k (limit 5 dni) | **8 000–15 000$** |
| Klucz Rico | 50 000$ | **3 000–6 000$** |
| hackingdevice | 5 000$ | **400–800$** |
| drill | 12 500$ | **1 500–3 000$** |
| hackingtablet | 10 000$ | **800–1 500$** |
| thermite | 2 000$ | **400–700$** |
| Pistolety (dirty) | 100–150k | **12 000–25 000$** |
| ammo-9 ×18 | 550 dirty | **80–150$** |
| kq_weed_fertilizer | 1 200$ | **300–500$** |

Limit zakupu pistoletów w sklepie — logika w `BabiczMarket_sv.lua` (pula dzienna).

---

## Bronie legalne — dwa pipeline'y

### `BabiczAmmunation` (dedykowany sklep, licencja)

| Kategoria | Obecnie (przykłady) | Docelowo v2 |
|---|---|---|
| Nóż | 20 000$ | 800–1 500$ |
| Pistolet | 100 000$ | 8 000–15 000$ |
| Pistolet .50 | 180 000$ | 15 000–25 000$ |
| Micro SMG | 240 000$ | 50 000–80 000$ |
| Karabin | 250 000$ | 120 000–200 000$ |
| Amunicja (szt.) | 50–310$ | 5–35$ |

### `rage_market` → `Items.Ammunation` (11 lokacji)

| Kategoria | Obecnie | Docelowo |
|---|---|---|
| Kamizelka | 249–250$ | 150–300$ |
| Wkład lekki | 5 000$ | 1 500–3 000$ |
| Broń biała | 1 200–3 600$ | 400–1 200$ |
| Pistolet | 90 000$ | 8 000–15 000$ |
| Limit pistoletu | 1 / 24h / gracz | zachować |

**Wdrożenie:** oba sklepy **zsynchronizować** do segmentów v2 (pistol 8–15k itd.).

---

## Kluby nocne i bar kasyna VIP

Premium względem monopolowego Liquor — poza budżetem żywienia.

| Sklep | Piwo | Whisky | Docelowo (×~0,25–0,35) |
|---|---:|---:|---|
| `Nightclub` | 60$ | 400$ | 15–25$ / 80–120$ |
| `CasinoVIP` | 85$ | 500$ | 20–30$ / 100–150$ |
| `Liquor` (monopol) | 10$ | 120$ | już w tabeli Food |

---

## Sklep techniczny 24/7 (`rage_market`)

Poza budżetem życia — narzędzia i RP.

| Item | Obecnie | Docelowo |
|---|---:|---:|
| nitro | 10 000$ | 2 000–4 000$ |
| platetape | 3 000$ | 600–1 200$ |
| carjack | 300$ | 80–150$ |
| spare_wheel | 250$ | 60–120$ |
| scuba | 1 600$ | 400–800$ |
| phone | 1 250–1 500$ | 400–700$ |

---

## Automaty (`ox_inventory` → `VendingMachineDrinks`)

| Item | Obecnie | Docelowo (= sklep 24/7) |
|---|---:|---:|
| sprunk / ecola | 22$ | **7$** |
| chips | 30$ | **3$** |

---

## Telefon (`lb-phone`)

| Usługa | Obecnie | Docelowo |
|---|---:|---:|
| Valet (odholowanie auta do gracza) | 100$ | 50–150$ |
| Promocja posta Birdy | 3 500$ | 500–1 000$ |
| Przelew max / transakcja | 100 000$ | bez zmian (anty-inflacja) |
| Limit dzienny / tygodniowy | 1M / 7M | monitorować po deployu |

---

## Blazing Tattoo (`c_blazingtattoo`)

| Element | Obecnie | Docelowo |
|---|---|---|
| Kurs paczkowy | 450–850$/pkg | **90–160$/pkg** (+ cooldown jak firmy) |
| Faktura tatuaż | dowolna kwota (UI) | widełki **150–800$/tatuaż** lub stała stawka |
| `estimatePerTattoo` | 750$ | **120–250$** (podpowiedź UI) |
| Split faktury | 90% firma / 10% DOJ | bez zmian |
| Składniki / menu klienta | brak | model **usługowy** (nie gastronomia) |

Pracownik **nie dostaje auto-%** od faktury — premie z bossmenu.

---

## Five Records (`c_fiverecords`)

| Element | Stan |
|---|---|
| Kurs paczkowy | tak (400–800$ → **80–150$**) |
| Sklep klienta | **nie** |
| `depositProducts` | **nie** |
| Faktury MDT | **nie** |

Firma **logistyczna** — tylko kurs + paycheck `c_*` + premie.

---

## Garaże — buyback i sprzedaż P2P

| Mechanizm | Obecnie | Docelowo |
|---|---|---|
| Sprzedaż dealerowi (`SellVehiclePercent`) | 65% ceny zakupu | **55–65%** (bez zmian lub −5%) |
| Sprzedaż gracz → gracz | dowolna cena; sprzedawca dostaje 100% | **bez limitu ceny**; **podatek 2% → society DOJ**; sprzedawca **98%** ceny |
| Współwłaściciel (`ManageCoOwnerPrice`) | 5 000$ | **1 500–3 000$** |
| `prevent_sell` (limited cars) | blokada | zachować |

**P2P — flow podatku:** kupujący płaci uzgodnioną cenę; przy `acceptSellVehicle` odliczyć `math.ceil(price * 0.02)` do DOJ (`BabiczBossMenu:DepositMoney`), sprzedawcy wpłacić resztę na bank. W UI potwierdzenia pokazać kwotę netto i podatek.

---

## Napad truckera (poza progresją v2)

| Parametr | Obecnie (komentarz `rage_heists/Config.lua`) | Docelowo (jeśli zostanie) |
|---|---|---|
| Łup | 20 000–50 000$ | **8 000–18 000$** (tier Fleeca/Pacific) |
| Gracze | 1–3, mnożnik do ×1.8 | bez zmian struktury |
| Progresja 14 napadów | **nie wchodzi** | osobny heist / event |

**Status:** poza główną progresją; rescale lub wyłączenie do decyzji właściciela.

---

## Rico — klucz i dark shop

Klucz do grupy Rico w Dark Shop: **50 000$** → docelowo **3 000–6 000$**. Zadanie `Tasks/Rico` — bricki pod `-1372, -310` (flow crime, bez osobnej tabeli wypłat w v2).

---

## Siłownia — sklep (`GymShop`)

| Item | Obecnie | Docelowo |
|---|---:|---:|
| protein_shake | 50$ | 25–40$ |
| sportlunch | 60$ | 30–45$ |
| junkenergy | 100$ | 40–60$ |

Karnety — sekcja Gym w kosztach życia. Sklep = opcjonalny boost statów, nie źródło zarobku.

---

## Premie bossmenu — implementacja

Koncepcja tierów — § Premie tygodniowe. **Kod obecnie:** `BabiczBossMenu:sendSalary` — dowolna kwota z society, **bez capu tygodniowego**.

| Wymaganie wdrożeniowe | Opis |
|---|---|
| Tier z `duty_time` | mapowanie godzin/tydz. → max premia (tabela v2) |
| Cap **10 000$/os./tydz.** | hard limit w `sendSalary` |
| Reset tygodniowy | licznik premii per employee per tydzień |
| Audit log | Discord webhook (już częściowo) |

---

## Systemowe

| Element | Obecnie | Docelowo |
|---|---|---|
| `StartingAccountMoney` (bank) | 2 500$ | **1 500–2 500$** (bez zmian lub −1000) |
| `rage_multicharacter` `OnStart` — gotówka | 5 000$ (`money` item) | **1 500–3 000$** (łącznie z bankiem ≤ ~5k startu) |
| `job_grades.salary` (SQL) | ignorowane przez paycheck | **wyłączyć** lub ustawić 0; paycheck = jedyne źródło |
| LSSD | job `police`, unit `bcso` | **ten sam paycheck** co LSPD (672–700$/h) |
| Job center UI `salary` | 6 000–26 000$ marketing | zsynchronizować z real $/h v2 |
| Przelewy P2P (`rage_banking`, `lb-phone`) | odbiorca dostaje pełną kwotę | **redystrybucja**, nie forma zarobku — limity w § Telefon |
| Komendy admin (`/giveaccountmoney`) | dowolna kwota | **poza ekonomią** — tylko staff |

---

## Plan testów po deployu

| Test | Metryka | Cel |
|---|---|---|
| Side job 6 h | $/h | 500–650$ (z zasiłkiem) |
| Frakcja spokojna 6 h | $/h | 650–750$ bez mandatów |
| Firma BS — kurs + sprzedaż | $/h pracownik | 700–1000$ |
| Crime mix 6 h | $/h po praniu | 700–1200$ |
| Kasyno 1 h | Δ saldo | ujemne EV |
| KQ weed 1 cykl | $/h ekwiwalent | ≤ crime typowy |
| 10 vs 50 online | mandaty/h LSPD | brak runaway society |

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

## Eventy sezonowe (poza stałą ekonomią)

| Event | Plik | Zarobek | Status v2 |
|---|---|---|---|
| Wielkanoc / znajdźki (`rage_znajdzki`) | sprzedaż jaj 150–200$/szt. | incydentalny | **Poza scope** — włączać tylko na event; nie w `server.cfg` |
| Dynie (`BabiczPumpkins`) | 100$/dynia | incydentalny | **Poza scope** — event sezonowy |
| Kaucje zwrot (side joby) | `BabiczJobs_sv.lua` | refund, nie netto | opisane w § Prace dorywcze |

---

## Audyt form zarobku (kompletność)

Przegląd codebase vs ten dokument (czerwiec 2026). **Wszystkie stałe źródła dochodu gracza są opisane** w sekcjach powyżej lub oznaczone jako poza scope.

| Kategoria | Źródła | Sekcja v2 |
|---|---|---|
| **Paycheck** | zasiłek, frakcje, firmy `c_*` | § Paycheck |
| **Side joby** | miner, tailor, slaughterer, lumberjack, hunter, beach_vendor | § Prace dorywcze |
| **KQ** | deliveries, powerwashing, trucker | § Prace dorywcze |
| **Frakcje** | minutówka, MDT 25–40%, EMS kursy, bossmenu premia/wypłata | § Frakcje, § Premie |
| **EMS dodatkowe** | `ambulance_localTasks`, `BabiczHospitality` | § EMS |
| **Firmy** | kursy paczkowe, paycheck, bossmenu | § Firmy |
| **Mechanik** | tuning 10%, MDT 40% | § Mechanik |
| **Crime** | drugsales, pralnia, lombard, KQ weed → sprzedaż | § Crime, § Narkotyki |
| **Napady aktywne** | NPC, ATM×2, kasetka, sejf, Fleeca | § Napady #1–9 |
| **Napady** | Pacific (loot do implementacji), 7 planowanych, trucker poza progresją | § Napady |
| **Zadania** | Fernando ×4, Rico bricki | § Zadania |
| **Hazard** | kasyno, zdrapki, kręgle | § Kasyno, § Zdrapki |
| **Garaże** | buyback dealer, sprzedaż P2P (+2% DOJ) | § Garaże |
| **Nieruchomości** | czynsz dla wynajmującego | § Mieszkania |
| **Start** | bank ESX + gotówka multichar | § Systemowe |
| **Redystrybucja** | banking, telefon, trade ESX | § Systemowe (nie grind) |

| Wykluczone świadomie | Powód |
|---|---|
| Śmieciarz | brak payout w kodzie |
| Tracker / boosting | brak payout |
| `inspir` heist | nie w `fxmanifest` / `server.cfg` |
| Eventy sezonowe | krótkotrwałe, poza balansem |
| Admin / bot | staff |
| Więzienie | tylko skrócenie wyroku |

**Luka kod ↔ dokument:** Bank Pacific — karta v2 gotowa, **wypłata w server do dopisania** przy wdrożeniu napadu.

---

## Zakres dokumentu

| Moduł | Status w v2 |
|---|---|
| Paycheck, side joby, frakcje, firmy, crime | Opisane |
| 14 napadów | Opisane (7 aktywnych + 7 planowanych) |
| Narkotyki, lombard, pralnia, KQ weed produkcja | Opisane |
| Pojazdy, sklepy 24/7, paliwo, wygląd | Opisane |
| Bronie: BabiczAmmunation + market Ammunation | Opisane |
| Dark Shop, kluby, Casino VIP, sklep techniczny | Opisane |
| Kasyno (`pickle_casinos`) | Opisane |
| Telefon (`lb-phone`) | Opisane |
| Mieszkania (segmenty, opłaty, bills) | Opisane |
| Siłownia (karnety + GymShop) | Opisane |
| Ubezpieczenie EMS, respawn | Opisane |
| FIB, mechanik (pełna analiza), hunter, Fernando | Opisane |
| Firmy — kursy, sklepy, Blazing Tattoo, Five Records | Opisane |
| Beach vendor | Opisane |
| Garaże P2P (+2% DOJ), buyback, współwłaściciel | Opisane |
| Audyt form zarobku | Opisane |
| Eventy sezonowe, kręgle, wynajem landlord | Opisane / poza scope |
| Start multichar (gotówka + bank) | Opisane |
| Kurs lawetą mechanika | Planowany (implementacja) |
| MDT police (230) + EMS (76) + mechanik (8) | Gotowe (wdrożenie) |
| Zdrapki, automaty, platetape | Opisane |
| Premie bossmenu — spec implementacji | Opisane |
| Plan testów po deployu | Opisane |
| Śmieciarz | **Poza scope** |
| Napad truckera | Poza progresją (karta rescale) |
| `vehicles.json` / qs-housing DB | Poza scope (właściciel) |

**Napad truckera** w komentarzu `rage_heists/Config.lua` nie wchodzi w progresję v2 — do ewentualnego dodania osobno.

---

## Status decyzji

| Temat | Status |
|---|---|
| Zasiłek 60$/h | Ustalone |
| Kotwica side job 500–600$/h | Ustalone |
| Model wypłat: minutówka + interakcje + premia tygodniowa | Ustalone |
| Minutówka: LSPD/FIB 672–700; EMS 460–500; DOJ 600–632 $/h | Ustalone |
| Spokojna zmiana frakcji > side job (~650–750 vs ~500–600) | Ustalone |
| Cooldowny napadów — osobny per typ napadu | Ustalone |
| Endgame aut: bulk 300–700k, perełki do 1.2 mln | Ustalone |
| Koszty życia: 80–115$/h @ 500; żywienie 45$/h | Ustalone |
| Skala typowy / aktywna sesja | Ustalone |
| DOJ: wyższa minutówka + podatki society | Ustalone |
| Mandaty i usługi — widełki ogólne | Ustalone |
| Mandaty MDT — pełna tabela (230 wykroczeń) | Ustalone |
| Premie tygodniowe (~36 h/tydz., max 10k) | Ustalone |
| Progresja 14 napadów (pełne karty) | Ustalone |
| Narkotyki — tabela per item | Ustalone |
| Lombard — pełna tabela | Ustalone |
| Narzędzia napadów | Ustalone |
| Pojazdy — segmenty i limity | Ustalone |
| Zdrapki — ceny i EV | Ustalone |
| Firmy — koszty operacyjne (świadomie wyższe) | Ustalone |
| Kursy firmowe 80–150$/pkg + cooldown 30–45 min | Ustalone |
| depositProducts 150% → 100–110% | Ustalone |
| Mechanik — tuning split 35/10/10 | Ustalone |
| Mechanik — kurs lawetą (przyszłość) | Planowany |
| Kasyno — MaxWager i chipy po deflacji | Ustalone |
| Dark Shop + dual Ammunation | Ustalone |
| Sprzedaż P2P aut — podatek **2% DOJ**, sprzedawca 98% | Ustalone |
| Beach vendor poza job center | Ustalone |
| Premie bossmenu — enforce tier + cap 10k | Ustalone (implementacja) |
| MDT EMS 76 pozycji | Ustalone |
| Mieszkania — segmenty cenowe | Ustalone |
| Hipoteka (5–8% / 30–60 min) | Ustalone |
| Czynsz co 2 tygodnie (0,6–1,6% wartości) | Ustalone |
| Miękki limit mandatów → society | **Odrzucone** — nie wdrażać |
| Śmieciarz | **Poza scope** — nie w job center, ignorować w v2 |
| Napady planowane (7 szt.) | Implementacja |
| `vehicles.json` — konwersja per model | Poza scope dokumentu (właściciel) |
| qs-housing DB — ceny per lokalizacja | Poza scope dokumentu (właściciel) |

---

## Podsumowanie

Dokument obejmuje **pełny zakres ekonomii serwera**: zarobki, crime, frakcje, koszty życia, nieruchomości, utrzymanie pojazdów i moduły pomocnicze. Kluczowe założenia:

- Kotwica side job **500–600$/h**; zasiłek 60$/h.
- Minutówka: LSPD/FIB 672–700$/h; EMS 460–500$/h; DOJ 600–632$/h.
- Spokojna zmiana frakcji ~650–750$/h.
- Koszty życia @ 500$/h: **80–115$/h** (16–22%); żywienie **45$/h** (9%).
- Endgame aut: większość 300–700k; perełki do 1.2 mln.
- Cooldown **per gracz per typ napadu** — rotacja crime.
- MDT — **230** police + **76** EMS + **8** mechanik w pliku wdrożeniowym.
- Kasyno, Dark Shop, telefon, P2P auta (podatek 2% DOJ) — w scope v2.
- Audyt form zarobku — kompletny; eventy sezonowe poza stałą ekonomią.
- Zdrapki — rozrywka, ujemne EV, max wygrana ~8000$.
- Firmy — **wyższe koszty operacyjne** (składniki 40–65%, DOJ 10%).
- Czynsz qs-housing — **co 2 tygodnie** (0,6–1,6% wartości).
- 14 napadów — pełne karty w dokumencie.
- Endgame auto max 1.2 mln; endgame dom max ~800k.
- 7 napadów planowanych — implementacja.

[`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md)
