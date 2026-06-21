# RageCity — docelowa ekonomia serwera (v2)

Dokument **dla zespołu** (design, balans, progresja): **docelowy stan** ekonomii po przeskalowaniu — widełki zarobków, kosztów, progresji i zasad balansu. Opisany logicznie, bez szczegółów plików.

**Wdrożenie techniczne** (co, gdzie, jak zmienić w kodzie): [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md).

**Deploy:** wszystkie zmiany wchodzą **jednocześnie** w jednym release — częściowe wdrożenie psuje balans.

---

## Decyzje przed deployem (zamknięte)

Wszystkie punkty poniżej są **ustalone** — wdrożenie techniczne: [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md).

| Temat | Decyzja |
|---|---|
| Start postaci | Bank **2500$** + gotówka **1500$** (łącznie **4000$**) |
| `job_grades.salary` (SQL) | **0** dla wszystkich grade — paycheck = jedyne źródło wypłaty |
| `towVehicle` (mechanik → parking PD) | **0$** — narzędzie frakcyjne; zarobek z holowania tylko przez **fakturę MDT** (80$/km, 40% FP) |
| `SellVehiclePercent` (buyback u dealera) | **65%** (bez zmian) |
| `ManageCoOwnerPrice` | **2000$** |
| Pralnia ręczna (trasa vana) | **350–900$**/punkt (zamiast 3000–7500$); strata 20–30% — bez capu i bez CD kursu |
| Sejf sklepu — wymagania | **`stethoscope`** (durability −15); CD per gracz **15–20 min** (kod: 60 min → `900000` ms) |
| Local dispatch | Częsty generator scen; uzupełnienie minutówki wg modelu § LSPD (nie druga pensja) |
| qs-housing config | `CreditEq` **0.06**, `CreditTime` **45 min**, czynsz **1%** co **14 dni** |
| MDT `data.ts` | Mock dev-only — produkcja czyta `rage_mdt/Config.lua` |
| `platetape.lua` / lombard animacja | **Bez zmian** (900–1280 ms) |
| KQ weed — yields / czasy | Patrz § KQ weed — tabela per odmiana (wdrożenie w `config.strains.lua`) |

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
| Mechanik (`mechanic`) | 168–175$ | 672–700$ | on duty |

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

### Mandaty LSPD / LSSD / FIB — realia patrolu

W praktyce mandaty **nie są tak częste**, jak sugerowałyby same widełki MDT:

- W patrolu zwykle są **dwaj funkcjonariusze**, ale mandat / wyrok wystawia **jeden FP** (wystawiający). Drugi asystuje — **nie dostaje** udziału z grzywny.
- Udział wystawiającego: **25%** kwoty grzywny → przy średnim mandacie ~100$ to **~25$** na wystawiającego.
- Spokojna zmiana to często **0–1 mandat na patrol na godzinę**; więcej dopiero przy dużej aktywności crime i eventach.
- **Minutówka (~680$/h) to główne źródło dochodu PD** — mandaty i local dispatch to uzupełnienie, nie druga pensja.

### Scenariusze (1 h)

| Online | Frakcja | Minutówka | Interakcje | Razem $/h |
|---:|---|---:|---:|---:|
| ~10 | LSPD, 0 mandatów | 680 | 0 | 680 |
| ~10 | LSPD, 1 mandat (patrol 2 os.) | 680 | ~25 | ~705 |
| ~10 | EMS, 3 kursy NPC | 480 | 240–375 | 720–855 |
| ~10 | EMS, 3 kursy + 1 revive | 480 | 320–455 | 800–935 |
| ~10 | DOJ, 0 spraw | 620 | 0 | 620 |
| ~10 | DOJ, 1 sprawa | 620 | 50–125 | 670–745 |
| ~25 | LSPD, 2–3 mandaty (cały serwer) | 680 | ~50–75 | ~730–755 |
| ~50 | LSPD, aktywna sesja + dispatch | 680 | 100–300 | ~780–980 |
| ~50 | EMS, aktywna sesja | 480 | 600–1100 | 1080–1580 |
| ~50 | DOJ, ~4 sprawy | 620 | 200–500 | 820–1120 |

| Scenariusz | Minutówka | Interakcje | Razem $/h |
|---|---:|---:|---:|
| LSPD, patrol 2 os., 1 mandat | 680 | ~25 (1× wystawiający) | ~705 |
| FIB, 1 mandat | 680 | ~25 | ~705 |
| EMS, 6× revive (~80$ udziału) | 480 | ~480 | ~960 |
| EMS, 3 kursy NPC (~80$ udziału) | 480 | ~240 | ~720 |
| DOJ, 2 sprawy × ~100$ (25%) | 620 | ~200 | ~820 |

### LSPD / LSSD / FIB

| Element | Wartość |
|---|---|
| Minutówka | 672–700$/h |
| Mandaty / wyroki MDT | 25% kwoty grzywny |
| Lokalne dispatch (NPC) | 550–1150$ / pełna interwencja → kieszeń |
| Laboratorium — depozyt | wartość dowodów → society police |
| Konwoje, sprawy | okazjonalnie |

#### Lokalne dispatch (interwencje NPC)

Generator scen na mapie — uzupełnienie minutówki LSPD. **Gotówka do kieszeni** funkcjonariusza (nie society).

| Scenariusz | Kod | Waga |
|---|---|---:|
| Podejrzana transakcja | 10-31 | 25% |
| Kradzież pojazdu | 10-35 | 25% |
| Bójka uliczna | 10-10 | 25% |
| Zakłócenie porządku | 10-16 | 25% |

| Wynik interwencji | Wypłata $ |
|---|---:|
| Pełny sukces (areszt + przeszukanie / rozdzielenie + areszt) | **550–1150$** (baza 300–900 + 150 areszt + 100 przeszukanie) |
| Częściowy (×0.55) | **165–495$** |
| Deeskalacja bez aresztu (×0.4) | **120–360$** |

| Parametr | Wartość |
|---|---|
| Spawns (generator) | **Częsty** — min. **4 min** między scenami; max **3** aktywne; po realnym dispatch **5 min** pauzy |
| Ukończenia per FP | **~0,3–0,6/h** przy 25–50 PD online (konkurencja); **~1–2/h** przy ≤10 PD online |
| Średnia ważona wypłata | **~400–500$** (pełny sukces ~35%, częściowy ~40%, deeskalacja ~25%) |
| Uzupełnienie minutówki | **+120–300$/h** typowo; do **+400–800$/h** przy małej liczbie PD i aktywnym reagowaniu |
| Loot przeszukania | narkotyki (deal), alkohol (intox) — nie $ |

**Dlaczego spawn częsty ≠ wysoki $/h:** generator tworzy sceny regularnie, ale wiele kończy się deeskalacją, przejmuje je inny patrol, albo FP nie zdąży — minutówka (~680$/h) pozostaje **głównym** dochodem PD; dispatch to uzupełnienie za aktywne reagowanie (zgodne z tabelą scenariuszy § Frakcje, wiersz ~50 online: interakcje **100–300$/h**).

#### Laboratorium policyjne (depozyt narkotyków)

3 lokacje (Mission Row, Davis, Vespucci). Funkcjonariusz **nie dostaje $** — wartość dowodów → **society police**.

| Item (przykład) | Wartość depozytu / szt. |
|---|---:|
| joint | 23–45$ |
| weed_packed | 73–91$ |
| coke_packed | 129–162$ |
| heroin_packed | 164–182$ |
| meth_packed | 137–171$ |

Skala: ~**30%** ceny rynkowej narkotyku. Test THC: 8 s, bez wypłaty.

### EMS

| Element | Wartość |
|---|---|
| Minutówka | 460–500$/h |
| Revive / leczenie | 25% kwoty |
| Kurs EMS → lokalny medyk | 200–500$ / kurs, 2–4/h solo |

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

Szczegółowa tabela — sekcja **Wyroki i mandaty MDT** poniżej.

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

### Konwersja `vehicles.json` (deploy)

**Bloker** — bez rescale progresja auta nie ma sensu po deflacji zarobków.

1. **Skrypt konwersji** (Python/Lua): `newPrice = clamp(round(oldPrice × factor), segmentMin, segmentMax)` per typ pojazdu.
2. **Domyślny mnożnik:** `×0,08` dla większości modeli osobowych; cap **1 200 000$** (perełki endgame).
3. **Segmenty:** zsynchronizować z tabelą powyżej (1000–1,2M osobowe; moto max 800k; łodzie/heli/plane wg typów).
4. **`VehiclePriceConfig.types`:** min/max jak w bloku `types` powyżej.
5. **Weryfikacja po skrypcie:** ręcznie 10 modeli (najtańszy, starter, mid, top, outlier); min ≥ 1000$, max ≤ 1,2M dla automobile.
6. **Po konwersji aut:** tuning flaty **×0,15**; `shopPricePercent` performance **2%** (zakres 1,5–2,5%).

