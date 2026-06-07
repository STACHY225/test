# RageCity — lista zmian ekonomii (wdrożenie)

Checklista zmian per skrypt. **Dokument planistyczny — bez wdrożenia w kodzie.**  
Odniesienie docelowe: [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md).

Legenda statusu: `Planowany` | `Do ustalenia` | `Gotowe do wdrożenia`

---

## 1. Paycheck i zasiłek

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | Stawki, warunek on duty, rozróżnienie jobów | 125$/15 min wszyscy; 250$/15 min frakcje/firmy (police, doj, ambulance, fib, c_*); bez on duty | Zasiłek 15$/15 min; LSPD/LSSD/EMS/FIB **30$/15 min** (120$/h); DOJ **35–38$/15 min**; tylko on duty | Planowany |
| `[core]/es_extended/server/paycheck.lua` lub nowy moduł | **Stypendium aktywnej służby** | brak | +70$/15 min (~280$/h) gdy on duty + aktywność (ruch / MDT / akcja służbowa); podłoga razem ~400$/h | Planowany |
| `[core]/es_extended/config.lua` | `PaycheckInterval`, `StartingAccountMoney` | 15 min; bank 2500$ | Bez zmian interwału; bank 2500$ OK | Planowany |
| SQL `job_grades.salary` | Grade salaries (jeśli reaktywowane) | Różne w DB | Zgodnie z nowym modelem lub wyłączone | Do ustalenia |

**Stypendium — logika wdrożenia:** co 15 min sprawdzenie: `on duty` + (przesunięcie >200 m w oknie **lub** mandat/revive/kurs **lub** strefa patrolu). Spełnione → wypłata stypendium; inaczej tylko paycheck 120$/h.

---

## 2. Job center (UI)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobcenter/Config.lua` | Wyświetlane `salary` per job | 6000–26000$ (marketing) | Zgodne z realnymi widełkami v2 (np. side job ~500$/h, frakcje ~800–2000$/h aktywna sesja) | Planowany |

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

## 6. Śmieciarz

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobs/Jobs/trashman.lua` | Pętla wypłaty | brak `reward` / sprzedaży | ~450–550$/h (do zaprojektowania) | Do implementacji |
| `[rage]/rage_jobs/Jobs/trashman.lua` | Kaucje | vehicleBail 1500; skinBail 500 | 400–600$ / 100–200$ | Planowany |
| `[rage]/rage_jobcenter/Config.lua` | Widoczność w UI | widoczny | ukryć do implementacji pętli | Do ustalenia |

---

## 7. Frakcje — MDT i mandaty

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_mdt/Config.lua` | `Config.Fines` — katalog mandatów (`policeFines`) | drobne 1000$+; ciężkie 25000–50000$ | Skala v2: drobne 30–150$; średnie 150–800$; ciężkie 800–3000$; max ~8000$ | Planowany |
| `[rage]/rage_mdt/Config.lua` | Faktury EMS (medical invoices) | 300–10000$ | Revive 200–500$, leczenie 150–400$ itd. | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer` | LSPD/FIB/EMS 25%; c_blackrepair/c_quiettech 40% | Bez zmian struktury; kwoty bazowe niższe | Planowany |
| `[rage]/rage_mdt/Config.lua` | `fineCompanyMultiplier` | 1.5–2.5× | Bez zmian | Planowany |
| `[rage]/rage_mdt/html/src/config/data.ts` | UI mandatów (jeśli duplikuje kwoty) | Wysokie | Zgodnie z Config.lua | Planowany |

### Mandaty — przykłady mapowania (policeFines)

| Wykroczenie | Obecnie | Docelowo |
|---|---:|---:|
| Napaść (Assault) | 1000$ | 80–120$ |
| Pobicie (Battery) | 2000$ | 120–200$ |
| Kradzież mała | 1000$ | 50–100$ |
| Kradzież średnia | 3000$ | 150–350$ |
| Rozbój | 10000$ | 400–900$ |
| GTA | 10000$ | 500–1000$ |
| Włamanie mieszkalne | 10000$ | 600–1200$ |
| Zabójstwo II stopnia | 25000$ | 3000–5000$ |

Pełna lista: wszystkie wpisy w `local policeFines` → przeskalować wg tabeli powyżej (×0.05–0.15 dla większości).

---

## 8. Frakcje — FIB

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | Grupa frakcji | fib → 250$/15 min | jak LSPD: 30$/15 min + stypendium (~400$/h podłoga) | Planowany |
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

---

## 10. Frakcje — DOJ i podatki

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/company_functions/server/BabiczCompanyShop_sv.lua` | Split sprzedaży | 90% firma / 10% DOJ | Bez zmian (10% na society DOJ) | Planowany |
| `[rage]/rage_mdt/Config.lua` | Split mandatów → DOJ | 10% części society | Bez zmian | Planowany |
| `[rage]/rage_bossmenu/BabiczBossMenu_sv.lua` | Premie tygodniowe — tier system | Ręczne wypłaty bossa | Implementacja tierów v2 (500–10000$, max 10k/os.) | Planowany |

