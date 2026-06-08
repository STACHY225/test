# RageCity — lista zmian ekonomii (wdrożenie)

Checklista zmian per skrypt. **Dokument planistyczny — bez wdrożenia w kodzie.**  
Odniesienie docelowe: [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md).

Legenda statusu: `Planowany` | `Do ustalenia` | `Gotowe do wdrożenia`

---

## 1. Paycheck i zasiłek

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | Stawki per frakcja, on duty | 125$/15 min cywile; 250$/15 min frakcje; bez on duty | Zasiłek 15$/15 min; LSPD/LSSD/FIB **168–175$/15 min**; EMS **115–125$/15 min**; DOJ **150–158$/15 min**; tylko on duty | Planowany |
| `[core]/es_extended/config.lua` | `PaycheckInterval`, `StartingAccountMoney` | 15 min; bank 2500$ | bez zmian | Planowany |
| SQL `job_grades.salary` | Grade salaries | różne w DB | zgodnie z modelem lub wyłączone | Do ustalenia |

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
| `[tebex]/kq_deliveries/config.lua` | `payPerMinute` per job | 330–500$/min (~20k+/h) | ~7–11$/min (~450–650$/h z zasiłkiem) | Planowany |
| `[tebex]/kq_deliveries/config.lua` | Bonusy, team %, level upgrades | Wysokie mnożniki | Obniżyć proporcjonalnie | Planowany |
| `[tebex]/kq_powerwashing/contracts.lua` | `reward` per kontrakt | 4800–24000$+ | ~800–2000$ per kontrakt | Planowany |
| `[tebex]/kq_powerwashing/config.lua` | Bonusy, teamWorkBonuses | do +50% | Obniżyć proporcjonalnie | Planowany |

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
| `[rage]/rage_mdt/Config.lua` | `Config.Fines` — `policeFines` | drobne 1000$+; ciężkie 25000–50000$ | **230 wykroczeń** — tabela poniżej | Gotowe do wdrożenia |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.ambulance` | 76 pozycji 300–10000$ | tabela §9 + `_mdt_ems_fines_v2_table.md` | Gotowe do wdrożenia |
| `[rage]/rage_mdt/Config.lua` | `Config.Fines.mechanic` | 8 usług 300–2000$ | tabela §12 + `_mdt_mechanic_fines_v2_table.md` | Gotowe do wdrożenia |
| `[rage]/rage_mdt/Config.lua` | Faktury EMS (medical invoices) | 300–10000$ | Revive 200–500$, leczenie 150–400$ itd. | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` | LSPD/FIB/EMS 25%; c_blackrepair/c_quiettech 40% | Bez zmian struktury; kwoty bazowe niższe | Planowany |
| `[rage]/rage_mdt/Config.lua` | `fineCompanyMultiplier` | 1.5–2.5× | Bez zmian | Planowany |
| `[rage]/rage_mdt/html/src/config/data.ts` | UI mandatów (jeśli duplikuje kwoty) | Wysokie | Zgodnie z Config.lua | Planowany |

### Pełna tabela policeFines (230 wykroczeń)

Odtwarzalna skryptem [docs/_gen_mdt_fines_table.py](_gen_mdt_fines_table.py) z [rage]/rage_mdt/Config.lua.

Ulgi mandatowe: ×0,10. Więzienie (`jail`) — bez zmian.

