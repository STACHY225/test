# RageCity — lista zmian ekonomii (wdrożenie)

Checklista zmian per skrypt. **Dokument planistyczny — bez wdrożenia w kodzie.**  
Odniesienie docelowe: [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md).

Legenda statusu: `Planowany` | `Do ustalenia` | `Gotowe do wdrożenia`

---

## 1. Paycheck i zasiłek

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[core]/es_extended/server/paycheck.lua` | Stawki, warunek on duty, rozróżnienie jobów | 125$/15 min wszyscy; 250$/15 min frakcje/firmy; bez on duty | Zasiłek 15$/15 min; frakcje 20–30$/15 min wg jobu; tylko on duty | Planowany |
| `[core]/es_extended/config.lua` | `PaycheckInterval`, `StartingAccountMoney` | 15 min; bank 2500$ | Bez zmian interwału; bank 2500$ OK | Planowany |
| SQL `job_grades.salary` | Grade salaries (jeśli reaktywowane) | Różne w DB | Zgodnie z nowym modelem lub wyłączone | Do ustalenia |

---

## 2. Job center (UI)

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobcenter/Config.lua` | Wyświetlane `salary` per job | 6000–26000$ (marketing) | Zgodne z realnymi widełkami v2 (np. side job ~500$/h, frakcje ~800–2000$/h aktywna sesja) | Planowany |

---

## 3. Prace dorywcze — rage_jobs

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_jobs/Jobs/miner.lua` | Ceny sprzedaży rud | copper 16–18, diamond 125–130 | Przeskalować do ~500–700$/h łącznie z zasiłkiem | Planowany |
| `[rage]/rage_jobs/Jobs/lumberjack.lua` | Cena sprzedaży | 530–550$/plank | ~500–650$/h | Planowany |
| `[rage]/rage_jobs/Jobs/tailor.lua` | Cena sprzedaży | 50–60$/szt. | ~450–550$/h | Planowany |
| `[rage]/rage_jobs/Jobs/slaughterer.lua` | Cena sprzedaży | 95–105$/szt. | ~450–600$/h | Planowany |
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

## 5. Frakcje — MDT i mandaty

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_mdt/Config.lua` | `Config.Fines` — katalog mandatów | 200–50000$+ | Widełki v2: drobne 30–80$, ciężkie do 3000$ | Planowany |
| `[rage]/rage_mdt/Config.lua` | Faktury EMS (medical invoices) | 300–10000$ | Revive 200–500$, leczenie 150–400$ itd. | Planowany |
| `[rage]/rage_mdt/Config.lua` | `finePercentForPlayer`, `fineCompanyMultiplier` | 25–40%, mnożnik 1.5–2.5 | Bez zmian struktury; kwoty bazowe niższe | Planowany |
| `[rage]/rage_mdt/html/src/config/data.ts` | UI mandatów (jeśli duplikuje kwoty) | Wysokie | Zgodnie z Config.lua | Planowany |

---

## 6. Frakcje — EMS

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | Kursy lokalne `rewardMoneyMin/Max` | 350–850$ | 200–500$ (zgodnie z usługami v2) | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | LocalMedic pricing | 50–10000$ dynamiczny | 500–1500$ (usługi v2) | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | Respawn fines, ubezpieczenie | 500/5000; 3500–12000$ | Przeskalować do nowej ekonomii | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/server/BabiczDeath_sv.lua` | Revive split (135% firma, 10% DOJ) | Procenty | Bez zmian struktury | Planowany |

---

## 7. Frakcje — DOJ i podatki

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/company_functions/server/BabiczCompanyShop_sv.lua` | Split sprzedaży | 90% firma / 10% DOJ | Bez zmian (10% na society DOJ) | Planowany |
| `[rage]/rage_mdt/Config.lua` | Split mandatów → DOJ | 10% części society | Bez zmian | Planowany |
| `[rage]/rage_bossmenu/BabiczBossMenu_sv.lua` | Premie tygodniowe — tier system | Ręczne wypłaty bossa | Implementacja tierów v2 (500–10000$, max 10k/os.) | Planowany |

---

## 8. Firmy prywatne

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_fractions/Fractions/c_burgershot/shared/BabiczBurgerShot_config.lua` | Ceny menu | 31–56$ | Spójne z kosztami życia v2 | Planowany |
| `[rage]/rage_fractions/Fractions/c_tequilala/shared/BabiczTequilala_config.lua` | Ceny menu | 34–56$ | j.w. | Planowany |
| `[rage]/rage_fractions/Fractions/company_functions/server/BabiczCompanyCourses_sv.lua` | Kursy firmowe → 50% do firmy | 50% | Bez zmian struktury | Planowany |

---

## 9. Napady — aktywne

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

## 10. Napady — planowane (implementacja)

| System | Opis | Docelowe wartości | Status |
|---|---|---|---|
| Boosting | Znajdź auto, dostarcz | 350–900$, solo | Planowany |
| Tracker | Ucieczka z nadajnikiem | 500–1400$, 1–2 os. | Planowany — istnieje `[rage]/BabiczTracker/Config.lua` (500–5000$) |
| Jubiler | Biżuteria → lombard | 1200–4500$ wartości skupu, 2–5 os. | Planowany |
| Lombard (napad) | Gotówka + biżuteria | 600–1800$ + biżuteria, 1–3 os. | Planowany |
| Jacht | Grupowy napad | 10000–25000$, 3–5 os. | Planowany |
| Humane Labs | Grupowy napad | 22000–55000$, 3–5 os. | Planowany |
| Cayo Perico | Endgame | 70000–130000$, 5–8 os. | Planowany |

---

## 11. Lombard (skup)

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Items` | Ceny skupu wszystkich itemów | diamond 155–160; phone 750–1250 | Tabela biżuterii v2 (15–160$) | Planowany |
| `[rage]/RageCity/Config.lua` → `Config.Pawnshop.Black` | multiplier | 0.95 | 0.95 (bez zmian) | Planowany |

