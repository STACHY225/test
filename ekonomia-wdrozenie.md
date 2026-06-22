# RageCity — wdrożenie ekonomii v2 (developer)

Checklista techniczna: **co, gdzie i na co zmienić** — pełne tabele cen, czasów i limitów.  
Model balansu (dlaczego, widełki): [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md).  
**QA po deployu:** [`ekonomia-qa-testy.md`](ekonomia-qa-testy.md) (testerzy) · [`ekonomia-qa-testy-dev.md`](ekonomia-qa-testy-dev.md) (lead)  
**Pełny rejestr (§A):** każda pozycja z configów — auto-generowany (`python docs/_gen_wdrozenie_full.py` → `_wdrozenie_full_registry.md`, scalany do tego pliku).  
**Katalogi ręczne:** §3A (rage_jobs), §11A (firmy), §15 (lombard), §16 (narkotyki), §19A (rage_market), §21B (pojazdy `types`).  
**MDT mandaty/wyroki (314 poz.):** pełne tabele **fine + jail** w **§7** (generator: `python docs/_gen_mdt_fines_table.py`).

**Deploy:** **jeden release** — wszystkie pozycje z §0 i poniższych sekcji wchodzą **razem**. Częściowe wdrożenie psuje ekonomię.


## Spis treści (skrót)

| § | Temat |
|---:|---|
| 0 | Blokery deploy |
| 1–1b | Paycheck, SQL salary |
| 2–6 | Job center, rage_jobs, KQ, hunter |
| 3A | rage_jobs — ceny per plik |
| 7–10 | MDT, FIB, EMS, DOJ |
| 11–11A | Firmy c_* + pełne menu/skladniki + sklepy PD/EMS |
| 12 | Mechanik |
| 13–15 | Napady, lombard |
| 16 | **Narkotyki — pełna tabela** |
| 17–18 | Pralnia, zadania Moris |
| 19–19A | Sklepy + **pełny rage_market** |
| 20–25 | Paliwo, pojazdy, garaże, housing |
| 21B | VehiclePriceConfig.types (lua) |
| 26–31 | Kasyno, dispatch, MDT ulgi |
| **A** | **Pełny rejestr zmian (każda pozycja)** |
| 32 | Indeks paczki |
| — | [**QA testy**](ekonomia-qa-testy.md) (testerzy) · [dev](ekonomia-qa-testy-dev.md) |

**Legenda kolumn:** Obecnie = stan w repo · Docelowo = v2 · Plik = ścieżka względem repo.

Legenda statusu: `Planowany` | `Gotowe do wdrożenia` | `Poza scope` | `Poza scope przygotowania`

---

## 0. Wymagane zmiany — blokery ekonomii

Bez poniższych punktów **nie robić czystki serwera** — gracze wzbogacą się za szybko.

| # | Bloker | Pliki / obszar | Co zrobić | Skutek jeśli pominięte |
|---:|---|---|---|---|
| 1 | Paycheck bez on duty | `[core]/es_extended/server/paycheck.lua` | Wypłata tylko `on duty`; stawki v2 (§1); `c_*` → 10–15$/15 min | AFK 1000$/h frakcje/firmy |
| 2 | KQ Deliveries / Powerwashing | `[tebex]/kq_deliveries/config.lua`, `kq_powerwashing/contracts.lua` | `payPerMinute` → 7–11; kontrakty → 800–2000$ | ~20–30k$/h side job |
| 3 | Kursy firm + deposit | `BabiczCompanyCourses_*`, `BabiczCompanyShop_sv.lua`, configi `c_*` | 80–150$/pkg; CD 30–45 min; `depositProducts` 100–110% (nie 150%) | ~9000$/kurs co ~20 min |
| 4 | Kasyno + zdrapki | `pickle_casinos/config.lua`, `rage_scratchcard/server.lua`, `rage_market` | MaxWager 5–15k; scratch max ~8k wygrana | Miliony z hazardu |
| 5 | Napady — loot + mnożniki | `rage_heists/**`, `Config.AttackerMultipliers` | Loot wg v2; mnożniki łagodne (max ~×1,35); CD per gracz per typ | Fleeca 60k+, spam |
| 6 | **Farm NPC** | `Heists/NPC/server/BabiczNPC_sv.lua` | `SetTimeout(15000)` → **`180000`** (3 min) między napadami per gracz | 1500–3500$/h sam NPC |
| 7 | Premie bossmenu | `BabiczBossMenu_sv.lua` → `sendSalary` | Tier + **cap 10 000$/os./tydz.** | Boss wylewa society |
| 8 | Start postaci | `es_extended/config.lua`, `rage_multicharacter/Config.lua` | Bank **2500$**; gotówka **1500$** (łącznie **4000$**) | Za dużo na start |
| 9 | **vehicles.json** | `[rage]/rage_vehicleshop/vehicles.json` | Segmenty v2 (auto max 1,2 mln) | Progresja auta rozjechana |
| 10 | **qs-housing DB + config** | `[tebex]/qs-housing/`, SQL | Ceny domów 15k–800k; CreditEq/RentTime (§25) | Pasywny dochód co 5 min |
| 11 | Pacific — wypłata | `Heists/BankPacific/server/` | Implementacja lootu 35–90k (§13) | Napad bez ekonomii / stary loot |
| 12 | MDT — ulgi + jail | `rage_mdt/Config.lua` | Ulgi max −2500$; jail z taryfikatora v2; usuń −5000/−10000/−25000 | Stare skale w wyrokach |
| 13 | Job center UI | `rage_jobcenter/Config.lua` | `salary` marketing = real $/h v2 | Mylący UI vs rzeczywistość |

**Poza scope v2 (bez zmian):** `BabiczHospitality`, `rage_znajdzki`, `BabiczPumpkins`, śmieciarz (`trashman.lua` stub).

---

## Decyzje przed deployem (zamknięte)

Pełny kontekst balansu: [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md) § Decyzje przed deployem.

| Temat | Wartość docelowa | Status wdrożenia |
|---|---|---|
| Start postaci | bank **2500$** + gotówka **1500$** | Planowany |
| SQL `job_grades.salary` | Przelicznik **$/h** wg § Paycheck v2 (MDT + bossmenu) | Planowany |
| `towVehicle` | **0$** (bez powiązania z MDT) | Gotowe do wdrożenia |
| `SellVehiclePercent` | **65%** | Planowany |
| `ManageCoOwnerPrice` | **2000$** | Planowany |
| Pralnia ręczna | `math.random(350, 900)` per punkt | Planowany |
| Sejf — `stethoscope`, CD | CD `900000` ms per gracz | Planowany |
| KQ bonusy | tabela §4 poniżej | Planowany |
| qs-housing | CreditEq **0.06**, CreditTime **45 min**, rent **1%/14d** | Planowany |
| MDT `data.ts` | mock przeglądarki — **nie** źródło produkcyjne | Gotowe |

**Poza scope przygotowania (po deployu):** depozyt lab PD (wartości w v2, bez zmian na deploy).

---

## 1. Paycheck i zasiłek

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | Stawki per frakcja, on duty | 125$/15 min cywile; 250$/15 min frakcje; bez on duty | Zasiłek 15$/15 min; LSPD/LSSD/FIB **168–175$/15 min**; EMS **115–125$/15 min**; DOJ **150–158$/15 min**; **mechanic 168–175$/15 min**; tylko on duty | Planowany |
| `[core]/es_extended/config.lua` | `PaycheckInterval`, `StartingAccountMoney` | 15 min; bank 2500$ | bank **2500$** (bez zmian kwoty) | Planowany |
| SQL `job_grades.salary` | Grade salaries | różne w DB | **Przelicznik $/h** — patrz §1b (nie `0`; paycheck tego nie używa) | Planowany |

---

## 1b. SQL `job_grades.salary` — przelicznik referencyjny

**Cel:** MDT (`grade_salary` → profil „Przelicznik $/h”) i bossmenu (`config.grades[].salary` → statystyki pracowników). **Paycheck tego nie czyta** — wypłata wyłącznie z `paycheck.lua`.

**Nie obejmuje** mandatów, kursów, premii — tylko szacunek minutówki za `duty_time`.

| Rodzina | Wzór `salary` ($/h) | Przykład SQL |
|---|---|---|
| `unemployed` | **60** | `UPDATE job_grades SET salary = 60 WHERE job_name = 'unemployed';` |
| Side joby | **550** | `UPDATE job_grades SET salary = 550 WHERE job_name IN ('miner','tailor','lumberjack','slaughterer','hunter','beach_vendor','trashman');` |
| `c_*` | **45 + grade × 3** | per job: `UPDATE job_grades SET salary = 45 + grade * 3 WHERE job_name LIKE 'c_%';` |
| `police`, `fib` | **650 + grade × 5** | `UPDATE job_grades SET salary = LEAST(720, 650 + grade * 5) WHERE job_name IN ('police','fib');` |
| `ambulance` | **450 + grade × 5** | `UPDATE job_grades SET salary = LEAST(720, 450 + grade * 5) WHERE job_name = 'ambulance';` |
| `doj` | **580 + grade × 5** | `UPDATE job_grades SET salary = LEAST(720, 580 + grade * 5) WHERE job_name = 'doj';` |
| `mechanic` | **650 + grade × 5** | `UPDATE job_grades SET salary = LEAST(720, 650 + grade * 5) WHERE job_name = 'mechanic';` |

Po UPDATE: restart resource `es_extended` lub serwer (cache jobów ESX). Bossmenu odświeża grades z `ESX.Jobs` przy cache refresh.

**Opcjonalnie (UX):** w bossmenu fallback `22000` w JS zastąpić `0` gdy brak salary — po ustawieniu SQL problem znika.

---

## 2. Job center (UI)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobcenter/Config.lua` | Wyświetlane `salary` per job | 6000–26000$ (marketing) | side job ~500–600$/h; frakcje spokojna zmiana ~650–750$/h; aktywna ~1000–2000$/h | Planowany |

---

## 3. Prace dorywcze — rage_jobs

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobs/Jobs/miner.lua` | Ceny sprzedaży rud + kaucje | copper 16–18; vehicleBail 1500; skinBail 500 | ~500–700$/h; kaucje 400–600$ / 100–200$ | Planowany |
| `[rage]/rage_jobs/Jobs/lumberjack.lua` | Cena sprzedaży + kaucje | 530–550$/plank; vehicleBail 2500 | ~500–650$/h; kaucje 600–900$ / 100–200$ | Planowany |
| `[rage]/rage_jobs/Jobs/tailor.lua` | Cena sprzedaży + kaucje | 50–60$/szt.; vehicleBail 1500 | ~450–550$/h; kaucje 400–600$ / 100–200$ | Planowany |
| `[rage]/rage_jobs/Jobs/slaughterer.lua` | Cena sprzedaży + kaucje | 95–105$/szt.; vehicleBail 500 | ~450–600$/h; kaucje 150–250$ / 100–200$ | Planowany |
| `[rage]/rage_jobs/Jobs/beach_vendor.lua` | Ceny produktów | 44–290$ | ~350–500$/h | Planowany |
| `[rage]/rage_jobs/Custom/server/BabiczBeachVendor_sv.lua` | Kursy | 40–80$ | W widełkach beach vendor | Planowany |
| `[rage]/rage_jobs/BabiczJobs_sv.lua` | `priceMultiplier` | 1.0 | 1.0 (po przeskalowaniu jobów) | Planowany |

---

## 3A. `rage_jobs` — ceny per plik


### `beach_vendor.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| orzeszki | 62-95 | 25-40 |
| kukurydza | 44-74 | 18-30 |
| lody | 170-220 | 35-55 |
| rogaliki | 220-290 | 45-70 |

### `tailor.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| clothe sale | 50-60 | 50-60 |
| vehicleBail | 1500 | 400-600 |
| skinBail | 500 | 100-200 |

### `slaughterer.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| packed_chicken | 95-105 | 95-105 |
| vehicleBail | 500 | 150-250 |
| skinBail | 500 | 100-200 |

### `lumberjack.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| plank | 530-550 | 530-550 |
| vehicleBail | 2500 | 600-900 |
| skinBail | 500 | 100-200 |

### `miner.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| copper | 16-18 | 16-18 |
| iron | 19-21 | 19-21 |
| gold | 60-62 | 60-62 |
| diamond | 125-130 | 125-130 |
| vehicleBail | 1500 | 400-600 |
| skinBail | 500 | 100-200 |

### `hunter.lua`

| Parametr | Obecnie | Docelowo |
|---|---|---|
| meat_boar | 128 | 45-55 |
| meat_deer | 183 | 65-75 |
| skin_boar | 85 | 30-38 |
| skin_deer | 116 | 40-48 |
| boar_tusks | 228 | 80-95 |
| deer_horns | 300 | 105-120 |
| vehicleBail | 2500 | 600-900 |

---

## 4. Prace dorywcze — KQ (priorytet)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/kq_deliveries/config.lua` | `payPerMinute` per job | 330–500$/min (~20k+/h) | **7 / 8 / 9 / 10 / 11** $/min (~450–650$/h z zasiłkiem) | Planowany |
| `[tebex]/kq_deliveries/config.lua` | Bonusy, team %, level upgrades | Wysokie mnożniki | Patrz tabela poniżej | Planowany |
| `[tebex]/kq_powerwashing/contracts.lua` | `reward` per kontrakt | 4800–24000$+ | ~800–2000$ per kontrakt | Planowany |
| `[tebex]/kq_powerwashing/config.lua` | Bonusy, teamWorkBonuses | do +50% | Jak Deliveries — max +10% team | Planowany |

### KQ — docelowe bonusy (Deliveries + Powerwashing)

| Parametr | Obecnie | Docelowo |
|---|---|---|
| `fiveStarBonus` | 10% | **3%** |
| `fourStarBonus` | 5% | **2%** |
| `vehicleCare` | 5% | **2%** |
| `teamWorkBonuses[2]` | 80% | **10%** |
| `salary_bonus_*` (level upgrades) | 5–30% | **2–8%** (max tier) |
| `maxVehicleDamagePenalty` | 1000$ | **150–250$** |
| `missingVehiclePenalty` | 1200$ | **200–300$** |

Cap łączny bonusów: **~15%** bazy `payPerMinute` / nagrody kontraktu.

---

## 5. Myśliwy (hunter)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobs/Jobs/hunter.lua` | `MeatPrices`, `vehicleBail` | meat 85–300$; vehicleBail 2500 | Ceny v2 (45–120$); kaucja 600–900$; ~450–600$/h | Planowany |
| `[rage]/rage_jobs/Custom/server/BabiczHunter_sv.lua` | Zwrot kaucji | logika bail | bez zmian struktury | Planowany |
| `[rage]/rage_jobs/Custom/client/BabiczHunter_cl.lua` | Flow joba | — | test po zmianie cen | Planowany |

---

## 6. Śmieciarz — poza scope v2

Praca **nie jest w job center** i nie ma działającej pętli wypłaty (`rage_jobs/Jobs/trashman.lua` to stub). **Nie wchodzi w ekonomię v2** — brak zmian do wdrożenia.

---

## 7. Frakcje — MDT i mandaty

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_mdt/Config.lua` | `policeFines` — `fine` + `jail` | stare skale / częściowo v2 w repo | **Tabela poniżej** — kolumny **Ustaw fine** i **Ustaw jail** | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.ambulance` (76 poz.) | 480–6800$ | **Tabela poniżej** — kolumna **Ustaw** | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.mechanic` (8 poz.) | 300–2000$ | **Tabela poniżej** — kolumna **Ustaw** | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.FinePlayerShare` | brak (flat 25%) | Progi 50/35/25/12/5%, cap **750$** | Planowany |
| `[rage]/rage_mdt/BabiczMdt_sv.lua` | `CalculateFinePlayerShare` | flat `finePercentForPlayer` | Progresywny split od sumy po zniżkach | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentProgressive` per job | brak | `true` police/fib/doj/ambulance · `false` mechanic | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` | LSPD/FIB/EMS/DOJ 25%; c_blackrepair/c_quiettech 40% | Flat tylko mechanik; reszta progresywnie | Planowany |
| `[rage]/rage_mdt/Config.lua` | `fineCompanyMultiplier` | 1.5–2.5× | Bez zmian | Planowany |
| `[rage]/rage_mdt/html/src/config/data.ts` | UI mandatów | mock dev-only | **Nie** źródło produkcyjne — tylko `Config.lua` | Gotowe |

### Pełna tabela policeFines / wyroków (230 pozycji)

Regeneracja: python docs/_gen_mdt_fines_table.py → _mdt_wyroki_wdrozenie.md.

Pełna lista **230 pozycji** `local policeFines` → `Config.Fines.police` / `fib` / `doj`.

**Kolumna „Ustaw”** = wartości docelowe v2 (wklej do `Config.lua`).
**Więzienie:** 1 miesiąc w MDT = 1 min w UI; realny czas = `floor(jail × 0.3)` sekund po akceptacji wyroku.
**Okoliczności łagodzące:** usunąć ulgi mandatowe −5000 / −10000 / −25000 oraz więzienną −120 mies.


#### Przestępstwa przeciwko życiu i zdrowiu

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Zabójstwo II stopnia | 2500$ | 50 | **2500$** | **50** |
| Zabójstwo II stopnia funkcjonariusza publicznego | 5000$ | 60 | **5000$** | **60** |
| Zabójstwo w afekcie (Voluntary Manslaughter) | 600$ | 22 | **600$** | **22** |
| Nieumyślne spowodowanie śmierci | 600$ | 22 | **600$** | **22** |
| Nieumyślne spowodowanie śmierci pojazdem (DUI) | 1500$ | 20 | **1500$** | **20** |
| Ciężki uszczerbek na zdrowiu | 300$ | 16 | **300$** | **16** |
| Ciężki uszczerbek na zdrowiu z użyciem broni palnej | 600$ | 24 | **600$** | **24** |
| Ciężki uszczerbek na zdrowiu funkcjonariusza | 600$ | 20 | **600$** | **20** |
| Napaść z użyciem niebezpiecznego narzędzia | 300$ | 15 | **300$** | **15** |
| Napaść (Assault) | 80$ | 6 | **80$** | **6** |
| Pobicie (Battery) | 160$ | 5 | **160$** | **5** |
| Pobicie z użyciem niebezpiecznego przedmiotu | 300$ | 16 | **300$** | **16** |
| Napaść na funkcjonariusza publicznego | 300$ | 16 | **300$** | **16** |
| Bezprawne pozbawienie wolności | 300$ | 12 | **300$** | **12** |
| Porwanie dla okupu | 2500$ | 22 | **2500$** | **22** |
| Porwanie z użyciem broni palnej | 3000$ | 30 | **3000$** | **30** |
| Porwanie funkcjonariusza publicznego | 5000$ | 40 | **5000$** | **40** |
| Usiłowanie zabójstwa | 2500$ | 38 | **2500$** | **38** |
| Usiłowanie zabójstwa funkcjonariusza | 5000$ | 50 | **5000$** | **50** |