---

## 11. Firmy prywatne

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/c_burgershot/shared/BabiczBurgerShot_config.lua` | Ceny menu | 31–56$ | Spójne z kosztami życia v2 | Planowany |
| `[rage]/rage_fractions/Fractions/c_tequilala/shared/BabiczTequilala_config.lua` | Ceny menu | 34–56$ | j.w. | Planowany |
| `[rage]/rage_fractions/Fractions/company_functions/server/BabiczCompanyCourses_sv.lua` | Kursy firmowe → 50% do firmy | 50% | Bez zmian struktury | Planowany |

---

## 12. Mechanik (frakcja)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/mechanic/shared/BabiczMechanic_shared.lua` | Lokalny mechanik NPC | 1000$ / 5000$ | 500–800$ / 1000–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/mechanic/server/BabiczTuningMenu_sv.lua` | Split tuning | firma 35% + DOJ 10% + zwrot 10% | Bez zmian struktury; ceny modów po vehicles.json | Planowany |
| `[rage]/rage_fractions/Fractions/mechanic/shared/BabiczMechanic_shared.lua` | Ceny modów (% ceny pojazdu) | % od obecnych cen | Przeliczyć po `vehicles.json` | Planowany |

---

## 13. Napady — aktywne

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_heists/Config.lua` | `AttackerMultipliers` | Agresywne (bank ×2.5) | Łagodniejsze mnożniki v2 | Planowany |
| `[rage]/rage_heists/Heists/NPC/shared/BabiczNPC_config.lua` | Gotówka, loot | money 45–195$ | 25–150$ + biżuteria niska | Planowany |
| `[rage]/rage_heists/Heists/ATM/shared/BabiczATM_config.lua` | Karta / hack | 500–2500$ / 200–500$ | 100–350$ / 200–700$ | Planowany |
| `[rage]/rage_heists/Heists/ShopKeeperCashRegister/shared/BabiczShopKeeperCashRegister_config.lua` | Kasetka | 2500–5000$ | 250–600$ | Planowany |
| `[rage]/rage_heists/Heists/ShopVault/shared/BabiczShopVault_shared.lua` | Sejf | 15000–40000$ | 800–2200$ | Planowany |
| `[rage]/rage_heists/Heists/Bank/shared/BabiczBank_shared.lua` | Fleeca | 60000–110000$ | 6000–16000$ | Planowany |
| `[rage]/rage_heists/Heists/BankPacific/shared/BabiczBankPacific_config.lua` | Pacific min group, loot | minGroup 1; loot w server flow | min 4 os.; 35000–90000$ | Planowany |

---

## 14. Napady — planowane (implementacja)