#### Przestępstwa przeciwko życiu i zdrowiu

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Zabójstwo II stopnia | 25000$ | **2500$** | 300 |
| Zabójstwo II stopnia funkcjonariusza publicznego | 50000$ | **5000$** | 360 |
| Zabójstwo w afekcie (Voluntary Manslaughter) | 10000$ | **600$** | 132 |
| Nieumyślne spowodowanie śmierci | 10000$ | **600$** | 132 |
| Nieumyślne spowodowanie śmierci pojazdem (DUI) | 15000$ | **1500$** | 120 |
| Ciężki uszczerbek na zdrowiu | 5000$ | **300$** | 96 |
| Ciężki uszczerbek na zdrowiu z użyciem broni palnej | 10000$ | **600$** | 144 |
| Ciężki uszczerbek na zdrowiu funkcjonariusza | 10000$ | **600$** | 120 |
| Napaść z użyciem niebezpiecznego narzędzia | 5000$ | **300$** | 72 |
| Napaść (Assault) | 1000$ | **80$** | 6 |
| Pobicie (Battery) | 2000$ | **160$** | 12 |
| Pobicie z użyciem niebezpiecznego przedmiotu | 5000$ | **300$** | 48 |
| Napaść na funkcjonariusza publicznego | 5000$ | **300$** | 48 |
| Bezprawne pozbawienie wolności | 5000$ | **300$** | 36 |
| Porwanie dla okupu | 25000$ | **2500$** | 132 |
| Porwanie z użyciem broni palnej | 30000$ | **3000$** | 180 |
| Porwanie funkcjonariusza publicznego | 50000$ | **5000$** | 240 |
| Usiłowanie zabójstwa | 25000$ | **2500$** | 225 |
| Usiłowanie zabójstwa funkcjonariusza | 50000$ | **5000$** | 300 |
#### Przestępstwa przeciwko mieniu

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Kradzież małej wartości (poniżej $950) | 1000$ | **80$** | 6 |
| Kradzież ($950 - $5000) | 3000$ | **175$** | 36 |
| Kradzież wielka (powyżej $5000) | 10000$ | **600$** | 48 |
| Kradzież z pojazdu | 2000$ | **160$** | 12 |
| Kradzież tożsamości | 10000$ | **600$** | 36 |
| Rozbój | 10000$ | **600$** | 60 |
| Rozbój z użyciem broni palnej | 25000$ | **2500$** | 108 |
| Rozbój z użyciem broni białej | 15000$ | **1500$** | 72 |
| Rozbój na instytucji finansowej | 50000$ | **5000$** | 156 |
| Włamanie do budynku mieszkalnego | 10000$ | **600$** | 72 |
| Włamanie do obiektu komercyjnego | 5000$ | **300$** | 36 |
| Włamanie z użyciem narzędzi | 10000$ | **600$** | 72 |
| Włamanie do pojazdu | 2000$ | **160$** | 12 |
| Kradzież pojazdu (GTA) | 10000$ | **600$** | 36 |
| Kradzież pojazdu uprzywilejowanego | 20000$ | **2000$** | 60 |
| Nielegalne posiadanie skradzionego pojazdu | 5000$ | **300$** | 12 |
| Wymuszenie / szantaż | 10000$ | **600$** | 48 |
| Wymuszenie z użyciem broni | 25000$ | **2500$** | 96 |
| Wymuszenie przez zorganizowaną grupę | 50000$ | **5000$** | 120 |
| Oszustwo (do $5000) | 3000$ | **175$** | 12 |
| Oszustwo (powyżej $5000) | 10000$ | **600$** | 36 |
| Oszustwo na szkodę instytucji publicznej | 25000$ | **2500$** | 60 |
| Fałszerstwo dokumentów | 5000$ | **300$** | 36 |
| Fałszerstwo pieniędzy / waluty | 25000$ | **2500$** | 72 |
| Wandalizm (szkoda poniżej $400) | 1000$ | **80$** | 12 |
| Wandalizm (szkoda powyżej $400) | 5000$ | **300$** | 36 |
| Zniszczenie mienia publicznego | 5000$ | **300$** | 36 |
| Zniszczenie pojazdu służbowego | 10000$ | **600$** | 48 |
| Pranie pieniędzy (do $50 000) | 20000$ | **2000$** | 48 |
| Pranie pieniędzy (powyżej $50 000) | 50000$ | **5000$** | 96 |
| Pranie pieniędzy przez zorganizowaną grupę | 100000$ | **4000$** | 120 |
#### Przestępstwa przeciwko porządkowi publicznemu

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Zakłócenie porządku publicznego | 500$ | **50$** | 3 |
| Wywołanie zbiegowiska / zamieszek | 2000$ | **160$** | 12 |
| Napaść słowna / groźby publiczne | 1000$ | **80$** | 6 |
| Groźba karalna | 1000$ | **80$** | 12 |
| Groźba z użyciem broni | 5000$ | **300$** | 36 |
| Groźba wobec funkcjonariusza publicznego | 5000$ | **300$** | 36 |
| Stalking / nękanie | 2000$ | **160$** | 48 |
| Użycie broni palnej podczas przestępstwa (+do kary) | 10000$ | **600$** | 120 |
| Oddanie strzału podczas przestępstwa (+do kary) | 20000$ | **2000$** | 180 |
| Napaść na funkcjonariusza (bez obrażeń) | 5000$ | **300$** | 48 |
| Pobicie funkcjonariusza (z obrażeniami) | 10000$ | **600$** | 84 |
| Pobicie funkcjonariusza z bronią | 25000$ | **2500$** | 144 |
| Usiłowanie zabójstwa funkcjonariusza | 50000$ | **5000$** | 300 |
| Ucieczka piesza przed funkcjonariuszem | 500$ | **50$** | 6 |
| Ucieczka pojazdem (niebezpieczna jazda) | 5000$ | **300$** | 36 |
| Ucieczka pojazdem z obrażeniami u innych | 15000$ | **1500$** | 84 |
| Ucieczka pojazdem ze śmiercią ofiary | 25000$ | **2500$** | 120 |
| Opór bierny przy zatrzymaniu | 500$ | **50$** | 3 |
| Czynny opór przy zatrzymaniu | 2000$ | **160$** | 12 |
| Atak na funkcjonariusza podczas zatrzymania | 10000$ | **600$** | 60 |
| Groźby terrorystyczne | 50000$ | **5000$** | 120 |
| Akt terrorystyczny bez ofiar | 100000$ | **4000$** | 240 |
| Udział w zorganizowanej grupie przestępczej | 25000$ | **2500$** | 60 |
| Kierowanie grupą przestępczą | 50000$ | **5000$** | 120 |
| Finansowanie działalności przestępczej | 100000$ | **4000$** | 144 |
#### Broń palna

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Posiadanie broni bez licencji (klasa I) | 5000$ | **300$** | 12 |
| Posiadanie broni klasy II przez cywila | 25000$ | **2500$** | 96 |
| Posiadanie broni przez osobę z zakazem | 25000$ | **2500$** | 60 |
| Posiadanie niezarejestrowanej broni | 10000$ | **600$** | 36 |
| Posiadanie broni klasy III bez zezwolenia | 50000$ | **5000$** | 96 |
| Posiadanie broni klasy IV (materiały wybuchowe) | 50000$ | **5000$** | 120 |
| Posiadanie ghost gun (bez numeru seryjnego) | 25000$ | **2500$** | 72 |
| Posiadanie tłumika bez zezwolenia | 20000$ | **2000$** | 48 |
| Posiadanie magazynka powyżej 10 nabojów | 5000$ | **300$** | 12 |
| Open carry bez uprawnień służbowych | 3000$ | **175$** | 12 |
| Concealed carry bez licencji CCW | 5000$ | **300$** | 36 |
| Noszenie broni załadowanej w pojeździe (bez CCW) | 2500$ | **150$** | 12 |
| Niezgłoszenie broni podczas kontroli drogowej | 1500$ | **120$** | 6 |
| Sprzedaż broni bez licencji FFL | 50000$ | **5000$** | 84 |
| Sprzedaż broni bez weryfikacji nabywcy | 25000$ | **2500$** | 60 |
| Sprzedaż broni osobie nieuprawnionej | 50000$ | **5000$** | 96 |
| Pośrednictwo w nielegalnym obrocie bronią | 50000$ | **5000$** | 120 |
| Niezgłoszenie kradzieży broni w terminie 48h | 3000$ | **175$** | 6 |
| Wejście do Gun Free Zone z bronią (nieumyślne) | 2500$ | **150$** | 6 |
| Wejście do Gun Free Zone z bronią (umyślne) | 10000$ | **600$** | 36 |
| Wniesienie broni do sądu lub więzienia | 25000$ | **2500$** | 60 |
| Bezpodstawne użycie broni palnej | 5000$ | **300$** | 60 |
#### Narkotyki

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Posiadanie marihuany przez niepełnoletniego | 500$ | **50$** | 0 |
| Posiadanie marihuany (28,5g - 500g) | 2000$ | **160$** | 6 |
| Posiadanie marihuany (powyżej 500g) | 10000$ | **600$** | 36 |
| Posiadanie marihuany z zamiarem dystrybucji | 25000$ | **2500$** | 48 |
| Sprzedaż marihuany bez licencji dispensary | 50000$ | **5000$** | 84 |
| Sprzedaż marihuany osobie niepełnoletniej | 50000$ | **5000$** | 120 |
| Używanie marihuany w miejscu publicznym / pojeździe | 250$ | **30$** | 0 |
| Uprawa powyżej 6 roślin marihuany | 5000$ | **300$** | 6 |
| Posiadanie Schedule I/II (mała ilość) | 10000$ | **600$** | 36 |
| Posiadanie Schedule I/II (duża ilość) | 25000$ | **2500$** | 60 |
| Posiadanie Schedule I/II z zamiarem dystrybucji | 50000$ | **5000$** | 84 |
| Posiadanie Schedule I/II w pobliżu szkoły (+300m) | 35000$ | **3500$** | 96 |
| Posiadanie Schedule III bez recepty | 5000$ | **300$** | 36 |
| Posiadanie Schedule IV bez recepty | 2000$ | **160$** | 12 |
| Posiadanie Schedule III z zamiarem dystrybucji | 25000$ | **2500$** | 72 |
| Wytwarzanie narkotyków (mała skala) | 50000$ | **5000$** | 84 |
| Wytwarzanie narkotyków (duża skala) | 100000$ | **4000$** | 144 |
| Laboratorium narkotykowe (meth lab / crack lab) | 100000$ | **4000$** | 180 |
| Wytwarzanie fentanylu / syntetyków opioidowych | 100000$ | **4000$** | 240 |
| Dystrybucja Schedule I/II (mała skala) | 50000$ | **5000$** | 84 |
| Dystrybucja Schedule I/II (duża skala) | 150000$ | **6000$** | 180 |
| Przemyt narkotyków przez granicę stanu | 100000$ | **4000$** | 240 |
| Kierowanie siatką dystrybucji narkotyków | 200000$ | **8000$** | 300 |
| Sprzedaż narkotyków niepełnoletniemu | 50000$ | **5000$** | 360 |
| Dystrybucja w pobliżu szkoły (+300m) | 100000$ | **4000$** | 156 |
| Posiadanie parafernaliów do Schedule I/II | 1500$ | **120$** | 6 |
| Sprzedaż parafernaliów narkotykowych | 5000$ | **300$** | 12 |
| Sprzedaż parafernaliów niepełnoletniemu | 25000$ | **2500$** | 36 |
#### Przestępstwa przeciwko wymiarowi sprawiedliwości

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Krzywoprzysięstwo | 5000$ | **300$** | 48 |
| Fałszywe zeznania w sprawie o zabójstwo | 25000$ | **2500$** | 120 |
| Utrudnianie postępowania karnego | 5000$ | **300$** | 36 |
| Niszczenie lub ukrywanie dowodów | 10000$ | **600$** | 60 |
| Zastraszanie świadków / ławy przysięgłych | 25000$ | **2500$** | 72 |
| Przekupstwo świadka | 25000$ | **2500$** | 72 |
| Fałszywe zgłoszenie przestępstwa | 2000$ | **160$** | 12 |
| Wręczenie łapówki funkcjonariuszowi | 25000$ | **2500$** | 48 |
| Przyjęcie łapówki przez funkcjonariusza | 50000$ | **5000$** | 96 |
| Przekupstwo sędziego lub prokuratora | 100000$ | **4000$** | 120 |
| Ucieczka z aresztu | 5000$ | **300$** | 36 |
| Ucieczka z zakładu karnego | 10000$ | **600$** | 60 |
| Pomoc w ucieczce z aresztu / więzienia | 10000$ | **600$** | 60 |
| Pomoc w ucieczce z konwoju | 10000$ | **600$** | 120 |
| Podszywanie się pod funkcjonariusza publicznego | 5000$ | **300$** | 36 |
| Podszywanie się z użyciem fałszywego umundurowania | 10000$ | **600$** | 48 |
| Składanie fałszywych zeznań | 5000$ | **300$** | 36 |
| Współudział w przestępstwie | 5000$ | **300$** | 60 |
| Fałszywe wezwanie służb | 2000$ | **160$** | 12 |
#### Bezpieczeństwo publiczne

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Nielegalne posiadanie materiałów wybuchowych | 25000$ | **2500$** | 72 |
| Wytwarzanie materiałów wybuchowych | 50000$ | **5000$** | 120 |
| Użycie materiałów wybuchowych bez ofiar | 100000$ | **4000$** | 180 |
| Podpalenie mienia (bez ofiar) | 10000$ | **600$** | 72 |
| Podpalenie budynku mieszkalnego | 25000$ | **2500$** | 96 |
| Podpalenie z ofiarami w środku | 50000$ | **5000$** | 144 |
| Podpalenie budynku publicznego | 50000$ | **5000$** | 120 |
#### Przestępstwa przeciwko państwu

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Ujawnienie tajemnicy służbowej | 25000$ | **2500$** | 60 |
| Nadużycie uprawnień służbowych | 10000$ | **600$** | 48 |
| Bezprawne pozbawienie wolności przez funkcjonariusza | 15000$ | **1500$** | 60 |
| Fałszowanie dokumentów służbowych | 10000$ | **600$** | 48 |
| Ujawnienie tajemnicy śledczej | 25000$ | **2500$** | 72 |
#### Wykroczenia drogowe

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Jazda bez prawa jazdy | 1000$ | **80$** | — |
| Jazda z zawieszonym prawem jazdy | 2500$ | **150$** | — |
| Jazda z cofniętym prawem jazdy | 5000$ | **300$** | 6 |
| Przekroczenie prędkości o 1-15 mph | 350$ | **35$** | — |
| Przekroczenie prędkości o 16-25 mph | 700$ | **60$** | — |
| Przekroczenie prędkości o 26-40 mph | 1500$ | **120$** | — |
| Przekroczenie prędkości o ponad 40 mph | 3000$ | **175$** | — |
| Przekroczenie prędkości w strefie szkolnej | 1400$ | **110$** | — |
| Przejazd na czerwonym świetle | 1000$ | **80$** | — |
| Niezatrzymanie się przed linią zatrzymania | 500$ | **50$** | — |
| Niezatrzymanie się przed przejściem dla pieszych | 1000$ | **80$** | — |
| Nieustąpienie pierwszeństwa pieszemu | 1500$ | **120$** | — |
| Nieustąpienie pierwszeństwa pojazd. uprzywilejowanemu | 1000$ | **80$** | — |
| Niezastosowanie się do znaku STOP | 700$ | **60$** | — |
| Niezastosowanie się do polecenia funkcjonariusza | 1500$ | **120$** | — |
| Jazda pod prąd | 700$ | **60$** | — |
| Brak zapiętych pasów bezpieczeństwa | 300$ | **30$** | — |
| Używanie telefonu podczas jazdy | 500$ | **50$** | — |
| Brawurowa / niebezpieczna jazda | 2000$ | **160$** | 12 |
| Niebezpieczna jazda z obrażeniami u innych | 10000$ | **600$** | 36 |
| Niedozwolone wyprzedzanie | 1000$ | **80$** | — |
| Jazda niepoprawnym pasem ruchu | 400$ | **40$** | — |
| DUI - BAC 0.08-0.14% (pierwsze) | 5000$ | **300$** | 6 |
| DUI - BAC 0.15%+ (pierwsze) | 10000$ | **600$** | 12 |
| DUI - drugie naruszenie | 10000$ | **600$** | 24 |
| DUI - z wypadkiem bez obrażeń | 10000$ | **600$** | 12 |
| DUI - z obrażeniami u innych | 15000$ | **1500$** | 36 |
| Odmowa badania alkotestem | 1500$ | **120$** | 12 |
| Spowodowanie kolizji | 500$ | **50$** | — |
| Spowodowanie wypadku | 1500$ | **120$** | — |
| Potrącenie pieszego | 2000$ | **160$** | — |
| Ucieczka z miejsca wypadku - bez ofiar (hit and run) | 5000$ | **300$** | 12 |
| Jazda pojazdem niezdatnym do ruchu | 2000$ | **160$** | — |
| Niesprawne oświetlenie | 350$ | **35$** | — |
| Niedozwolony kolor świateł (niebieski/czerwony) | 1500$ | **120$** | — |
| Niedozwolone przyciemnienie przedniej szyby | 700$ | **60$** | — |
| Niedozwolone modyfikacje pojazdu | 2000$ | **160$** | — |
| Nadmierny hałas układu wydechowego | 500$ | **50$** | — |
| Jazda pojazdem niezarejestrowanym | 1500$ | **120$** | — |
| Nieczytelne tablice rejestracyjne | 350$ | **35$** | — |
| Parkowanie przy czerwonym krawężniku | 500$ | **50$** | — |
| Parkowanie na przejściu dla pieszych | 700$ | **60$** | — |
| Parkowanie na miejscu dla niepełnosprawnych | 1500$ | **120$** | — |
| Brak dokumentów pojazdu podczas kontroli | 300$ | **30$** | — |
| Niezgłoszenie broni podczas kontroli drogowej | 1500$ | **120$** | 6 |
| Jazda bez kasku (motocykl / quad) | 500$ | **50$** | — |
| Niedopuszczalny lane splitting | 700$ | **60$** | — |
| Udział w nielegalnym wyścigu | 5000$ | **300$** | 12 |
| Niezatrzymanie pojazdu do kontroli | 1500$ | **120$** | — |
| Ucieczka pojazdem przed służbami | 5000$ | **300$** | 36 |
#### Naruszenia gospodarcze

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Prowadzenie działalności bez rejestracji | 5000$ | **300$** | 6 |
| Prowadzenie działalności bez licencji branżowej | 15000$ | **1500$** | 12 |
| Podanie fałszywych danych przy rejestracji | 10000$ | **600$** | 36 |
| Prowadzenie działalności po cofnięciu licencji | 25000$ | **2500$** | 36 |
| Zaniżenie przychodów w deklaracji podatkowej | 25000$ | **2500$** | 36 |
| Ukrywanie przychodów / podwójna księgowość | 100000$ | **4000$** | 60 |
| Oszustwo gospodarcze na szkodę kontrahentów | 50000$ | **5000$** | 60 |
| Kartel / porozumienie antykonkurencyjne | 100000$ | **4000$** | 84 |
| Zatrudnianie bez umowy (praca na czarno) | 10000$ | **600$** | 6 |
#### Okoliczności łagodzące