#### Przestępstwa przeciwko mieniu

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Kradzież małej wartości (poniżej $950) | 80$ | 6 | **80$** | **6** |
| Kradzież ($950 - $5000) | 175$ | 12 | **175$** | **12** |
| Kradzież wielka (powyżej $5000) | 600$ | 16 | **600$** | **16** |
| Kradzież z pojazdu | 160$ | 5 | **160$** | **5** |
| Kradzież tożsamości | 600$ | 12 | **600$** | **12** |
| Rozbój | 600$ | 15 | **600$** | **15** |
| Rozbój z użyciem broni palnej | 2500$ | 18 | **2500$** | **18** |
| Rozbój z użyciem broni białej | 1500$ | 15 | **1500$** | **15** |
| Włamanie do budynku mieszkalnego | 600$ | 15 | **600$** | **15** |
| Włamanie do obiektu komercyjnego | 300$ | 12 | **300$** | **12** |
| Włamanie z użyciem narzędzi | 600$ | 15 | **600$** | **15** |
| Włamanie do pojazdu | 160$ | 5 | **160$** | **5** |
| Kradzież pojazdu cywilnego | 600$ | 12 | **600$** | **12** |
| Kradzież pojazdu uprzywilejowanego | 2000$ | 15 | **2000$** | **15** |
| Nielegalne posiadanie skradzionego pojazdu | 300$ | 5 | **300$** | **5** |
| Wymuszenie / szantaż | 600$ | 16 | **600$** | **16** |
| Wymuszenie z użyciem broni | 2500$ | 16 | **2500$** | **16** |
| Wymuszenie przez zorganizowaną grupę | 5000$ | 20 | **5000$** | **20** |
| Oszustwo (do $5000) | 175$ | 5 | **175$** | **5** |
| Oszustwo (powyżej $5000) | 600$ | 12 | **600$** | **12** |
| Oszustwo na szkodę instytucji publicznej | 2500$ | 15 | **2500$** | **15** |
| Fałszerstwo dokumentów | 300$ | 12 | **300$** | **12** |
| Fałszerstwo pieniędzy / waluty | 2500$ | 15 | **2500$** | **15** |
| Wandalizm (szkoda poniżej $400) | 80$ | 5 | **80$** | **5** |
| Wandalizm (szkoda powyżej $400) | 300$ | 12 | **300$** | **12** |
| Zniszczenie mienia publicznego | 300$ | 12 | **300$** | **12** |
| Zniszczenie pojazdu służbowego | 600$ | 16 | **600$** | **16** |
| Pranie pieniędzy (do $50 000) | 2000$ | 16 | **2000$** | **16** |
| Pranie pieniędzy (powyżej $50 000) | 5000$ | 16 | **5000$** | **16** |
| Pranie pieniędzy przez zorganizowaną grupę | 4000$ | 20 | **4000$** | **20** |

#### Przestępstwa przeciwko porządkowi publicznemu

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Zakłócenie porządku publicznego | 50$ | 3 | **50$** | **3** |
| Wywołanie zbiegowiska / zamieszek | 160$ | 5 | **160$** | **5** |
| Napaść słowna / groźby publiczne | 80$ | 6 | **80$** | **6** |
| Groźba karalna | 80$ | 5 | **80$** | **5** |
| Groźba z użyciem broni | 300$ | 12 | **300$** | **12** |
| Groźba wobec funkcjonariusza publicznego | 300$ | 12 | **300$** | **12** |
| Stalking / nękanie | 160$ | 16 | **160$** | **16** |
| Użycie broni palnej podczas przestępstwa (+do kary) | 600$ | 20 | **600$** | **20** |
| Oddanie strzału podczas przestępstwa (+do kary) | 2000$ | 30 | **2000$** | **30** |
| Napaść na funkcjonariusza (bez obrażeń) | 300$ | 16 | **300$** | **16** |
| Pobicie funkcjonariusza (z obrażeniami) | 600$ | 15 | **600$** | **15** |
| Pobicie funkcjonariusza z bronią | 2500$ | 24 | **2500$** | **24** |
| Usiłowanie zabójstwa funkcjonariusza | 5000$ | 50 | **5000$** | **50** |
| Ucieczka piesza przed funkcjonariuszem | 50$ | 6 | **50$** | **6** |
| Ucieczka pojazdem (niebezpieczna jazda) | 300$ | 12 | **300$** | **12** |
| Ucieczka pojazdem z obrażeniami u innych | 1500$ | 15 | **1500$** | **15** |
| Ucieczka pojazdem ze śmiercią ofiary | 2500$ | 20 | **2500$** | **20** |
| Opór bierny przy zatrzymaniu | 50$ | 3 | **50$** | **3** |
| Czynny opór przy zatrzymaniu | 160$ | 5 | **160$** | **5** |
| Atak na funkcjonariusza podczas zatrzymania | 600$ | 15 | **600$** | **15** |
| Groźby terrorystyczne | 5000$ | 20 | **5000$** | **20** |
| Akt terrorystyczny bez ofiar | 4000$ | 40 | **4000$** | **40** |
| Udział w zorganizowanej grupie przestępczej | 2500$ | 15 | **2500$** | **15** |
| Kierowanie grupą przestępczą | 5000$ | 20 | **5000$** | **20** |
| Finansowanie działalności przestępczej | 4000$ | 24 | **4000$** | **24** |

#### Broń palna

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Posiadanie broni bez licencji (klasa I) | 300$ | 5 | **300$** | **5** |
| Posiadanie broni klasy II przez cywila | 2500$ | 16 | **2500$** | **16** |
| Posiadanie broni przez osobę z zakazem | 2500$ | 15 | **2500$** | **15** |
| Posiadanie niezarejestrowanej broni | 600$ | 12 | **600$** | **12** |
| Posiadanie broni klasy III bez zezwolenia | 5000$ | 16 | **5000$** | **16** |
| Posiadanie broni klasy IV (materiały wybuchowe) | 5000$ | 20 | **5000$** | **20** |
| Posiadanie ghost gun (bez numeru seryjnego) | 2500$ | 15 | **2500$** | **15** |
| Posiadanie tłumika bez zezwolenia | 2000$ | 16 | **2000$** | **16** |
| Posiadanie magazynka powyżej 10 nabojów | 300$ | 5 | **300$** | **5** |
| Open carry bez uprawnień służbowych | 175$ | 5 | **175$** | **5** |
| Concealed carry bez licencji CCW | 300$ | 12 | **300$** | **12** |
| Noszenie broni załadowanej w pojeździe (bez CCW) | 150$ | 5 | **150$** | **5** |
| Niezgłoszenie broni podczas kontroli drogowej | 120$ | 6 | **120$** | **6** |
| Sprzedaż broni bez licencji FFL | 5000$ | 15 | **5000$** | **15** |
| Sprzedaż broni bez weryfikacji nabywcy | 2500$ | 15 | **2500$** | **15** |
| Sprzedaż broni osobie nieuprawnionej | 5000$ | 16 | **5000$** | **16** |
| Pośrednictwo w nielegalnym obrocie bronią | 5000$ | 20 | **5000$** | **20** |
| Niezgłoszenie kradzieży broni w terminie 48h | 175$ | 6 | **175$** | **6** |
| Wejście do Gun Free Zone z bronią (nieumyślne) | 150$ | 6 | **150$** | **6** |
| Wejście do Gun Free Zone z bronią (umyślne) | 600$ | 12 | **600$** | **12** |
| Wniesienie broni do sądu lub więzienia | 2500$ | 15 | **2500$** | **15** |
| Bezpodstawne użycie broni palnej | 300$ | 15 | **300$** | **15** |

#### Narkotyki

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Posiadanie narkotyków miękkich | 190$ | 5 | **190$** | **5** |
| Posiadanie narkotyków twardych | 350$ | 10 | **350$** | **10** |
| Posiadanie hurtowych ilości narkotyków I | 825$ | 15 | **825$** | **15** |
| Posiadanie hurtowych ilości narkotyków II | 975$ | 20 | **975$** | **20** |
| Posiadanie hurtowych ilości narkotyków III | 1150$ | 25 | **1150$** | **25** |
| Posiadanie hurtowych ilości narkotyków IV | 1275$ | 30 | **1275$** | **30** |
| Posiadanie hurtowych ilości narkotyków V | 1450$ | 35 | **1450$** | **35** |
| Używanie narkotyków w miejscu publicznym | 30$ | 0 | **30$** | **—** |
| Handel narkotykami miękkimi | 320$ | 10 | **320$** | **10** |
| Handel narkotykami twardymi | 620$ | 15 | **620$** | **15** |
| Handel hurtowy narkotykami miękkimi | 825$ | 15 | **825$** | **15** |
| Handel hurtowy narkotykami twardymi | 1150$ | 25 | **1150$** | **25** |
| Sprzedaż narkotyków niepełnoletniemu | 2000$ | 20 | **2000$** | **20** |
| Produkcja narkotyków miękkich | 300$ | 8 | **300$** | **8** |
| Produkcja narkotyków twardych | 800$ | 15 | **800$** | **15** |
| Prowadzenie laboratorium narkotykowego | 900$ | 15 | **900$** | **15** |

#### Napady

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Napad na obywatela | 200$ | 5 | **200$** | **5** |
| Napad na bankomat | 350$ | 10 | **350$** | **10** |
| Kradzież bankomatu | 900$ | 12 | **900$** | **12** |
| Napad na kasetkę | 550$ | 12 | **550$** | **12** |
| Napad tracker | 1200$ | 15 | **1200$** | **15** |
| Napad na sejf sklepu | 2400$ | 15 | **2400$** | **15** |
| Napad na jubilera | 4500$ | 22 | **4500$** | **22** |
| Napad na sejf banku | 5500$ | 20 | **5500$** | **20** |
| Napad na lombard | 4200$ | 18 | **4200$** | **18** |
| Napad na jacht | 5500$ | 28 | **5500$** | **28** |
| Napad na bank Pacific | 7000$ | 30 | **7000$** | **30** |
| Napad na Humane Labs | 7500$ | 35 | **7500$** | **35** |
| Napad na posesję Cayo Perico | 8000$ | 40 | **8000$** | **40** |

#### Przestępstwa przeciwko wymiarowi sprawiedliwości

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Krzywoprzysięstwo | 300$ | 16 | **300$** | **16** |
| Fałszywe zeznania w sprawie o zabójstwo | 2500$ | 20 | **2500$** | **20** |
| Utrudnianie postępowania karnego | 300$ | 12 | **300$** | **12** |
| Niszczenie lub ukrywanie dowodów | 600$ | 15 | **600$** | **15** |
| Zastraszanie świadków / ławy przysięgłych | 2500$ | 15 | **2500$** | **15** |
| Przekupstwo świadka | 2500$ | 15 | **2500$** | **15** |
| Fałszywe zgłoszenie przestępstwa | 160$ | 5 | **160$** | **5** |
| Wręczenie łapówki funkcjonariuszowi | 2500$ | 16 | **2500$** | **16** |
| Przyjęcie łapówki przez funkcjonariusza | 5000$ | 16 | **5000$** | **16** |
| Przekupstwo sędziego lub prokuratora | 4000$ | 20 | **4000$** | **20** |
| Ucieczka z aresztu | 300$ | 12 | **300$** | **12** |
| Ucieczka z zakładu karnego | 600$ | 15 | **600$** | **15** |
| Pomoc w ucieczce z aresztu / więzienia | 600$ | 15 | **600$** | **15** |
| Pomoc w ucieczce z konwoju | 600$ | 20 | **600$** | **20** |
| Podszywanie się pod funkcjonariusza publicznego | 300$ | 12 | **300$** | **12** |
| Podszywanie się z użyciem fałszywego umundurowania | 600$ | 16 | **600$** | **16** |
| Składanie fałszywych zeznań | 300$ | 12 | **300$** | **12** |
| Współudział w przestępstwie | 300$ | 15 | **300$** | **15** |
| Fałszywe wezwanie służb | 160$ | 5 | **160$** | **5** |

#### Bezpieczeństwo publiczne

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Nielegalne posiadanie materiałów wybuchowych | 2500$ | 15 | **2500$** | **15** |
| Wytwarzanie materiałów wybuchowych | 5000$ | 20 | **5000$** | **20** |
| Użycie materiałów wybuchowych bez ofiar | 4000$ | 30 | **4000$** | **30** |
| Podpalenie mienia (bez ofiar) | 600$ | 15 | **600$** | **15** |
| Podpalenie budynku mieszkalnego | 2500$ | 16 | **2500$** | **16** |
| Podpalenie z ofiarami w środku | 5000$ | 24 | **5000$** | **24** |
| Podpalenie budynku publicznego | 5000$ | 20 | **5000$** | **20** |

#### Przestępstwa przeciwko państwu

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Ujawnienie tajemnicy służbowej | 2500$ | 15 | **2500$** | **15** |
| Nadużycie uprawnień służbowych | 600$ | 16 | **600$** | **16** |
| Bezprawne pozbawienie wolności przez funkcjonariusza | 1500$ | 15 | **1500$** | **15** |
| Fałszowanie dokumentów służbowych | 600$ | 16 | **600$** | **16** |
| Ujawnienie tajemnicy śledczej | 2500$ | 15 | **2500$** | **15** |

#### Wykroczenia drogowe

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Jazda bez prawa jazdy | 80$ | — | **80$** | **—** |
| Jazda z zawieszonym prawem jazdy | 150$ | — | **150$** | **—** |
| Jazda z cofniętym prawem jazdy | 300$ | 6 | **300$** | **6** |
| Przekroczenie prędkości o 1-15 mph | 35$ | — | **35$** | **—** |
| Przekroczenie prędkości o 16-25 mph | 60$ | — | **60$** | **—** |
| Przekroczenie prędkości o 26-40 mph | 120$ | — | **120$** | **—** |
| Przekroczenie prędkości o ponad 40 mph | 175$ | — | **175$** | **—** |
| Przekroczenie prędkości w strefie szkolnej | 110$ | — | **110$** | **—** |
| Przejazd na czerwonym świetle | 80$ | — | **80$** | **—** |
| Niezatrzymanie się przed linią zatrzymania | 50$ | — | **50$** | **—** |
| Niezatrzymanie się przed przejściem dla pieszych | 80$ | — | **80$** | **—** |
| Nieustąpienie pierwszeństwa pieszemu | 120$ | — | **120$** | **—** |
| Nieustąpienie pierwszeństwa pojazd. uprzywilejowanemu | 80$ | — | **80$** | **—** |
| Niezastosowanie się do znaku STOP | 60$ | — | **60$** | **—** |
| Niezastosowanie się do polecenia funkcjonariusza | 120$ | — | **120$** | **—** |
| Jazda pod prąd | 60$ | — | **60$** | **—** |
| Brak zapiętych pasów bezpieczeństwa | 30$ | — | **30$** | **—** |
| Używanie telefonu podczas jazdy | 50$ | — | **50$** | **—** |
| Brawurowa / niebezpieczna jazda | 160$ | 5 | **160$** | **5** |
| Niebezpieczna jazda z obrażeniami u innych | 600$ | 12 | **600$** | **12** |
| Niedozwolone wyprzedzanie | 80$ | — | **80$** | **—** |
| Jazda niepoprawnym pasem ruchu | 40$ | — | **40$** | **—** |
| DUI - BAC 0.08-0.14% (pierwsze) | 300$ | 6 | **300$** | **6** |
| DUI - BAC 0.15%+ (pierwsze) | 600$ | 5 | **600$** | **5** |
| DUI - drugie naruszenie | 600$ | 8 | **600$** | **8** |
| DUI - z wypadkiem bez obrażeń | 600$ | 5 | **600$** | **5** |
| DUI - z obrażeniami u innych | 1500$ | 12 | **1500$** | **12** |
| Odmowa badania alkotestem | 120$ | 5 | **120$** | **5** |
| Spowodowanie kolizji | 50$ | — | **50$** | **—** |
| Spowodowanie wypadku | 120$ | — | **120$** | **—** |
| Potrącenie pieszego | 160$ | — | **160$** | **—** |
| Ucieczka z miejsca wypadku - bez ofiar (hit and run) | 300$ | 5 | **300$** | **5** |
| Jazda pojazdem niezdatnym do ruchu | 160$ | — | **160$** | **—** |
| Niesprawne oświetlenie | 35$ | — | **35$** | **—** |
| Niedozwolony kolor świateł (niebieski/czerwony) | 120$ | — | **120$** | **—** |
| Niedozwolone przyciemnienie przedniej szyby | 60$ | — | **60$** | **—** |
| Niedozwolone modyfikacje pojazdu | 160$ | — | **160$** | **—** |
| Nadmierny hałas układu wydechowego | 50$ | — | **50$** | **—** |
| Jazda pojazdem niezarejestrowanym | 120$ | — | **120$** | **—** |
| Nieczytelne tablice rejestracyjne | 35$ | — | **35$** | **—** |
| Parkowanie przy czerwonym krawężniku | 50$ | — | **50$** | **—** |
| Parkowanie na przejściu dla pieszych | 60$ | — | **60$** | **—** |
| Parkowanie na miejscu dla niepełnosprawnych | 120$ | — | **120$** | **—** |
| Brak dokumentów pojazdu podczas kontroli | 30$ | — | **30$** | **—** |
| Niezgłoszenie broni podczas kontroli drogowej | 120$ | 6 | **120$** | **6** |
| Jazda bez kasku (motocykl / quad) | 50$ | — | **50$** | **—** |
| Niedopuszczalny lane splitting | 60$ | — | **60$** | **—** |
| Udział w nielegalnym wyścigu | 300$ | 5 | **300$** | **5** |
| Niezatrzymanie pojazdu do kontroli | 120$ | — | **120$** | **—** |
| Ucieczka pojazdem przed służbami | 300$ | 12 | **300$** | **12** |

#### Naruszenia gospodarcze

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Prowadzenie działalności bez rejestracji | 300$ | 6 | **300$** | **6** |
| Prowadzenie działalności bez licencji branżowej | 1500$ | 5 | **1500$** | **5** |
| Podanie fałszywych danych przy rejestracji | 600$ | 12 | **600$** | **12** |
| Prowadzenie działalności po cofnięciu licencji | 2500$ | 12 | **2500$** | **12** |
| Zaniżenie przychodów w deklaracji podatkowej | 2500$ | 12 | **2500$** | **12** |
| Ukrywanie przychodów / podwójna księgowość | 4000$ | 15 | **4000$** | **15** |
| Oszustwo gospodarcze na szkodę kontrahentów | 5000$ | 15 | **5000$** | **15** |
| Kartel / porozumienie antykonkurencyjne | 4000$ | 15 | **4000$** | **15** |
| Zatrudnianie bez umowy (praca na czarno) | 600$ | 6 | **600$** | **6** |

#### Okoliczności łagodzące

| Wykroczenie | Obecnie fine | Obecnie jail | **Ustaw fine** | **Ustaw jail** |
|---|---:|---:|---:|---:|
| Ulga mandatowa -250$ | -250$ | — | **-250$** | **—** |
| Ulga mandatowa -500$ | -500$ | — | **-500$** | **—** |
| Ulga mandatowa -750$ | -750$ | — | **-750$** | **—** |
| Ulga mandatowa -1000$ | -1000$ | — | **-1000$** | **—** |
| Ulga mandatowa -1500$ | — | — | **-1500$** | **—** |
| Ulga mandatowa -2000$ | — | — | **-2000$** | **—** |
| Ulga mandatowa -2500$ | -2500$ | — | **-2500$** | **—** |
| Ulga więzienna -1 miesiąc | — | -1 | **—** | **-1** |
| Ulga więzienna -6 miesięcy | — | -6 | **—** | **-6** |
| Ulga więzienna -12 miesięcy | — | -12 | **—** | **-12** |
| Ulga więzienna -24 miesiące | — | -24 | **—** | **-24** |
| Ulga więzienna -36 miesięcy | — | -36 | **—** | **-36** |
| Ulga więzienna -60 miesięcy | — | -60 | **—** | **-60** |
### Pełna tabela faktur EMS (76 pozycji)

Plik: `Config.Fines.ambulance` · split wystawiającego: **progresywny** (jak LSPD).