| System | Plik | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| Boosting | (nowy / heists) | — | 350–900$, solo | Planowany |
| Tracker | `[rage]/BabiczTracker/Config.lua` | neededMoney 500$; rewards 500–5000$ | start 200–400$; nagroda 500–1400$ | Planowany |
| Jubiler | (nowy) | — | 1200–4500$ wartości skupu, 2–5 os. | Planowany |
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

## 19. Sklepy, bronie i narzędzia

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_market/Config.lua` | Jedzenie, narzędzia, broń, dark shop | woda 20$; lockpick 1200$; fixkit 5000$; thermite 2000$ | Tabela kosztów życia + narzędzia v2 | Planowany |
| `[core]/ox_inventory/data/shops.lua` | Automaty, siłownia | 22–100$ | Spójne z v2 | Planowany |
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
| `[rage]/rage_vehicleshop/vehicles.json` | Cena per model | Setki tys. – 20M+ | Segmenty v2 (auto 1k–1.2M itd.) | Planowany |
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
| `[tebex]/qs-housing/config/main.lua` | `RentTime`, scheduler | co 5 min / monthly | 0,3–0,8% wartości / tydzień | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `BankFee`, `BrokerFee`, `Taxes` | 10% / 5% / 5% | bez zmian | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `Config.Bills` | woda 30–150; internet 80–300; prąd 50$/kWh | 15–40 / 25–80 / 12–20$/kWh | Planowany |
| `[tebex]/qs-housing/config/main.lua` | Upgrade'y (alarm, kamery…) | 10000–60000$ | 2500–22000$ (tabela v2) | Planowany |
| `[tebex]/qs-housing/config/main.lua` | `MetaKeyCreatePrice` | 500$ | 150–300$ | Planowany |
| `[tebex]/qs-housing/config/furniture.lua` | Ceny mebli IKEA | 61–2500$+ | 30–800$ | Planowany |
| Baza danych / `Config.Houses` | Ceny nieruchomości | DB-driven, wysokie | Segmenty 15k–800k (tabela v2) | Planowany |

---

## 26. Kolejność wdrożenia (zalecana)

1. Paycheck (`paycheck.lua`)  
2. KQ Deliveries + Powerwashing  
3. rage_jobs (+ hunter, kaucje)  
4. MDT mandaty (pełna tabela) + EMS usługi + FIB  
5. Napady aktywne + mnożniki  
6. Pralnia + lombard (33 itemy) + narkotyki + DrugSelling  
7. rage_market + ox_fuel + wygląd + siłownia  
8. Usługi + garaże (w tym myjnia)  
9. Pojazdy (`vehicles.json` + `VehiclePriceConfig` + tablica)  
10. Mechanik (lokalny + tuning)  
11. Premie bossmenu (implementacja tierów)  
12. Fernando + Tracker + napady planowane  
13. qs-housing (config + DB)  
14. Job center UI  
15. Śmieciarz (implementacja pętli lub ukrycie)  

---

## 27. Ryzyka wdrożeniowe

| Ryzyko | Mitygacja |
|---|---|
| Wdrożenie częściowe | Wszystkie zmiany w jednym deployu lub feature flag |
| KQ bez zmian | Priorytet #2 — niszczy kotwicę 500$/h |
| UI job center ≠ real pay | Zsynchronizować w tym samym deployu |
| Mandaty tylko w Config.lua, nie w UI MDT | Sprawdzić oba źródła |
| vehicles.json (~10k linii) | Skrypt masowej konwersji / segment algorithm |
| qs-housing DB | Backup + skrypt masowej konwersji cen |
| Fernando teksty vs kwoty | shared + server + client w jednym commicie |
| Myjnia hardcoded w BabiczGarages_sv | zmiana mnożnika 12 → 4–8 |

---

*Ostatnia aktualizacja: zgodnie z `ekonomia-zmiany-v2.md` (pełny zakres modułów). Wdrożenie w kodzie — po zatwierdzeniu przez zespół.*