| Wykroczenie | Obecnie | Docelowo | Więzienie (mies.) |
|---|---:|---:|---:|
| Ulga mandatowa -250$ | -250$ | **-25$** | — |
| Ulga mandatowa -500$ | -500$ | **-50$** | — |
| Ulga mandatowa -750$ | -750$ | **-75$** | — |
| Ulga mandatowa -1000$ | -1000$ | **-100$** | — |
| Ulga mandatowa -2500$ | -2500$ | **-250$** | — |
| Ulga mandatowa -5000$ | -5000$ | **-500$** | — |
| Ulga mandatowa -10000$ | -10000$ | **-1000$** | — |
| Ulga mandatowa -25000$ | -25000$ | **-2500$** | — |
| Ulga więzienna -1 miesiąc | 0$ | **0$** | -1 |
| Ulga więzienna -6 miesięcy | 0$ | **0$** | -6 |
| Ulga więzienna -12 miesięcy | 0$ | **0$** | -12 |
| Ulga więzienna -24 miesiące | 0$ | **0$** | -24 |
| Ulga więzienna -36 miesięcy | 0$ | **0$** | -36 |
| Ulga więzienna -60 miesięcy | 0$ | **0$** | -60 |
| Ulga więzienna -120 miesięcy | 0$ | **0$** | -120 |

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