| Usługa | Obecnie | **Ustaw** |
|---|---:|---:|
| Złamanie palca dłoni | 480$ | **70$** |
| Złamanie śródręcza | 720$ | **80$** |
| Złamanie nadgarstka | 900$ | **100$** |
| Złamanie kości promieniowej | 1800$ | **225$** |
| Złamanie kości łokciowej | 1800$ | **225$** |
| Złamanie przedramienia (obie kości) | 2700$ | **275$** |
| Złamanie kości ramiennej | 1500$ | **175$** |
| Złamanie barku / obojczyka | 1320$ | **150$** |
| Złamanie palca stopy | 480$ | **70$** |
| Złamanie śródstopia | 900$ | **100$** |
| Złamanie kostki | 1200$ | **150$** |
| Złamanie kości piszczelowej | 2700$ | **275$** |
| Złamanie kości strzałkowej | 2100$ | **200$** |
| Złamanie podudzia (obie kości) | 3600$ | **350$** |
| Złamanie kolana | 3300$ | **325$** |
| Złamanie kości udowej | 4800$ | **475$** |
| Złamanie miednicy | 5200$ | **400$** |
| Złamanie żeber (za każde) | 750$ | **100$** |
| Złamanie kręgosłupa | 6800$ | **550$** |
| Złamanie czaszki | 6400$ | **500$** |
| Rana cięta | 720$ | **80$** |
| Rana kłuta | 1200$ | **150$** |
| Rana tłuczona | 600$ | **90$** |
| Rana szarpana | 1320$ | **150$** |
| Otarcia naskórka | 360$ | **50$** |
| Rana postrzałowa – wlot | 2100$ | **200$** |
| Rana postrzałowa – wylot | 2700$ | **275$** |
| Postrzał mnogi (za każdy) | 2400$ | **250$** |
| Oparzenie I stopnia | 1200$ | **150$** |
| Oparzenie II stopnia | 2100$ | **200$** |
| Oparzenie III stopnia | 3600$ | **350$** |
| Odmrożenia | 2400$ | **250$** |
| Zmiażdżenie kończyny | 4200$ | **425$** |
| RTG (jedna partia ciała) | 1500$ | **175$** |
| RTG rozszerzone | 2400$ | **250$** |
| USG | 1500$ | **175$** |
| Tomografia komputerowa | 3000$ | **300$** |
| Rezonans magnetyczny | 3500$ | **350$** |
| Badanie krwi | 600$ | **90$** |
| Badanie toksykologiczne | 900$ | **100$** |
| Badanie alkoholu | 480$ | **70$** |
| Badanie narkotykowe | 900$ | **100$** |
| Badania psychologiczne (na broń palną) | 10000$ | **800$** |
| Konsultacja psychologiczna | 3000$ | **300$** |
| Konsultacja psychiatryczna | 2700$ | **275$** |
| Konsultacja neurologiczna | 2400$ | **250$** |
| Konsultacja ortopedyczna | 2400$ | **250$** |
| Konsultacja chirurgiczna | 2700$ | **275$** |
| Konsultacja kardiologiczna | 2400$ | **250$** |
| Konsultacja internistyczna | 2100$ | **200$** |
| Podanie leków przeciwbólowych | 300$ | **40$** |
| Podanie leków uspokajających | 300$ | **40$** |
| Podanie antybiotyku | 300$ | **40$** |
| Podanie adrenaliny | 480$ | **70$** |
| Podanie narkozy | 720$ | **80$** |
| Opatrzenie rany | 480$ | **70$** |
| Szycie rany (za szew) | 360$ | **50$** |
| Nastawienie złamania | 2400$ | **250$** |
| Unieruchomienie kończyny | 3000$ | **300$** |
| Reanimacja (RKO) | 1000$ | **125$** |
| Intubacja | 1800$ | **225$** |
| Operacja chirurgiczna | 1500$ | **175$** |
| Zespół ratownictwa medycznego | 1500$ | **175$** |
| Wezwanie chirurga | 2400$ | **250$** |
| Wezwanie specjalisty | 2100$ | **200$** |
| Transport karetką | 2100$ | **200$** |
| Transport lotniczy | 7200$ | **600$** |
| Nieuzasadnione wezwanie EMS | 3000$ | **300$** |
| Fałszywe zgłoszenie | 4800$ | **475$** |
| Odmowa współpracy z EMS | 1500$ | **175$** |
| Utrudnianie czynności medycznych | 2400$ | **250$** |
| Ucieczka z miejsca leczenia | 4800$ | **475$** |
| Zniewaga funkcjonariusza EMS | 1500$ | **175$** |
| Agresja wobec personelu medycznego | 9000$ | **700$** |
| Zniszczenie mienia EMS | 3000$ | **300$** |
| Leczenie pod eskortą LSPD/LSSD | 2400$ | **250$** |
### Pełna tabela usług mechanika (8 pozycji)

Plik: `Config.Fines.mechanic` · split wystawiającego: **40%** flat (`finePercentForPlayer`).

| Usługa | Obecnie | **Ustaw** |
|---|---:|---:|
| Myjnia ręczna | 400$ | **80$** |
| Naprawa blacharsko-lakiernicza | 1000$ | **150$** |
| Kompletny zestaw naprawczy pojazdu | 1800$ | **275$** |
| Usługa holowania (stawka za km) | 400$ | **80$** |
| Usługa odblokowania pojazdu (palnik) | 1500$ | **225$** |
| Kompleksowa pomoc drogowa | 2000$ | **300$** |
| Przegląd techniczny pojazdu | 1000$ | **150$** |
| Tankowanie paliwa (dojazd) | 300$ | **60$** |

## 8. Frakcje — FIB

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | `police`, `fib` | 250$/15 min | 168–175$/15 min (672–700$/h) | Planowany |
| `[core]/es_extended/server/paycheck.lua` | `ambulance` | 250$/15 min | 115–125$/15 min (460–500$/h) | Planowany |
| `[core]/es_extended/server/paycheck.lua` | `doj` | 250$/15 min | 150–158$/15 min (600–632$/h) | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.fib` | `policeFines` | ten sam katalog po przeskalowaniu | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` (fib) | 25% flat | progresywny (jak police) | Planowany |

---

## 9. Frakcje — EMS

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | Kursy lokalne `rewardMoneyMin/Max` | 350–850$ | 200–500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | LocalMedic pricing | 50–10000$ dynamiczny | 500–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | `HealthInsurance.Prices` | 3500 / 6000 / 12000$ | 800–1200 / 1400–2000 / 2500–3500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | `respawnFine`, `respawnFineWhileMedicsOnline` | 500 / 5000$ | 150–300 / 800–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/server/BabiczDeath_sv.lua` | Revive split (135% firma, 10% DOJ) | Procenty | Bez zmian struktury | Planowany |

### Faktury EMS w MDT

Pełna tabela **76 pozycji** z kolumną **Ustaw**: **§7** (sekcja „Pełna tabela faktur EMS”). Regeneracja: `python docs/_gen_mdt_fines_table.py`.

---

## 10. Frakcje — DOJ i podatki

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/company_functions/server/BabiczCompanyShop_sv.lua` | Split sprzedaży | 90% firma / 10% DOJ | Bez zmian (10% na society DOJ) | Planowany |
| `[rage]/rage_mdt/Config.lua` | Split mandatów → DOJ | 10% części society | Bez zmian | Planowany |
| `[rage]/rage_bossmenu/BabiczBossMenu_sv.lua` | Premie tygodniowe — tier system | Ręczne wypłaty bossa | Implementacja tierów v2 (500–10000$, max 10k/os.) | Planowany |

---

## 11. Firmy prywatne — kursy i sklepy

**Koszty operacyjne świadomie wyższe** — składniki 40–65% ceny menu, DOJ 10%, paycheck pracowników 40–60$/h.

Moduły: `company_functions/server/BabiczCompanyCourses_sv.lua`, `BabiczCompanyShop_sv.lua` · configi per firma `Fractions/c_*/shared/`.

### Kursy firmowe (7 jobów)

| Plik / job | Nagroda / paczka | Max pkg | Society | Cooldown |
|---|---|---|---|---|
| `c_burgershot` … `mechanic` (config `C.Course.Reward`) | 400–800$ (Tattoo 450–850) | 10 | 50% | **brak** → **30–45 min** po kursie |
| `BabiczCompanyCourses_sv.lua` | logika wypłaty | — | `floor(reward×0.5)` | dodać per gracz |

**Docelowo:** **80–150$/paczka** (Tattoo **90–160$**) — § Firmy w `ekonomia-zmiany-v2.md`.

### Sklep klienta + depositProducts

| Plik | Mechanizm | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `BabiczCompanyShop_sv.lua` | Sprzedaż klientowi | 90% firma / 10% DOJ | bez zmian struktury | Planowany |
| `BabiczCompanyShop_sv.lua` | `depositProducts` | **150%** ceny menu → society | **100–110%** (po obniżeniu menu) | Planowany |
| `BabiczBurgerShot_config.lua` / `BabiczTequilala_config.lua` | `C.Shop` ceny menu | 31–56$ | cheeseburger **34$**, burgir **40$** | Planowany |
| `rage_fractions/Config.lua` | `c_*_ingredients_shop` | meat 13$, cheese 5$… | **40–65%** docelowej ceny menu | Planowany |
| `rage_fractions/Config.lua` | `c_*_shop` (ecola/sprunk) | 18$ | spójne z market v2 | Planowany |
| `[core]/es_extended/server/paycheck.lua` | Paycheck `c_*` | **250$/15 min** (1000$/h) | **10–15$/15 min** (40–60$/h), on duty | Planowany |
| `ox_inventory/data/crafting.lua` | burgershot / tequilala craft | — | test po zmianie składników | Planowany |
| `.../c_blazingtattoo/...` | Faktury tatuaż, kurs | 750$/est.; 450–850$/pkg | §26 + v2 § Blazing Tattoo | Planowany |
| `.../c_fiverecords/...` | Kurs only | 400–800$/pkg | 80–150$/pkg | Planowany |

---

## 11A. `rage_fractions` — menu i składniki (pełna lista)


### Burger Shot — `BabiczBurgerShot_config.lua` → `C.Shop.items`

| Item | Obecnie | Docelowo |
|---|---|---|
| mint_lemonade | 34 | 13 |
| orange_juice | 32 | 12 |
| apple_juice | 31 | 12 |
| banana_smoothie | 39 | 14 |
| tropical_juice | 36 | 14 |
| veegi_burger | 43 | 32 |
| becon_burger | 56 | 35 |
| cheeseburger | 48 | 34 |
| frytki | 38 | 27 |
| schakemalinowy | 46 | 32 |

### Tequi-la-la — `BabiczTequilala_config.lua` → `C.Shop.items`

| Item | Obecnie | Docelowo |
|---|---|---|
| woda_po_studencie | 34 | 14 |
| bum | 36 | 20-38 |
| wino_z_kartonu | 36 | 16 |
| hot_cat | 43 | 38 |
| burgir | 48 | 40 |
| woda_alko | 46 | 20-38 |
| szampon | 56 | 28 |
| rozwod | 56 | 20-38 |

### Składniki — `rage_fractions/Config.lua`

**Docelowa zasada:** składniki = **40–65%** docelowej ceny menu (nie 150% przy `depositProducts`).

| Sklep | Item | Obecnie | Docelowo (przykład) |
|---|---|---|---|
| c_burgershot_ingredients | meat | 13 | 12-14 |
| c_burgershot_ingredients | cheese | 5 | 5-6 |
| c_burgershot_ingredients | tortilla | 5 | 5 |
| c_burgershot_ingredients | potato | 6 | 6-8 |
| c_tequilala_ingredients | beer | 12 | 4-6 |
| c_tequilala_ingredients | whisky | 18 | 12-18 |
| c_*_shop | ecola/sprunk | 18 | 7-10 |

### Kursy firm — `C.Course.Reward` we wszystkich `c_*_config.lua`

| Job | Obecnie min-max | Docelowo | CD | Society split |
|---|---|---|---|---|
| c_burgershot | 400-800 | 80-150 | 30-45 min | 50% |
| c_tequilala | 400-800 | 80-150 | 30-45 min | 50% |
| c_fiverecords | 400-800 | 80-150 | 30-45 min | 50% |
| c_quiettech | 400-800 | 80-150 | 30-45 min | 50% |
| c_blackrepair | 400-800 | 80-150 | 30-45 min | 50% |
| c_blazingtattoo | 450-850 | 90-160 | 30-45 min | 50% |
| mechanic (LSC) | 400-800 | 80-150 | 30-45 min | 50% |

### `BabiczCompanyShop_sv.lua` — hardcoded

| Mechanizm | Obecnie | Docelowo |
|---|---|---|
| Sprzedaż klientowi → society | 90% | 90% |
| Sprzedaż → DOJ | 10% | 10% |
| `depositProducts` | **×1.5** menu | **×1.0–1.1** |

### Frakcje publiczne — sklepy (`rage_fractions/Config.lua`)

Paycheck **nie** jest w tym pliku — §1 `paycheck.lua`.

| Sklep | Item (przykłady) | Obecnie | Docelowo v2 |
|---|---|---:|---|
| `police_shop` / `fib_shop` | gps, radio, handcuffs, bodycam | 10$ | bez zmian |
| | blowtorch | 5000$ | 800–1500$ |
| | arplate_light / heavy | 3500 / 4500$ | 1500–3000$ |
| | police_stormram | 5000$ | 1500–3000$ |
| `police_shop_weapons` | ammo-9 | 2$ | bez zmian |
| | at_suppressor_light | 20000$ | zsynchronizować z Ammunation |
| `ambulance_shop` | bandage / firstaidkit / defibrillator | 100 / 400 / 800$ | bez zmian |
| | stethoscope | 20$ | bez zmian |
| `doj_shop` | radio, gps, flashlight | 10–100$ | bez zmian |

MDT, dispatch, faktury — §7–9.


---

## 12. Mechanik (frakcja)

Jobs: `mechanic`, `c_blackrepair`, `c_quiettech` · tuning w `BabiczTuningMenu_*`, laweta `BabiczFlatbed_*`.

### Paycheck i NPC

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | `mechanic` on duty | 125$/15 min (cywil) | **168–175$/15 min** (672–700$/h) | Planowany |
| `BabiczMechanic_shared.lua` → `LocalMechanic` | `price` / `mechanicsOnlinePrice` | 1000$ / 5000$ | 500–800$ / 1000–1500$ | Planowany |

### Tuning — ceny i split

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `BabiczTuningMenu_sv.lua` | `GetModPrice` (flaty) | nitro 75k, parachute 1.2M… | **×0,15** po `vehicles.json` | Planowany |
| `BabiczTuningMenu_sv.lua` | `ShopPricePercents` | performance **5%** | **2%** po deflacji aut | Planowany |
| `BabiczTuningMenus_cl.lua` | `shopPricePercent` w menu | 5% / 0,25% | zsynchronizować z server | Planowany |
| `BabiczTuningMenu_sv.lua` | Split invoice | 35% + 10% DOJ + **10% mechanik** | bez zmian struktury | Planowany |
| `rage_mdt/Config.lua` | `finePercentForPlayer` LSC | **25%** | **40%** (jak sub-shopy) | Planowany |
| `rage_mdt/Config.lua` | `Config.Fines.mechanic` | 300–2000$ | 60–700$ (skala v2) | Planowany |

### Kursy i laweta

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `BabiczMechanic_shared.lua` → `C.Course.Reward` | Kurs paczkowy LSC | 400–800$/pkg | **80–150$/pkg** + cooldown | Planowany |
| `BabiczCompanyCourses_sv.lua` | Cooldown kursu | brak | 30–45 min per gracz | Planowany |
| `BabiczFlatbed_*` + `LocalMechanic.Locations` | **Kurs lawetą** (nowy) | brak ekonomii | 150–350$/kurs, 50% society, CD 15–20 min | **Planowany (implementacja)** |
| `BabiczMechanic_sv.lua` | `towVehicle` | 0$ dla mechanika | **0$** — narzędzie frakcyjne; holowanie MDT = osobna płatna usługa (80$/km, 40% FP) | Gotowe do wdrożenia |

### Sklep frakcji

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `rage_fractions/Config.lua` → `mechanic.Shops` | fixkit, carokit, blowtorch | 1500 / 800 / 4500$ | 250–600 / 400–800 / 800–1500$ | Planowany |
| `rage_market/Config.lua` | fixkit cywil | 5000$ | 400–800$ | Planowany |

### Pełna tabela MDT usług mechanika (8 pozycji)

Generator: [`_gen_mdt_fines_table.py`](_gen_mdt_fines_table.py) · [`_mdt_mechanic_fines_v2_table.md`](_mdt_mechanic_fines_v2_table.md)

Algorytm mechanik: ≤500$ ×0,20; ≤2000$ ×0,15; powyżej ×0,12. LSC `finePercentForPlayer`: docelowo 40% (jak sub-shopy).

#### Usługi mechaniczne

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Myjnia ręczna | 400$ | **80$** |
| Naprawa blacharsko-lakiernicza | 1000$ | **150$** |
| Kompletny zestaw naprawczy pojazdu | 1800$ | **275$** |
| Usługa holowania (stawka za km) | 400$ | **80$** |
| Usługa odblokowania pojazdu (palnik) | 1500$ | **225$** |
| Kompleksowa pomoc drogowa | 2000$ | **300$** |
| Przegląd techniczny pojazdu | 1000$ | **150$** |
| Tankowanie paliwa (dojazd) | 300$ | **60$** |

---

## 13. Napady — aktywne

**Cooldown:** każdy typ napadu ma **osobny cooldown per gracz** — wymusza rotację (szczegóły: `ekonomia-zmiany-v2.md` § Cooldowny napadów).

### Mapowanie cooldownów (kod → docelowy)

| Heist | Plik | Obecny CD | Docelowy | Zmiana w kodzie |
|---|---|---|---|---|
| Fleeca | `Heists/Bank/server/BabiczBank_sv.lua` | **2 h** (`7200000`) per gracz | **15 min** (`900000`) | `SetTimeout(7200000)` → `900000` |
| Sejf sklepu | `Heists/ShopVault/server/BabiczShopVault_sv.lua` | **1 h** per gracz + 10 min global | **15 min** per gracz + 10 min global | `3600000` → `900000` |
| ATM hack | `Heists/ATM/server/BabiczATM_sv.lua` | brak per gracz; 2 min global `hackTimeout` | **10 min per gracz** | dodać `PlayerTimeouts[identifier.."_atm_hack"]` |
| Sejf — wymagania | `Heists/ShopVault/server/BabiczShopVault_sv.lua` | `stethoscope` durability −15 | bez zmian mechanizmu | opis w v2 §7 |

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_heists/Config.lua` | Cooldown per heist / per gracz | częściowo | Osobny CD na każdy typ napadu | Planowany |
| `[rage]/rage_heists/Config.lua` | `AttackerMultipliers` | Agresywne (bank ×2.5); brak `trucker` | Mnożniki v2 + **`trucker`** §14a | Planowany |
| `[rage]/rage_heists/Heists/NPC/shared/BabiczNPC_config.lua` | Gotówka, loot | money 45–195$ | 25–150$ + biżuteria niska | Planowany |
| `[rage]/rage_heists/Heists/NPC/server/BabiczNPC_sv.lua` | Cooldown między napadami | `SetTimeout(15000)` (**15 s**) | **`SetTimeout(180000)`** (3 min) per `source` | **Bloker §0** |
| `[rage]/rage_heists/Heists/ATM/shared/BabiczATM_config.lua` | Karta / hack | 500–2500$ / 200–500$ | 100–350$ / 200–700$ | Planowany |
| `[rage]/rage_heists/Heists/ShopKeeperCashRegister/shared/BabiczShopKeeperCashRegister_config.lua` | Kasetka | 2500–5000$ | 250–600$ | Planowany |
| `[rage]/rage_heists/Heists/ShopVault/shared/BabiczShopVault_shared.lua` | Sejf + Sandy bonus | 15000–40000$; +5000 Sandy | 800–2200$; **usunąć** `rewardBonus` Sandy | Planowany |
| `[rage]/rage_heists/Heists/Bank/shared/BabiczBank_shared.lua` | Fleeca | 60000–110000$ | 4500–12000$ | Planowany |
| `[rage]/rage_heists/Heists/BankPacific/shared/BabiczBankPacific_config.lua` | `minGroup` | 1 | **4** | Planowany |
| `[rage]/rage_heists/Heists/BankPacific/server/BabiczBankPacific_sv.lua` | **Wypłata lootu** | **brak** `addAccountMoney` / itemów | Patrz **§13a** poniżej | **Bloker §0** |

### 13a. Bank Pacific — implementacja wypłaty (server)

Gameplay jest aktywny; **ekonomia ustalona w v2 §12**. Do dopisania w `BabiczBankPacific_sv.lua` po sukcesie napadu:

```lua
-- Pseudokod — po zakończeniu flow, przed resetem heistu:
local total = math.floor(math.random(35000, 90000) * (Config.AttackerMultipliers.bank_pacific[attackersCount] or 1))
local blackShare = math.floor(total * 0.70)
local jewelryValue = total - blackShare  -- itemy z puli Config.Pawnshop / biżuteria v2
-- Podział blackShare między uczestników (równe lub wg udziału w heist)
-- jewelryValue → losowe itemy (necklace_gold, diamond…) o łącznej wartości skupu ≈ jewelryValue
-- PlayerTimeouts[identifier.."_pacific"] = os.time(); SetTimeout(5400000, ...)  -- 90 min CD per gracz
```

| Parametr | Wartość |
|---|---|
| Łup łączny | 35 000–90 000$ × mnożnik grupy |
| Skład | ~70% `black_money`, ~30% biżuteria (lombard) |
| CD per gracz | 60–90 min (`PlayerTimeouts`, klucz `_pacific`) |
| `minGroup` | 4 w shared config |

---

## 14. Napady — planowane (implementacja)

| System | Plik | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| Trucker (napad — ciężarówka) | (nowy / `rage_heists`) | komentarz 20–50k | **3500–8000$** black; 1–3 os.; CD 35–45 min | Planowany |
| Boosting | (nowy / heists) | — | 350–900$, solo | Planowany |
| Tracker | `[rage]/BabiczTracker/Config.lua` | neededMoney 500$; rewards 500–5000$ | start 200–400$; nagroda 500–1400$ | Planowany |
| Jubiler | (nowy) | — | 2800–6500$ wartości skupu, 2–5 os. | Planowany |
| Lombard (napad) | (nowy) | — | 600–1800$ + biżuteria, 1–3 os. | Planowany |
| Jacht | (nowy) | — | 10000–25000$, 3–5 os. | Planowany |
| Humane Labs | (nowy) | — | 22000–55000$, 3–5 os. | Planowany |
| Cayo Perico | (nowy) | — | 70000–130000$, 5–8 os. | Planowany |

### 14a. Napad truckera (dostawa ciężarówki)

**≠ KQ Trucker (RS Haul)** — to crime w `rage_heists`, nie job z `kq_deliveries`.

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_heists/Config.lua` | `AttackerMultipliers.trucker` | brak w aktywnym Config | `{ [1]=1.0, [2]=1.15, [3]=1.25 }` | Planowany |
| `[rage]/rage_heists/Config.lua` | Komentarz BASE_LOOT #9 | 20 000–50 000$ | **3500–8000$** (dokumentacja) | Planowany |
| `[rage]/rage_heists/Heists/Trucker/` (nowy) | Flow + wypłata | brak | Po dostawie: `addAccountMoney("black_money", reward)` | Planowany |