---

## 12. Narkotyki

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `Config.Drugs` | Ceny sprzedaży 17 itemów | 75–1050$ | Tabela v2 (25–220$) | Planowany |
| `[rage]/RageCity/server/drugsales.lua` | Logika sprzedaży | quality multiplier | Bez zmian struktury | Planowany |
| `[rage]/rage_drugs/Config.lua` | Zbiory / craft | Yields, czasy | Ewentualna korekta yields po testach | Do ustalenia |

---

## 13. Pralnia

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/RageCity/Config.lua` → `MoneyWash.Automatic` | Strata, prędkość | 30% strata; 175$/s | 35–45% strata; 25–75$/s | Planowany |
| `[rage]/RageCity/server/moneywash.lua` | Trasa ręczna | 3000–7500$/stop; 5% strata | 3000–7500$/stop; 20–30% strata | Planowany |

---

## 14. Sklepy i narzędzia

| Plik | Co zmienić | Obecnie (przykłady) | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_market/Config.lua` | Jedzenie, narzędzia, broń, dark shop | woda 20$; lockpick 1200$; fixkit 5000$; thermite 2000$ | Tabela kosztów życia + narzędzia v2 | Planowany |
| `[core]/ox_inventory/data/shops.lua` | Automaty, siłownia | 22–100$ | Spójne z v2 | Planowany |
| `[core]/ox_fuel/config.lua` | Paliwo, kanistry | priceTick 8; kanister 1500$ | Przeskalować proporcjonalnie | Do ustalenia |
| `[rage]/BabiczAmmunation/Config.lua` | Bronie legalne | 80000–280000$ | 8000–300000$ (tabela v2) | Planowany |

---

## 15. Pojazdy

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_vehicleshop/vehicles.json` | Cena per model | Setki tys. – 20M+ | Segmenty v2 (auto 1k–1.2M itd.) | Planowany |
| `[rage]/rage_vehicleshop/BabiczVehicleShop_cl.lua` | `VehiclePriceConfig.types` | min/max obecne | min/max v2 (types w dokumencie) | Planowany |
| `[rage]/rage_vehicleshop/Config.lua` | Tablica rejestracyjna, test drive | plate 750000$; test 80$ | Przeskalować (tablica — do ustalenia) | Do ustalenia |

---

## 16. Usługi

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[rage]/rage_garages/Config.lua` | Tow, impound | 800$ / 5000$ | 300–700$ / 1500–3000$ | Planowany |
| `[rage]/rage_garages/BabiczGarages_sv.lua` | Naprawa przy wyciągnięciu | 1500$ hardcoded | 500–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/mechanic/shared/BabiczMechanic_shared.lua` | Lokalny mechanik | 1000$ / 5000$ | 500–1500$ | Planowany |
| `[rage]/rage_fractions/Fractions/ambulance/shared/BabiczAmbulance_shared.lua` | Lokalny medyk | patrz EMS | 500–1500$ | Planowany |

---

## 17. Mieszkania

| Plik | Co zmienić | Obecnie | Docelowo | Status |
|---|---|---|---|---|
| `[tebex]/qs-housing/config/main.lua` | Raty, czynsze, opłaty | Niezsynchronizowane | Osobna iteracja po głównym wdrożeniu | Do ustalenia |
| Baza danych (ceny domów) | Ceny nieruchomości | DB-driven | Audyt + przeskalowanie | Do ustalenia |

---

## 18. Kolejność wdrożenia (zalecana)

1. Paycheck (`paycheck.lua`)  
2. KQ Deliveries + Powerwashing  
3. rage_jobs  
4. MDT mandaty + EMS usługi  
5. Napady aktywne + mnożniki  
6. Pralnia + lombard + narkotyki  
7. rage_market + usługi + garaże  
8. Pojazdy (`vehicles.json` + `VehiclePriceConfig`)  
9. Job center UI  
10. Premie bossmenu (implementacja tierów)  
11. Napady planowane  
12. qs-housing  

---

## 19. Ryzyka wdrożeniowe

| Ryzyko | Mitygacja |
|---|---|
| Wdrożenie częściowe | Wszystkie zmiany w jednym deployu lub feature flag |
| KQ bez zmian | Priorytet #2 — niszczy kotwicę 500$/h |
| UI job center ≠ real pay | Zsynchronizować w tym samym deployu |
| Mandaty tylko w Config.lua, nie w UI MDT | Sprawdzić oba źródła |
| vehicles.json (~10k linii) | Skrypt masowej konwersji / segment algorithm |

---

*Ostatnia aktualizacja: zgodnie z `ekonomia-zmiany-v2.md`. Wdrożenie w kodzie — po zatwierdzeniu przez zespół.*