Generator: [`_gen_mdt_fines_table.py`](_gen_mdt_fines_table.py) · [`_mdt_ems_fines_v2_table.md`](_mdt_ems_fines_v2_table.md)

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
| `BabiczTuningMenu_sv.lua` | `GetModPrice` (flaty) | nitro 75k, parachute 1.2M… | ×0,1–0,2 po `vehicles.json` | Planowany |
| `BabiczTuningMenu_sv.lua` | `ShopPricePercents` | performance **5%** | **1,5–2,5%** po deflacji aut | Planowany |
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
| `BabiczMechanic_sv.lua` | `towVehicle` | 0$ dla mechanika | opcjonalnie powiązać z MDT holowanie | Do ustalenia |

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

**Cooldown:** każdy typ napadu ma **osobny cooldown per gracz** — wymusza rotację między napadami (szczegóły tierów w `ekonomia-zmiany-v2.md` § Cooldowny napadów).

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_heists/Config.lua` | Cooldown per heist / per gracz | częściowo | Osobny CD na każdy typ napadu | Planowany |
| `[rage]/rage_heists/Config.lua` | `AttackerMultipliers` | Agresywne (bank ×2.5) | Łagodniejsze mnożniki v2 | Planowany |
| `[rage]/rage_heists/Heists/NPC/shared/BabiczNPC_config.lua` | Gotówka, loot | money 45–195$ | 25–150$ + biżuteria niska | Planowany |
| `[rage]/rage_heists/Heists/ATM/shared/BabiczATM_config.lua` | Karta / hack | 500–2500$ / 200–500$ | 100–350$ / 200–700$ | Planowany |
| `[rage]/rage_heists/Heists/ShopKeeperCashRegister/shared/BabiczShopKeeperCashRegister_config.lua` | Kasetka | 2500–5000$ | 250–600$ | Planowany |
| `[rage]/rage_heists/Heists/ShopVault/shared/BabiczShopVault_shared.lua` | Sejf | 15000–40000$ | 800–2200$ | Planowany |
| `[rage]/rage_heists/Heists/Bank/shared/BabiczBank_shared.lua` | Fleeca | 60000–110000$ | 4500–12000$ | Planowany |
| `[rage]/rage_heists/Heists/BankPacific/shared/BabiczBankPacific_config.lua` | Pacific min group, loot | minGroup 1; loot w server flow | min 4 os.; 35000–90000$ | Planowany |

---

## 14. Napady — planowane (implementacja)

| System | Plik | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| Boosting | (nowy / heists) | — | 350–900$, solo | Planowany |
| Tracker | `[rage]/BabiczTracker/Config.lua` | neededMoney 500$; rewards 500–5000$ | start 200–400$; nagroda 500–1400$ | Planowany |
| Jubiler | (nowy) | — | 2800–6500$ wartości skupu, 2–5 os. | Planowany |
| Lombard (napad) | (nowy) | — | 600–1800$ + biżuteria, 1–3 os. | Planowany |
| Jacht | (nowy) | — | 10000–25000$, 3–5 os. | Planowany |
| Humane Labs | (nowy) | — | 22000–55000$, 3–5 os. | Planowany |
| Cayo Perico | (nowy) | — | 70000–130000$, 5–8 os. | Planowany |

---

## 15. Lombard (skup) — pełna tabela

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Items` | Wszystkie 33 itemy | diamond 155–160; phone 750–1250; gold_chain 800–1250 | Tabela v2 (15–160$) — patrz `ekonomia-zmiany-v2.md` § Biżuteria i lombard | Planowany |
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Black` | multiplier | 0.95 | 0.95 (bez zmian) | Planowany |
| `[rage]/RageCity/server/platetape.lua` / lombard flow | Czas sprzedaży (`time`) | 900–1280 ms | opcjonalnie skrócić po deflacji | Do ustalenia |

---

## 16. Narkotyki i sprzedaż

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Drugs` | Ceny sprzedaży 17 itemów | 75–1050$ | Tabela v2 (25–220$) | Planowany |
| `[rage]/RageCity/Config.lua` → `Config.DrugSelling` | copsRequired, callCopsChance, Zones | 2 / 65% / 1 strefa | bez zmian parametrów | Planowany |
| `[rage]/RageCity/server/drugsales.lua` | Logika sprzedaży | quality multiplier | Bez zmian struktury | Planowany |
| `[rage]/rage_drugs/Config.lua` | Zbiory / craft | Yields, czasy | Ewentualna korekta yields po testach | Do ustalenia |
| `[tebex]/kq_weed/config.extras.lua` | Produkcja KQ weed | yields | produkcja bez zmian; ceny w Config.Drugs | Planowany |