**Wzór wypłaty (server, po dostarczeniu ciężarówki):**

```lua
local count = attackersInRange -- 1..3
local total = math.floor(math.random(3500, 8000) * (Config.AttackerMultipliers.trucker[count] or 1))
local share = math.floor(total / count)
-- każdy uczestnik: addAccountMoney("black_money", share)
-- PlayerTimeouts[identifier.."_trucker"] + SetTimeout(45*60000, ...)
```

| Parametr | Wartość |
|---|---|
| Łup łączny | 3500–8000$ × mnożnik grupy |
| Na osobę (solo) | pełna kwota (3500–8000$) |
| Na osobę (3 os., baza 5500$) | ~1458–3333$ |
| CD per gracz | 35–45 min |
| Policja min. | 2 |

Karta balansu: `ekonomia-zmiany-v2.md` §9b.

---

## 15. Lombard (skup) — pełna tabela

**Plik:** `[rage]/RageCity/Config.lua` → `Config.Pawnshop` · dark: `Config.Pawnshop.Black.multiplier = 0.95` · czas sprzedaży `time` 900–1280 ms — **bez zmian**.

| Item | Obecnie min–max | Docelowo min–max |
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
| phone | 750–1250$ | 40–70$ (dark_lombard) |
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

| Plik | Co zmienić | Status |
|---|---|---|
| `Config.Pawnshop.Items` | Wszystkie pozycje wg tabeli | Planowany |
| `Config.Pawnshop.Black.multiplier` | 0.95 | Gotowe |
| `server/platetape.lua` | animacja sprzedaży | Gotowe |

---

## 16. Narkotyki i sprzedaż — pełna tabela

**Plik:** `[rage]/RageCity/Config.lua` → `Config.Drugs` · wypłata: `black_money` w `server/drugsales.lua` (mnożnik jakości: `0.5 + 0.5 × quality/100`).

| Item | Ilość | Szansa % | Obecnie $/szt. | Docelowo $/szt. |
|---|---|---|---|---|
| joint | 1-2 | 70 | 75-150 | 25-45 |
| magic_mushrooms_dry | 1-8 | 65 | 87-147 | 20-40 |
| weed_packed | 1-3 | 65 | 243-303 | 55-85 |
| kq_weed_joint_og_kush | 1-4 | 67 | 175-235 | 40-65 |
| kq_weed_joint_purple_haze | 1-4 | 67 | 240-300 | 55-80 |
| kq_weed_joint_white_widow | 1-4 | 67 | 250-330 | 60-85 |
| kq_weed_joint_blue_dream | 1-4 | 67 | 280-360 | 65-95 |
| heroin_opium | 1-4 | 60 | 339-395 | 70-100 |
| coke_packed | 1-4 | 55 | 431-541 | 90-130 |
| meth_packed | 1-5 | 50 | 455-570 | 95-140 |
| heroin_packed | 1-3 | 55 | 545-605 | 100-145 |
| kq_weed_bag_og_kush | 1-3 | 65 | 370-480 | 80-120 |
| kq_weed_bag_purple_haze | 1-3 | 65 | 480-580 | 100-150 |
| kq_weed_bag_white_widow | 1-4 | 65 | 500-640 | 110-160 |
| kq_weed_bag_blue_dream | 1-3 | 65 | 560-700 | 120-175 |
| coke | 1-2 | 60 | 750-950 | 130-190 |
| coke_bag | 1-3 | 65 | 850-1050 | 150-220 |

### `Config.DrugSelling` — parametry

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| copsRequired | 1 | **2** | Config.lua |
| callCopsChance | 65 | 65 | Config.lua |
| Zones | 1 × radius 2600 | bez zmian | Config.lua |
| blacklistedJobs | police, doj, fib | bez zmian | Config.lua |
| avatarChance | 35 | bez zmian | Config.lua |

### Czasy (`drugsales.lua` / client)

| Etap | Obecnie | Docelowo |
|---|---|---|
| Progress bar sprzedaży | 5 s | bez zmian |
| Server ready po akceptacji | 4.5 s | bez zmian |
| Timeout sesji | 30 s | bez zmian |
| Heat decay | -2 / 120 s | bez zmian |
| Pack / joint | 5 s | bez zmian |

### `Config.RicoDrugBricks` — przeliczyć po rescale `Config.Drugs`

Formuła: `avg(Config.Drugs[source].price) × requiredCount × 0.9` (±5%). Przykład po v2: `weed_brick` (1000× weed) → **~50-85k$** zamiast 47-52k przy starych cenach.


### KQ weed — `[tebex]/kq_weed/config.strains.lua`

| Odmiana | Parametr | Obecnie | Docelowo |
|---|---|---|---|
| og_kush | baseGrowTime | 20 | 28 |
| og_kush | wateringTime | 9 | 10 |
| og_kush | collectionLifespan | 25 | 25 |
| og_kush | yield.buds min-max | 6-8 | 4-5 |
| purple_haze | baseGrowTime | 25 | 32 |
| purple_haze | wateringTime | 12 | 12 |
| purple_haze | collectionLifespan | 30 | 28 |
| purple_haze | yield.buds min-max | 5-6 | 4-5 |
| white_widow | baseGrowTime | 30 | 36 |
| white_widow | wateringTime | 15 | 14 |
| white_widow | collectionLifespan | 20 | 22 |
| white_widow | yield.buds min-max | 5-6 | 3-4 |
| blue_dream | baseGrowTime | 35 | 40 |
| blue_dream | wateringTime | 18 | 16 |
| blue_dream | collectionLifespan | 60 | 35 |
| blue_dream | yield.buds min-max | 5-7 | 3-5 |

**Bez zmian:** `budsPerBaggie = 1`, `budsPerBrick = 20`, `config.fertilizers.lua` mnożniki.


### `rage_drugs/Config.lua` — zbiory i craft

**Status: Planowany** — obecne mnożniki i krótkie `spawnDelay` pozwalają farmić **2–4×** szybciej niż norma v2; wdrożyć tabelę z `ekonomia-zmiany-v2.md` § Produkcja narkotyków.

| Strefa | Parametr | Obecnie | Docelowo |
|---|---|---|---|
| Weed harvest | duration | 8500 | 10000–11000 |
| Weed harvest | spawnDelay | 25–30 | 35–40 |
| Weed harvest | multiplier | 1–2 / 1–3 | 1–2 |
| Weed process | multiplier | 1–4 | 1–2 |
| Weed process | duration | 8500 | 10000 |
| Koka harvest | duration / spawnDelay | 11500 / 30 | 12000–13000 / 40–45 |
| Koka process | multiplier | 1–3 | 1–2 |
| Grzyby harvest | spawnDelay | 15 | 25–30 |

**Norma QA:** weed pole **700–1000 $/h** · koka **850–1200 $/h** (pełny łańcuch + ulica).

### `rage_meth/` — lab mobilny

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `Config.lua` → `NeededItems.acetone` | koszt wejścia | 5 | **6–8** | Planowany |
| `BabiczMeth_sv.lua` | yield / cook | random(5,15) | **random(3,8)** | Planowany |
| `Config.lua` | accetonTime, wearMask, nauseaTime | 20000 | **25000–30000** | Planowany |

**Norma QA:** pełny cykl **≥25 min** · **900–1400 $/h** po sprzedaży (E12).

---

## 17. Pralnia

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `MoneyWash.Automatic` | Strata, prędkość | 30% strata; 175$/s | 35–45% strata; 25–75$/s | Planowany |
| `[rage]/RageCity/server/moneywash.lua` | Trasa ręczna | 3000–7500$/stop; 5% strata | **`math.random(350, 900)`**/stop; strata **20–30%** | Planowany |

---

## 18. Zadania Moris — progresja crime

### 18a. Fernando (tutorial)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Tasks/Moris/shared/1_Fernando.lua` | `moneyAmount` (zad. 2) | 25000$ dirty | 4000–6000$ | Planowany |
| `[rage]/RageCity/Tasks/Moris/server/1_Fernando.lua` | Wypłaty zad. 1–4 | 250–12000$ mixed | 80–2500$ wg tabeli v2 | Planowany |
| `[rage]/RageCity/Tasks/Moris/client/1_Fernando.lua` | Teksty UI (kwoty) | hardcoded 25000 | zsynchronizować z shared | Planowany |

### 18b. Carlos (pojazdy / przemyt)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `.../shared/2_Carlos.lua` | Zad. 1 rozwóz — `t2` / `finishReward` | 10 000–15 000$ | 40–80$ dirty/stop × 9; CD 45 min | Planowany |
| `.../shared/2_Carlos.lua` | Zad. 2 boosting — `t1` `vehicles[].reward` | 3200–13000$ clean | 350–900$ tier auta; CD 45 min | Planowany |
| `.../server/2_Carlos.lua` | Wypłata boosting `t1` | pełna kwota z config | rescale + kara −20% uszkodzenie | Planowany |
| `.../shared/2_Carlos.lua` | **Zad. 3 tracker — `t3`** | patrz tabela poniżej | rescale wg v2 § Carlos zad. 3 | Planowany |
| `.../server/2_Carlos.lua` | Wypłata tracker `t3` | `addAccountMoney("black_money")` | bez zmian typu; kwoty z config | Planowany |
| `.../server/2_Carlos.lua` | `BabiczTasks:buyTrackerItem` | lockpick 1320$ · hack 4400$ | 200–300$ · 450–650$ | Planowany |
| `.../shared/2_Carlos.lua` | Zad. 4 przemyt — `t4` | (brak) | 900–1600$ black; CD 90 min | Planowany |
| `[rage]/BabiczTracker/` | Heist powtarzalny | 500–5000$; opłata 500$ | wypłaty jak v2 napad #6; quest bez opłaty | Planowany |
| `.../client/2_Carlos.lua` | UI / SMS / etapy | t1 boosting + **t3 tracker** aktywne | t2 rozwóz + t4 przemyt | Planowany |

**`TaskConfig[t3]` — tracker (obecny kod vs docelowo):**

| Klucz | Obecnie | Docelowo |
|---|---|---|
| `cooldown` | `45 * 60` | `60 * 60` |
| `copsNeeded` | `0` | `1` |
| `lockpickPrice` | `1320` | `200–300` |
| `hackingdevicePrice` | `4400` | `450–650` |
| `deactivateTime` | `20` (test) | `180–240` |
| `vehicles[].reward` | 7500–16500$ | **500–1200$** per tier (tabela w v2) |

**Uwaga:** boosting w kodzie to `Moris_Carlos_1` (`t1`), tracker to `Moris_Carlos_3` (`t3`) — numeracja fabularna ≠ indeksy `t1`–`t4` (patrz mapowanie w v2 § Carlos).

### 18c. Moris (haracz / fabuła)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `.../shared/3_Moris.lua` | Config zadań 1–3 | (brak) | bonusy wg tabeli v2 | Planowany |
| `.../server/3_Moris.lua` | Wypłaty, karta debetowa, broń | (brak) | zad.1: bonus 300–600$ + ATM 400–800$; pistolet 1200–2000$ | Planowany |
| `.../client/3_Moris.lua` | Menu / etapy | Coming soon | haracz → syzyf → pogróżki | Planowany |
| `rage_heists` kasetka | Loot w quest Moris 1/3 | 250–600$ | bez zmian skali; osobny flag quest | Planowany |
| `rage_heists` lombard | Loot w quest Moris 2 | planowany 600–1800$ | biżuteria flavor bez itemu | Planowany |
| Sklep XYZ | Ten sam los w zad. 1 i 3 | — | persist per gracz w `tasks` | Planowany |

| Zadanie | Bonus quest | Loot napadu | Cooldown quest |
|---|---|---|---|
| 1 Pierwszy haracz | 300–600$ clean | kasetka + karta 400–800$ | 24 h |
| 2 Syzyfowa robota | 500–900$ black | lombard 600–1800$ + biżuteria flavor | 24 h |
| 3 Śmieszne pogróżki | 400–700$ black | kasetka; hack = flavor | 24 h (po zad. 1) |

---

## 19. Sklepy, jedzenie i picie

> **Pełny katalog `rage_market` (wszystkie itemy):** §19A poniżej.

Źródło statystyk: `[rage]/rage_hud/Config.lua` → `Config.Status`, `Config.Food`.  
**Budżet żywienie @ 500$/h: 45$/h** (profil Minimum: 4× woda + 2× kanapka + 1× chips).  
**Koszt życia łącznie @ 500$/h: 80–115$/h** (16–22% dochodu) — § Koszty życia v2.

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_market/Config.lua` → `Market` | Napoje/jedzenie sklep | woda 20$; sandwich 36$ | woda 7$; sandwich 7$; chips 3$ (loadout 45$/h) | Planowany |
| `[rage]/rage_market/Config.lua` → `Liquor` | Alkohol | beer 45$; whisky 300$ | poza kosztami życia — tabela v2 | Planowany |
| `[core]/ox_inventory/data/shops.lua` | Automaty | woda 18$; ecola 22$ | jak sklep (woda 7$) | Planowany |
| `[rage]/rage_fractions/.../BabiczBurgerShot_config.lua` | Menu Burger Shot | cheeseburger 48$ | 34$ (~0,55$/pkt) | Planowany |
| `[rage]/rage_fractions/.../BabiczTequilala_config.lua` | Menu Tequi-la-la | burgir 48$ | 40$ (premium RP) | Planowany |
| `[core]/ox_fuel/config.lua` | Paliwo | priceTick 8$ | 2–4$ (składnik kosztów życia 20–30$/h) | Planowany |
| `[rage]/rage_market/Config.lua` | Narzędzia, dark shop | lockpick 1200$ | poza bieżącymi kosztami życia | Planowany |
| `[rage]/rage_market/Config.lua` | Zdrapki (`scratchticketn/p/d`) | 1000 / 3500 / 10000$ | **75 / 200 / 500$** | Planowany |
| `[rage]/rage_scratchcard/server.lua` | `chance`, `price`, `multiplier` | max wygrana miliony | BigWin 12% → 400–800$; Premium 8% → 1–2k$; Deluxe 4% → 5–8k$ | Planowany |
| `[rage]/BabiczAmmunation/Config.lua` | Bronie legalne | 80000–280000$ | 8000–300000$ (tabela v2) | Planowany |

---

## 19A. `rage_market` — pełny katalog cen

**Plik:** `[rage]/rage_market/Config.lua` → `Items.*` · limity globalne w `BabiczMarket_sv.lua`.


### Market 24/7 (`Items.Market`)

| Item | Obecnie | Docelowo |
|---|---|---|
| water | 20 | 7 |
| ecola | 22 | 7 |
| sprunk | 22 | 7 |
| coffee | 90 | 12 |
| junkenergy | 120 | 14 |
| sandwich | 36 | 7 |
| chips | 37 | 3 |
| donut | 34 | 3 |
| hotdog | 38 | 6 |
| phone | 1500 | 500-1000 |
| lighter | 15 | 15 |
| cigarette | 10 | 10 |
| scratchticketn | 1000 | 75 |
| scratchticketp | 3500 | 200 |
| scratchticketd | 10000 | 500 |
| beer | 46 | 10 |
| vine | 185 | 22 |
| potato | 8 | 3 |
| sirloin_steak | 75 | 25 |
| farming_chickenbreast | 28 | 10 |
| meat | 30 | 12 |
| fish | 20 | 8 |
| bread | 7 | 2 |
| tomato | 10 | 4 |
| cheese | 15 | 6 |
| egg | 9 | 3 |
| raw_bacon | 18 | 7 |
| rice | 11 | 4 |
| cucumber | 9 | 3 |
| seaweed | 12 | 5 |
| sausage_raw | 19 | 7 |
| apple | 8 | 3 |
| banana | 8 | 3 |
| lemon | 9 | 3 |
| mint | 8 | 3 |
| orange | 10 | 4 |
| tortilla | 12 | 5 |

### Liquor (`Items.Liquor`)

| Item | Obecnie | Docelowo |
|---|---|---|
| beer | 45 | 10 |
| rhum | 120 | 45 |
| vine | 180 | 22 |
| champagne | 200 | 30 |
| whisky | 300 | 55 |
| vodka | 250 | 45 |

### Sklep techniczny (`Items.Tools`)

| Item | Obecnie | Docelowo |
|---|---|---|
| fixkit | 5000 | 250-600 |
| carokit | 1200 | 400-800 |
| carjack | 300 | 80-150 |
| spare_wheel | 250 | 60-120 |
| nitro | 10000 | 2000-4000 |
| platetape | 3000 | 600-1200 |
| handcuffs | 5000 | bez zmian |
| lockpick | 1200 | 80-200 |
| phone | 1250 | 400-700 |
| radio | 850 | 300-700 |
| scuba | 1600 | 400-800 |

### Dark Shop (`Items.DarkShop`)

| Item | Obecnie | Docelowo | Uwagi |
|---|---|---|---|
| key (black market) | 100000 | 8000-15000 | limit 1/5d |
| key (Rico) | 50000 | 3000-6000 |  |
| stethoscope | 60 | 60 | już OK |
| hackingdevice | 5000 | 400-800 |  |
| drill | 12500 | 1500-3000 |  |
| hackingtablet | 10000 | 800-1500 |  |
| thermite | 2000 | 400-700 |  |
| WEAPON_PISTOL (dirty) | 100000-150000 | 12000-25000 | black_money + pool |
| ammo-9 x18 | 550 | 80-150 | black_money |
| kq_weed_fertilizer_blackmarket | 1200 | 300-500 |  |

### Ammunation (`Items.Ammunation`) — skrót

| Item | Obecnie | Docelowo |
|---|---|---|
| vest | 249-250 | 150-300 |
| arplate_light | 5000 | 1500-3000 |
| WEAPON_SWITCHBLADE | 1200 | 400-1200 |
| WEAPON_BAT | 1800 | 400-1200 |
| WEAPON_KNIFE | 2400 | 400-1200 |
| WEAPON_PISTOL | 90000 | 8000-15000 |
| WEAPON_SNSPISTOL | 140000 | 8000-15000 |
| WEAPON_PISTOL50 | 160000 | 15000-25000 |
| ammo-9 x20 | 600 | 5-35/szt. |
| at_flashlight | 2500 | bez zmian |

### Weed shop (`Items.Weed`)

| Item | Obecnie | Docelowo |
|---|---|---|
| empty_baggy | 20 | 20 |
| rolling_paper | 15 | 15 |
| kq_weed_seed_* | 200 | 150-250 |
| kq_weed_pot | 1000 | 400-700 |
| kq_weed_tent | 12000 | 3000-6000 |
| kq_weed_table | 10000 | 2500-5000 |
| kq_weed_press | 50000 | 12000-20000 |
| kq_weed_watering_system | 14000 | 3500-7000 |
| kq_weed_fertilizer_speed/yield | 700 | 300-500 |
| kq_weed_fertilizer_resistance | 800 | 350-550 |
| kq_weed_pesticide_spray | 45 | 45 |

### Nightclub / CasinoVIP / Pharmacy / BlackMarket

Nightclub: beer 60→15-25$, whisky 400→80-120$. CasinoVIP: beer 85→20-30$, whisky 560→100-150$. Pharmacy: bandage 250→bez zmian, cpr 2500 limit 1, stethoscope 80→80. BlackMarket: hackingdevice 4000→400-800, hackingtablet 8000→800-1500, key cocaine lab 75000→8000-15000.


---

## 20. Paliwo (`ox_fuel`)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/ox_fuel/config.lua` | `priceTick` | 8$ | 2–4$ | Planowany |
| `[core]/ox_fuel/config.lua` | `petrolCan.price` | 1500$ | 400–700$ | Planowany |
| `[core]/ox_fuel/config.lua` | `petrolCan.refillPrice` | 1000$ | 250–500$ | Planowany |
| `[core]/ox_fuel/config.lua` | `petrolCan.ecoPetrolCanPrice` | 800$ | 200–400$ | Planowany |

