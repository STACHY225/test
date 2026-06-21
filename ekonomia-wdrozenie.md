# RageCity — wdrożenie ekonomii v2 (developer)

Checklista techniczna: **co, gdzie i jak zmienić** w kodzie / DB.  
Model balansu (dlaczego, widełki, progresja): [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md).

**Deploy:** **jeden release** — wszystkie pozycje z §0 i poniższych sekcji wchodzą **razem**. Częściowe wdrożenie psuje ekonomię.

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
| `[rage]/rage_mdt/Config.lua` | `policeFines` — grzywny | drobne 1000$+; ciężkie 25000–50000$ | **×0,10** — tabela poniżej (tylko kolumna Docelowo $) | Gotowe do wdrożenia |
| `[rage]/rage_mdt/Config.lua` | `policeFines` — `jail` | 300+ mies. zabójstwo itd. | Wartości z **taryfikatora v2** § Wyroki (np. zabójstwo II → **50** mies.) | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.ambulance` | 76 pozycji 300–10000$ | Pełna tabela docelowa — [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md) § EMS — faktury medyczne | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.mechanic` | 8 usług 300–2000$ | Pełna tabela docelowa — [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md) § Mechanik — usługi MDT | Planowany |
| `[rage]/rage_mdt/Config.lua` | Faktury EMS (medical invoices) | 300–10000$ | Revive 200–500$, leczenie 150–400$ itd. | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` | LSPD/FIB/EMS 25%; c_blackrepair/c_quiettech 40% | Bez zmian struktury; kwoty bazowe niższe | Planowany |
| `[rage]/rage_mdt/Config.lua` | `fineCompanyMultiplier` | 1.5–2.5× | Bez zmian | Planowany |
| `[rage]/rage_mdt/html/src/config/data.ts` | UI mandatów | mock dev-only | **Nie** źródło produkcyjne — tylko `Config.lua` | Gotowe |

### Pełna tabela policeFines (230 wykroczeń)

Odtwarzalna skryptem [docs/_gen_mdt_fines_table.py](_gen_mdt_fines_table.py) z [rage]/rage_mdt/Config.lua.

**Grzywny:** kolumna **Docelowo** = ×0,10 obecnej kwoty (230 pozycji).  
**Więzienie (`jail`):** **wyłącznie** z taryfikatora v2 § Wyroki w [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md) — kolumna jail **usunięta** z tabeli poniżej (stare wartości Config były mylące).  
**Okoliczności łagodzące:** osobna skala — max **−2500$** mandat, max **−60** mies.; usunąć pozycje −5000 / −10000 / −25000 i −120 mies.

#### Przestępstwa przeciwko życiu i zdrowiu

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Zabójstwo II stopnia | 25000$ | **2500$** |
| Zabójstwo II stopnia funkcjonariusza publicznego | 50000$ | **5000$** |
| Zabójstwo w afekcie (Voluntary Manslaughter) | 10000$ | **600$** |
| Nieumyślne spowodowanie śmierci | 10000$ | **600$** |
| Nieumyślne spowodowanie śmierci pojazdem (DUI) | 15000$ | **1500$** |
| Ciężki uszczerbek na zdrowiu | 5000$ | **300$** |
| Ciężki uszczerbek na zdrowiu z użyciem broni palnej | 10000$ | **600$** |
| Ciężki uszczerbek na zdrowiu funkcjonariusza | 10000$ | **600$** |
| Napaść z użyciem niebezpiecznego narzędzia | 5000$ | **300$** |
| Napaść (Assault) | 1000$ | **80$** |
| Pobicie (Battery) | 2000$ | **160$** |
| Pobicie z użyciem niebezpiecznego przedmiotu | 5000$ | **300$** |
| Napaść na funkcjonariusza publicznego | 5000$ | **300$** |
| Bezprawne pozbawienie wolności | 5000$ | **300$** |
| Porwanie dla okupu | 25000$ | **2500$** |
| Porwanie z użyciem broni palnej | 30000$ | **3000$** |
| Porwanie funkcjonariusza publicznego | 50000$ | **5000$** |
| Usiłowanie zabójstwa | 25000$ | **2500$** |
| Usiłowanie zabójstwa funkcjonariusza | 50000$ | **5000$** |
#### Przestępstwa przeciwko mieniu

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Kradzież małej wartości (poniżej $950) | 1000$ | **80$** |
| Kradzież ($950 - $5000) | 3000$ | **175$** |
| Kradzież wielka (powyżej $5000) | 10000$ | **600$** |
| Kradzież z pojazdu | 2000$ | **160$** |
| Kradzież tożsamości | 10000$ | **600$** |
| Rozbój | 10000$ | **600$** |
| Rozbój z użyciem broni palnej | 25000$ | **2500$** |
| Rozbój z użyciem broni białej | 15000$ | **1500$** |
| Rozbój na instytucji finansowej | 50000$ | **5000$** |
| Włamanie do budynku mieszkalnego | 10000$ | **600$** |
| Włamanie do obiektu komercyjnego | 5000$ | **300$** |
| Włamanie z użyciem narzędzi | 10000$ | **600$** |
| Włamanie do pojazdu | 2000$ | **160$** |
| Kradzież pojazdu (GTA) | 10000$ | **600$** |
| Kradzież pojazdu uprzywilejowanego | 20000$ | **2000$** |
| Nielegalne posiadanie skradzionego pojazdu | 5000$ | **300$** |
| Wymuszenie / szantaż | 10000$ | **600$** |
| Wymuszenie z użyciem broni | 25000$ | **2500$** |
| Wymuszenie przez zorganizowaną grupę | 50000$ | **5000$** |
| Oszustwo (do $5000) | 3000$ | **175$** |
| Oszustwo (powyżej $5000) | 10000$ | **600$** |
| Oszustwo na szkodę instytucji publicznej | 25000$ | **2500$** |
| Fałszerstwo dokumentów | 5000$ | **300$** |
| Fałszerstwo pieniędzy / waluty | 25000$ | **2500$** |
| Wandalizm (szkoda poniżej $400) | 1000$ | **80$** |
| Wandalizm (szkoda powyżej $400) | 5000$ | **300$** |
| Zniszczenie mienia publicznego | 5000$ | **300$** |
| Zniszczenie pojazdu służbowego | 10000$ | **600$** |
| Pranie pieniędzy (do $50 000) | 20000$ | **2000$** |
| Pranie pieniędzy (powyżej $50 000) | 50000$ | **5000$** |
| Pranie pieniędzy przez zorganizowaną grupę | 100000$ | **4000$** |
#### Przestępstwa przeciwko porządkowi publicznemu

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Zakłócenie porządku publicznego | 500$ | **50$** |
| Wywołanie zbiegowiska / zamieszek | 2000$ | **160$** |
| Napaść słowna / groźby publiczne | 1000$ | **80$** |
| Groźba karalna | 1000$ | **80$** |
| Groźba z użyciem broni | 5000$ | **300$** |
| Groźba wobec funkcjonariusza publicznego | 5000$ | **300$** |
| Stalking / nękanie | 2000$ | **160$** |
| Użycie broni palnej podczas przestępstwa (+do kary) | 10000$ | **600$** |
| Oddanie strzału podczas przestępstwa (+do kary) | 20000$ | **2000$** |
| Napaść na funkcjonariusza (bez obrażeń) | 5000$ | **300$** |
| Pobicie funkcjonariusza (z obrażeniami) | 10000$ | **600$** |
| Pobicie funkcjonariusza z bronią | 25000$ | **2500$** |
| Usiłowanie zabójstwa funkcjonariusza | 50000$ | **5000$** |
| Ucieczka piesza przed funkcjonariuszem | 500$ | **50$** |
| Ucieczka pojazdem (niebezpieczna jazda) | 5000$ | **300$** |
| Ucieczka pojazdem z obrażeniami u innych | 15000$ | **1500$** |
| Ucieczka pojazdem ze śmiercią ofiary | 25000$ | **2500$** |
| Opór bierny przy zatrzymaniu | 500$ | **50$** |
| Czynny opór przy zatrzymaniu | 2000$ | **160$** |
| Atak na funkcjonariusza podczas zatrzymania | 10000$ | **600$** |
| Groźby terrorystyczne | 50000$ | **5000$** |
| Akt terrorystyczny bez ofiar | 100000$ | **4000$** |
| Udział w zorganizowanej grupie przestępczej | 25000$ | **2500$** |
| Kierowanie grupą przestępczą | 50000$ | **5000$** |
| Finansowanie działalności przestępczej | 100000$ | **4000$** |
#### Broń palna

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Posiadanie broni bez licencji (klasa I) | 5000$ | **300$** |
| Posiadanie broni klasy II przez cywila | 25000$ | **2500$** |
| Posiadanie broni przez osobę z zakazem | 25000$ | **2500$** |
| Posiadanie niezarejestrowanej broni | 10000$ | **600$** |
| Posiadanie broni klasy III bez zezwolenia | 50000$ | **5000$** |
| Posiadanie broni klasy IV (materiały wybuchowe) | 50000$ | **5000$** |
| Posiadanie ghost gun (bez numeru seryjnego) | 25000$ | **2500$** |
| Posiadanie tłumika bez zezwolenia | 20000$ | **2000$** |
| Posiadanie magazynka powyżej 10 nabojów | 5000$ | **300$** |
| Open carry bez uprawnień służbowych | 3000$ | **175$** |
| Concealed carry bez licencji CCW | 5000$ | **300$** |
| Noszenie broni załadowanej w pojeździe (bez CCW) | 2500$ | **150$** |
| Niezgłoszenie broni podczas kontroli drogowej | 1500$ | **120$** |
| Sprzedaż broni bez licencji FFL | 50000$ | **5000$** |
| Sprzedaż broni bez weryfikacji nabywcy | 25000$ | **2500$** |
| Sprzedaż broni osobie nieuprawnionej | 50000$ | **5000$** |
| Pośrednictwo w nielegalnym obrocie bronią | 50000$ | **5000$** |
| Niezgłoszenie kradzieży broni w terminie 48h | 3000$ | **175$** |
| Wejście do Gun Free Zone z bronią (nieumyślne) | 2500$ | **150$** |
| Wejście do Gun Free Zone z bronią (umyślne) | 10000$ | **600$** |
| Wniesienie broni do sądu lub więzienia | 25000$ | **2500$** |
| Bezpodstawne użycie broni palnej | 5000$ | **300$** |
#### Narkotyki

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Posiadanie marihuany przez niepełnoletniego | 500$ | **50$** |
| Posiadanie marihuany (28,5g - 500g) | 2000$ | **160$** |
| Posiadanie marihuany (powyżej 500g) | 10000$ | **600$** |
| Posiadanie marihuany z zamiarem dystrybucji | 25000$ | **2500$** |
| Sprzedaż marihuany bez licencji dispensary | 50000$ | **5000$** |
| Sprzedaż marihuany osobie niepełnoletniej | 50000$ | **5000$** |
| Używanie marihuany w miejscu publicznym / pojeździe | 250$ | **30$** |
| Uprawa powyżej 6 roślin marihuany | 5000$ | **300$** |
| Posiadanie Schedule I/II (mała ilość) | 10000$ | **600$** |
| Posiadanie Schedule I/II (duża ilość) | 25000$ | **2500$** |
| Posiadanie Schedule I/II z zamiarem dystrybucji | 50000$ | **5000$** |
| Posiadanie Schedule I/II w pobliżu szkoły (+300m) | 35000$ | **3500$** |
| Posiadanie Schedule III bez recepty | 5000$ | **300$** |
| Posiadanie Schedule IV bez recepty | 2000$ | **160$** |
| Posiadanie Schedule III z zamiarem dystrybucji | 25000$ | **2500$** |
| Wytwarzanie narkotyków (mała skala) | 50000$ | **5000$** |
| Wytwarzanie narkotyków (duża skala) | 100000$ | **4000$** |
| Laboratorium narkotykowe (meth lab / crack lab) | 100000$ | **4000$** |
| Wytwarzanie fentanylu / syntetyków opioidowych | 100000$ | **4000$** |
| Dystrybucja Schedule I/II (mała skala) | 50000$ | **5000$** |
| Dystrybucja Schedule I/II (duża skala) | 150000$ | **6000$** |
| Przemyt narkotyków przez granicę stanu | 100000$ | **4000$** |
| Kierowanie siatką dystrybucji narkotyków | 200000$ | **8000$** |
| Sprzedaż narkotyków niepełnoletniemu | 50000$ | **5000$** |
| Dystrybucja w pobliżu szkoły (+300m) | 100000$ | **4000$** |
| Posiadanie parafernaliów do Schedule I/II | 1500$ | **120$** |
| Sprzedaż parafernaliów narkotykowych | 5000$ | **300$** |
| Sprzedaż parafernaliów niepełnoletniemu | 25000$ | **2500$** |
#### Przestępstwa przeciwko wymiarowi sprawiedliwości

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Krzywoprzysięstwo | 5000$ | **300$** |
| Fałszywe zeznania w sprawie o zabójstwo | 25000$ | **2500$** |
| Utrudnianie postępowania karnego | 5000$ | **300$** |
| Niszczenie lub ukrywanie dowodów | 10000$ | **600$** |
| Zastraszanie świadków / ławy przysięgłych | 25000$ | **2500$** |
| Przekupstwo świadka | 25000$ | **2500$** |
| Fałszywe zgłoszenie przestępstwa | 2000$ | **160$** |
| Wręczenie łapówki funkcjonariuszowi | 25000$ | **2500$** |
| Przyjęcie łapówki przez funkcjonariusza | 50000$ | **5000$** |
| Przekupstwo sędziego lub prokuratora | 100000$ | **4000$** |
| Ucieczka z aresztu | 5000$ | **300$** |
| Ucieczka z zakładu karnego | 10000$ | **600$** |
| Pomoc w ucieczce z aresztu / więzienia | 10000$ | **600$** |
| Pomoc w ucieczce z konwoju | 10000$ | **600$** |
| Podszywanie się pod funkcjonariusza publicznego | 5000$ | **300$** |
| Podszywanie się z użyciem fałszywego umundurowania | 10000$ | **600$** |
| Składanie fałszywych zeznań | 5000$ | **300$** |
| Współudział w przestępstwie | 5000$ | **300$** |
| Fałszywe wezwanie służb | 2000$ | **160$** |
#### Bezpieczeństwo publiczne

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Nielegalne posiadanie materiałów wybuchowych | 25000$ | **2500$** |
| Wytwarzanie materiałów wybuchowych | 50000$ | **5000$** |
| Użycie materiałów wybuchowych bez ofiar | 100000$ | **4000$** |
| Podpalenie mienia (bez ofiar) | 10000$ | **600$** |
| Podpalenie budynku mieszkalnego | 25000$ | **2500$** |
| Podpalenie z ofiarami w środku | 50000$ | **5000$** |
| Podpalenie budynku publicznego | 50000$ | **5000$** |
#### Przestępstwa przeciwko państwu

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Ujawnienie tajemnicy służbowej | 25000$ | **2500$** |
| Nadużycie uprawnień służbowych | 10000$ | **600$** |
| Bezprawne pozbawienie wolności przez funkcjonariusza | 15000$ | **1500$** |
| Fałszowanie dokumentów służbowych | 10000$ | **600$** |
| Ujawnienie tajemnicy śledczej | 25000$ | **2500$** |
#### Wykroczenia drogowe

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Jazda bez prawa jazdy | 1000$ | **80$** |
| Jazda z zawieszonym prawem jazdy | 2500$ | **150$** |
| Jazda z cofniętym prawem jazdy | 5000$ | **300$** |
| Przekroczenie prędkości o 1-15 mph | 350$ | **35$** |
| Przekroczenie prędkości o 16-25 mph | 700$ | **60$** |
| Przekroczenie prędkości o 26-40 mph | 1500$ | **120$** |
| Przekroczenie prędkości o ponad 40 mph | 3000$ | **175$** |
| Przekroczenie prędkości w strefie szkolnej | 1400$ | **110$** |
| Przejazd na czerwonym świetle | 1000$ | **80$** |
| Niezatrzymanie się przed linią zatrzymania | 500$ | **50$** |
| Niezatrzymanie się przed przejściem dla pieszych | 1000$ | **80$** |
| Nieustąpienie pierwszeństwa pieszemu | 1500$ | **120$** |
| Nieustąpienie pierwszeństwa pojazd. uprzywilejowanemu | 1000$ | **80$** |
| Niezastosowanie się do znaku STOP | 700$ | **60$** |
| Niezastosowanie się do polecenia funkcjonariusza | 1500$ | **120$** |
| Jazda pod prąd | 700$ | **60$** |
| Brak zapiętych pasów bezpieczeństwa | 300$ | **30$** |
| Używanie telefonu podczas jazdy | 500$ | **50$** |
| Brawurowa / niebezpieczna jazda | 2000$ | **160$** |
| Niebezpieczna jazda z obrażeniami u innych | 10000$ | **600$** |
| Niedozwolone wyprzedzanie | 1000$ | **80$** |
| Jazda niepoprawnym pasem ruchu | 400$ | **40$** |
| DUI - BAC 0.08-0.14% (pierwsze) | 5000$ | **300$** |
| DUI - BAC 0.15%+ (pierwsze) | 10000$ | **600$** |
| DUI - drugie naruszenie | 10000$ | **600$** |
| DUI - z wypadkiem bez obrażeń | 10000$ | **600$** |
| DUI - z obrażeniami u innych | 15000$ | **1500$** |
| Odmowa badania alkotestem | 1500$ | **120$** |
| Spowodowanie kolizji | 500$ | **50$** |
| Spowodowanie wypadku | 1500$ | **120$** |
| Potrącenie pieszego | 2000$ | **160$** |
| Ucieczka z miejsca wypadku - bez ofiar (hit and run) | 5000$ | **300$** |
| Jazda pojazdem niezdatnym do ruchu | 2000$ | **160$** |
| Niesprawne oświetlenie | 350$ | **35$** |
| Niedozwolony kolor świateł (niebieski/czerwony) | 1500$ | **120$** |
| Niedozwolone przyciemnienie przedniej szyby | 700$ | **60$** |
| Niedozwolone modyfikacje pojazdu | 2000$ | **160$** |
| Nadmierny hałas układu wydechowego | 500$ | **50$** |
| Jazda pojazdem niezarejestrowanym | 1500$ | **120$** |
| Nieczytelne tablice rejestracyjne | 350$ | **35$** |
| Parkowanie przy czerwonym krawężniku | 500$ | **50$** |
| Parkowanie na przejściu dla pieszych | 700$ | **60$** |
| Parkowanie na miejscu dla niepełnosprawnych | 1500$ | **120$** |
| Brak dokumentów pojazdu podczas kontroli | 300$ | **30$** |
| Niezgłoszenie broni podczas kontroli drogowej | 1500$ | **120$** |
| Jazda bez kasku (motocykl / quad) | 500$ | **50$** |
| Niedopuszczalny lane splitting | 700$ | **60$** |
| Udział w nielegalnym wyścigu | 5000$ | **300$** |
| Niezatrzymanie pojazdu do kontroli | 1500$ | **120$** |
| Ucieczka pojazdem przed służbami | 5000$ | **300$** |
#### Naruszenia gospodarcze

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Prowadzenie działalności bez rejestracji | 5000$ | **300$** |
| Prowadzenie działalności bez licencji branżowej | 15000$ | **1500$** |
| Podanie fałszywych danych przy rejestracji | 10000$ | **600$** |
| Prowadzenie działalności po cofnięciu licencji | 25000$ | **2500$** |
| Zaniżenie przychodów w deklaracji podatkowej | 25000$ | **2500$** |
| Ukrywanie przychodów / podwójna księgowość | 100000$ | **4000$** |
| Oszustwo gospodarcze na szkodę kontrahentów | 50000$ | **5000$** |
| Kartel / porozumienie antykonkurencyjne | 100000$ | **4000$** |
| Zatrudnianie bez umowy (praca na czarno) | 10000$ | **600$** |
#### Okoliczności łagodzące

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Ulga mandatowa -250$ | -250$ | **-250$** |
| Ulga mandatowa -500$ | -500$ | **-500$** |
| Ulga mandatowa -750$ | -750$ | **-750$** |
| Ulga mandatowa -1000$ | -1000$ | **-1000$** |
| Ulga mandatowa -1500$ | — | **-1500$** (dodać) |
| Ulga mandatowa -2000$ | — | **-2000$** (dodać) |
| Ulga mandatowa -2500$ | -2500$ | **-2500$** |
| Ulga mandatowa -5000$ | -5000$ | **usunąć** |
| Ulga mandatowa -10000$ | -10000$ | **usunąć** |
| Ulga mandatowa -25000$ | -25000$ | **usunąć** |
| Ulga więzienna -1 miesiąc | 0$ | **0$** |
| Ulga więzienna -6 miesięcy | 0$ | **0$** |
| Ulga więzienna -12 miesięcy | 0$ | **0$** |
| Ulga więzienna -24 miesiące | 0$ | **0$** |
| Ulga więzienna -36 miesięcy | 0$ | **0$** |
| Ulga więzienna -60 miesięcy | 0$ | **0$** |
| Ulga więzienna -120 miesięcy | 0$ | **usunąć** |

---

## 8. Frakcje — FIB

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | `police`, `fib` | 250$/15 min | 168–175$/15 min (672–700$/h) | Planowany |
| `[core]/es_extended/server/paycheck.lua` | `ambulance` | 250$/15 min | 115–125$/15 min (460–500$/h) | Planowany |
| `[core]/es_extended/server/paycheck.lua` | `doj` | 250$/15 min | 150–158$/15 min (600–632$/h) | Planowany |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.fib` | `policeFines` | ten sam katalog po przeskalowaniu | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` (fib) | 25% | 25% (bez zmian) | Planowany |

---

## 9. Frakcje — EMS

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | Kursy lokalne `rewardMoneyMin/Max` | 350–850$ | 200–500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | LocalMedic pricing | 50–10000$ dynamiczny | 500–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | `HealthInsurance.Prices` | 3500 / 6000 / 12000$ | 800–1200 / 1400–2000 / 2500–3500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | `respawnFine`, `respawnFineWhileMedicsOnline` | 500 / 5000$ | 150–300 / 800–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/server/BabiczDeath_sv.lua` | Revive split (135% firma, 10% DOJ) | Procenty | Bez zmian struktury | Planowany |