---

## 17. Pralnia

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `MoneyWash.Automatic` | Strata, prędkość | 30% strata; 175$/s | 35–45% strata; 25–75$/s | Planowany |
| `[rage]/RageCity/server/moneywash.lua` | Trasa ręczna | 3000–7500$/stop; 5% strata | 3000–7500$/stop; 20–30% strata | Planowany |

---

## 18. Zadania Fernando (Moris)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Tasks/Moris/shared/1_Fernando.lua` | `moneyAmount` (zad. 2) | 25000$ dirty | 4000–6000$ | Planowany |
| `[rage]/RageCity/Tasks/Moris/server/1_Fernando.lua` | Wypłaty zad. 1–4 | 250–12000$ mixed | 80–2500$ wg tabeli v2 | Planowany |
| `[rage]/RageCity/Tasks/Moris/client/1_Fernando.lua` | Teksty UI (kwoty) | hardcoded 25000 | zsynchronizować z shared | Planowany |

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

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_vehicleshop/vehicles.json` | Cena per model | Setki tys. – 20M+ | Segmenty v2 (auto 1k–1.2M itd.) | **Poza scope dokumentu** — właściciel serwera |
| `[rage]/rage_vehicleshop/BabiczVehicleShop_cl.lua` | `VehiclePriceConfig.types` | min/max obecne | min/max v2 | Planowany |
| `[rage]/rage_vehicleshop/Config.lua` | `Config.Prices.customPlate` | 750000$ | 15000–40000$ | Planowany |
| `[rage]/rage_vehicleshop/Config.lua` | `testDrivePrice` | 80$ | 50–100$ | Planowany |

---

## 22. Usługi i garaże

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_garages/Config.lua` | Tow, impound | 800$ / 5000$ | 300–700$ / 1500–3000$ | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Naprawa przy wyciągnięciu | 1500$ hardcoded | 500–1500$ | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Myjnia `dirtLevel * 12` | ×12$ | ×4–8$ | Planowany |
| `[rage]/rage_garages/Config.lua` | Współwłaściciel (jeśli jest) | 5000$ | 1500–3000$ | Do ustalenia |

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

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/qs-housing/config/main.lua` | `CreditEq`, `CreditTime` | 30% co 5 min | 5–8% co 30–60 min | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `RentTime`, scheduler | co 5 min / monthly | **0,6–1,6% wartości co 2 tygodnie** | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `BankFee`, `BrokerFee`, `Taxes` | 10% / 5% / 5% | bez zmian | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `Config.Bills` | woda 30–150; internet 80–300; prąd 50$/kWh | 15–40 / 25–80 / 12–20$/kWh | Planowany |
| `[tebex]/qs-housing/config/main.lua` | Upgrade'y (alarm, kamery…) | 10000–60000$ | 2500–22000$ (tabela v2) | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `MetaKeyCreatePrice` | 500$ | 150–300$ | Planowany |
| `[tebex]/qs-housing/config/furniture.lua` | Ceny mebli IKEA | 61–2500$+ | 30–800$ | Planowany |
| Baza danych / `Config.Houses` | Ceny nieruchomości | DB-driven, wysokie | Segmenty 15k–800k (tabela v2) | **Poza scope dokumentu** — właściciel serwera |

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
| `[tebex]/kq_weed/config.strains.lua` | yields / czasy | 6–8 bud; 20 min | monitorować; nawóz z dark shop | Po testach |
| `[rage]/rage_market/Config.lua` | Klucz Rico (DarkShop) | 50 000$ | 3 000–6 000$ | Planowany |

### Garaże P2P

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_garages/Config.lua` | `SellVehiclePercent` | 65% | 55–65% | Planowany |
| `[rage]/rage_garages/Config.lua` | `ManageCoOwnerPrice` | 5000$ | 1500–3000$ | Planowany |
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
| `[core]/es_extended/config.lua` | `StartingAccountMoney` | bank 2500$ | 1500–2500$ | Planowany |
| `[rage]/rage_multicharacter/Config.lua` | `OnStart` → `money` item | 5000$ | 1500–3000$ (łącznie z bankiem) | Planowany |
| SQL `job_grades.salary` | Grade salaries | różne | 0 lub wyłączone | Do ustalenia |
| `[rage]/rage_bossmenu/BabiczBossMenu_sv.lua` | `sendSalary` | bez capu | tier + max 10k/tydz. | Planowany |