---

## 21. Pojazdy i salon

**Bloker §0 pkt 9** — bez rescale `vehicles.json` progresja auta nie ma sensu po deflacji zarobków.

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_vehicleshop/vehicles.json` | Cena per model (~10k linii) | Setki tys. – 20M+ | Segmenty v2: osobowe max **1,2 mln**, większość 300–700k | **Wymagane w deploy** |
| `[rage]/rage_vehicleshop/BabiczVehicleShop_cl.lua` | `VehiclePriceConfig.types` | min/max obecne | min/max v2 | Planowany |
| `[rage]/rage_vehicleshop/Config.lua` | `Config.Prices.customPlate` | 750000$ | 15000–40000$ | Planowany |
| `[rage]/rage_vehicleshop/Config.lua` | `testDrivePrice` | 80$ | 50–100$ | Planowany |

Po rescale aut — tuning (`BabiczTuningMenu_sv.lua`): flaty **×0,15**; `shopPricePercent` performance **2%**.

### 21a. Skrypt konwersji `vehicles.json`

**Kroki:**

1. Backup `vehicles.json` + commit przed konwersją.
2. Dla każdego modelu: `newPrice = min(max(round(oldPrice * 0.08), typeMin), typeMax)` — typ z pola `category` / mapowania na `VehiclePriceConfig.types`.
3. Cap automobile: **max 1 200 000$**; ręcznie obniżyć outliery > cap.
4. Zapis JSON; restart resource / cache vehicleshop.
5. **Weryfikacja:** 10 losowych modeli + najtańszy + najdroższy w DB; porównanie z segmentami v2.

**Pseudokod (Python):**

```python
import json
from pathlib import Path

TYPES = {
    "automobile": (1000, 1_200_000),
    "bike": (1500, 800_000),
    # ... jak v2 § Skala cen pojazdów
}
FACTOR = 0.08

data = json.loads(Path("vehicles.json").read_text(encoding="utf-8"))
for entry in data:
    t = entry.get("category", "automobile")
    lo, hi = TYPES.get(t, (1000, 1_200_000))
    old = entry["price"]
    entry["price"] = max(lo, min(hi, round(old * FACTOR, -2)))
Path("vehicles.json").write_text(json.dumps(data, indent=2), encoding="utf-8")
```

Po konwersji: zsynchronizować `BabiczVehicleShop_cl.lua` → `VehiclePriceConfig.types` i tuning (§12).

---

## 21B. `VehiclePriceConfig.types` — blok do wklejenia

**Plik:** `[rage]/rage_vehicleshop/BabiczVehicleShop_cl.lua` (~linia 380).


**Obecnie:** automobile min **12000** / max **7800000** · boat max **12M** · plane max **25M**.


**Docelowo (v2):**

```lua

types = {
    automobile = {
        minPrice = 1000,
        maxPrice = 1200000,
        roundTo = 500,
        driftMinRatio = 0.0486,
        driftMaxRatio = 0.1487,
        classFloorRatio = {
            [11] = 0.0160,
            [12] = 0.0210,
            [20] = 0.0235,
        },
        priceCurveEase = 1.38,
        classInfluence = 0.35,
        classInfluenceTopFade = 0.78,
    },
    bike       = { minPrice = 1500,    maxPrice = 800000,  roundTo = 500   },
    bicycle    = { minPrice = 100,     maxPrice = 1000,    roundTo = 10    },
    boat       = { minPrice = 1000000, maxPrice = 2500000, roundTo = 10000 },
    submarine  = { minPrice = 500000,  maxPrice = 1500000, roundTo = 10000 },
    heli       = { minPrice = 2000000, maxPrice = 4000000, roundTo = 10000 },
    plane      = { minPrice = 6000000, maxPrice = 8000000, roundTo = 10000 },
    trailer    = { minPrice = 2500,    maxPrice = 100000,  roundTo = 500   },
    train      = { minPrice = 0,       maxPrice = 0,       roundTo = 1     },
},

```


**Domy:** min segment **15 000$** · max cap **800 000$** (DB `houselocations`, nie `VehiclePriceConfig`).


---

## 22. Usługi i garaże

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_garages/Config.lua` | Tow, impound | 800$ / 5000$ | 300–700$ / 1500–3000$ | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Naprawa przy wyciągnięciu | 1500$ hardcoded | 500–1500$ | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Myjnia `dirtLevel * 12` | ×12$ | ×4–8$ | Planowany |
| `[rage]/rage_garages/Config.lua` | Współwłaściciel (`ManageCoOwnerPrice`) | 5000$ | **2000$** | Planowany |

---

## 23. Wygląd postaci

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_multicharacter/Config.lua` | Sklep odzieżowy (`price`) | 200$ | 80–150$ | Planowany |
| `[rage]/rage_multicharacter/Config.lua` | Fryzjer | 180$ | 60–120$ | Planowany |
| `[rage]/rage_multicharacter/Config.lua` | Tatuaż | 750$ | 250–500$ | Planowany |
| `[rage]/rage_multicharacter/Config.lua` | Maski | 50$ | 25–50$ | Planowany |

---

## 24. Siłownia

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Gym.Membership.Prices` | Karnety 7/14/30/90 dni | 500 / 900 / 1500 / 4000$ | 150–250 / 250–400 / 400–700 / 1000–1800$ | Planowany |
| `[core]/ox_inventory/data/shops.lua` | Sklep siłowni (jeśli osobny) | — | spójne z Config.Gym | Planowany |

---

## 25. Mieszkania (`qs-housing`)

### Problem (dlaczego bloker §0)

| Mechanizm | Obecnie | Skutek po deflacji zarobków |
|---|---|---|
| `CreditEq` + `CreditTime` | **30%** wartości hipoteki **co 5 min** | Spłata całego kredytu w ~17 min — absurd |
| `RentTime` / scheduler | Czynsz **co 5 min** | Właściciel dostaje tysiące $/h pasywnie |
| Ceny w DB | Setki tysięcy – miliony | Dom droższy niż cała ekonomia v2 |

### Config — pliki

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/qs-housing/config/main.lua` | `CreditEq`, `CreditTime` | 0.3 co 5 min | **`CreditEq = 0.06`**, **`CreditTime = 45`** min | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `RentTime`, `RentScheduler` | co 5 min / monthly | **1%** wartości nieruchomości **co 14 dni** (real time) | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `BankFee`, `BrokerFee`, `Taxes` | 10% / 5% / 5% | bez zmian | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `Config.Bills` | woda 30–150; internet 80–300; prąd 50$/kWh | 15–40 / 25–80 / 12–20$/kWh | Planowany |
| `[tebex]/qs-housing/config/main.lua` | Upgrade'y (alarm, kamery…) | 10000–60000$ | 2500–22000$ (tabela v2) | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `MetaKeyCreatePrice` | 500$ | 150–300$ | Planowany |
| `[tebex]/qs-housing/config/furniture.lua` | Ceny mebli IKEA | 61–2500$+ | 30–800$ | Planowany |

### Baza danych — ceny nieruchomości

Tabela `houselocations` / `Config.Houses` — **konwersja per lokalizacja** przed deployem (w tym samym release co reszta v2).

| Segment | Docelowa cena | Przykład |
|---|---:|---|
| **MIN globalny (po rescale)** | **15 000$** | żaden dom taniej |
| **MAX globalny (hard cap)** | **800 000$** | LEAST(FLOOR(price×0.08), 800000) |
| **Najtańsza melina** (garaż, komórka, slums) | **15 000–20 000$** | np. najtańsze lokacje Sandy / industrial |
| Starter | 25 000–50 000$ | pierwsze mieszkanie |
| Standard | 50 000–100 000$ | typowe mieszkanie w mieście |
| Dom średni / duży | 100 000–400 000$ | |
| Premium | 400 000–650 000$ | |
| **Najdroższa willa** (hard cap) | **650 000–800 000$** | max lokacje Vinewood / wybrzeże |

**Zasada:** żadna nieruchomość **> 800 000$** po rescale. Endgame dom ≤ endgame auto (1,2 mln).

**SQL (szkic audytu):**

```sql
-- Przykład: mapowanie starych cen → segment (dostosować progi do realnej DB)
UPDATE houselocations SET price = LEAST(FLOOR(price * 0.08), 800000) WHERE price > 100000;
-- Ręczna weryfikacja najtańszych / najdroższych lokacji po skrypcie
```

**Czynsz po zmianie (przykład):** willa 700 000$ × 1% co 14 dni = **7000$** dla właściciela — ~250$/dzień pasywnie, nie tysiące co 5 min.

---

## 26. Moduły satelitarne

### Kasyno (`pickle_casinos`)

**Plik:** `[tebex]/pickle_casinos/config.lua` · **cel v2:** hazard = sink/rozrywka, nie źródło dochodu (ujemne EV).

#### Limity globalne (linie 8–14)

| Klucz | Obecnie | **Ustaw** | Uwagi |
|---|---:|---:|---|
| `Config.MaxWager` | `500000` | **`10000`** | max stawka na rundę/spin (widełki v2: 5–15k) |
| `Config.MaximumChipPurchaseAmount` | `1000000` | **`75000`** | max zakup żetonów naraz (widełki: 50–100k) |
| `Config.MaximumItemPurchaseAmount` | `1000` | `1000` | bez zmian |

#### `Config.Chips` — nominały żetonów (linie 72–83)

| Obecnie `value` | **Ustaw `value`** |
|---:|---:|
| 1 | 1 |
| 5 | 5 |
| 10 | 10 |
| 100 | 50 |
| 1000 | 100 |
| 10000 | 500 |
| 25000 | 1000 |
| 100000 | 2500 |
| 250000 | 5000 |
| 1000000 | **25000** |

Gotowy blok (zachować kolory):

```lua
Config.Chips = {
    {color = "#15803D", value = 1},
    {color = "#C2410C", value = 5},
    {color = "#BE185D", value = 10},
    {color = "#3F3F46", value = 50},
    {color = "#0369A1", value = 100},
    {color = "#15803D", value = 500},
    {color = "#C2410C", value = 1000},
    {color = "#BE185D", value = 2500},
    {color = "#3F3F46", value = 5000},
    {color = "#0369A1", value = 25000},
}
```

#### `Config.Casinos` — bez zmian (linie 141–149)

| Klucz | Wartość | Status |
|---|---|---|
| `vipExpireTime` | `86400 * 7` (7 dni) | bez zmian |
| `tax` | `10` | bez zmian |
| `taxRecipients.police` | `50` | bez zmian |
| `taxRecipients.ambulance` | `50` | bez zmian |
| `profitPercent` | `5` | bez zmian |

#### `Config.LuckyWheel` — nagrody (linie 669–821)

Zasada: **`amount` / `casinoCost` × 0,08** (zaokrąglij do pełnych setek). Żetony (`chips`) — ten sam mnożnik. **Szanse (`chance`) bez zmian.**

| Sekcja | Typ | Obecnie | **Ustaw** |
|---|---|---:|---:|
| `vehicle` | Zorrusso `casinoCost` | 250000 | **20000** |
| `mystery` | money | 50000 / 40000 / 60000 / 25000 / 15000 | **4000 / 3200 / 4800 / 2000 / 1200** |
| `mystery` | chips | 75000 … 20000 | **6000 … 1600** (×0,08) |
| `mystery` | pojazdy `casinoCost` | 360000–535000 | **28800–42800** (×0,08) |
| `discount` | money | 5000 | **400** |
| `clothing` | chips | 1000 | **80** |
| `money_50k` | money | 50000 | **4000** |
| `money_40k` | money | 40000 | **3200** |
| `money_30k` | money | 30000 | **2400** |
| `money_20k` | money | 20000 | **1600** |
| `chips_25k` | chips | 25000 | **2000** |
| `chips_20k` | chips | 20000 | **1600** |
| `chips_15k` | chips | 15000 | **1200** |
| `chips_10k` | chips | 10000 | **800** |
| `rp_15k` / `rp_10k` / `rp_7k` / `rp_5k` / `rp_2k` | chips | 15000 … 2500 | **1200 … 200** |

`spinCooldown`, `paidCooldown`, `rotateVehicle` — **bez zmian**.

#### Gry stołowe / sloty / wyścigi — bez zmian mnożników

| Sekcja | Plik (linie) | Co robić |
|---|---|---|
| `Config.GameTimes` | 33–40 | bez zmian (15 s) |
| `Config.WheelMultiplier` / `Config.WheelBonus` | 332–366 | bez zmian (mnożniki; cap wynika z `MaxWager`) |
| `Config.Slots` (`bananabonanza`, `spacewarriors`, `candybonanza`) | 368–585 | **`pay` mnożniki bez zmian** — max wygrana = stawka × pay × `MaxWager` |
| `Config.HorseRacing.odds` | 605–666 | bez zmian (mnożniki 0,5–40) |

#### Podsumowanie deploy kasyna

| # | Akcja | Plik |
|---:|---|---|
| 1 | `MaxWager = 10000` | `config.lua` |
| 2 | `MaximumChipPurchaseAmount = 75000` | `config.lua` |
| 3 | Przepisać `Config.Chips` (tabela wyżej) | `config.lua` |
| 4 | Przeskalować `Config.LuckyWheel.sections.*.rewards` (×0,08) | `config.lua` |
| 5 | Po deploy: test max spin slotów / ruletki ≤ 10k | QA |

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/pickle_casinos/config.lua` | `MaxWager` | 500 000$ | **10 000$** | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `MaximumChipPurchaseAmount` | 1 000 000$ | **75 000$** | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `Config.Chips` | max 1M | max **25 000$** | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `Config.LuckyWheel` nagrody | do 535k | ×0,08 | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `tax`, `taxRecipients` | 10%; PD/EMS 50/50 | bez zmian | Planowany |

### Dark Shop (`rage_market`)

> Szczegóły per item: **§19A**.

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_market/Config.lua` → `Items.DarkShop` | Klucze, narzędzia, broń dirty | 2k–150k | tabela v2 § Dark Shop | Planowany |
| `[rage]/rage_market/BabiczMarket_sv.lua` | Limit pistoletów / okno czasowe | aktywne | po rescale cen | Planowany |

### Bronie — `BabiczAmmunation` + market

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/BabiczAmmunation/Config.lua` | Ceny broni i ammo | 20k–280k | segmenty v2 | Planowany |
| `[rage]/rage_market/Config.lua` → `Items.Ammunation` | Kamizelki, pistolety, biała | 250–90k | segmenty v2 | Planowany |

### Kluby, VIP, techniczny, automaty

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_market/Config.lua` | `Nightclub`, `CasinoVIP` | 60–500$ | ×0,25–0,35 | Planowany |
| `[rage]/rage_market/Config.lua` | Sklep techniczny (nitro, platetape…) | nitro 10k; tape 3k | tabela v2 | Planowany |
| `[core]/ox_inventory/data/shops.lua` | `VendingMachineDrinks` | 22–30$ | jak 24/7 (7$/3$) | Planowany |

### Telefon (`lb-phone`)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/lb-phone/config/config.lua` | `Config.Valet.Price` | 100$ | 50–150$ | Planowany |
| `[tebex]/lb-phone/config/config.lua` | `Config.PromoteBirdy.Cost` | 3500$ | 500–1000$ | Planowany |

### Blazing Tattoo / Five Records

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `.../c_blazingtattoo/.../BabiczBlazingTattoo_config.lua` | `estimatePerTattoo`, kurs | 750$; 450–850$/pkg | 120–250$; 90–160$/pkg | Planowany |
| `.../c_blazingtattoo/server/BabiczBlazingTattoo_sv.lua` | Faktura, split 90/10 | dowolna kwota | widełki 150–800$ | Planowany |
| `.../c_fiverecords/.../BabiczFiveRecords_config.lua` | Kurs | 400–800$/pkg | 80–150$/pkg | Planowany |

### Beach vendor / KQ weed / Rico

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobs/Jobs/beach_vendor.lua` | Ceny itemów | 44–290$ | 18–70$ | Planowany |
| `[tebex]/kq_weed/config.strains.lua` | yields / czasy per odmiana | OG 6–8 bud / 20 min itd. | Tabela v2 § KQ weed (OG 4–5 / 28 min … Blue Dream 3–5 / 40 min) | Planowany |
| `[rage]/rage_market/Config.lua` | Klucz Rico (DarkShop) | 50 000$ | 3 000–6 000$ | Planowany |

### Garaże P2P

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_garages/Config.lua` | `SellVehiclePercent` | 65% | **65%** (bez zmian) | Planowany |
| `[rage]/rage_garages/Config.lua` | `ManageCoOwnerPrice` | 5000$ | **2000$** | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | P2P `acceptSellVehicle` — podatek | sprzedawca 100% ceny | **2% → DOJ** (`TriggerEvent("BabiczBossMenu:DepositMoney", "doj", math.ceil(price * 0.02))`); sprzedawca `addAccountMoney("bank", price - tax)` | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | P2P `prepareSellVehicle` / UI | brak info o podatku | w ofercie i potwierdzeniu: cena, podatek 2%, kwota netto | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Webhook sprzedaży P2P | tylko cena | dodać linię: podatek DOJ + netto sprzedawcy | Planowany |