### Pełna tabela faktur EMS (76 pozycji)

Pełna tabela docelowa: [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md) § EMS — faktury medyczne. Generator (sync): [`_gen_mdt_fines_table.py`](_gen_mdt_fines_table.py).

Algorytm EMS: ≤600$ ×0,15; ≤2000$ ×0,12; ≤5000$ ×0,10; powyżej ×0,08 (cap 1200$). Udział wystawiającego EMS: 25% (bez zmian struktury).

#### Złamania

| Usługa / wykroczenie | Obecnie | Docelowo |
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

#### Rany i obrażenia

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
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

#### Diagnostyka

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| RTG (jedna partia ciała) | 1500$ | **175$** |
| RTG rozszerzone | 2400$ | **250$** |
| USG | 1500$ | **175$** |
| Tomografia komputerowa | 3000$ | **300$** |
| Rezonans magnetyczny | 3500$ | **350$** |
| Badanie krwi | 600$ | **90$** |
| Badanie toksykologiczne | 900$ | **100$** |
| Badanie alkoholu | 480$ | **70$** |
| Badanie narkotykowe | 900$ | **100$** |

#### Konsultacje specjalistyczne

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Badania psychologiczne (na broń palną) | 10000$ | **800$** |
| Konsultacja psychologiczna | 3000$ | **300$** |
| Konsultacja psychiatryczna | 2700$ | **275$** |
| Konsultacja neurologiczna | 2400$ | **250$** |
| Konsultacja ortopedyczna | 2400$ | **250$** |
| Konsultacja chirurgiczna | 2700$ | **275$** |
| Konsultacja kardiologiczna | 2400$ | **250$** |
| Konsultacja internistyczna | 2100$ | **200$** |