### Audyt — pozostałe źródła (bez rescale lub poza scope)

| Plik / resource | Co | Docelowo | Status |
|---|---|---|---|
| `[rage]/rage_heists/Heists/BankPacific/server/` | Wypłata lootu | zgodnie z kartą v2 (35–90k) — **brak w kodzie** | Do implementacji |
| `[rage]/BabiczHospitality/` | EMS random ped 100–500$ | scalić z widełkami kursu EMS lub wyłączyć | Do ustalenia |
| `[tebex]/rtx_bowling/` | Wygrane kręgli | EV ujemne; bez zmian grindu | Monitorować |
| `[rage]/rage_znajdzki/`, `BabiczPumpkins/` | Eventy sezonowe | poza stałą ekonomią | Poza scope |
| `[tebex]/qs-housing/` | Czynsz → właściciel | rescale interwału/kwoty (§ housing v2) | Planowany |

### Napad truckera

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_heists/Config.lua` | Heist #9 trucker | 20–50k (komentarz) | 8–18k lub wyłączyć | Poza progresją v2 |

---

## 27. Kolejność wdrożenia (zalecana)

1. Paycheck (`paycheck.lua`)  
2. KQ Deliveries + Powerwashing  
3. rage_jobs (+ hunter, kaucje)  
4. MDT mandaty (pełna tabela — §7) + EMS usługi + FIB  
5. Napady aktywne + cooldown per gracz + mnożniki  
6. Pralnia + lombard (33 itemy) + narkotyki + DrugSelling  
7. rage_market + zdrapki + ox_fuel + wygląd + siłownia  
8. Usługi + garaże (w tym myjnia)  
9. Firmy (menu + składniki + kursy 80–150$ + depositProducts 100–110%)  
10. Mechanik (paycheck + NPC + tuning flat/% + MDT usługi) — ceny modów po `vehicles.json`  
10b. Kurs lawetą mechanika (nowa implementacja)  
11. Premie bossmenu (implementacja tierów)  
12. Fernando + Tracker + napady planowane  
13. qs-housing (config; hipoteka + czynsz co 2 tyg.)  
14. Job center UI  
15. Kasyno + Dark Shop + Ammunation (oba) + lb-phone  
16. Beach vendor + kluby + automaty + GymShop  
17. Blazing Tattoo / Five Records  
18. Pojazdy (`vehicles.json` + DB housing) — **właściciel serwera**  
19. Plan testów (§ v2)  

---

## 28. Ryzyka wdrożeniowe

| Ryzyko | Mitygacja |
|---|---|
| Wdrożenie częściowe | Wszystkie zmiany w jednym deployu lub feature flag |
| KQ bez zmian | Priorytet #2 — niszczy kotwicę 500–600$/h |
| UI job center ≠ real pay | Zsynchronizować w tym samym deployu |
| Mandaty tylko w Config.lua, nie w UI MDT | Sprawdzić oba źródła |
| vehicles.json (~10k linii) | Konwersja per model — **właściciel serwera** (segmenty w v2) |
| qs-housing DB | Ceny per lokalizacja — **właściciel serwera** (widełki segmentów w v2) |
| Zdrapki bez rescale | Obecne mnożniki → wygrane w milionach; wdrożyć razem z market |
| Kasyno bez rescale | MaxWager 500k niszczy deflację |
| Dark Shop / broń | Ceny 100k+ poza skalą v2 |
| KQ weed yields | Produkcja może > crime bez testów |
| Premie bez capu | Boss może wypłacić całą kasę society |
| P2P auta | Podatek 2% DOJ + webhook; wolny rynek nadal może przenosić bogactwo — monitorować |
| Fernando teksty vs kwoty | shared + server + client w jednym commicie |
| Myjnia hardcoded w BabiczGarages_sv | zmiana mnożnika 12 → 4–8 |

---

*Ostatnia aktualizacja: pełny zakres ekonomii serwera w `ekonomia-zmiany-v2.md` + generatory MDT (`_gen_mdt_fines_table.py`). Wdrożenie w kodzie — po zatwierdzeniu przez zespół.*