### Siłownia — GymShop

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/ox_inventory/data/shops.lua` | `GymShop` | 50–100$ | 25–60$ | Planowany |

### Systemowe

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/config.lua` | `StartingAccountMoney` | bank 2500$ | **2500$** | Planowany |
| `[rage]/rage_multicharacter/Config.lua` | `OnStart` → `money` item | 5000$ | **1500$** (łącznie z bankiem **4000$**) | Planowany |
| SQL `job_grades.salary` | Grade salaries | różne | **Przelicznik $/h** — patrz §1b | Planowany |
| `[rage]/rage_bossmenu/BabiczBossMenu_sv.lua` | `sendSalary` | bez capu | tier + max 10k/tydz. | Planowany |

### Audyt — pozostałe źródła

| Plik / resource | Co | Docelowo | Status |
|---|---|---|---|
| `[rage]/rage_heists/Heists/BankPacific/server/` | Wypłata lootu | §13a — **wymagane przed deployem** | Planowany |
| `[tebex]/rtx_bowling/config.lua` | Opłata za grę | `PayForBowling = false`; fee 100$ | **`PayForBowling = true`**; **`bowlingstartfee = 40$** (25–50$); brak wypłat $ | Planowany |
| `[rage]/BabiczHospitality/` | EMS random ped | — | **Poza scope v2** |
| `[rage]/rage_znajdzki/`, `BabiczPumpkins/` | Eventy sezonowe | — | **Poza scope v2** |

### Napad truckera — wdrożenie

Patrz **§14a**. Docelowa wypłata za dostarczenie ciężarówki: **3500–8000$** `black_money` (łącznie). Stary komentarz 20–50k **nie stosować**.

---


---



---



---


---

# A. Pełny rejestr zmian ekonomii v2

Wygenerowano z configów repo + cele z `ekonomia-zmiany-v2.md`. Regeneracja: `python docs/_gen_wdrozenie_full.py`.

**Format:** każdy wiersz = jedna zmiana do wdrożenia.


## A.Market — rage_market (37 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Market` | water (Butelka wody) | NAPOJE | 20$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | ecola (E-Cola) | NAPOJE | 22$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | sandwich (Kanapka) | JEDZENIE | 36$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | sprunk (Sprunk) | NAPOJE | 22$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | coffee (Kawa) | NAPOJE | 90$ | 12 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | junkenergy (Energetyk Junke) | NAPOJE | 120$ | 14 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | chips (Chipsy) | JEDZENIE | 37$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | donut (Donut) | JEDZENIE | 34$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | hotdog (Hotdog) | JEDZENIE | 38$ | 6 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | phone (Telefon) | PRZEDMIOTY | 1500$ | 500-1000 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | lighter (Zapalniczka) | PRZEDMIOTY | 15$ | 15 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | cigarette (Papieros) | PRZEDMIOTY | 10$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | scratchticketn (Zdrapka BigWin) | PRZEDMIOTY | 1000$ | 75 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | scratchticketp (Zdrapka Premium) | PRZEDMIOTY | 3500$ | 200 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | scratchticketd (Zdrapka Deluxe) | PRZEDMIOTY | 10000$ | 500 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | beer (Piwo) | ALKOHOLE | 46$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | vine (Wino) | ALKOHOLE | 185$ | 22 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | potato (Ziemniak) | SKŁADNIKI | 8$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | sirloin_steak (Surowy Stek) | SKŁADNIKI | 75$ | 25 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | farming_chickenbreast (Surowy Kurczak) | SKŁADNIKI | 28$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | meat (Surowa wołowina) | SKŁADNIKI | 30$ | 12 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | fish (Surowa ryba) | SKŁADNIKI | 20$ | 8 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | bread (Chleb) | SKŁADNIKI | 7$ | 2 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | tomato (Pomidor) | SKŁADNIKI | 10$ | 4 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | cheese (Ser) | SKŁADNIKI | 15$ | 6 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | egg (Jajko) | SKŁADNIKI | 9$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | raw_bacon (Surowy bekon) | SKŁADNIKI | 18$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | rice (Ryż) | SKŁADNIKI | 11$ | 4 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | cucumber (Ogórek) | SKŁADNIKI | 9$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | seaweed (Nori) | SKŁADNIKI | 12$ | 5 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | sausage_raw (Surowa kiełbasa) | SKŁADNIKI | 19$ | 7 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | apple (Jabłko) | SKŁADNIKI | 8$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | banana (Banan) | SKŁADNIKI | 8$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | lemon (Cytryna) | SKŁADNIKI | 9$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | mint (Mięta) | SKŁADNIKI | 8$ | 3 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | orange (Pomarańcza) | SKŁADNIKI | 10$ | 4 | `[rage]/rage_market/Config.lua` |
| `Items.Market` | tortilla (Tortilla) | SKŁADNIKI | 12$ | 5 | `[rage]/rage_market/Config.lua` |


## A.Liquor — rage_market (6 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Liquor` | beer (Piwo) | NAPOJE | 45$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.Liquor` | rhum (Rum) | NAPOJE | 120$ | 45 | `[rage]/rage_market/Config.lua` |
| `Items.Liquor` | vine (Wino) | NAPOJE | 180$ | 22 | `[rage]/rage_market/Config.lua` |
| `Items.Liquor` | champagne (Szampan) | NAPOJE | 200$ | 30 | `[rage]/rage_market/Config.lua` |
| `Items.Liquor` | whisky (Whisky) | NAPOJE | 300$ | 55 | `[rage]/rage_market/Config.lua` |
| `Items.Liquor` | vodka (Wódka) | NAPOJE | 250$ | 45 | `[rage]/rage_market/Config.lua` |


## A.Nightclub — rage_market (5 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Nightclub` | beer (Piwo) | NAPOJE | 60$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.Nightclub` | vine (Wino) | NAPOJE | 250$ | 22 | `[rage]/rage_market/Config.lua` |
| `Items.Nightclub` | champagne (Szampan) | NAPOJE | 280$ | ×0,25–0,35 (v2 kluby) | `[rage]/rage_market/Config.lua` |
| `Items.Nightclub` | whisky (Whisky) | NAPOJE | 400$ | ×0,25–0,35 (v2 kluby) | `[rage]/rage_market/Config.lua` |
| `Items.Nightclub` | vodka (Wódka) | NAPOJE | 350$ | ×0,25–0,35 (v2 kluby) | `[rage]/rage_market/Config.lua` |


## A.CasinoVIP — rage_market (6 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.CasinoVIP` | beer (Piwo) | NAPOJE | 85$ | 10 | `[rage]/rage_market/Config.lua` |
| `Items.CasinoVIP` | vine (Wino) | NAPOJE | 350$ | 22 | `[rage]/rage_market/Config.lua` |
| `Items.CasinoVIP` | champagne (Szampan) | NAPOJE | 400$ | ×0,25–0,35 (v2 VIP) | `[rage]/rage_market/Config.lua` |
| `Items.CasinoVIP` | whisky (Whisky) | NAPOJE | 560$ | ×0,25–0,35 (v2 VIP) | `[rage]/rage_market/Config.lua` |
| `Items.CasinoVIP` | vodka (Wódka) | NAPOJE | 490$ | ×0,25–0,35 (v2 VIP) | `[rage]/rage_market/Config.lua` |
| `Items.CasinoVIP` | niebieskispecjal (Niebieski Spieciał) | NAPOJE | 680$ | ×0,25–0,35 (v2 VIP) | `[rage]/rage_market/Config.lua` |


## A.Tools — rage_market (46 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Tools` | fixkit (Zestaw naprawczy) | DO POJAZDU | 5000$ | 250-600 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | carokit (Zestaw blacharski) | DO POJAZDU | 1200$ | 400-800 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | handcuffs (Kajdanki) | NARZĘDZIA | 5000$ | bez zmian | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | lockpick (Wytrych) | NARZĘDZIA | 1200$ | 80-200 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | WEAPON_HAMMER (Młotek) | NARZĘDZIA | 800$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | WEAPON_CROWBAR (Łom) | NARZĘDZIA | 1000$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | WEAPON_WRENCH (Klucz) | NARZĘDZIA | 1200$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | phone (Telefon) | PRZEDMIOTY | 1250$ | 500-1000 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | radio (Radio) | PRZEDMIOTY | 850$ | 300-700 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | bag (Torba) | PRZEDMIOTY | 900$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | headbag (Worek na głowę) | PRZEDMIOTY | 1500$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | cement (Cement) | PRZEDMIOTY | 350$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | carjack (Podnośnik) | DO POJAZDU | 300$ | 80-150 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | spare_wheel (Koło zapasowe) | DO POJAZDU | 250$ | 60-120 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | nitro (Butla nitro) | DO POJAZDU | 10000$ | 2000-4000 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | platetape (SuperDuperTape) | DO POJAZDU | 3000$ | 600-1200 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | binoculars (Lornetka) | PRZEDMIOTY | 220$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | scuba (Sprzęt do nurkowania) | PRZEDMIOTY | 1600$ | 400-800 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | notepad (Notatnik) | PRZEDMIOTY | 120$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | spray_can (Spray z farbą) | PRZEDMIOTY | 600$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | cleaningkit (Zestaw do czyszczenia) | PRZEDMIOTY | 250$ | ×0,2 | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | folding_chair (Składane krzesło) | REKWIZYTY | 195$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | pool_rack (Stojak na kije bilardowe) | REKWIZYTY | 420$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | pool_table (Stół do bilarda) | REKWIZYTY | 1580$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | arm_wrestle_table (Beczka do siłowania na rękę) | REKWIZYTY | 520$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | camp_table (Stolik turystyczny) | REKWIZYTY | 270$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | camp_chair (Krzesło turystyczne) | REKWIZYTY | 195$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | work_light (Reflektor roboczy) | REKWIZYTY | 240$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | grill (Grill) | REKWIZYTY | 360$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | kitchen_table (Stół kuchenny) | REKWIZYTY | 520$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | garden_table (Stolik ogrodowy) | REKWIZYTY | 330$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | garden_chair (Krzesło ogrodowe) | REKWIZYTY | 210$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | wooden_barrel (Drewniana beczka) | REKWIZYTY | 285$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | boombox (Boombox) | REKWIZYTY | 240$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | beach_umbrella (Parasol plażowy) | REKWIZYTY | 225$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | beach_bucket (Wiadro) | REKWIZYTY | 150$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | tent (Namiot) | REKWIZYTY | 540$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | pavilion (Pawilon) | REKWIZYTY | 2500$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | mattress (Materac) | REKWIZYTY | 270$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | field_bed (Łóżko polowe) | REKWIZYTY | 330$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | campfire (Ognisko) | REKWIZYTY | 285$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | table_lamp (Lampa stołowa) | REKWIZYTY | 210$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | candles (Świeczki) | REKWIZYTY | 105$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | pizza_box (Pudełko pizzy) | REKWIZYTY | 120$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | flower_vase (Wazon z kwiatami) | REKWIZYTY | 165$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |
| `Items.Tools` | funeral_wreath (Wieniec pogrzebowy) | REKWIZYTY | 180$ | ×0,15–0,25 (sink RP) | `[rage]/rage_market/Config.lua` |


## A.Pharmacy — rage_market (5 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Pharmacy` | bandage (Bandaż) | MEDYKAMENTY | 250$ | bez zmian lub v2 | `[rage]/rage_market/Config.lua` |
| `Items.Pharmacy` | cpr (Zestaw CPR) | MEDYKAMENTY | 2500$ | bez zmian lub v2 | `[rage]/rage_market/Config.lua` |
| `Items.Pharmacy` | stethoscope (Stetoskop) | PRZEDMIOTY | 80$ | bez zmian lub v2 | `[rage]/rage_market/Config.lua` |
| `Items.Pharmacy` | empty_baggy (Samarka) | PRZEDMIOTY | 20$ | bez zmian lub v2 | `[rage]/rage_market/Config.lua` |
| `Items.Pharmacy` | wheelchair (Wózek inwalidzki) | PRZEDMIOTY | 5000$ | bez zmian lub v2 | `[rage]/rage_market/Config.lua` |


## A.BlackMarket — rage_market (3 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.BlackMarket` | key (Klucz do laboratorium kokainy) | NARKOTYKI | 75000$ | v2 § Dark Shop | `[rage]/rage_market/Config.lua` |
| `Items.BlackMarket` | hackingdevice (Program do hakowania) | PRZEDMIOTY | 4000$ | v2 § Dark Shop | `[rage]/rage_market/Config.lua` |
| `Items.BlackMarket` | hackingtablet (Tablet do hakowania) | PRZEDMIOTY | 8000$ | v2 § Dark Shop | `[rage]/rage_market/Config.lua` |


## A.DarkShop — rage_market (17 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.DarkShop` | key (Klucz do black marketu) | KLUCZE | 100000$ | 8000-15000 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | key (Klucz do grupy Rico) | KLUCZE | 50000$ | 3000-6000 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | stethoscope (Stetoskop) | NAPADY | 60$ | 60 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | hackingdevice (Program do hakowania) | NAPADY | 5000$ | 400-800 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | drill (Wiertło) | NAPADY | 12500$ | 1500-3000 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | hackingtablet (Tablet do hakowania) | NAPADY | 10000$ | 800-1500 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | thermite (Termit) | NAPADY | 2000$ | 400-700 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_PISTOL (Pistolet) | BROŃ PALNA | 100000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_SNSPISTOL_MK2 (Pistolet SNS MK2) | BROŃ PALNA | 150000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_VINTAGEPISTOL (Pistolet Vintage) | BROŃ PALNA | 140000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_PISTOL (Pistolet) | BROŃ PALNA | 150000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_SNSPISTOL_MK2 (Pistolet SNS MK2) | BROŃ PALNA | 225000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | WEAPON_VINTAGEPISTOL (Pistolet Vintage) | BROŃ PALNA | 210000$ | **ustalić wg v2** | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | ammo-9 (18x Amunicja 9mm) | AMUNICJA | 550$ | 80-150 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | empty_baggy (Samarka) | NARKOTYKI | 18$ | 18 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | rolling_paper (Bletki) | NARKOTYKI | 8$ | 8 | `[rage]/rage_market/Config.lua` |
| `Items.DarkShop` | kq_weed_fertilizer_blackmarket (Nawóz) | NARKOTYKI | 1200$ | 300-500 | `[rage]/rage_market/Config.lua` |


## A.Ammunation — rage_market (31 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Ammunation` | vest ((Męska) Kamizelka kuloodporna) | KAMIZELKI | 250$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | vest ((Damska) Kamizelka kuloodporna) | KAMIZELKI | 249$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | arplate_light (Lekki wkład) | KAMIZELKI | 5000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_SWITCHBLADE (Scyzoryk) | BROŃ BIAŁA | 1200$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_BAT (Kij baseballowy) | BROŃ BIAŁA | 1800$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_KNIFE (Nóż do masła) | BROŃ BIAŁA | 2400$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_KNUCKLE (Kastet) | BROŃ BIAŁA | 2000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_MACHETE (Maczeta) | BROŃ BIAŁA | 3600$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_PISTOL (Pistolet) | BROŃ PALNA | 90000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_SNSPISTOL (Pistolet SNS) | BROŃ PALNA | 140000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | WEAPON_PISTOL50 (Pistolet .50) | BROŃ PALNA | 160000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | ammo-9 (20x Amunicja 9mm) | AMUNICJA | 600$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | ammo-45 (20x Amunicja .45 ACP) | AMUNICJA | 650$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | ammo-50 (20x Amunicja .50 AE) | AMUNICJA | 700$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_flashlight (Latarka taktyczna) | AKCESORIA | 2500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_clip_extended_pistol (Pistolet: Powiększony magazynek) | AKCESORIA | 6000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_compensator (Pistolet: Kompensator) | AKCESORIA | 5000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_scope_holo (Pistolet: Celownik holograficzny) | AKCESORIA | 4000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_camo (Kamuflaż) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_brushstroke (Pociągnięcia Pędzla) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_woodland (Leśne) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_skull (Czaszka) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_sessanta (Sessanta) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_perseus (Perseus) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_leopard (leopard) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_zebra (Zebra) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_geometric (Geometryczne) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_boom (Boom) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_patriotic (Patriotyczne) | MALOWANIE | 3500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_pearl (Perłowy .50) | MALOWANIE | 15000$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |
| `Items.Ammunation` | at_skin_wood (Drewniane Heavy/SNS) | MALOWANIE | 12500$ | v2 § Bronie legalne | `[rage]/rage_market/Config.lua` |


## A.Weed — rage_market (15 poz.)

| Sekcja | Item | Kategoria | Obecnie | Docelowo | Plik |
|---|---|---|---|---|---|
| `Items.Weed` | empty_baggy (Samarka) | PAKUNEK | 20$ | 20 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | rolling_paper (Bletki) | PAKUNEK | 15$ | 15 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_pesticide_spray (Spray pestycydowy) | NAWÓZ | 45$ | 45 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_fertilizer_speed (Nawóz (szybkość wzrostu)) | NAWÓZ | 700$ | 300-500 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_fertilizer_yield (Nawóz (ilość plonu)) | NAWÓZ | 700$ | 300-500 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_fertilizer_resistance (Nawóz (wzmocnienie odporności)) | NAWÓZ | 800$ | 350-550 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_seed_og_kush (Nasiono OG Kush) | NASIONA | 200$ | 150-250 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_seed_purple_haze (Nasiono Purple Haze) | NASIONA | 200$ | 150-250 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_seed_white_widow (Nasiono White Widow) | NASIONA | 200$ | 150-250 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_seed_blue_dream (Nasiono Blue Dream) | NASIONA | 200$ | 150-250 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_pot (Doniczka) | SPRZĘT | 1000$ | 400-700 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_table (Stół do pakowania) | SPRZĘT | 10000$ | 2500-5000 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_press (Prasa) | SPRZĘT | 50000$ | 12000-20000 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_tent (Namiot uprawowy) | SPRZĘT | 12000$ | 3000-6000 | `[rage]/rage_market/Config.lua` |
| `Items.Weed` | kq_weed_watering_system (System nawadniania roślin) | SPRZĘT | 14000$ | 3500-7000 | `[rage]/rage_market/Config.lua` |


## A.ChinaMarket

| Item | Obecnie | Docelowo | Plik |
|---|---|---|---|
| lithium | 250 | ×0,2 lub 50-80 | ChinaMarket |
| methlab | 750 | ×0,2 lub 150-250 | ChinaMarket |
| hydrochloric_acid | 500 | ×0,2 lub 100-150 | ChinaMarket |


## A.ox_inventory shops

| Sklep | Item | Obecnie | Docelowo | Plik |
|---|---|---|---|---|
| VendingMachineDrinks | sprunk | 22 | 7 | [core]/ox_inventory/data/shops.lua |
| VendingMachineDrugs | chips | 30 | 3 | [core]/ox_inventory/data/shops.lua |
| GymShop | protein_shake | 50 | 25-40 | [core]/ox_inventory/data/shops.lua |
| GymShop | sportlunch | 60 | 30-45 | [core]/ox_inventory/data/shops.lua |
| GymShop | junkenergy | 100 | 40-60 | [core]/ox_inventory/data/shops.lua |
| GymShop | water | 18 | 7 | [core]/ox_inventory/data/shops.lua |


## A.Paycheck i start

| Parametr | Obecnie | Docelowo | Plik |
|---|---|---|---|
| cywil unemployed | 125$/15min | 15$/15min | paycheck.lua |
| police/fib/mechanic on duty | 250$/15min | 168-175$/15min | paycheck.lua |
| ambulance on duty | 250$/15min | 115-125$/15min | paycheck.lua |
| doj on duty | 250$/15min | 150-158$/15min | paycheck.lua |
| c_* on duty | 250$/15min | 10-15$/15min | paycheck.lua |
| warunek on duty | brak | wymagany | paycheck.lua |
| StartingAccountMoney.bank | 2500 | 2500 | es_extended/config.lua |
| OnStart money item | 5000 | 1500 | rage_multicharacter/Config.lua |


## A.Napady aktywne