#### Farmakologia i zabiegi

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
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

#### Transport i personel EMS

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Zespół ratownictwa medycznego | 1500$ | **175$** |
| Wezwanie chirurga | 2400$ | **250$** |
| Wezwanie specjalisty | 2100$ | **200$** |
| Transport karetką | 2100$ | **200$** |
| Transport lotniczy | 7200$ | **600$** |

#### Wykroczenia medyczne

| Usługa / wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Nieuzasadnione wezwanie EMS | 3000$ | **300$** |
| Fałszywe zgłoszenie | 4800$ | **475$** |
| Odmowa współpracy z EMS | 1500$ | **175$** |
| Utrudnianie czynności medycznych | 2400$ | **250$** |
| Ucieczka z miejsca leczenia | 4800$ | **475$** |
| Zniewaga funkcjonariusza EMS | 1500$ | **175$** |
| Agresja wobec personelu medycznego | 9000$ | **700$** |
| Zniszczenie mienia EMS | 3000$ | **300$** |
| Leczenie pod eskortą LSPD/LSSD | 2400$ | **250$** |

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

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Items` | Wszystkie 33 itemy | diamond 155–160; phone 750–1250; gold_chain 800–1250 | Tabela v2 (15–160$) — patrz `ekonomia-zmiany-v2.md` § Biżuteria i lombard | Planowany |
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Black` | multiplier | 0.95 | 0.95 (bez zmian) | Planowany |
| `[rage]/RageCity/server/platetape.lua` / lombard flow | Czas sprzedaży (`time`) | 900–1280 ms | **bez zmian** | Gotowe |