Szczegóły techniczne: [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §21, §21a.

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
| Trucker (KQ / RS Haul) | 540–790$ | 60$ | 600–850$ |

*Wiersz „Trucker” = legalna praca KQ. **Napad truckera** (crime, dostawa skradzionej ciężarówki) = **3500–8000$**/kurs — §9b.*

**Ważne:** KQ Deliveries i Powerwashing — obecnie ~20k–30k+/h; wymagają ~40× obniżki `payPerMinute` **oraz** obniżenia bonusów (patrz tabelę poniżej), inaczej kotwica 500$/h nie będzie respektowana.

### KQ — docelowe `payPerMinute` i bonusy

**`payPerMinute` per job (Deliveries):** mapowanie z obecnych 330–500 → **7 / 8 / 9 / 10 / 11** $/min (≈420–660$/h aktywności + zasiłek 60$/h).

**Bonusy (cap łączny ~15% bazy):**

| Parametr | Obecnie (przykład) | Docelowo |
|---|---|---|
| `fiveStarBonus` | 10% | **3%** |
| `fourStarBonus` | 5% | **2%** |
| `vehicleCare` | 5% | **2%** |
| `teamWorkBonuses[2]` | 80% | **10%** |
| `salary_bonus_*` (level upgrades) | 5–30% | **2–8%** (max tier) |
| `maxVehicleDamagePenalty` | 1000$ | **150–250$** |
| `missingVehiclePenalty` | 1200$ | **200–300$** |

Te same wartości `teamWorkBonuses` dla **kq_powerwashing** (max +10% do nagrody kontraktu).

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

**Szacunki $/h w kartach napadów** poniżej przyjmują **~25% straty prania** jako średnią (między ręczną 20–30% a automatyczną 35–45% — patrz § Pralnia).

---

## Napady — progresja

### Kolejność (od najłatwiejszego)

NPC → Bankomat (karta) → Boosting → Bankomat (hack) → Kasetka → Tracker → Sejf sklepu → Jubiler → Fleeca → **Trucker (ciężarówka)** → Lombard → Jacht → Pacific → Humane Labs → Cayo Perico

> **Uwaga:** **KQ Trucker** (RS Haul, job center) to **legalna praca dorywcza** (~600–850$/h) — to nie to samo co **napad truckera** (crime, karta §9b poniżej).

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
| 9b | **Trucker (ciężarówka)** | 1 | 3 | 3500$ | 8000$ | black_money | Planowany |
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
| Szybkie (solo) | NPC (**3 min** CD), ATM karta, boosting | 3–15 min |
| Średnie | ATM hack, kasetka, tracker, sejf sklepu | 8–20 min |
| Grupowe | Jubiler, Fleeca, **trucker (ciężarówka)**, lombard | 30–45 min |
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
    trucker           = { [1] = 1.0,  [2] = 1.15, [3] = 1.25 },
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
| Cooldown | **3 min per gracz** (obecnie 15 s — zbyt krótko, umożliwia farm) |
| Czas akcji | ~30–60 s |
| Wymagania | Broń zwiększa szansę (80% vs 20% bez broni) |
| Sprzedaż lootu | Lombard / dark_lombard (Lester ×0.95) |
| Szacunek $/h (solo, po CD 3 min) | **~400–700$/h** gotówka + biżuteria — entry-level w **miksie** crime, nie solo farm |

**Anty-farm:** bez wydłużenia cooldownu napad na NPC przy loot 25–150$ daje **1500–3500$/h** — powyżej widełek crime. Cooldown **3 min** + rotacja na ATM / kasetkę / narkotyki utrzymuje NPC w roli wprowadzenia, nie głównego źródła.

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
| Status | Planowany (`rage_heists` lub quest **Carlos — Boosting**) |
| Gracze | 1 (solo) |
| Mnożnik | ×1.0 |
| Łup | 350–900$ gotówki |
| Typ nagrody | Czysta gotówka |
| Policja min. | 1 |
| Cooldown | ~15 min |
| Czas akcji | ~10–20 min (szukanie auta + dostawa) |
| Wymagania | Brak thermite/hack; opcjonalnie lockpick |
| Mechanika | Otrzymaj wskazane auto w mieście → dostarcz do punktu |
| Quest | Carlos zad. 2 — ten sam tier wypłat; tutorial przed powtarzalnym heistem |

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
| Quest | Moris zad. 1 i 3 — kasetka w ramach fabuły; loot wg tabeli, bonus quest osobno |

**Loot na osobę:**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 250–600$ | 250–600$ |
| 2 | ×1.15 | 287–690$ | 143–345$ |

---

### 6. Tracker

| Parametr | Wartość |
|---|---|
| Status | Planowany — `[rage]/BabiczTracker` + questy **Carlos** (tracker, przemyt) |
| Gracze | 1–2 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.10 |
| Łup | 500–1400$ black_money |
| Typ nagrody | black_money |
| Policja min. | 1 |
| Cooldown | ~20 min |
| Czas akcji | ~15–25 min |
| Wymagania | Opłata startowa ~200–400$ (heist); **brak opłaty** w questach Carlosa |
| Mechanika | Weź auto z nadajnikiem → ucieczka przed LSPD przez określony czas → punkt zrzutu |
| Uwagi | Obecny config trackera: 500–5000$, opłata 500$ — rescale; przemyt Carlosa: **900–1600$**, cooldown 90 min |
| Quest | Carlos zad. 3–4 — wprowadzenie przed powtarzalnym heistem |

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
| Cooldown | **15 min** per gracz + **10 min** globalny serwerowy |
| Czas akcji | 180–240 s |
| Wymagania | **`stethoscope`** (opcjonalnie — zużywa durability −15; brak itemu = brak sinku) |
| Uwagi | Wyraźnie wyższy próg niż kasetka; **usunąć** `rewardBonus` Sandy (+5000$) |

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
| Cooldown | **15 min** per gracz per bank (kod: 2 h → `900000` ms) |
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

### 9b. Napad truckera (dostawa ciężarówki)

Kradzież ciężarówki z ładunkiem → dostawa pod wskazany punkt. **Crime**, nie mylić z jobem KQ Trucker (RS Haul).

| Parametr | Wartość |
|---|---|
| Status | Planowany — szkic lootu w `rage_heists/Config.lua`; **flow do implementacji** |
| Gracze | 1–3 |
| Mnożnik | 1 os. ×1.0 · 2 os. ×1.15 · 3 os. ×1.25 |
| **Wypłata (łup łączny)** | **3500–8000$** `black_money` |
| Typ nagrody | black_money (wymaga prania) |
| Policja min. | 2 |
| Cooldown | **35–45 min per gracz** |
| Czas akcji | ~15–25 min (znajdź ciężarówkę + dostawa) |
| Wymagania | Broń opcjonalnie; umiejętność jazdy ciężarówką |
| Pozycja w progresji | Po Fleeca / równolegle z Lombard — tier grupowy, krótszy niż bank |

**Loot na osobę (brutto, przed praniem):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 3500–8000$ | 3500–8000$ |
| 2 | ×1.15 | 4025–9200$ | 2012–4600$ |
| 3 | ×1.25 | 4375–10000$ | 1458–3333$ |

*Po praniu (25% straty), solo, baza 5500$: ok. **4125$** czystego. Przy CD ~45 min + ~20 min akcji ≈ **~3400$/h** efektywnego — mieści się w widełkach crime (700–1200$/h typowo w miksie z innymi napadami).*

**Balans:** niższy próg niż Fleeca (4500–12k, min. 2 os. + hack), wyższy niż tracker (500–1400). Solo możliwe, ale mniej opłacalne na osobę niż duo.

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
| Quest | Moris zad. 2 — napad fabularny; biżuteria „mamy Morisa” bez itemu w ekwipunku |

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
| Status | **Aktywny gameplay, brak wypłaty lootu w kodzie** — ekonomia ustalona poniżej; implementacja w [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §13 |
| Gracze | 4–6 (docelowo `minGroup = 4`) |
| Mnożnik | 4 os. ×1.0 · 5 os. ×1.2 · 6 os. ×1.35 |
| Łup łączny | 35000–90000$ |
| Skład łupu | **~70%** `black_money` + **~30%** biżuteria (wartość skupu w lombardzie) |
| Policja min. | 5 |
| Cooldown | **60–90 min per gracz** (osobny licznik, tier endgame) |
| Czas akcji | ~20–40 min |
| Wymagania | Hackingdevice, hackingtablet, thermite, broń |
| Po praniu (~25% straty) | ok. **6500–17000$** czystego na osobę przy 4 napastnikach |

**Loot na osobę (brutto, przed praniem):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 4 | ×1.0 | 35000–90000$ | 8750–22500$ |
| 5 | ×1.2 | 42000–108000$ | 8400–21600$ |
| 6 | ×1.35 | 47250–121500$ | 7875–20250$ |

**Przykład podziału łupu (baza 60000$, 4 os., ×1.0):**

| Składnik | Kwota | Uwagi |
|---|---:|---|
| black_money | 42000$ | 10500$/os. → ~7875$ czystego po praniu |
| Biżuteria (skup) | 18000$ | np. 4× necklace_gold + diamenty; ~4500$/os. po sprzedaży |
| **Razem na osobę** | 15000$ brutto | ~**11800$** efektywnego po praniu + skup |

**Implementacja (docelowa):** po zakończeniu flow w `BankPacific/server` — `addAccountMoney("black_money", …)` + itemy z puli biżuterii v2; mnożnik z `Config.AttackerMultipliers.bank_pacific`; brak wypłaty = bug do naprawy przed deployem.

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
| Ręczna trasa | 20–30% | **350–900$/punkt** (obecnie 3000–7500$) |

**Ręczna trasa — rola:** konwersja **brudnej gotówki z napadów**, nie osobny grind. ~24 lokacji × jednorazowo na kurs; przy pełnym obiegu teoretycznie ~8–21k black przed stratą — w praktyce limituje ilość `black_money` z crime, nie liczba punktów sama w sobie.

**Kalkulacje napadów** w dokumencie używają **~25% straty prania** jako średniej (patrz § Crime).

---

## Narkotyki

Docelowy zarobek: **800–1500$/h** typowo | **1500–2200$/h** aktywna sesja — **górna połowa** widełek crime (miks 700–1200$/h); wyższe ryzyko policji, setup i rotacja sprzedaży.

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

### Segmenty cenowe i ekstremy (po rescale DB)

| Segment | Cena | Czas @ 500$/h | Uwagi |
|---|---:|---:|---|
| **Najtańsza melina** (garaż / komórka / slums) | **15 000–20 000$** | 30–40 h | entry housing |
| Mieszkanie starter | 25 000–50 000$ | 50–100 h | pierwsza nieruchomość |
| Mieszkanie standard | 50 000–100 000$ | 100–200 h | porównywalne z dobrym autem |
| Dom średni | 100 000–200 000$ | 200–400 h | |
| Dom duży | 200 000–400 000$ | 400–800 h | |
| Premium / willa | 400 000–650 000$ | 800–1300 h | poniżej endgame auta (1,2 mln) |
| **Najdroższa willa** (max, rzadkie lokacje) | **650 000–800 000$** | 1300–1600 h | hard cap nieruchomości |

**Zasada:** endgame dom **nie powinien przewyższać** endgame auta osobowego (1,2 mln).

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
| `CreditEq` (spłata %) | 30% co 5 min | **6% (`0.06`) co 45 min** |
| Czynsz (wynajem) | co 5 min / miesięcznie | **1% wartości nieruchomości co 14 dni** (real time) |
| Dochód wynajmującego | czynsz → konto bankowe właściciela (`qs-housing`) | **bez zmian mechanizmu**; po rescale czynszu — pasywny dochód, nie grind |
| Max domów na gracza | 5 | 5 |
| Konwersja DB | — | `price = LEAST(FLOOR(price × 0.08), 800000)` + ręczna weryfikacja melin/willi |

Obecny model (30% co 5 min) jest zbyt agresywny po deflacji — wymaga zmiany w config **i audytu DB** (szczegóły techniczne: [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §25).

**Problem po czystce bez zmiany qs-housing:** właściciel nieruchomości dostaje **pasywny dochód** (czynsz → konto bankowe) w skali starej ekonomii — szybkie bogacenie się bez grindu. Po rescale czynsz co 14 dni (1% wartości) = np. willa 700k → **7000$ / 2 tyg.** dla wynajmującego (~25$/h ekwiwalent), nie tysiące co kilka minut.

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
| Współwłaściciel garażu | 5000$ | **2000$** |

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

Po deflacji `vehicles.json`: obniżyć **flaty ×0,15** oraz **shopPricePercent** performance z **5% → 2%** (widełki 1,5–2,5%).

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
| **Tow** (`towVehicle` w `BabiczMechanic_sv`) | 10 s → auto do impound · **0$** dla mechanika — **decyzja:** narzędzie frakcyjne, nie zarobek |
| **Flatbed** (`BabiczFlatbed_*`) | załaduj/rozładuj · **0$** · `flatbed3` · mechanic + police on duty |
| **MDT „holowanie za km”** | faktura **80$/km** — mechanik wystawiający dostaje **40%** (`finePercentForPlayer`) |

Odholowanie z `rage_garages` (odbiór auta): docelowo **300–700$** (osobna sekcja kosztów życia).

### Kurs paczkowy LSC (obecny)

Identyczny moduł co Burger Shot: port → LSC HQ · **400–800$/paczka** · 50% society · **brak cooldownu**.

**Docelowo:** **80–150$/paczka** + cooldown **30–45 min** (jak pozostałe firmy). Tematycznie słaby dla mechanika — docelowo zastąpić lub uzupełnić kursem lawety.

### Kurs lawetą — zepsute auta lokalne (docelowy)

**Zastępuje** paczkowy kurs LSC jako główna aktywność mechanika on duty. Mechanik jedzie `flatbed3` do jednej z **13 lokacji** `C.LocalMechanic` — na miejscu **zepsute auto NPC** do załadunku i dowozu do LSC.

| Parametr | Docelowo |
|---|---|
| Start | garaż LSC / sub-shop |
| Cel | losowa lokacja lokalnego mechanika (lista w `BabiczMechanic_shared.lua`) |
| Pojazd misji | NPC „awaria” (engine/body low) — **nie** naprawa u NPC, tylko holowanie |
| Akcja | flatbed: załaduj → dowieź do LSC → rozładuj |
| Nagroda | **150–350$** clean + **50%** → society `mechanic` |
| Cooldown | **15–20 min** / gracz |
| Wymagania | on duty, flatbed w garażu frakcji |
| Szacunek | 3–4 kursy/h → **450–1400$/h** + minutówka + tuning/MDT |

Kurs paczkowy LSC (400–800$/pkg) — **wyłączyć lub obniżyć do 80–150$** po wdrożeniu kursu lawetą.

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

## Zadania Moris — progresja crime

Trzy linie questów u **Moris** (`Tasks/Moris`): **Fernando** (tutorial) → **Carlos** (mechaniki pojazdowe / narkotyki) → **Moris** (haracz, fabuła, napady sklepowe). Nie zastępują napadów — **wprowadzają** w te same mechaniki co progresja 14 napadów.

| Zasada | Wartość |
|---|---|
| Rola w ekonomii | Progresja + okazjonalny dochód; **nie** główne źródło $/h po ukończeniu linii |
| Loot z napadu w trakcie questu | Wg skali v2 (kasetka, sejf, lombard); quest daje **bonus fabularny** obok |
| Cooldown questu | Osobny od cooldownu heistu — ukończenie zadania nie omija limitów powtarzalnych napadów poza questem |
| Kolejność odblokowania | Fernando → Carlos (min. poziom 2) → zadania u samego Morisa |

### Fernando — tutorial

| Zadanie | Obecnie | Docelowo |
|---|---|---|
| 1 — paczki | 250–750$ dirty / paczka | 80–200$ dirty |
| 2 — pralnia (wypłata) | 8000–12000$ czyste + 25000$ dirty do prania | 1500–2500$ czyste + 4000–6000$ dirty |
| 3 — sprzedaż weed_packed | 5000–8000$ dirty + sprzedaż | 1200–2000$ dirty + sprzedaż |
| 4 — poszukiwacz | 7000–10000$ | 1500–2500$ |
| 5 — ślad plantacji | (exp / flow) | bez osobnej wypłaty — progresja do Carlosa |
| Cooldown zad. 2 | 24 h | 24 h |

### Carlos — pojazdy i przemyt

Wymaga ukończenia linii Fernando. Zadania uczą rotacji crime: dostawy → boosting → tracker → przemyt.

| # | Zadanie | Flow (skrót) | Czas | Nagroda quest (docelowo) | Cooldown | Powiązanie z napadami |
|---:|---|---|---|---:|---:|---|
| 1 | **Rozwóz narkotyków** | 9 lokalizacji na mapie | 30–45 min | **40–80$ dirty / stop** → łącznie **360–720$** black | 45 min | Street sales / KQ weed |
| 2 | **Boosting** | Znajdź model auta w mieście → punkt A → powrót po nagrodę | 15–25 min | **350–900$** clean (tier auta; niski 350–500$, wysoki 700–900$) | 45 min | Napad #3 Boosting |
| 3 | **Tracker** | Obszar na mapie → auto z nadajnikiem → ucieczka przed LSPD → punkt A → powrót | 20–30 min | **500–1200$** black | 60 min | Napad #6 Tracker |
| 4 | **Przemyt (+ tracker)** | Zlokalizuj auto → punkt A (paczka) → punkt B (przekazanie) → ucieczka → schowaj w C → Moris | 30–45 min | **900–1600$** black | 90 min | Tracker + crime logistyka |

**Boosting (Carlos) — obecny stan kodu:** wypłaty **3200–13000$** per auto (`shared/2_Carlos.lua`) — wymaga rescale do tierów powyżej. Kara za uszkodzenie auta: **−20%** nagrody (już w logice serwera).

**Tracker / przemyt:** integracja z `[rage]/BabiczTracker` lub logika questowa — wypłaty jak napady #6, bez opłaty startowej 500$ z obecnego trackera.

### Moris — haracz i fabuła

Odblokowanie po linii Carlosa. Zadania łańcuchowe — sklep z zad. 1 wraca w zad. 3.

| # | Zadanie | Flow (skrót) | Czas | Bonus quest | Loot napadu (osobno) | Inne |
|---:|---|---|---|---:|---|---|
| 1 | **Pierwszy haracz** | Sklep XYZ → kasjer odmawia → Moris (groźba, broń?) → napad kasetka + przeszukanie → karta debetowa → wypłata | 25–40 min | **300–600$** clean | Kasetka **250–600$** (50–70% dirty); karta **400–800$** clean | Brak broni: waypoint Dark Shop **lub** pistolet od Morisa **1200–2000$** |
| 2 | **Syzyfowa robota** | Przeszukanie pedów w mieście → ucieczka + dialog → lombard → napad na lombard → „biżuteria mamy” (flavor, nie item) | 45–70 min | **500–900$** black | Lombard **600–1800$** + biżuteria skup **200–500$** ekwiwalent | Powrót bez dialogu = konieczność odnalezienia peda |
| 3 | **Śmieszne pogróżki** | Ten sam sklep co zad. 1 → napad → hack komputera → wydruk maili → Moris | 30–45 min | **400–700$** black | Kasetka **250–600$**; hack bez dodatkowego $ (dowód fabularny) | Wymaga ukończenia zad. 1 |

**Ekonomia broni w zad. 1:** sprzedaż pistoletu przez Morisa to **wejście w crime**, nie zysk gracza — cena poniżej Ammunation (8–15k), ale powyżej Dark Shop dirty.

**Szacunek $/h (aktywna sesja, z lootem napadu):**

| Linia | Typowy quest + napad | Uwagi |
|---|---:|---|
| Carlos 1 | ~500–900$/h | 9 stopów, bez policji |
| Carlos 2–3 | ~700–1100$/h | ryzyko LSPD |
| Carlos 4 | ~900–1400$/h | najdłuższy, najwyższy bonus |
| Moris 1–3 | ~600–1200$/h | jednorazowa / 24h fabuła; kasetka+lombard wg cooldownów heistów poza questem |

Po ukończeniu linii questów gracz powinien przechodzić na **rotację napadów + narkotyki** (700–1200$/h typowo) — questy nie mogą być farmione w nieskończoność z wyższym $/h niż crime.

---

## Wyroki i mandaty MDT (LSPD / LSSD / FIB / DOJ)

Katalog w `rage_mdt/Config.lua` — **230 pozycji** wspólnych dla police, fib, doj (tabela poniżej). **EMS (76)** i **mechanik (8)** — osobne katalogi faktur (sekcje poniżej, pełne listy w tym dokumencie).

### Zasady systemu wyroków

| Zasada | Wartość |
|---|---|
| Mandat vs wyrok | **Mandat** — pojedyncza grzywna (drożne wykroczenia). **Wyrok** — koszyk pozycji + więzienie (`Config.SentenceJobs`: police, fib, doj) |
| Jednostka więzienia | **1 miesiąc wyroku = 1 minuta** w UI MDT i komunikatach |
| Czas realny odsiadki | Po akceptacji wyroku: **`floor(jail × 0.3)` sekund** w więzieniu (np. 50 mies. → 15 min realnie) |
| Składanie wyroku | Pozycje z koszyka **sumują** grzywnę i `jail`; można dodać wiele przestępstw |
| Zniżka (DOJ/LSPD) | **0–30%** na grzywnę (UI MDT) |
| Ubezpieczenie EMS | **−50%** grzywny (`health_insurance`) |
| Split grzywny | **25%** na bank wystawiającego, reszta → society frakcji |
| Wykroczenia drogowe | Tylko grzywna (bez więzienia), wyjątki: DUI, ucieczka, brawura |
| Kategoria Napady | **1 wpis = 1 typ napadu** — nie łączyć z ogólnym rozbojem; zgodność z progresją 14 napadów |
| Okoliczności łagodzące | Osobna kategoria — ulgi **odejmują** z sumy (patrz tabela) |

### Skala grzywien (docelowa — wdrożenie w `Config.lua`)

| Tier | Grzywna | Przykłady |
|---|---:|---|
| Drobne wykroczenie | 30–80$ | pasy, telefon, parkowanie |
| Standard | 80–200$ | prędkość, czerwone światło, napaść |
| Poważne | 200–600$ | rozbój, włamanie, broń bez licencji |
| Ciężkie | 600–2500$ | zabójstwo w afekcie, porwanie, napad Fleeca |
| Najcięższe | 2500–8000$ | zabójstwo FFP, Cayo, Humane |

### Skala więzienia (miesiące w MDT → minuty realne ×0.3)

| Tier | Miesiące (MDT) | Realnie (~) | Przykłady |
|---|---:|---:|---|
| Wykroczenie z jail | 3–6 | 1–2 min | DUI pierwsze, ucieczka piesza |
| Standard | 5–16 | 1.5–5 min | kradzież, włamanie, broń |
| Ciężkie | 15–30 | 4.5–9 min | napad lombard, tracker |
| Bardzo ciężkie | 30–50 | 9–15 min | zabójstwo, Pacific, Cayo |

### Okoliczności łagodzące

Obecne ulgi mandatowe sięgają **−25000$** (skala sprzed deflacji). Docelowo — **tylko** pozycje z tabeli poniżej (max **−2500$** mandat, max **−60** mies. więzienia). Stare pozycje −5000 / −10000 / −25000 **usunąć** z Config.

| Typ | Obecnie (max) | Docelowo |
|---|---:|---:|
| Ulga mandatowa | −25000$ | **−250 / −500 / −750 / −1000 / −1500 / −2000 / −2500** |
| Ulga więzienna | −120 mies. | **−1 / −6 / −12 / −24 / −36 / −60** mies. |

### Taryfikator — pełna lista (LSPD / FIB / DOJ)

**Łącznie pozycji LSPD/FIB/DOJ:** 230.

#### Przestępstwa przeciwko życiu i zdrowiu

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Zabójstwo II stopnia | **2500$** | **50** |
| Zabójstwo II stopnia funkcjonariusza publicznego | **5000$** | **60** |
| Zabójstwo w afekcie (Voluntary Manslaughter) | **600$** | **22** |
| Nieumyślne spowodowanie śmierci | **600$** | **22** |
| Nieumyślne spowodowanie śmierci pojazdem (DUI) | **1500$** | **20** |
| Ciężki uszczerbek na zdrowiu | **300$** | **16** |
| Ciężki uszczerbek na zdrowiu z użyciem broni palnej | **600$** | **24** |
| Ciężki uszczerbek na zdrowiu funkcjonariusza | **600$** | **20** |
| Napaść z użyciem niebezpiecznego narzędzia | **300$** | **15** |
| Napaść (Assault) | **80$** | **6** |
| Pobicie (Battery) | **160$** | **5** |
| Pobicie z użyciem niebezpiecznego przedmiotu | **300$** | **16** |
| Napaść na funkcjonariusza publicznego | **300$** | **16** |
| Bezprawne pozbawienie wolności | **300$** | **12** |
| Porwanie dla okupu | **2500$** | **22** |
| Porwanie z użyciem broni palnej | **3000$** | **30** |
| Porwanie funkcjonariusza publicznego | **5000$** | **40** |
| Usiłowanie zabójstwa | **2500$** | **38** |
| Usiłowanie zabójstwa funkcjonariusza | **5000$** | **50** |

#### Przestępstwa przeciwko mieniu

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Kradzież małej wartości (poniżej $950) | **80$** | **6** |
| Kradzież ($950 - $5000) | **175$** | **12** |
| Kradzież wielka (powyżej $5000) | **600$** | **16** |
| Kradzież z pojazdu | **160$** | **5** |
| Kradzież tożsamości | **600$** | **12** |
| Rozbój | **600$** | **15** |
| Rozbój z użyciem broni palnej | **2500$** | **18** |
| Rozbój z użyciem broni białej | **1500$** | **15** |
| Włamanie do budynku mieszkalnego | **600$** | **15** |
| Włamanie do obiektu komercyjnego | **300$** | **12** |
| Włamanie z użyciem narzędzi | **600$** | **15** |
| Włamanie do pojazdu | **160$** | **5** |
| Kradzież pojazdu cywilnego | **600$** | **12** |
| Kradzież pojazdu uprzywilejowanego | **2000$** | **15** |
| Nielegalne posiadanie skradzionego pojazdu | **300$** | **5** |
| Wymuszenie / szantaż | **600$** | **16** |
| Wymuszenie z użyciem broni | **2500$** | **16** |
| Wymuszenie przez zorganizowaną grupę | **5000$** | **20** |
| Oszustwo (do $5000) | **175$** | **5** |
| Oszustwo (powyżej $5000) | **600$** | **12** |
| Oszustwo na szkodę instytucji publicznej | **2500$** | **15** |
| Fałszerstwo dokumentów | **300$** | **12** |
| Fałszerstwo pieniędzy / waluty | **2500$** | **15** |
| Wandalizm (szkoda poniżej $400) | **80$** | **5** |
| Wandalizm (szkoda powyżej $400) | **300$** | **12** |
| Zniszczenie mienia publicznego | **300$** | **12** |
| Zniszczenie pojazdu służbowego | **600$** | **16** |
| Pranie pieniędzy (do $50 000) | **2000$** | **16** |
| Pranie pieniędzy (powyżej $50 000) | **5000$** | **16** |
| Pranie pieniędzy przez zorganizowaną grupę | **4000$** | **20** |

#### Przestępstwa przeciwko porządkowi publicznemu

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Zakłócenie porządku publicznego | **50$** | **3** |
| Wywołanie zbiegowiska / zamieszek | **160$** | **5** |
| Napaść słowna / groźby publiczne | **80$** | **6** |
| Groźba karalna | **80$** | **5** |
| Groźba z użyciem broni | **300$** | **12** |
| Groźba wobec funkcjonariusza publicznego | **300$** | **12** |
| Stalking / nękanie | **160$** | **16** |
| Użycie broni palnej podczas przestępstwa (+do kary) | **600$** | **20** |
| Oddanie strzału podczas przestępstwa (+do kary) | **2000$** | **30** |
| Napaść na funkcjonariusza (bez obrażeń) | **300$** | **16** |
| Pobicie funkcjonariusza (z obrażeniami) | **600$** | **15** |
| Pobicie funkcjonariusza z bronią | **2500$** | **24** |
| Usiłowanie zabójstwa funkcjonariusza | **5000$** | **50** |
| Ucieczka piesza przed funkcjonariuszem | **50$** | **6** |
| Ucieczka pojazdem (niebezpieczna jazda) | **300$** | **12** |
| Ucieczka pojazdem z obrażeniami u innych | **1500$** | **15** |
| Ucieczka pojazdem ze śmiercią ofiary | **2500$** | **20** |
| Opór bierny przy zatrzymaniu | **50$** | **3** |
| Czynny opór przy zatrzymaniu | **160$** | **5** |
| Atak na funkcjonariusza podczas zatrzymania | **600$** | **15** |
| Groźby terrorystyczne | **5000$** | **20** |
| Akt terrorystyczny bez ofiar | **4000$** | **40** |
| Udział w zorganizowanej grupie przestępczej | **2500$** | **15** |
| Kierowanie grupą przestępczą | **5000$** | **20** |
| Finansowanie działalności przestępczej | **4000$** | **24** |

#### Broń palna

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Posiadanie broni bez licencji (klasa I) | **300$** | **5** |
| Posiadanie broni klasy II przez cywila | **2500$** | **16** |
| Posiadanie broni przez osobę z zakazem | **2500$** | **15** |
| Posiadanie niezarejestrowanej broni | **600$** | **12** |
| Posiadanie broni klasy III bez zezwolenia | **5000$** | **16** |
| Posiadanie broni klasy IV (materiały wybuchowe) | **5000$** | **20** |
| Posiadanie ghost gun (bez numeru seryjnego) | **2500$** | **15** |
| Posiadanie tłumika bez zezwolenia | **2000$** | **16** |
| Posiadanie magazynka powyżej 10 nabojów | **300$** | **5** |
| Open carry bez uprawnień służbowych | **175$** | **5** |
| Concealed carry bez licencji CCW | **300$** | **12** |
| Noszenie broni załadowanej w pojeździe (bez CCW) | **150$** | **5** |
| Niezgłoszenie broni podczas kontroli drogowej | **120$** | **6** |
| Sprzedaż broni bez licencji FFL | **5000$** | **15** |
| Sprzedaż broni bez weryfikacji nabywcy | **2500$** | **15** |
| Sprzedaż broni osobie nieuprawnionej | **5000$** | **16** |
| Pośrednictwo w nielegalnym obrocie bronią | **5000$** | **20** |
| Niezgłoszenie kradzieży broni w terminie 48h | **175$** | **6** |
| Wejście do Gun Free Zone z bronią (nieumyślne) | **150$** | **6** |
| Wejście do Gun Free Zone z bronią (umyślne) | **600$** | **12** |
| Wniesienie broni do sądu lub więzienia | **2500$** | **15** |
| Bezpodstawne użycie broni palnej | **300$** | **15** |

#### Narkotyki

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Posiadanie narkotyków miękkich | **190$** | **5** |
| Posiadanie narkotyków twardych | **350$** | **10** |
| Posiadanie hurtowych ilości narkotyków I | **825$** | **15** |
| Posiadanie hurtowych ilości narkotyków II | **975$** | **20** |
| Posiadanie hurtowych ilości narkotyków III | **1150$** | **25** |
| Posiadanie hurtowych ilości narkotyków IV | **1275$** | **30** |
| Posiadanie hurtowych ilości narkotyków V | **1450$** | **35** |
| Używanie narkotyków w miejscu publicznym | **30$** | — |
| Handel narkotykami miękkimi | **320$** | **10** |
| Handel narkotykami twardymi | **620$** | **15** |
| Handel hurtowy narkotykami miękkimi | **825$** | **15** |
| Handel hurtowy narkotykami twardymi | **1150$** | **25** |
| Sprzedaż narkotyków niepełnoletniemu | **2000$** | **20** |
| Produkcja narkotyków miękkich | **300$** | **8** |
| Produkcja narkotyków twardych | **800$** | **15** |
| Prowadzenie laboratorium narkotykowego | **900$** | **15** |

#### Napady

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Napad na obywatela | **200$** | **5** |
| Napad na bankomat | **350$** | **10** |
| Kradzież bankomatu | **900$** | **12** |
| Napad na kasetkę | **550$** | **12** |
| Napad tracker | **1200$** | **15** |
| Napad na sejf sklepu | **2400$** | **15** |
| Napad na jubilera | **4500$** | **22** |
| Napad na sejf banku | **5500$** | **20** |
| Napad na lombard | **4200$** | **18** |
| Napad na jacht | **5500$** | **28** |
| Napad na bank Pacific | **7000$** | **30** |
| Napad na Humane Labs | **7500$** | **35** |
| Napad na posesję Cayo Perico | **8000$** | **40** |

#### Przestępstwa przeciwko wymiarowi sprawiedliwości

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Krzywoprzysięstwo | **300$** | **16** |
| Fałszywe zeznania w sprawie o zabójstwo | **2500$** | **20** |
| Utrudnianie postępowania karnego | **300$** | **12** |
| Niszczenie lub ukrywanie dowodów | **600$** | **15** |
| Zastraszanie świadków / ławy przysięgłych | **2500$** | **15** |
| Przekupstwo świadka | **2500$** | **15** |
| Fałszywe zgłoszenie przestępstwa | **160$** | **5** |
| Wręczenie łapówki funkcjonariuszowi | **2500$** | **16** |
| Przyjęcie łapówki przez funkcjonariusza | **5000$** | **16** |
| Przekupstwo sędziego lub prokuratora | **4000$** | **20** |
| Ucieczka z aresztu | **300$** | **12** |
| Ucieczka z zakładu karnego | **600$** | **15** |
| Pomoc w ucieczce z aresztu / więzienia | **600$** | **15** |
| Pomoc w ucieczce z konwoju | **600$** | **20** |
| Podszywanie się pod funkcjonariusza publicznego | **300$** | **12** |
| Podszywanie się z użyciem fałszywego umundurowania | **600$** | **16** |
| Składanie fałszywych zeznań | **300$** | **12** |
| Współudział w przestępstwie | **300$** | **15** |
| Fałszywe wezwanie służb | **160$** | **5** |

#### Bezpieczeństwo publiczne

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Nielegalne posiadanie materiałów wybuchowych | **2500$** | **15** |
| Wytwarzanie materiałów wybuchowych | **5000$** | **20** |
| Użycie materiałów wybuchowych bez ofiar | **4000$** | **30** |
| Podpalenie mienia (bez ofiar) | **600$** | **15** |
| Podpalenie budynku mieszkalnego | **2500$** | **16** |
| Podpalenie z ofiarami w środku | **5000$** | **24** |
| Podpalenie budynku publicznego | **5000$** | **20** |

#### Przestępstwa przeciwko państwu

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Ujawnienie tajemnicy służbowej | **2500$** | **15** |
| Nadużycie uprawnień służbowych | **600$** | **16** |
| Bezprawne pozbawienie wolności przez funkcjonariusza | **1500$** | **15** |
| Fałszowanie dokumentów służbowych | **600$** | **16** |
| Ujawnienie tajemnicy śledczej | **2500$** | **15** |

#### Wykroczenia drogowe

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Jazda bez prawa jazdy | **80$** | — |
| Jazda z zawieszonym prawem jazdy | **150$** | — |
| Jazda z cofniętym prawem jazdy | **300$** | **6** |
| Przekroczenie prędkości o 1-15 mph | **35$** | — |
| Przekroczenie prędkości o 16-25 mph | **60$** | — |
| Przekroczenie prędkości o 26-40 mph | **120$** | — |
| Przekroczenie prędkości o ponad 40 mph | **175$** | — |
| Przekroczenie prędkości w strefie szkolnej | **110$** | — |
| Przejazd na czerwonym świetle | **80$** | — |
| Niezatrzymanie się przed linią zatrzymania | **50$** | — |
| Niezatrzymanie się przed przejściem dla pieszych | **80$** | — |
| Nieustąpienie pierwszeństwa pieszemu | **120$** | — |
| Nieustąpienie pierwszeństwa pojazd. uprzywilejowanemu | **80$** | — |
| Niezastosowanie się do znaku STOP | **60$** | — |
| Niezastosowanie się do polecenia funkcjonariusza | **120$** | — |
| Jazda pod prąd | **60$** | — |
| Brak zapiętych pasów bezpieczeństwa | **30$** | — |
| Używanie telefonu podczas jazdy | **50$** | — |
| Brawurowa / niebezpieczna jazda | **160$** | **5** |
| Niebezpieczna jazda z obrażeniami u innych | **600$** | **12** |
| Niedozwolone wyprzedzanie | **80$** | — |
| Jazda niepoprawnym pasem ruchu | **40$** | — |
| DUI - BAC 0.08-0.14% (pierwsze) | **300$** | **6** |
| DUI - BAC 0.15%+ (pierwsze) | **600$** | **5** |
| DUI - drugie naruszenie | **600$** | **8** |
| DUI - z wypadkiem bez obrażeń | **600$** | **5** |
| DUI - z obrażeniami u innych | **1500$** | **12** |
| Odmowa badania alkotestem | **120$** | **5** |
| Spowodowanie kolizji | **50$** | — |
| Spowodowanie wypadku | **120$** | — |
| Potrącenie pieszego | **160$** | — |
| Ucieczka z miejsca wypadku - bez ofiar (hit and run) | **300$** | **5** |
| Jazda pojazdem niezdatnym do ruchu | **160$** | — |
| Niesprawne oświetlenie | **35$** | — |
| Niedozwolony kolor świateł (niebieski/czerwony) | **120$** | — |
| Niedozwolone przyciemnienie przedniej szyby | **60$** | — |
| Niedozwolone modyfikacje pojazdu | **160$** | — |
| Nadmierny hałas układu wydechowego | **50$** | — |
| Jazda pojazdem niezarejestrowanym | **120$** | — |
| Nieczytelne tablice rejestracyjne | **35$** | — |
| Parkowanie przy czerwonym krawężniku | **50$** | — |
| Parkowanie na przejściu dla pieszych | **60$** | — |
| Parkowanie na miejscu dla niepełnosprawnych | **120$** | — |
| Brak dokumentów pojazdu podczas kontroli | **30$** | — |
| Niezgłoszenie broni podczas kontroli drogowej | **120$** | **6** |
| Jazda bez kasku (motocykl / quad) | **50$** | — |
| Niedopuszczalny lane splitting | **60$** | — |
| Udział w nielegalnym wyścigu | **300$** | **5** |
| Niezatrzymanie pojazdu do kontroli | **120$** | — |
| Ucieczka pojazdem przed służbami | **300$** | **12** |

#### Naruszenia gospodarcze

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Prowadzenie działalności bez rejestracji | **300$** | **6** |
| Prowadzenie działalności bez licencji branżowej | **1500$** | **5** |
| Podanie fałszywych danych przy rejestracji | **600$** | **12** |
| Prowadzenie działalności po cofnięciu licencji | **2500$** | **12** |
| Zaniżenie przychodów w deklaracji podatkowej | **2500$** | **12** |
| Ukrywanie przychodów / podwójna księgowość | **4000$** | **15** |
| Oszustwo gospodarcze na szkodę kontrahentów | **5000$** | **15** |
| Kartel / porozumienie antykonkurencyjne | **4000$** | **15** |
| Zatrudnianie bez umowy (praca na czarno) | **600$** | **6** |

#### Okoliczności łagodzące

| Wykroczenie / przestępstwo | Grzywna $ | Więzienie (mies.) |
|---|---:|---:|
| Ulga mandatowa -250$ | **-250$** | — |
| Ulga mandatowa -500$ | **-500$** | — |
| Ulga mandatowa -750$ | **-750$** | — |
| Ulga mandatowa -1000$ | **-1000$** | — |
| Ulga mandatowa -1500$ | **-1500$** | — |
| Ulga mandatowa -2000$ | **-2000$** | — |
| Ulga mandatowa -2500$ | **-2500$** | — |
| Ulga więzienna -1 miesiąc | — | **-1** |
| Ulga więzienna -6 miesięcy | — | **-6** |
| Ulga więzienna -12 miesięcy | — | **-12** |
| Ulga więzienna -24 miesiące | — | **-24** |
| Ulga więzienna -36 miesięcy | — | **-36** |
| Ulga więzienna -60 miesięcy | — | **-60** |


### EMS — faktury medyczne (76 pozycji)

Plik: `[rage]/rage_mdt/Config.lua` → `Config.Fines.ambulance`. **Split wystawiającego: 25%** (bez zmian struktury).

#### Złamania

| Usługa | Grzywna $ |
|---|---:|
| Złamanie palca dłoni | **70$** |
| Złamanie śródręcza | **80$** |
| Złamanie nadgarstka | **100$** |
| Złamanie kości promieniowej | **225$** |
| Złamanie kości łokciowej | **225$** |
| Złamanie przedramienia (obie kości) | **275$** |
| Złamanie kości ramiennej | **175$** |
| Złamanie barku / obojczyka | **150$** |
| Złamanie palca stopy | **70$** |
| Złamanie śródstopia | **100$** |
| Złamanie kostki | **150$** |
| Złamanie kości piszczelowej | **275$** |
| Złamanie kości strzałkowej | **200$** |
| Złamanie podudzia (obie kości) | **350$** |
| Złamanie kolana | **325$** |
| Złamanie kości udowej | **475$** |
| Złamanie miednicy | **400$** |
| Złamanie żeber (za każde) | **100$** |
| Złamanie kręgosłupa | **550$** |
| Złamanie czaszki | **500$** |

#### Rany i obrażenia

| Usługa | Grzywna $ |
|---|---:|
| Rana cięta | **80$** |
| Rana kłuta | **150$** |
| Rana tłuczona | **90$** |
| Rana szarpana | **150$** |
| Otarcia naskórka | **50$** |
| Rana postrzałowa – wlot | **200$** |
| Rana postrzałowa – wylot | **275$** |
| Postrzał mnogi (za każdy) | **250$** |
| Oparzenie I stopnia | **150$** |
| Oparzenie II stopnia | **200$** |
| Oparzenie III stopnia | **350$** |
| Odmrożenia | **250$** |
| Zmiażdżenie kończyny | **425$** |

#### Diagnostyka

| Usługa | Grzywna $ |
|---|---:|
| RTG (jedna partia ciała) | **175$** |
| RTG rozszerzone | **250$** |
| USG | **175$** |
| Tomografia komputerowa | **300$** |
| Rezonans magnetyczny | **350$** |
| Badanie krwi | **90$** |
| Badanie toksykologiczne | **100$** |
| Badanie alkoholu | **70$** |
| Badanie narkotykowe | **100$** |

#### Konsultacje specjalistyczne

| Usługa | Grzywna $ |
|---|---:|
| Badania psychologiczne (na broń palną) | **800$** |
| Konsultacja psychologiczna | **300$** |
| Konsultacja psychiatryczna | **275$** |
| Konsultacja neurologiczna | **250$** |
| Konsultacja ortopedyczna | **250$** |
| Konsultacja chirurgiczna | **275$** |
| Konsultacja kardiologiczna | **250$** |
| Konsultacja internistyczna | **200$** |

#### Farmakologia i zabiegi

| Usługa | Grzywna $ |
|---|---:|
| Podanie leków przeciwbólowych | **40$** |
| Podanie leków uspokajających | **40$** |
| Podanie antybiotyku | **40$** |
| Podanie adrenaliny | **70$** |
| Podanie narkozy | **80$** |
| Opatrzenie rany | **70$** |
| Szycie rany (za szew) | **50$** |
| Nastawienie złamania | **250$** |
| Unieruchomienie kończyny | **300$** |
| Reanimacja (RKO) | **125$** |
| Intubacja | **225$** |
| Operacja chirurgiczna | **175$** |

#### Transport i personel EMS

| Usługa | Grzywna $ |
|---|---:|
| Zespół ratownictwa medycznego | **175$** |
| Wezwanie chirurga | **250$** |
| Wezwanie specjalisty | **200$** |
| Transport karetką | **200$** |
| Transport lotniczy | **600$** |

#### Wykroczenia medyczne

| Usługa | Grzywna $ |
|---|---:|
| Nieuzasadnione wezwanie EMS | **300$** |
| Fałszywe zgłoszenie | **475$** |
| Odmowa współpracy z EMS | **175$** |
| Utrudnianie czynności medycznych | **250$** |
| Ucieczka z miejsca leczenia | **475$** |
| Zniewaga funkcjonariusza EMS | **175$** |
| Agresja wobec personelu medycznego | **700$** |
| Zniszczenie mienia EMS | **300$** |
| Leczenie pod eskortą LSPD/LSSD | **250$** |

### Mechanik — usługi MDT (8 pozycji)

Plik: `[rage]/rage_mdt/Config.lua` → `Config.Fines.mechanic`. **Split wystawiającego: 40%** dla `mechanic`, `c_blackrepair`, `c_quiettech` (LSC docelowo **40%**, jak sub-shopy — patrz § Frakcja mechanik).

#### Usługi mechaniczne

| Usługa | Grzywna $ |
|---|---:|
| Myjnia ręczna | **80$** |
| Naprawa blacharsko-lakiernicza | **150$** |
| Kompletny zestaw naprawczy pojazdu | **275$** |
| Usługa holowania (stawka za km) | **80$** |
| Usługa odblokowania pojazdu (palnik) | **225$** |
| Kompleksowa pomoc drogowa | **300$** |
| Przegląd techniczny pojazdu | **150$** |
| Tankowanie paliwa (dojazd) | **60$** |

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

Ceny **sprzedaży ulicznej** — tabela Narkotyki. Poniżej **docelowe parametry produkcji** (`[tebex]/kq_weed/config.strains.lua`) — wdrożenie razem z resztą v2.

### Tabela per odmiana (docelowo)

| Odmiana | `baseGrowTime` | `wateringTime` | `collectionLifespan` | Yield buds (min–max) | `budsPerBrick` | `needsConditions` |
|---|---:|---:|---:|---|---:|---|
| OG Kush | **28** min | **10** min | 25 min | **4–5** | 20 | nie |
| Purple Haze | **32** min | **12** min | 28 min | **4–5** | 20 | nie |
| White Widow | **36** min | **14** min | 22 min | **3–4** | 20 | tak |
| Blue Dream | **40** min | **16** min | 35 min | **3–5** | 20 | tak |

**Bez zmian struktury:** szanse mutacji nasion, `budsPerBaggie = 1`, nawozy (`config.fertilizers.lua`) — mnożniki jak obecnie; cena nawozu yield w dark shop **300–500$**.

### Szacunek $/h (solo, 2–3 rośliny, sprzedaż bag/joint v2)

| Odmiana | Cykl ~ | Budów / cykl | Przychód / cykl (bag v2) | ≈ $/h |
|---|---|---:|---:|---:|
| OG Kush | ~35 min | 4–5 | 320–500$ | **550–850$** |
| Purple Haze | ~40 min | 4–5 | 400–600$ | **600–900$** |
| White Widow | ~45 min | 3–4 | 330–640$ | **450–750$** |
| Blue Dream | ~50 min | 3–5 | 360–875$ | **430–800$** |

**Cel balansu:** produkcja KQ weed **nie przewyższa** typowego crime (700–1200$/h) przy solo farmie; wyższe odmiany wymagają warunków (grow tent / lockup) i dłuższego cyklu. Ryzyko: policja, konfiskata, koszt setupu (nasiona, nawóz, woda).

**Pliki wdrożenia:** `config.strains.lua` (yields + czasy), `config.lua` / dark shop (nawozy), `rage_drugs` / street sales (ceny sprzedaży — tabela Narkotyki).

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

Ekonomia = **sink + rozrywka** (jak zdrapki / kasyno) — **ujemne EV**, brak grindu.

| Parametr | Obecnie | Docelowo v2 |
|---|---|---|
| `Config.PayForBowling` | `false` | **`true`** — opłata za start gry |
| `bowlingstartfee` (per lokacja) | 100$ | **40$** (widełki **25–50$**) |
| Wypłata gotówki za wynik / strike | brak w resource (tylko fee) | **bez wypłat** — nie dodawać nagród pieniężnych |
| `AddMoneyRTX` | hook ESX | **nie używać** do nagród; ewentualnie tylko refund przy błędzie |
| Leaderboard | kosmetyka / RP | bez nagrody $ |

**Szacunek:** gra ~10–15 min → opłata 40$ ≈ **160–240$/h sink** przy ciągłej grze — akceptowalne jako rozrywka, nie źródło dochodu.

**Plik:** `[tebex]/rtx_bowling/config.lua` — ustawić `PayForBowling = true` i `bowlingstartfee` we wszystkich aktywnych lokacjach.

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
| Sprzedaż dealerowi (`SellVehiclePercent`) | 65% ceny zakupu | **65%** (bez zmian) |
| Sprzedaż gracz → gracz | dowolna cena; sprzedawca dostaje 100% | **bez limitu ceny**; **podatek 2% → society DOJ**; sprzedawca **98%** ceny |
| Współwłaściciel (`ManageCoOwnerPrice`) | 5 000$ | **2 000$** |
| `prevent_sell` (limited cars) | blokada | zachować |

**P2P — flow podatku:** kupujący płaci uzgodnioną cenę; przy `acceptSellVehicle` odliczyć `math.ceil(price * 0.02)` do DOJ (`BabiczBossMenu:DepositMoney`), sprzedawcy wpłacić resztę na bank. W UI potwierdzenia pokazać kwotę netto i podatek.

---

## Napad truckera — podsumowanie

Pełna karta: **§9b** powyżej. Docelowa wypłata za **dostarczenie ciężarówki**: **3500–8000$** `black_money` (łącznie, przed mnożnikiem grupy), CD **35–45 min** per gracz.

| | KQ Trucker (RS Haul) | Napad truckera (crime) |
|---|---|---|
| Typ | Legalna praca dorywcza | Napad / kradzież + dostawa |
| Zarobek | ~600–850$/h (+ zasiłek) | 3500–8000$/kurs + pranie |
| Plik | `kq_deliveries/config.lua` | `rage_heists` (implementacja) |

---

## Napady — status implementacji

Ekonomia (łup, cooldown, mnożniki) jest ustalona dla **wszystkich** tierów — także tych bez pełnego skryptu. Wypłaty w kodzie muszą odpowiadać kartom poniżej przed deployem.

| Napad | Gameplay | Wypłata w kodzie | Łup docelowy |
|---|---|---|---|
| NPC, ATM, kasetka, sejf, Fleeca | Aktywny | Tak (rescale) | wg kart §1–9 |
| **Bank Pacific** | Aktywny | **Brak** — do implementacji | 35–90k$/napad |
| Boosting, Tracker, **Trucker**, Jubiler, Lombard, Jacht, Humane, Cayo | Planowany | Nie / częściowo | wg kart § Napady |

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

## Premie bossmenu — wymagania systemu

Koncepcja tierów — § Premie tygodniowe. Obecnie premia z bossmenu nie ma capu tygodniowego — docelowo:

| Wymaganie | Opis |
|---|---|
| Tier z `duty_time` | mapowanie godzin/tydz. → max premia (tabela v2) |
| Cap **10 000$/os./tydz.** | hard limit w `sendSalary` |
| Reset tygodniowy | licznik premii per employee per tydzień |
| Audit log | Discord webhook (już częściowo) |

---

## Systemowe

| Element | Obecnie | Docelowo |
|---|---|---|
| `StartingAccountMoney` (bank) | 2 500$ | **2 500$** |
| `rage_multicharacter` `OnStart` — gotówka | 5 000$ (`money` item) | **1 500$** (łącznie z bankiem **4 000$** startu) |
| `job_grades.salary` (SQL) | ignorowane przez paycheck | **0** dla wszystkich grade; paycheck = jedyne źródło |
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

**Jeden release** — wszystkie moduły razem. Kolejność prac technicznych (dla devów): [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §0 i §27.

---

## Eventy sezonowe (poza stałą ekonomią)

**Nie uwzględniane** w balansie v2 — wyłączone z planu wdrożenia i testów po deployu.

| Event / resource | Uwagi |
|---|---|
| Wielkanoc / `rage_znajdzki` | krótki event — poza scope |
| `BabiczPumpkins` / dynie | sezonowy — poza scope |
| Kaucje zwrot (side joby) | refund, nie netto — § Prace dorywcze |
| `BabiczHospitality` | opcjonalny EMS — **poza scope v2**, bez zmian |

---

## Mapa źródeł dochodu

| Kategoria | Źródła | Gdzie w dokumencie |
|---|---|---|
| **Paycheck** | zasiłek, frakcje, firmy `c_*` | § Paycheck |
| **Side joby** | miner, tailor, slaughterer, lumberjack, hunter, beach_vendor | § Prace dorywcze |
| **KQ** | deliveries, powerwashing, trucker | § Prace dorywcze |
| **Frakcje** | minutówka, MDT 25–40%, EMS kursy, LSPD local dispatch, bossmenu | § Frakcje, § Premie |
| **EMS dodatkowe** | kursy lokalne, BabiczHospitality | § EMS |
| **Firmy** | kursy paczkowe, paycheck, bossmenu | § Firmy |
| **Mechanik** | tuning 10%, MDT 40% | § Mechanik |
| **Crime** | drugsales, pralnia, lombard, KQ weed → sprzedaż | § Crime, § Narkotyki |
| **Napady** | NPC, ATM, kasetka, sejf, Fleeca, Pacific, planowane | § Napady |
| **Zadania** | Fernando ×5, Carlos ×4, Moris ×3, Rico | § Zadania Moris |
| **Hazard** | kasyno, zdrapki, kręgle | § Kasyno, § Zdrapki |
| **Garaże** | buyback dealer, sprzedaż P2P (+2% DOJ) | § Garaże |
| **Nieruchomości** | czynsz dla wynajmującego | § Mieszkania |
| **Start** | bank + gotówka na nowej postaci | § Systemowe |
| **Redystrybucja** | przelewy bankowe, telefon | § Systemowe (nie grind) |

| Poza stałą ekonomią | Powód |
|---|---|
| Śmieciarz | brak wypłaty w obecnym jobie |
| Napad truckera (ciężarówka) | crime — dostawa 3500–8000$ black | §9b |
| `BabiczHospitality`, `rage_znajdzki`, `BabiczPumpkins` | poza scope v2 |
| Eventy sezonowe | krótkotrwałe |
| Admin | tylko staff |
| Więzienie | skrócenie wyroku, bez $ |

**Bank Pacific:** gameplay aktywny — **wypłata lootu do zaimplementowania** przed deployem (karta §12).

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
| FIB, mechanik (pełna analiza), hunter, zadania Moris (Fernando / Carlos / Moris) | Opisane |
| Firmy — kursy, sklepy, Blazing Tattoo, Five Records | Opisane |
| Beach vendor | Opisane |
| Garaże P2P (+2% DOJ), buyback, współwłaściciel | Opisane |
| Mapa źródeł dochodu | Opisane |
| Eventy sezonowe, kręgle, wynajem | Opisane / poza scope |
| Start (gotówka + bank) | Opisane |
| Kurs lawetą mechanika (zepsute auta) | Ustalone |
| MDT police (230) + EMS (76) + mechanik (8) | Pełne tabele w v2 § Wyroki i mandaty MDT |
| Zdrapki, automaty, platetape | Opisane |
| Premie bossmenu — wymagania systemu | Opisane |
| Plan testów po deployu | Opisane |
| Śmieciarz | **Poza scope** |
| Napad truckera (ciężarówka) | 3500–8000$ black, 1–3 os. | Ustalone |
| Ceny pojazdów per model + housing DB | Osobna praca konwersji |

**Deploy:** jeden release, wszystkie moduły — patrz [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §0 (blokery) i §27.

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
| Wyroki MDT — zasady + pełny taryfikator | Ustalone |
| LSPD local dispatch + laboratorium | Ustalone |
| Mechanik — kurs lawetą (zepsute auta lokalne) | Ustalone |
| Premie tygodniowe (~36 h/tydz., max 10k) | Ustalone |
| Progresja 14 napadów (pełne karty) | Ustalone |
| Zadania Moris — Carlos ×4, Moris ×3 | Ustalone |
| Narkotyki — tabela per item | Ustalone |
| Lombard — pełna tabela | Ustalone |
| Narzędzia napadów | Ustalone |
| Pojazdy — segmenty i limity | Ustalone |
| Zdrapki — ceny i EV | Ustalone |
| Firmy — koszty operacyjne (świadomie wyższe) | Ustalone |
| Kursy firmowe 80–150$/pkg + cooldown 30–45 min | Ustalone |
| depositProducts 150% → 100–110% | Ustalone |
| Mechanik — tuning split 35/10/10 | Ustalone |
| Mechanik — kurs lawetą (zepsute auta lokalne) | Ustalone |
| Kasyno — MaxWager i chipy po deflacji | Ustalone |
| Dark Shop + dual Ammunation | Ustalone |
| Sprzedaż P2P aut — podatek **2% DOJ**, sprzedawca 98% | Ustalone |
| Beach vendor poza job center | Ustalone |
| Premie bossmenu — tier + cap 10k | Ustalone |
| MDT EMS 76 pozycji | Pełna tabela w v2 § EMS — faktury medyczne |
| Mieszkania — segmenty cenowe | Ustalone |
| Hipoteka (`CreditEq` **0.06**, co **45 min**) | Ustalone |
| Czynsz co **14 dni** (**1%** wartości nieruchomości) | Ustalone |
| KQ weed — yields / czasy per odmiana | Ustalone |
| Kręgle — opłata 25–50$, brak wypłat | Ustalone |
| Miękki limit mandatów → society | **Odrzucone** — nie wdrażać |
| Śmieciarz | **Poza scope** — nie w job center |
| Napad truckera (ciężarówka) | **3500–8000$** black, CD 35–45 min | Ustalone |
| NPC farm — cooldown 3 min | **Ustalone** |
| Mandaty — 1 wystawiający FP na patrol | **Ustalone** (realia RP) |
| Napady planowane (7 szt.) | Implementacja |
| Pacific — wypłata lootu | Implementacja przed deployem |
| Ceny pojazdów per model (`vehicles.json`) | Osobna praca konwersji (w deploy) |
| qs-housing — ceny per lokalizacja | Osobna praca konwersji DB (w deploy) |

---

## Podsumowanie

Dokument obejmuje **pełny zakres ekonomii serwera**: zarobki, crime, frakcje, koszty życia, nieruchomości, utrzymanie pojazdów i moduły pomocnicze. Kluczowe założenia:

- Kotwica side job **500–600$/h**; zasiłek 60$/h.
- Minutówka: LSPD/FIB 672–700$/h; EMS 460–500$/h; DOJ 600–632$/h.
- Spokojna zmiana frakcji ~650–750$/h.
- Koszty życia @ 500$/h: **80–115$/h** (16–22%); żywienie **45$/h** (9%).
- Endgame aut: większość 300–700k; perełki do 1.2 mln.
- Cooldown **per gracz per typ napadu** — rotacja crime; NPC **3 min** (anty-farm).
- Mandaty PD — **1 wystawiający FP** na patrol; minutówka > mandaty w typowej zmianie.
- MDT — **230** wyroków/mandatów LSPD + **76** EMS + **8** mechanik; zasady więzienia (×0.3 real time).
- LSPD local dispatch + laboratorium depozytu — uzupełnienie $/h frakcji.
- Mechanik — kurs lawetą do zepsutych aut lokalnych (150–350$/kurs).
- Kasyno, Dark Shop, telefon, P2P auta (podatek 2% DOJ) — w scope v2.
- Eventy sezonowe poza stałą ekonomią.
- Zdrapki — rozrywka, ujemne EV, max wygrana ~8000$.
- Firmy — **wyższe koszty operacyjne** (składniki 40–65%, DOJ 10%).
- Czynsz qs-housing — **1% wartości co 14 dni** (real time).
- KQ weed — yields i czasy per odmiana (§ KQ weed).
- Kręgle — opłata startu 25–50$, brak nagród pieniężnych.
- 14 napadów — pełne karty w dokumencie.
- Zadania Moris: Fernando → Carlos (×4) → Moris (×3) — progresja crime, wypłaty ≤ tier napadów.
- Endgame auto max 1.2 mln; endgame dom max ~800k.
- 7 napadów planowanych — do implementacji.