| Napad | Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|---|
| NPC | money.Count | 45-195 | 25-150 | BabiczNPC_config + sv CD 15s→180s |
| NPC | cooldown per gracz | 15s | 180s | BabiczNPC_sv.lua SetTimeout |
| ATM karta | CreditCard.reward | 500-2500 | 100-350 | BabiczATM_config.lua |
| ATM hack | Hack.reward+piles | 134-358×6-14 | 200-700 łącznie | BabiczATM_config.lua |
| ATM hack | CD per gracz | brak | 10 min | server |
| Kasetka | reward | 2500-5000 | 250-600 | ShopKeeperCashRegister_config.lua |
| Kasetka | robTimeout | 600000ms | ~480000ms | ten sam plik |
| Sejf | reward | 15000-40000 | 800-2200 | ShopVault_shared.lua |
| Sejf | rewardBonus Sandy | +5000 | usunąć | ShopVault_shared.lua |
| Sejf | robTimeout/CD | 600000ms | 15min+10min global | server |
| Fleeca | reward | 60000-110000 | 4500-12000 | BabiczBank_shared.lua |
| Fleeca | AttackerMultipliers | [2]=1.6… | [2]=1.0,[3]=1.2,[4]=1.35 | rage_heists/Config.lua |
| Pacific | wypłata lootu | brak | 35000-90000 | BankPacific/server — implementacja |
| Config | AttackerMultipliers wszystkie | stare | v2 blok Lua | rage_heists/Config.lua |


## A.Napady planowane

| Napad | Klucz | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| Boosting | loot | — | 350-900 clean | planowany |
| Tracker | loot | — | 500-1400 dirty | planowany |
| Trucker crime | loot | — | 3500-8000 dirty | §14a |
| Jubiler | biżuteria skup | — | 2800-6500 | planowany |
| Lombard napad | loot | — | 600-1800+items | planowany |
| Jacht | loot | — | 10000-25000 | planowany |
| Humane | loot | — | 22000-55000 | planowany |
| Cayo | loot | — | 70000-130000 | planowany |


## A.Pralnia

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| Automatic.lossPercent | 0.30 | 0.35-0.45 | RageCity/Config.lua MoneyWash |
| Automatic.amountPerSecond | 175 | 25-75 | ten sam |
| ręczna trasa / punkt | 3000-7500 | random(350,900) | moneywash.lua |
| ręczna strata | 5% | 20-30% | moneywash.lua |


## A.KQ Deliveries

| Parametr | Obecnie | Docelowo | Plik |
|---|---|---|---|
| payPerMinute (job) | 450 | 7-11 | kq_deliveries/config.lua |
| payPerMinute (job) | 350 | 7-11 | kq_deliveries/config.lua |
| payPerMinute (job) | 400 | 7-11 | kq_deliveries/config.lua |
| payPerMinute (job) | 330 | 7-11 | kq_deliveries/config.lua |
| payPerMinute (job) | 500 | 7-11 | kq_deliveries/config.lua |
| fiveStarBonus | 10-30% | 3% | kq_deliveries/config.lua |
| fourStarBonus | 5% | 2% | kq_deliveries/config.lua |
| vehicleCare | 5% | 2% | kq_deliveries/config.lua |
| teamWorkBonuses[2] | 20-80% | 10% | kq_deliveries/config.lua |
| maxVehicleDamagePenalty | 600-1500 | 150-250 | kq_deliveries/config.lua |
| missingVehiclePenalty | 1200 | 200-300 | kq_deliveries/config.lua |
| salary_bonus tiers | 5-30% | 2-8% | kq_deliveries/config.lua |


## A.KQ Powerwashing — kontrakty (103 poz.)

| Kontrakt | Obecnie | Docelowo | Plik |
|---|---|---|---|
| contract #1 | 4×2550=10200$ | ~892$ (widełka 800-2000) | contracts.lua |
| contract #2 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #3 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #4 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #5 | 4×900=3600$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #6 | 4×4050=16200$ | ~1418$ (widełka 800-2000) | contracts.lua |
| contract #7 | 4×3300=13200$ | ~1155$ (widełka 800-2000) | contracts.lua |
| contract #8 | 4×5100=20400$ | ~1785$ (widełka 800-2000) | contracts.lua |
| contract #9 | 4×3750=15000$ | ~1312$ (widełka 800-2000) | contracts.lua |
| contract #10 | 4×5850=23400$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #11 | 4×2100=8400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #12 | 4×2100=8400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #13 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #14 | 4×2400=9600$ | ~840$ (widełka 800-2000) | contracts.lua |
| contract #15 | 4×2250=9000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #16 | 4×1800=7200$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #17 | 4×2550=10200$ | ~892$ (widełka 800-2000) | contracts.lua |
| contract #18 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #19 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #20 | 4×3150=12600$ | ~1102$ (widełka 800-2000) | contracts.lua |
| contract #21 | 4×1950=7800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #22 | 4×1350=5400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #23 | 4×2400=9600$ | ~840$ (widełka 800-2000) | contracts.lua |
| contract #24 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #25 | 4×3750=15000$ | ~1312$ (widełka 800-2000) | contracts.lua |
| contract #26 | 4×6000=24000$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #27 | 4×5400=21600$ | ~1890$ (widełka 800-2000) | contracts.lua |
| contract #28 | 4×5400=21600$ | ~1890$ (widełka 800-2000) | contracts.lua |
| contract #29 | 4×4800=19200$ | ~1680$ (widełka 800-2000) | contracts.lua |
| contract #30 | 4×4500=18000$ | ~1575$ (widełka 800-2000) | contracts.lua |
| contract #31 | 4×3900=15600$ | ~1365$ (widełka 800-2000) | contracts.lua |
| contract #32 | 4×6300=25200$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #33 | 4×3300=13200$ | ~1155$ (widełka 800-2000) | contracts.lua |
| contract #34 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #35 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #36 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #37 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #38 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #39 | 4×2250=9000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #40 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #41 | 4×2850=11400$ | ~997$ (widełka 800-2000) | contracts.lua |
| contract #42 | 4×3150=12600$ | ~1102$ (widełka 800-2000) | contracts.lua |
| contract #43 | 4×3150=12600$ | ~1102$ (widełka 800-2000) | contracts.lua |
| contract #44 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #45 | 4×1350=5400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #46 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #47 | 4×1650=6600$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #48 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #49 | 4×3900=15600$ | ~1365$ (widełka 800-2000) | contracts.lua |
| contract #50 | 4×4350=17400$ | ~1522$ (widełka 800-2000) | contracts.lua |
| contract #51 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #52 | 4×4950=19800$ | ~1732$ (widełka 800-2000) | contracts.lua |
| contract #53 | 4×1650=6600$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #54 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #55 | 4×6900=27600$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #56 | 4×2850=11400$ | ~997$ (widełka 800-2000) | contracts.lua |
| contract #57 | 4×6000=24000$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #58 | 4×3450=13800$ | ~1208$ (widełka 800-2000) | contracts.lua |
| contract #59 | 4×3300=13200$ | ~1155$ (widełka 800-2000) | contracts.lua |
| contract #60 | 4×2550=10200$ | ~892$ (widełka 800-2000) | contracts.lua |
| contract #61 | 4×4800=19200$ | ~1680$ (widełka 800-2000) | contracts.lua |
| contract #62 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #63 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #64 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #65 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #66 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #67 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #68 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #69 | 4×3000=12000$ | ~1050$ (widełka 800-2000) | contracts.lua |
| contract #70 | 4×7800=31200$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #71 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #72 | 4×1200=4800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #73 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #74 | 4×4500=18000$ | ~1575$ (widełka 800-2000) | contracts.lua |
| contract #75 | 4×4950=19800$ | ~1732$ (widełka 800-2000) | contracts.lua |
| contract #76 | 4×2100=8400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #77 | 4×3900=15600$ | ~1365$ (widełka 800-2000) | contracts.lua |
| contract #78 | 4×5550=22200$ | ~1942$ (widełka 800-2000) | contracts.lua |
| contract #79 | 4×4050=16200$ | ~1418$ (widełka 800-2000) | contracts.lua |
| contract #80 | 4×3750=15000$ | ~1312$ (widełka 800-2000) | contracts.lua |
| contract #81 | 4×2400=9600$ | ~840$ (widełka 800-2000) | contracts.lua |
| contract #82 | 4×2100=8400$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #83 | 4×2400=9600$ | ~840$ (widełka 800-2000) | contracts.lua |
| contract #84 | 4×4350=17400$ | ~1522$ (widełka 800-2000) | contracts.lua |
| contract #85 | 4×3300=13200$ | ~1155$ (widełka 800-2000) | contracts.lua |
| contract #86 | 4×4500=18000$ | ~1575$ (widełka 800-2000) | contracts.lua |
| contract #87 | 4×4200=16800$ | ~1470$ (widełka 800-2000) | contracts.lua |
| contract #88 | 4×2250=9000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #89 | 4×1650=6600$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #90 | 4×3300=13200$ | ~1155$ (widełka 800-2000) | contracts.lua |
| contract #91 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #92 | 4×2550=10200$ | ~892$ (widełka 800-2000) | contracts.lua |
| contract #93 | 4×6300=25200$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #94 | 4×3900=15600$ | ~1365$ (widełka 800-2000) | contracts.lua |
| contract #95 | 4×2850=11400$ | ~997$ (widełka 800-2000) | contracts.lua |
| contract #96 | 4×1500=6000$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #97 | 4×1950=7800$ | ~800$ (widełka 800-2000) | contracts.lua |
| contract #98 | 4×2700=10800$ | ~945$ (widełka 800-2000) | contracts.lua |
| contract #99 | 4×6600=26400$ | ~2000$ (widełka 800-2000) | contracts.lua |
| contract #100 | 4×2550=10200$ | ~892$ (widełka 800-2000) | contracts.lua |
| contract #101 | 4×2850=11400$ | ~997$ (widełka 800-2000) | contracts.lua |
| contract #102 | 4×2850=11400$ | ~997$ (widełka 800-2000) | contracts.lua |
| contract #103 | 4×7200=28800$ | ~2000$ (widełka 800-2000) | contracts.lua |


## A.Paliwo

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| priceTick | 8 | 2-4 | ox_fuel/config.lua |
| petrolCan.price | 1500 | 400-700 | ox_fuel/config.lua |
| petrolCan.refillPrice | 1000 | 250-500 | ox_fuel/config.lua |
| petrolCan.ecoPetrolCanPrice | 800 | 200-400 | ox_fuel/config.lua |


## A.Garaże

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| Tow.price | 800 | 300-700 | rage_garages/Config.lua |
| ManageCoOwnerPrice | 5000 | 2000 | rage_garages/Config.lua |
| SellVehiclePercent | 65% | 65% | bez zmian |
| impound/tow helikopter | 5000 | 1500-3000 | rage_garages/Config.lua |
| repair on retrieve | 1500 | 500-1500 | BabiczGarages_sv.lua |
| myjnia dirt× | 12 | 4-8 | BabiczGarages_sv.lua |
| P2P tax DOJ | 0% | 2% | BabiczGarages_sv.lua acceptSellVehicle |


## A.qs-housing

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| CreditEq | 0.3 | 0.06 | qs-housing/config/main.lua |
| CreditTime | 5 min | 45 min | main.lua |
| RentTime/scheduler | 5 min | 1%/14d RT | main.lua |
| Bills woda | 30-150 | 15-40 | main.lua |
| Bills internet | 80-300 | 25-80 | main.lua |
| Bills prąd | 50/kWh | 12-20/kWh | main.lua |
| MetaKeyCreatePrice | 500 | 150-300 | main.lua |
| Upgrade alarm | 10000 | 2500-4000 | main.lua |
| Upgrade kamery | 35000 | 8000-12000 | main.lua |
| Upgrade czujnik | 45000 | 10000-15000 | main.lua |
| Upgrade sejf | 50000 | 12000-18000 | main.lua |
| Upgrade meble+ | 60000 | 15000-22000 | main.lua |
| Upgrade dzwonek | 15000 | 3500-5500 | main.lua |
| Upgrade Aura | 30000 | 7000-11000 | main.lua |
| Upgrade dekoracje | 25000 | 6000-9000 | main.lua |
| DB houselocations.price | stare | ×0.08 cap 800k | SQL |
| MIN dom | — | 15000 | audyt DB |
| MAX dom | — | 800000 | audyt DB |
| furniture IKEA | 61-2500+ | 30-800 | furniture.lua |


## A.Hazard

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| MaxWager | 500000 | 5000-15000 | pickle_casinos/config.lua |
| MaximumChipPurchaseAmount | 1000000 | 50000-100000 | pickle_casinos/config.lua |
| Chips max nominał | 1000000 | 10000-25000 | pickle_casinos/config.lua |
| tax | 10% | 10% | bez zmian |
| scratchticketn win | do milionów | max ~800 | rage_scratchcard/server.lua |
| scratchticketp win | — | max ~2000 | rage_scratchcard/server.lua |
| scratchticketd win | — | max ~8000 | rage_scratchcard/server.lua |
| PayForBowling | false | true | rtx_bowling/config.lua |
| bowlingstartfee | 100 | 40 | rtx_bowling/config.lua |


## A.lb-phone

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| Valet.Price | 100 | 50-150 | lb-phone/config/config.lua |
| PromoteBirdy.Cost | 3500 | 500-1000 | lb-phone/config/config.lua |


## A.Wygląd

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| Clothe.price | 200 | 80-150 | rage_multicharacter/Config.lua |
| Barber.price | 180 | 60-120 | rage_multicharacter/Config.lua |
| Tattoo.price | 750 | 250-500 | rage_multicharacter/Config.lua |
| Mask.price | 50 | 25-50 | rage_multicharacter/Config.lua |


## A.Siłownia karnety

| Karnet | Obecnie | Docelowo | Plik |
|---|---|---|---|
| 7 dni | 500 | 150-250 | RageCity/Config.lua Gym |
| 14 dni | 900 | 250-400 | ten sam |
| 30 dni | 1500 | 400-700 | ten sam |
| 90 dni | 4000 | 1000-1800 | ten sam |


## A.EMS (poza MDT)

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| health_insurance 7d | 3500 | 800-1200 | BabiczAmbulance_shared.lua |
| health_insurance 14d | 6000 | 1400-2000 | ten sam |
| health_insurance 30d | 12000 | 2500-3500 | ten sam |
| local medic price | 50 | 500-1500 | ten sam |
| priceWhileMedicsOnline | 5000 | 800-1500 | ten sam |
| NPC course reward | 350-850 | 200-500 | ten sam |


## A.Mechanik NPC/tuning

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| LocalMechanic.price | 1000 | 500-800 | BabiczMechanic_shared.lua |
| LocalMechanic.mechanicsOnlinePrice | 5000 | 1000-1500 | ten sam |
| tuning flaty | 10k-1.2M | ×0.15 | BabiczTuningMenu_sv.lua |
| ShopPricePercents performance | 5% | 2% | BabiczTuningMenu_sv.lua |
| towVehicle | 0 | 0 | bez zmian |
| tow course (nowy) | brak | 150-350 | BabiczMechanic_shared C.TowCourse |
| fixkit shop mechanic | 1500 | 250-600 | rage_fractions/Config.lua |
| blowtorch shop | 4500 | 800-1500 | rage_fractions/Config.lua |


## A.MDT (podsumowanie — pełne listy w §7 i v2)

| Obszar | Obecnie | Docelowo | Plik |
|---|---|---|---|
| policeFines grzywny ×230 | 1000-50000 | ×0.10 wg §7 | rage_mdt/Config.lua |
| policeFines jail | 300+ mies | taryfikator v2 | rage_mdt/Config.lua |
| ulgi mandatowe | do -25000 | max -2500 | rage_mdt/Config.lua |
| ambulance fines ×76 | 50-475 | v2 § EMS | rage_mdt/Config.lua |
| mechanic fines ×8 | 60-1800 | 60-300 | rage_mdt/Config.lua |
| FinePlayerShare | flat 25% | progresywny+cap750 | Config+BabiczMdt_sv.lua |
| mechanic finePercent | 25% | 40% flat | rage_mdt/Config.lua |


## A.Zadania Moris/Carlos

| Zadanie | Obecnie | Docelowo | Plik |
|---|---|---|---|
| Fernando t1 paczka | 250-750 | 80-200 | 1_Fernando.lua |
| Fernando t2 finish | 8-12k | 1500-2500 | 1_Fernando.lua |
| Fernando t2 dirty | 25000 | 4000-6000 | 1_Fernando.lua |
| Fernando t3 | 5-8k | 1200-2000 | 1_Fernando.lua |
| Fernando t4 | 7-10k | 1500-2500 | 1_Fernando.lua |
| Carlos t1 boosting | 3200-13000 | 350-900 | 2_Carlos.lua t1 |
| Carlos t2 rozwóz | 10-15k | 360-720 | 2_Carlos.lua t2 |
| Carlos t3 tracker | 7500-16500 | 480-1200 | 2_Carlos.lua t3 |
| Carlos t3 lockpick | 1320 | 200-300 | 2_Carlos.lua |
| Carlos t3 hack | 4400 | 450-650 | 2_Carlos.lua |
| Carlos t3 CD | 45min | 60min | 2_Carlos.lua |
| Carlos t3 deactivateTime | 20s | 180-240s | 2_Carlos.lua |
| Moris zad 1-3 bonus | brak | 300-900 | 3_Moris.lua |


## A.Pawnshop — 33 poz.

| Item | Obecnie | Docelowo | Plik |
|---|---|---|---|
| diamond | 155-160$ | 100-160 | Config.Pawnshop.Items |
| phone | 750-1250$ | 40-70 | Config.Pawnshop.Items |
| watch | 450-800$ | 30-55 | Config.Pawnshop.Items |
| watch_silver | 650-1050$ | 40-65 | Config.Pawnshop.Items |
| watch_gold | 950-1500$ | 55-85 | Config.Pawnshop.Items |
| silver_ring | 180-280$ | 15-25 | Config.Pawnshop.Items |
| emerald_ring | 900-1450$ | 55-90 | Config.Pawnshop.Items |
| sapphire_ring | 850-1350$ | 55-90 | Config.Pawnshop.Items |
| ruby_ring | 880-1400$ | 55-90 | Config.Pawnshop.Items |
| thornburn_turquoise_ring | 980-1550$ | 60-95 | Config.Pawnshop.Items |
| earrings | 160-260$ | 12-20 | Config.Pawnshop.Items |
| earrings_silver | 260-420$ | 20-35 | Config.Pawnshop.Items |
| earrings_gold | 420-680$ | 40-70 | Config.Pawnshop.Items |
| earrings_ruby | 760-1220$ | 50-80 | Config.Pawnshop.Items |
| earrings_emerald | 820-1300$ | 55-85 | Config.Pawnshop.Items |
| earrings_turquoise | 700-1120$ | 45-72 | Config.Pawnshop.Items |
| earrings_diamond | 980-1600$ | 80-130 | Config.Pawnshop.Items |
| bracelet | 220-340$ | 18-30 | Config.Pawnshop.Items |
| bracelet_silver | 320-520$ | 22-38 | Config.Pawnshop.Items |
| bracelet_gold | 520-860$ | 40-70 | Config.Pawnshop.Items |
| necklace_silver | 500-820$ | 35-60 | Config.Pawnshop.Items |
| necklace_gold | 760-1220$ | 55-90 | Config.Pawnshop.Items |
| necklace_emerald | 1000-1620$ | 90-150 | Config.Pawnshop.Items |
| necklace_sapphire | 960-1540$ | 85-140 | Config.Pawnshop.Items |
| necklace_ruby | 980-1580$ | 88-145 | Config.Pawnshop.Items |
| wallet | 240-360$ | 15-25 | Config.Pawnshop.Items |
| powerbank | 210-330$ | 12-22 | Config.Pawnshop.Items |
| headphones | 260-420$ | 18-30 | Config.Pawnshop.Items |
| smartwatch | 550-900$ | 35-60 | Config.Pawnshop.Items |
| gold_ring | 650-1050$ | 45-75 | Config.Pawnshop.Items |
| gold_chain | 800-1250$ | 55-90 | Config.Pawnshop.Items |
| vintage_camera | 700-1150$ | 45-75 | Config.Pawnshop.Items |
| designer_bag | 900-1400$ | 70-120 | Config.Pawnshop.Items |


