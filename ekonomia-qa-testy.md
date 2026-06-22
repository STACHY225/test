# RageCity — testy QA ekonomii v2

Checklista testów **po pełnym deployu** ekonomii v2. Uruchamiać na serwerze testowym ze **wszystkimi** zmianami z [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §0 — częściowy deploy unieważnia wyniki.

**Widełki i zasady balansu:** [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md)

---

## Metodologia

| Element | Wartość |
|---|---|
| Sesja standardowa | **60 min** aktywnej gry (chyba że test podaje inaczej) |
| Pomiar dochodu | `(stan kont na koniec) − (stan na start) − (koszty nietętnione)` |
| Kaucje jobów | zwrot kaucji = **0 netto** (nie wliczać do zarobku) |
| Crime | wartości **po praniu** (~25% straty), chyba że kolumna mówi „brutto dirty” |
| Warunek FAIL | wynik **> 120%** górnej granicy widełki v2 **w 2 kolejnych** powtórzeniach tego samego testu |
| Admin / giveaway | **wyłączone**; start testu od typowego loadoutu postaci v2 |

### Legenda kolumn w tabelach

| Kolumna | Znaczenie |
|---|---|
| **ID** | Skrót do raportu bugów |
| **Priorytet** | P0 = bloker otwarcia · P1 = wysoki · P2 = regresja |
| **Wynik** | `PASS` / `FAIL` / `SKIP` / puste |
| **$/h lub $** | zmierzona wartość testera |
| **Uwagi** | dowód (screenshot, log, timestamp) |

### Szablon raportu pojedynczego testu

```
ID: 
Data: 
Tester: 
Build / commit: 
Online PD / EMS: 
Wynik: PASS | FAIL
Zmierzone: 
Oczekiwane: 
Uwagi: 
```

---

## Kolejność przed otwarciem serwera

1. **K** — anti-exploit / deploy  
2. **B8, B9, A10, K5** — paycheck i AFK  
3. **A7, A8** — KQ  
4. **C2, C3** — firmy  
5. **D6, D7, I2, I4** — crime endgame  
6. **E3, E7** — narkotyki  
7. **G1–G4** — hazard  
8. Reszta — regresja balansu  

---

## A. Kotwica — prace dorywcze

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| A1 | P1 | Miner — typowa sesja | on duty, pełna pętla rud, sprzedaż, bez AFK | 60 min | **500–700$/h** (w tym zasiłek ~60) | >840$/h | | | |
| A2 | P1 | Lumberjack | jak A1 | 60 min | **500–650$/h** | >780$/h | | | |
| A3 | P2 | Tailor | jak A1 | 60 min | **450–550$/h** | >660$/h | | | |
| A4 | P2 | Slaughterer | jak A1 | 60 min | **450–600$/h** | >720$/h | | | |
| A5 | P1 | Hunter | polowanie + sprzedaż (ceny v2) | 60 min | **450–600$/h** | >720$/h | | | |
| A6 | P2 | Beach vendor | sprzedaż NPC + kursy | 60 min | **350–500$/h** | >600$/h | | | |
| A7 | P0 | KQ Deliveries | 1 job tier środkowy, solo, bez abuse team bonus | 60 min | **450–650$/h** | >780$/h | | | |
| A8 | P0 | KQ Powerwashing | 3–4 kontrakty w widełkach v2 | 60 min | **500–750$/h** | >900$/h | | | |
| A9 | P1 | KQ Trucker (legalny) | pełne trasy RS Haul | 60 min | **600–850$/h** | >1020$/h | | | |
| A10 | P0 | AFK side job | on duty, postać bez ruchu | 60 min | **≤60$/h** lub **0** (off duty) | >100$/h | | | |

---

## B. Paycheck i frakcje

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| B1 | P1 | LSPD spokojna | on duty, 0 mandatów, 0 dispatch | 60 min | **650–750$/h** | >900$/h | | | |
| B2 | P1 | LSPD aktywna | on duty, reagowanie na dispatch | 60 min | **780–980$/h** | >1200$/h | | | |
| B3 | P2 | LSPD mandaty | ~3 mandaty drobne/h, solo wystawiający | 60 min | minutówka + mandaty **≤30%** dochodu | mandaty >30% | | | |
| B4 | P1 | EMS spokojna | on duty, 0 akcji | 60 min | **460–500$/h** | >600$/h | | | |
| B5 | P1 | EMS aktywna | ~4 revive + 2 kursy NPC/h | 60 min | **1000–1500$/h** | >1800$/h | | | |
| B6 | P2 | DOJ spokojna | on duty, 0 spraw | 60 min | **580–700$/h** | >840$/h | | | |
| B7 | P1 | Mechanik LSC | on duty, 0 usług MDT | 60 min | **650–700$/h** minutówka | >840$/h | | | |
| B8 | P0 | AFK frakcja | on duty, bez ruchu (AFK) | 60 min | **0$/h** z duty | >50$/h | | | |
| B9 | P0 | Off duty frakcja | off duty całą godzinę | 60 min | **60$/h** zasiłek lub 0 | >100$/h | | | |
| B10 | P1 | Firma `c_*` spokojna | on duty, bez kursów | 60 min | **40–60$/h** | >80$/h | | | |
| B11 | P1 | Cap udziału FP | 1 duży wyrok (fine 5000$+) | 1× | udział wystawiającego **≤750$** | >750$ | | | |
| B12 | P0 | MDT ulgi | szukaj −5000 / −10000 / −25000 | 1× | pozycje **nie istnieją** w UI | ulga mandatowa >2500$ | | | |

---

## C. Firmy prywatne

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| C1 | P1 | Pracownik BS — kurs | 1 pełny kurs, 10 paczek | ~30 min | pracownik **800–1500$** + ~50% society | pracownik >1800$ | | | |
| C2 | P0 | CD kursu firmowego | 2 pełne kursy z rzędu | 45 min | **2. kurs zablokowany** do CD 30–45 min | 2 wypłaty bez CD | | | |
| C3 | P0 | `depositProducts` | craft 10× cheeseburger, deposit do sklepu | 1× | society **≈ menu × 0,85**/szt. (nie ×1,5) | society >110% wartości menu | | | |
| C4 | P2 | Sprzedaż klientowi | 10 sprzedaży z menu BS | ~30 min | 90% society, 10% DOJ; marża **8–12$/burger** | strata przy każdej sprzedaży | | | |
| C5 | P1 | Właściciel aktywny | 2 pracowników + właściciel, pełna zmiana | 60 min | właściciel **900–1400$/h** netto | >1700$/h | | | |
| C6 | P1 | Cap premii bossmenu | 2 premie 10k w tym samym tygodniu | 1× | **2. odrzucona**; max **10k/os./tydz.** | >10k sumarycznie | | | |

---

## D. Crime — napady (pojedynczo)

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| D1 | P1 | NPC solo | napad co 3 min CD, skup biżuterii | 60 min | **400–700$/h** po skupie | >900$/h | | | |
| D2 | P2 | ATM — karta | karty z NPC, wypłaty z bankomatów | 60 min | **600–1200$/h** | >1500$/h | | | |
| D3 | P2 | ATM — hack | hack + pranie | 60 min | **800–1500$/h** po praniu | >1800$/h | | | |
| D4 | P1 | Kasetka solo | lockpick, CD respektowany | 60 min | **1000–1800$/h** brutto dirty | >2200$/h | | | |
| D5 | P1 | Sejf sklepu | solo, **bez** Sandy `rewardBonus` | 60 min | **1500–2500$/h** brutto dirty | >3000$/h | | | |
| D6 | P0 | Fleeca duo | 2 os., pełny flow, CD **35 min** | 60 min | w mikście crime: **2000–3500$/h**/os. po praniu | >4500$/h tylko z Fleeca | | | |
| D7 | P0 | CD Fleeca | 2. napad przed upływem CD | 70 min | **2. zablokowany** | 2 wypłaty | | | |
| D8 | P1 | Tracker | heist / Carlos tier średni | 60 min | **700–1100$/h** efektywne | >1400$/h | | | |
| D9 | P1 | Boosting | tier średni auta, clean | 60 min | **1000–1800$/h** | >2200$/h | | | |
| D10 | P2 | Trucker crime | solo, CD ~45 min | 90 min | **~3000–3400$/h** efektywne | >4000$/h | | | |
| D11 | P1 | Pacific | grupa 4+, pełny loot | 120 min | **15–35k$/os.** po praniu; nie grind | powtórka <90 min CD | | | |
| D12 | P0 | Rotacja CD | ten sam typ napadu w pętli 1 h | 60 min | **max 1 wypłata** per typ | >1 wypłata / typ | | | |

---

## E. Narkotyki

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| E1 | P1 | Ulica — weed | joint / `weed_packed` | 60 min | **800–1200$/h** brutto dirty | >1500$/h | | | |
| E2 | P1 | Ulica — hard | coke / meth mix | 60 min | **1000–1500$/h** brutto | >1800$/h | | | |
| E3 | P0 | Ulica — aktywna | optymalny routing, solo | 60 min | **≤2200$/h** brutto | >2500$/h | | | |
| E4 | P1 | CD / tempo dealów | spam dealów pod rząd | 15 min | **≥20 s** między udanymi (jeśli wdrożone) | <10 s sustained | | | |
| E5 | P0 | `copsRequired` | sprzedaż przy 0–1 PD online | 1× | **zablokowane** przy <2 PD | sprzedaż bez PD | | | |
| E6 | P1 | KQ weed solo | 2–3 rośliny OG Kush | 60 min | **550–850$/h** brutto | >1000$/h | | | |
| E7 | P0 | Weed + ulica | produkcja + sprzedaż łącznie | 60 min | **≤2200$/h** brutto łącznie | >2500$/h | | | |
| E8 | P1 | Rico brick | skup brick po rescale v2 | 1× | **50–85k$**; 2. w 24h zablokowany (jeśli CD) | >100k lub brak limitu | | | |
| E9 | P2 | Mnożnik jakości | deal quality 100% vs 50% | porównanie | wyższa jakość **≤2×** bazy | >2,5× | | | |

---

## F. Pralnia

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| F1 | P1 | Pralnia automatyczna | 10k `black_money` | 1× | zwrot **55–65%** wartości | >70% | | | |
| F2 | P1 | Pralnia ręczna | 1 kurs, max punktów w limicie v2 | 45 min | **≤6750$** czystego; strata **20–30%** | >8000$ lub >10 punktów | | | |
| F3 | P1 | CD pralnia ręczna | 2 pełne kursy z rzędu | 60 min | **2. zablokowany** <45 min | 2 pełne kursy | | | |
| F4 | P0 | Pralnia bez lootu | start bez `black_money` | 1× | **brak** generowania $ | dodatnie $ | | | |

---

## G. Hazard i sinki

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| G1 | P0 | `MaxWager` | max stawka ruletka / slot | 1× | **≤10 000$** | >15k | | | |
| G2 | P0 | Zdrapka Deluxe | 20× `scratchticketd` | 30 min | średni wynik **ujemny**; max pojedyncza **≤8000$** | >10k lub zysk >0 | | | |
| G3 | P1 | Kasyno 1h | 50 spinów po 5k | 60 min | strata **10–30%** obrótu | zysk >0 po 1h | | | |
| G4 | P1 | Horse racing EV | 100 zakładów po 1k (log / symulacja) | — | **EV ≤ −8%** | EV ≥ 0 | | | |
| G5 | P2 | Koło fortuny | 10 spinów | 15 min | nagrody wg tabeli ×0,08 | pojedyncza >50k | | | |
| G6 | P2 | Kręgle | 3 gry z opłatą startową | 30 min | **sink** ~40$/gra; brak wypłat | dodatni $/h | | | |

---

## H. Koszty życia i progresja

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| H1 | P1 | Profil Minimum | 4× woda + 2× kanapka + 1× chips/h | 60 min | koszt **~45$/h** | >55$/h | | | |
| H2 | P1 | Koszty łącznie | jedzenie + paliwo + drobne @ ~500$/h dochodu | 60 min | **80–115$/h** (16–22%) | >150$/h | | | |
| H3 | P2 | Pierwsze auto | side job od zera (bank + gotówka start) | ~3 h | **~1,5k$** na auto entry | <2 h do 1,5k | | | |
| H4 | P2 | Odsprzedaż pojazdu | kup 10k, sprzedaj dealerowi | 1× | zwrot **6500$** (65%) | >7000$ | | | |
| H5 | P1 | Housing — kredyt | spłata kredytu | 45 min | **6%** salda / cykl (nie 30%/5 min) | >10%/cykl | | | |
| H6 | P2 | Czynsz | willa 700k, najemca 14 dni RL | 14 dni | właściciel **7000$** (1%) | >10k | | | |
| H7 | P2 | Cap pasywny wynajem | max nieruchomości na wynajmie | 30 dni | ekwiwalent **≤150$/h** | >200$/h | | | |

---

## I. Crime — sesja mieszana

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| I1 | P0 | Typowy crime mix | NPC → ATM → kasetka → kilka dealów | 60 min | **700–1200$/h** po praniu | >1440$/h | | | |
| I2 | P0 | Aktywny crime mix | sejf + hack + narkotyki + boosting | 60 min | **1800–2800$/h** po praniu | >3360$/h | | | |
| I3 | P1 | Dominacja jednego źródła | powtórka I2 — % z najlepszego źródła | — | **<50%** dochodu z jednego źródła | ≥60% z jednego | | | |
| I4 | P0 | Ekipa Fleeca | 4 os., rotacja napadów + inne crime | 120 min | **≤2800$/h**/os. średnio | >3500$/h sustained | | | |
| I5 | P2 | Wejście w crime | 3h side job → lockpick + hack → 1. napad | 3 h | narzędzia **~1500$**; 1. łup je pokrywa | narzędzia >3h grind | | | |

---

## J. Zadania Moris

| ID | P | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---:|---|---|---|---:|---|
| J1 | P1 | Fernando zad. 2 | pralnia (tutorial) | 1× | **1500–2500$** clean + 4–6k dirty | >5k clean | | | |
| J2 | P1 | Carlos boosting | tier niski auta | 45 min | **350–900$** clean | >1200$ | | | |
| J3 | P1 | Carlos tracker | tier średni | 60 min | **500–1200$** black | >1500$ | | | |
| J4 | P2 | Moris haracz zad. 1 | pełny flow | 40 min | bonus **300–600$** + loot kasetki | bonus >1000$ | | | |
| J5 | P1 | Po ukończeniu linii | powtórka Carla boosting | 60 min | **brak** lub ≤ tier napadu #3 | >1500$ sam quest | | | |

---

## K. Anti-exploit i regresja

| ID | P | Test | Kroki | Oczekiwany rezultat | FAIL jeśli | Wynik | Uwagi |
|---|---|---|---|---|---|---|---|
| K1 | P0 | Pełny deploy | checklista §0 `ekonomia-wdrozenie.md` | **wszystkie 13 blokerów** wdrożone | częściowy deploy | | |
| K2 | P0 | Duplikat wypłaty heist | reconnect w trakcie wypłaty | **1×** wypłata | 2× | | |
| K3 | P0 | Society bez craftu | `depositProducts` bez itemów w EQ | **odrzucone** | society rośnie | | |
| K4 | P1 | Stack wyroków MDT | 10 drobnych pozycji w 1 wyroku | cap FP **750$** | >750$ | | |
| K5 | P0 | Paycheck off duty `c_*` | off duty 1h w firmie | **0$** z paycheck | >0$ | | |
| K6 | P1 | Pacific bez lootu | ukończony napad Pacific | **wypłata** black + biżuteria wg v2 | brak wypłaty | | |
| K7 | P1 | Sejf Sandy bonus | napad na lokację Sandy | loot **bez +5000$** bonus | +5000$ | | |
| K8 | P0 | NPC CD | 2 napady NPC <3 min | **2. zablokowany** | 2 wypłaty | | |

---

## Podsumowanie przebiegu testów

| Sekcja | Testów | PASS | FAIL | SKIP |
|---|:---:|:---:|:---:|:---:|
| A — Side joby | 10 | | | |
| B — Paycheck / frakcje | 12 | | | |
| C — Firmy | 6 | | | |
| D — Napady | 12 | | | |
| E — Narkotyki | 9 | | | |
| F — Pralnia | 4 | | | |
| G — Hazard | 6 | | | |
| H — Koszty / progresja | 7 | | | |
| I — Crime mix | 5 | | | |
| J — Questy Moris | 5 | | | |
| K — Anti-exploit | 8 | | | |
| **Razem** | **84** | | | |

**Kryterium otwarcia serwera:** wszystkie **P0** = PASS; **P1** = PASS lub znany FAIL z planem hotfixu w 48h.

| Pole | Wartość |
|---|---|
| Data zakończenia QA | |
| Wersja / commit | |
| Lead tester | |
| Decyzja | `GO` / `NO-GO` |
| Znane wyjątki | |

---

## Changelog

| Data | Autor | Zmiana |
|---|---|---|
| 2026-06-22 | — | Utworzenie checklisty (84 testy) |