---

## 16. Narkotyki i sprzedaż

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Drugs` | Ceny sprzedaży 17 itemów | 75–1050$ | Tabela v2 (25–220$) | Planowany |
| `[rage]/RageCity/Config.lua` → `Config.DrugSelling` | copsRequired, callCopsChance, Zones | 2 / 65% / 1 strefa | bez zmian parametrów | Planowany |
| `[rage]/RageCity/server/drugsales.lua` | Logika sprzedaży | quality multiplier | Bez zmian struktury | Planowany |
| `[rage]/rage_drugs/Config.lua` | Zbiory / craft | Yields, czasy | **bez zmian na deploy** | Gotowe |
| `[tebex]/kq_weed/config.strains.lua` | yields, `baseGrowTime`, `wateringTime` | patrz obecny config | v2 § KQ weed — tabela per odmiana | Planowany |

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
| `.../shared/2_Carlos.lua` | Zad. 1 rozwóz — 9 lokacji | (brak — do implementacji) | 40–80$ dirty/stop; CD 45 min | Planowany |
| `.../shared/2_Carlos.lua` | Zad. 2 boosting — `vehicles[].reward` | 3200–13000$ clean | 350–900$ tier auta; CD 45 min | Planowany |
| `.../server/2_Carlos.lua` | Wypłata boosting | pełna kwota z config | rescale + kara −20% uszkodzenie | Planowany |
| `.../shared/2_Carlos.lua` | Zad. 3 tracker | (brak) | 500–1200$ black; CD 60 min | Planowany |
| `.../shared/2_Carlos.lua` | Zad. 4 przemyt | (brak) | 900–1600$ black; CD 90 min | Planowany |
| `[rage]/BabiczTracker/` | Integracja tracker/przemyt | 500–5000$; opłata 500$ | wypłaty questowe bez opłaty; heist wg v2 | Planowany |
| `.../client/2_Carlos.lua` | UI / SMS / etapy | boosting (t1) | + 3 nowe zadania | Planowany |

**Uwaga:** w kodzie boosting jest `Moris_Carlos_1` — docelowa numeracja: t1 rozwóz, t2 boosting (wymaga reindex lub nowe ID).

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

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/pickle_casinos/config.lua` | `MaxWager` | 500 000$ | 5 000–15 000$ | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `MaximumChipPurchaseAmount` | 1 000 000$ | 50 000–100 000$ | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `Config.Chips` nominały | do 1M | max 10–25k | Planowany |
| `[tebex]/pickle_casinos/config.lua` | `tax`, `taxRecipients` | 10%; PD/EMS 50/50 | bez zmian | Planowany |

### Dark Shop (`rage_market`)

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
| Carlos boosting 3200–13000$ | Rescale w tym samym commicie co §18 |

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

---

*Ostatnia aktualizacja: §0 blokery, Pacific §13a, qs-housing §25, NPC farm, trucker §14a (3500–8000$), spójność ulg/jail. Deploy = jeden release.*