## A.Config.Drugs — 17 poz.

| Item | Ilość | Szansa | Obecnie → docelowo $/szt. | Plik |
|---|---|---|---|---|
| joint | ilość 1-2 | szansa 70% | $75-150 → 25-45 | Config.Drugs |
| weed_packed | ilość 1-3 | szansa 65% | $243-303 → 55-85 | Config.Drugs |
| kq_weed_joint_og_kush | ilość 1-4 | szansa 67% | $175-235 → 40-65 | Config.Drugs |
| kq_weed_joint_purple_haze | ilość 1-4 | szansa 67% | $240-300 → 55-80 | Config.Drugs |
| kq_weed_joint_white_widow | ilość 1-4 | szansa 67% | $250-330 → 60-85 | Config.Drugs |
| kq_weed_joint_blue_dream | ilość 1-4 | szansa 67% | $280-360 → 65-95 | Config.Drugs |
| kq_weed_bag_og_kush | ilość 1-3 | szansa 65% | $370-480 → 80-120 | Config.Drugs |
| kq_weed_bag_purple_haze | ilość 1-3 | szansa 65% | $480-580 → 100-150 | Config.Drugs |
| kq_weed_bag_white_widow | ilość 1-4 | szansa 65% | $500-640 → 110-160 | Config.Drugs |
| kq_weed_bag_blue_dream | ilość 1-3 | szansa 65% | $560-700 → 120-175 | Config.Drugs |
| coke_packed | ilość 1-4 | szansa 55% | $431-541 → 90-130 | Config.Drugs |
| coke | ilość 1-2 | szansa 60% | $750-950 → 130-190 | Config.Drugs |
| coke_bag | ilość 1-3 | szansa 65% | $850-1050 → 150-220 | Config.Drugs |
| heroin_opium | ilość 1-4 | szansa 60% | $339-395 → 70-100 | Config.Drugs |
| heroin_packed | ilość 1-3 | szansa 55% | $545-605 → 100-145 | Config.Drugs |
| magic_mushrooms_dry | ilość 1-8 | szansa 65% | $87-147 → 20-40 | Config.Drugs |
| meth_packed | ilość 1-5 | szansa 50% | $455-570 → 95-140 | Config.Drugs |


## A.Config.DrugSelling

| Klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|
| copsRequired | 1 | 2 | Config.DrugSelling |
| callCopsChance | 65 | 65 (bez zmian) | Config.DrugSelling |
| avatarChance | 35 | 35 (bez zmian) | Config.DrugSelling |


## A.RicoDrugBricks — 9 poz.

| Brick | Obecnie | Docelowo | Plik |
|---|---|---|---|
| weed_brick | 46683-51597$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| coke_brick | 415530-459270$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| heroin_brick | 491625-543375$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| opium_brick | 313785-346815$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| meth_brick | 438188-484313$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| kq_weed_brick_og_kush | 7268-8033$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| kq_weed_brick_purple_haze | 9063-10017$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| kq_weed_brick_white_widow | 9747-10773$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |
| kq_weed_brick_blue_dream | 10773-11907$ | formuła: avg(Drugs)×count×0.9 | Config.RicoDrugBricks |


## A.Firmy c_* menu i kursy (26 poz.)

| Job | Item/klucz | Obecnie | Docelowo | Plik |
|---|---|---|---|---|
| shared | C.Course.Reward | 400-800 | 80-150 + CD 30-45min | [rage]\rage_fractions\Fractions\c_blackrepair\shared\BabiczBlackRepair_config.lua |
| c_blazingtattoo | C.Course.Reward | 450-850 | 90-160 + CD 30-45min | [rage]\rage_fractions\Fractions\c_blazingtattoo\shared\BabiczBlazingTattoo_config.lua |
| c_burgershot | ecola | 21 | v2 §11A | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | sprunk | 21 | v2 §11A | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | mint_lemonade | 34 | 13 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | orange_juice | 32 | 12 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | apple_juice | 31 | 12 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | banana_smoothie | 39 | 14 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | tropical_juice | 36 | 14 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | veegi_burger | 43 | 32 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | becon_burger | 56 | 35 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | cheeseburger | 48 | 34 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | frytki | 38 | 27 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | schakemalinowy | 46 | 32 | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| c_burgershot | C.Course.Reward | 400-800 | 80-150 + CD 30-45min | [rage]\rage_fractions\Fractions\c_burgershot\shared\BabiczBurgerShot_config.lua |
| shared | C.Course.Reward | 400-800 | 80-150 + CD 30-45min | [rage]\rage_fractions\Fractions\c_fiverecords\shared\BabiczFiveRecords_config.lua |
| shared | C.Course.Reward | 400-800 | 80-150 + CD 30-45min | [rage]\rage_fractions\Fractions\c_quiettech\shared\BabiczQuietTech_config.lua |
| c_tequilala | woda_po_studencie | 34 | 14 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | bum | 36 | 20-38 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | wino_z_kartonu | 36 | 16 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | hot_cat | 43 | 38 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | burgir | 48 | 40 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | woda_alko | 46 | 20-38 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | szampon | 56 | 28 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | rozwod | 56 | 20-38 | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |
| c_tequilala | C.Course.Reward | 400-800 | 80-150 + CD 30-45min | [rage]\rage_fractions\Fractions\c_tequilala\shared\BabiczTequilala_config.lua |


## A.rage_fractions sklepy (141 poz.)

| Item | Obecnie | Typ | Docelowo | Plik |
|---|---|---|---|---|
| ecola | 18 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| sprunk | 18 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| water | 9 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| beer | 12 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| whisky | 18 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| vodka | 14 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| champagne | 22 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| vine | 16 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| orange | 8 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| apple | 4 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| lemon | 6 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| bread | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| sausage_raw | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| meat | 13 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| cheese | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| tomato | 7 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| tortilla | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_FLASHLIGHT | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_NIGHTSTICK | 50 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_STUNGUN | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| handcuffs | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| bodycam | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| blowtorch | 5000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| scuba | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| parachute | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| binoculars | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_light | 3500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_heavy | 4500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| police_stormram | 5000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-9 | 2 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-45 | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_COMBATPISTOL | 0 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_HEAVYPISTOL | 0 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| at_flashlight | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| at_suppressor_light | 20000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_heavy | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| spy_microphone | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| evidence_laptop | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fingerprint_scanner | 50 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| evidence_box | 25 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| baggy_empty | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| hydrogen_peroxide | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fingerprint_brush | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_FLASHLIGHT | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_NIGHTSTICK | 50 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_STUNGUN | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| handcuffs | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| bodycam | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| blowtorch | 5000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| scuba | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| parachute | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| binoculars | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_light | 3500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_heavy | 4500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| police_stormram | 5000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-9 | 2 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-45 | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_COMBATPISTOL | 0 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_HEAVYPISTOL | 0 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-beanbag | 3 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_BEANBAG | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_BEANBAG2 | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| at_flashlight | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| at_suppressor_light | 20000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| arplate_heavy | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-shotgun | 8 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo-rifle | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_PUMPSHOTGUN | 17500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_SMG | 20000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_CARBINERIFLE | 22500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| spy_microphone | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| evidence_laptop | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fingerprint_scanner | 50 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| evidence_box | 25 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| baggy_empty | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| hydrogen_peroxide | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fingerprint_brush | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| bandage | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| firstaidkit | 400 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| defibrillator | 800 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| paramedic_bag | 500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| water | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| wheelchair | 500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_FLASHLIGHT | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_FIREEXTINGUISHER | 500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_STUNGUN | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| stethoscope | 20 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| blowtorch | 5000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| scuba | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| parachute | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 150 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 750 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fixkit | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| carokit | 800 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| cleaningkit | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| blowtorch | 4500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| carjack | 250 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| spare_wheel | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| nitro | 7000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_WRENCH | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 10 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_FLASHLIGHT | 100 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_STUNGUN | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ecola | 18 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| sprunk | 18 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| tortilla | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| meat | 13 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| raw_bacon | 11 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| cheese | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| tomato | 7 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| potato | 6 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| water | 9 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| banana | 6 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| orange | 8 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| apple | 4 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| lemon | 6 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| mint | 2 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 150 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| radio | 750 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| fixkit | 1500 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| carokit | 800 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| cleaningkit | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| blowtorch | 2200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| carjack | 250 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| spare_wheel | 200 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| nitro | 7000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_WRENCH | 1000 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| gps | 0 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| doughnut | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| water | 2 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_PISTOL | 250 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_STUNGUN | 150 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| WEAPON_NIGHTSTICK | 150 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |
| ammo | 5 | składnik/frakcja | v2 §11A | rage_fractions/Config.lua |


## A.rage_jobs (11 poz.)

| Plik | Klucz | Obecnie | Docelowo | Ścieżka |
|---|---|---|---|---|
| hunter.lua | vehicleBail | 2500 | v2 §3A bail | [rage]\rage_jobs\Jobs\hunter.lua |
| lumberjack.lua | vehicleBail | 2500 | v2 §3A bail | [rage]\rage_jobs\Jobs\lumberjack.lua |
| lumberjack.lua | skinBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\lumberjack.lua |
| miner.lua | vehicleBail | 1500 | v2 §3A bail | [rage]\rage_jobs\Jobs\miner.lua |
| miner.lua | skinBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\miner.lua |
| slaughterer.lua | vehicleBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\slaughterer.lua |
| slaughterer.lua | skinBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\slaughterer.lua |
| tailor.lua | vehicleBail | 1500 | v2 §3A bail | [rage]\rage_jobs\Jobs\tailor.lua |
| tailor.lua | skinBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\tailor.lua |
| trashman.lua | vehicleBail | 1500 | v2 §3A bail | [rage]\rage_jobs\Jobs\trashman.lua |
| trashman.lua | skinBail | 500 | v2 §3A bail | [rage]\rage_jobs\Jobs\trashman.lua |


## A.kq_weed strains

| Odmiana | Parametr | Obecnie | Docelowo | Plik |
|---|---|---|---|---|
| og_kush | baseGrowTime | 20 | 28 | config.strains.lua |
| og_kush | wateringTime | 9 | 10 | config.strains.lua |
| og_kush | collectionLifespan | 25 | 25 | config.strains.lua |
| og_kush | yield.buds | 6-8 | 4-5 | config.strains.lua |
| purple_haze | baseGrowTime | 25 | 32 | config.strains.lua |
| purple_haze | wateringTime | 12 | 12 | config.strains.lua |
| purple_haze | collectionLifespan | 30 | 28 | config.strains.lua |
| purple_haze | yield.buds | 5-6 | 4-5 | config.strains.lua |
| white_widow | baseGrowTime | 30 | 36 | config.strains.lua |
| white_widow | wateringTime | 15 | 14 | config.strains.lua |
| white_widow | collectionLifespan | 20 | 22 | config.strains.lua |
| white_widow | yield.buds | 5-6 | 3-4 | config.strains.lua |
| blue_dream | baseGrowTime | 35 | 40 | config.strains.lua |
| blue_dream | wateringTime | 18 | 16 | config.strains.lua |
| blue_dream | collectionLifespan | 60 | 35 | config.strains.lua |
| blue_dream | yield.buds | 5-7 | 3-5 | config.strains.lua |


## A.Kod / SQL / sync

| Element | Obecnie | Docelowo | Plik |
|---|---|---|---|
| job_grades.salary SQL | różne | wzory §1b | MySQL |
| bossmenu sendSalary cap | brak | 10000/tydz | BabiczBossMenu_sv.lua |
| rage_hud Config.Food | sync market | jak v2 Market | rage_hud/Config.lua |
| vehicles.json per model | stare | ×0.08+cap | rage_vehicleshop/vehicles.json |
| VehiclePriceConfig.types | 12k-7.8M auto | 1k-1.2M | BabiczVehicleShop_cl.lua |
| customPlate | 750000 | 15000-40000 | rage_vehicleshop/Config.lua |
| testDrivePrice | 80 | 50-100 | rage_vehicleshop/Config.lua |
| rage_drugs harvest/craft | obecne | bez zmian | rage_drugs/Config.lua |
| depositDrugs PD | obecne | sync po Drugs v2 | BabiczPolice_shared.lua |
| localDispatch | 300-900 baza | 550-1150 OK | BabiczPolice_shared.lua |
| company depositProducts | ×1.5 | ×1.0-1.1 | BabiczCompanyShop_sv.lua |
| company cleaning reward | 30-80 | ×0,25 lub zostawić | c_*_config Cleaning |
| jobcenter salary UI | 6k-26k | real $/h | rage_jobcenter/Config.lua |


## A.Poza scope v2 (bez zmian)

BabiczHalloween, BabiczPumpkins, BabiczZoneCapture, [unused], BabiczAmmunation, BabiczAntyTroll, BabiczKD, BabiczHospitality, BabiczQueue, BabiczTracker, bd_collectibles, rage_admin, rage_ammunation, rage_carradio2, rage_interactions, rage_society, rage_znajdzki, trashman.


---


## 27. Kolejność prac (jeden release)

Wszystko wchodzi w **jednym deployu** — poniższa kolejność to **kolejność prac w repo / testów**, nie etapy produkcyjne.

1. §0 — blokery (paycheck on duty, KQ, NPC CD, bossmenu cap)  
2. MDT — grzywny ×0,10, jail z taryfikatora v2, ulgi (§7, §31)  
3. Napady aktywne + mnożniki + **Pacific §13a**  
4. rage_jobs + KQ powerwashing + hunter  
5. Pralnia + lombard + narkotyki  
6. rage_market + zdrapki + ox_fuel + wygląd + siłownia  
7. Firmy (kursy, składniki, depositProducts)  
8. Mechanik (tuning, kurs lawety)  
9. Zadania Moris + Tracker  
10. **vehicles.json** + **qs-housing DB** (§21, §25)  
11. Kasyno + Dark Shop + Ammunation + lb-phone + job center UI  
12. Plan testów (v2 § Plan testów po deployu)  

---

## 28. Ryzyka wdrożeniowe

| Ryzyko | Mitygacja |
|---|---|
| Wdrożenie częściowe | **Jeden deploy** — §0 obowiązkowe w całości |
| Farm NPC (15 s CD) | `BabiczNPC_sv.lua` → 3 min — §0 pkt 6 |
| Pacific bez wypłaty | §13a przed deployem |
| KQ bez zmian | §0 pkt 2 |
| qs-housing DB + config | §25 — w tym samym release |
| vehicles.json (~10k linii) | §21 — konwersja segmentów v2 |
| Zdrapki / kasyno bez rescale | §0 pkt 4 |
| Premie bez capu | §0 pkt 7 |
| Mandaty — tylko w Config.lua | `data.ts` = mock dev-only; produkcja czyta Config.lua | §7 |
| Carlos boosting 3200–13000$ · tracker 7500–16500$ | Rescale w tym samym commicie co §18 |

---


## 29. LSPD — local dispatch i laboratorium

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `.../police/shared/BabiczPolice_shared.lua` | `C.localDispatch.reward` | 300–900 + bonusy | bez zmian skali v2 | Gotowe |
| `.../police/server/BabiczPolice_localDispatch_sv.lua` | Wypłata `addMoney` | 550–1150 full | bez zmian | Gotowe |
| `.../police/shared/BabiczPolice_shared.lua` | `C.DrugLab.depositDrugs` | 30% ceny rynku | bez zmian na deploy | Poza scope przygotowania |
| `.../police/server/BabiczPolice_lab_sv.lua` | Depozyt → society | police bossmenu | bez zmian | Gotowe |

---

## 30. Mechanik — kurs lawetą (zepsute auta)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `.../mechanic/shared/BabiczMechanic_shared.lua` | `C.TowCourse` (nowy) | brak | 150–350$/kurs, CD 15–20 min, 50% society | Planowany |
| `.../mechanic/server/` | Logika kursu + spawn NPC awaria | brak | jedna z 13 lokacji LocalMechanic | Planowany |
| `.../mechanic/client/` | UI / target flatbed | flatbed 0$ | integracja z kursem | Planowany |
| `.../mechanic/shared/BabiczMechanic_shared.lua` | `C.Course` paczki | 400–800$/pkg | wyłączyć lub 80–150$ | Planowany |

---

## 31. MDT — wyroki (ulgi + jail)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_mdt/Config.lua` | `policeFines` — grzywny | 230 poz. | ×0,10 — tabela §7 | Gotowe do wdrożenia |
| `[rage]/rage_mdt/Config.lua` | `policeFines` — `jail` | 300+ mies. zabójstwo | Taryfikator v2 § Wyroki (np. zabójstwo II → **50** mies.) | Planowany |
| `[rage]/rage_mdt/Config.lua` | Okoliczności łagodzące | ulga do −25000$; −120 mies. | −250…−2500$; −1…−60 mies.; **usuń** wyższe tiery | Planowany |
| `[rage]/rage_mdt/BabiczMdt_sv.lua` | `jail * 0.3` | 30% czasu | bez zmian | Gotowe |
| `[rage]/rage_mdt/BabiczMdt_sv.lua` | `CalculateFinePlayerShare` | flat 25% | Progresywny split — §7 | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.FinePlayerShare` | brak | Progi + cap 750$ — §7 | Planowany |

### Progresywny split mandatów (implementacja)

**Plik:** `[rage]/rage_mdt/Config.lua` → `Config.FinePlayerShare`

```lua
Config.FinePlayerShare = {
    cap = 750,
    brackets = {
        { limit = 150,  percent = 0.50 },
        { limit = 400,  percent = 0.35 },
        { limit = 1500, percent = 0.25 },
        { limit = 5000, percent = 0.12 },
        { limit = math.huge, percent = 0.05 },
    },
}
```

**Plik:** `[rage]/rage_mdt/BabiczMdt_sv.lua` — po obliczeniu `fine` (zniżka DOJ, ubezpieczenie EMS):

1. `forPlayer = CalculateFinePlayerShare(fine, cache.config)`
2. `forSociety = fine - forPlayer`
3. Cap stosuje się **przed** wypłatą do banku FP

**Per job (`Config.Jobs`):**

| Job | `finePercentProgressive` | Uwagi |
|---|---|---|
| police, fib, doj, ambulance | `true` | progi z `Config.FinePlayerShare` |
| mechanic, c_blackrepair, c_quiettech | `false` | flat `finePercentForPlayer` (LSC docelowo 40%) |

---

## 32. Indeks paczki — co audytowano


### W scope wdrożenia v2 (zmiany w tym dokumencie)

`RageCity`, `rage_market`, `rage_jobs`, `rage_fractions`, `rage_mdt`, `rage_heists`, `rage_garages`, `rage_vehicleshop`, `rage_multicharacter`, `rage_bossmenu`, `rage_jobcenter`, `rage_hud`, `rage_drugs`, `rage_meth`, `rage_scratchcard`, `rage_banking`, `es_extended`, `ox_fuel`, `ox_inventory`, `kq_deliveries`, `kq_powerwashing`, `kq_weed`, `qs-housing`, `pickle_casinos`, `lb-phone`, `rtx_bowling`, `rcore_pool`


### Poza scope (bez zmian ekonomii v2)

BabiczHalloween, BabiczPumpkins, BabiczZoneCapture, [unused], BabiczAmmunation, BabiczAntyTroll, BabiczKD, BabiczHospitality, BabiczQueue, BabiczTracker, bd_collectibles, rage_admin, rage_ammunation, rage_carradio2, rage_interactions, rage_society, rage_znajdzki, trashman.

**Regeneracja tabel:** `python docs/_gen_wdrozenie_catalog.py` → `_wdrozenie_catalog_generated.md` (merge do §3A, §11A, §16, §19A, §21B).


---

*Ostatnia aktualizacja: §0 blokery, Pacific §13a, qs-housing §25, NPC farm, trucker §14a (3500–8000$), spójność ulg/jail. Deploy = jeden release.*
