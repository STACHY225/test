# RageCity — testy QA ekonomii v2

Checklista testów **po pełnym deployu** ekonomii v2. Uruchamiać na serwerze testowym ze **wszystkimi** zmianami z [`ekonomia-wdrozenie.md`](ekonomia-wdrozenie.md) §0 — częściowy deploy unieważnia wyniki.

**Widełki i zasady balansu:** [`ekonomia-zmiany-v2.md`](ekonomia-zmiany-v2.md)

**Szacowany czas QA (1 tester, sekwencyjnie):** ~**12–16 h** · z **3 testerami** równolegle (A/B/C + D/E + G/K): ~**5–7 h** · testy **T6** i **RL** opcjonalne przed startem.

---

## Metodologia

### Typy testów (kolumna **Typ**)

| Typ | Czas aktywny | Kiedy używać | Jak oceniać |
|---|---:|---|---|
| **spot** | sekundy–2 min | Config, UI, jednorazowa akcja, cap kwoty | wartość bezpośrednia — **bez** ekstrapolacji |
| **15m** | **15 min** | Side job, crime loop, narkotyki, hazard | `$/h = zarobek_netto ÷ min × 60` |
| **20m** | **20 min** | Aktywna frakcja, crime mix skrócony | j.w. |
| **30m** | **30 min** | Paycheck, AFK, minutówka (2× interwał 15 min) | j.w. lub suma 2 wypłat |
| **1×run** | 1 ukończona akcja | Napad, quest, kurs firmowy — trwa dłużej niż próbka | kwota **za run** (nie $/h) |
| **CD** | do upływu CD | Tylko czy **blokada** działa | PASS/FAIL bez $/h |
| **T6** | 60 min | **Opcjonalny** — tylko gdy próbka 15–20m na granicy FAIL | pełna sesja referencyjna |
| **RL** | dni RL | Housing czynsz, pasywny wynajem | poza blokiem pre-launch |

### Zasady skracania

1. **Minimum 2 pełne cykle** pętli joba w próbce 15m (np. 2 dostawy KQ, 2 trasy miner) — inaczej wynik **SKIP**.
2. **Paycheck / AFK:** wystarczą **2 interwały** (30m), nie 60m.
3. **Napady:** w próbce 15m mierz **1–2 udane akcje** tego samego typu; $/h licz tylko gdy CD na to pozwala — w przeciwnym razie **1×run**.
4. **Fleeca / Pacific / trucker crime:** testuj **łup za 1 run** + osobny test **CD** — nie godzinę farmy.
5. **FAIL:** wynik **> 120%** górnej granicy — w próbkach 15–20m wystarczy **1 FAIL**; pełna godzina (T6) tylko przy wątpliwości.
6. Kaucje jobów = **0 netto**. Crime domyślnie **po praniu** (~25% straty), chyba że „brutto dirty”.

### Pomiar

```
zarobek_netto = (konto_koniec − konto_start) − koszty_bez_zwrotu
$/h           = zarobek_netto ÷ minuty_aktywne × 60
```

| Element | Wartość |
|---|---|
| Admin / giveaway | **wyłączone** |
| Start postaci | v2: bank 2500$ + gotówka 1500$ |

### Legenda kolumn

| Kolumna | Znaczenie |
|---|---|
| **ID** | Skrót do raportu |
| **P** | P0 bloker · P1 wysoki · P2 regresja |
| **Typ** | spot / 15m / 20m / 30m / 1×run / CD / T6 / RL |
| **Czas** | Szacowany czas testera |
| **Wynik** | PASS / FAIL / SKIP |

### Szablon raportu

```
ID: 
Typ: 
Czas aktywny:     min
Zarobek netto:    $
$/h (jeśli dot.): $
Wynik: PASS | FAIL | SKIP
Online PD/EMS: 
Uwagi: 
```

---

## Kolejność przed otwarciem serwera

1. **K** (spot + CD) — ~30 min  
2. **B8, B9, A10, K5** (30m) — ~1,5 h  
3. **A7, A8** (15m) — ~30 min  
4. **C2, C3** (CD + spot) — ~20 min  
5. **D6, D7, D12, I2** (1×run + 20m) — ~2 h  
6. **E3, E7** (15m) — ~30 min  
7. **G1–G3** (spot + 15m) — ~20 min  
8. Reszta P1/P2 — równolegle  

---

## A. Kotwica — prace dorywcze

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| A1 | P1 | 15m | Miner | on duty, ≥2 pełne pętle rud | **15 min** | **500–700$/h** | >840$/h | | | |
| A2 | P1 | 15m | Lumberjack | jak A1 | **15 min** | **500–650$/h** | >780$/h | | | |
| A3 | P2 | 15m | Tailor | jak A1 | **15 min** | **450–550$/h** | >660$/h | | | |
| A4 | P2 | 15m | Slaughterer | jak A1 | **15 min** | **450–600$/h** | >720$/h | | | |
| A5 | P1 | 15m | Hunter | ≥2 cykle polowania + sprzedaż | **15 min** | **450–600$/h** | >720$/h | | | |
| A6 | P2 | 15m | Beach vendor | sprzedaż NPC w próbce | **15 min** | **350–500$/h** | >600$/h | | | |
| A7 | P0 | 15m | KQ Deliveries | 1 job środkowy, solo, ≥2 kontrakty | **15 min** | **450–650$/h** | >780$/h | | | |
| A8 | P0 | 15m | KQ Powerwashing | ≥2 kontrakty | **15 min** | **500–750$/h** | >900$/h | | | |
| A9 | P1 | 15m | KQ Trucker | ≥1 pełna trasa + część 2. | **15–20 min** | **600–850$/h** | >1020$/h | | | |
| A10 | P0 | 30m | AFK side job | on duty, bez ruchu, 2× paycheck | **30 min** | **≤60$/h** lub 0 off duty | >100$/h | | | |

---

## B. Paycheck i frakcje

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| B1 | P1 | 30m | LSPD spokojna | on duty, 0 mandatów | **30 min** | **650–750$/h** | >900$/h | | | |
| B2 | P1 | 20m | LSPD aktywna | on duty, dispatch w próbce | **20 min** | **780–980$/h** | >1200$/h | | | |
| B3 | P2 | spot | LSPD mandaty | 3 mandaty drobne, policz % udziału | **~10 min** | mandaty **≤30%** łącznej kwoty | >30% | | | nie wymaga 1h patrolu |
| B4 | P1 | 30m | EMS spokojna | on duty, 0 akcji | **30 min** | **460–500$/h** | >600$/h | | | |
| B5 | P1 | 20m | EMS aktywna | max akcji w 20m (revive + kursy) | **20 min** | **1000–1500$/h** | >1800$/h | | | |
| B6 | P2 | 30m | DOJ spokojna | on duty, 0 spraw | **30 min** | **580–700$/h** | >840$/h | | | |
| B7 | P1 | 30m | Mechanik LSC | on duty, 0 MDT | **30 min** | **650–700$/h** | >840$/h | | | |
| B8 | P0 | 30m | AFK frakcja | on duty, AFK, 2× paycheck | **30 min** | **0$/h** z duty | >50$/h | | | |
| B9 | P0 | 30m | Off duty frakcja | off duty, 2× interwał | **30 min** | **60$/h** lub 0 | >100$/h | | | |
| B10 | P1 | 30m | Firma `c_*` | on duty, bez kursów | **30 min** | **40–60$/h** | >80$/h | | | |
| B11 | P1 | spot | Cap udziału FP | 1 wyrok fine 5000$+ | **2 min** | udział **≤750$** | >750$ | | | |
| B12 | P0 | spot | MDT ulgi | szukaj −5000/−10000/−25000 | **2 min** | pozycje **brak** | ulga >2500$ | | | |

---

## C. Firmy prywatne

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| C1 | P1 | 1×run | Pracownik BS — kurs | 1 kurs, 10 paczek | **~20 min** | pracownik **800–1500$** + ~50% society | >1800$ | | | |
| C2 | P0 | CD | CD kursu | 2. kurs zaraz po 1. | **~25 min** | **2. zablokowany** 30–45 min | 2 wypłaty | | | |
| C3 | P0 | spot | `depositProducts` | 10× cheeseburger → deposit | **5 min** | society **≈ 34×0,85$/szt.** | >110% menu | | | |
| C4 | P2 | spot | Sprzedaż klientowi | 3 sprzedaże (nie 10) | **~10 min** | 90/10 split; marża **8–12$/burger** | strata <0 | | | |
| C5 | P1 | 20m | Właściciel aktywny | 2 pracowników, próbka 20m | **20 min** | **900–1400$/h** netto | >1700$/h | | | |
| C6 | P1 | spot | Cap premii | 2× premia 10k / tydzień | **2 min** | **2. odrzucona** | >10k/os. | | | |

---

## D. Crime — napady (pojedynczo)

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| D1 | P1 | 15m | NPC solo | max napadów przy CD 3 min | **15 min** | **400–700$/h** po skupie | >900$/h | | | ~5 napadów max |
| D2 | P2 | 15m | ATM karta | karty + wypłaty w próbce | **15 min** | **600–1200$/h** | >1500$/h | | | |
| D3 | P2 | 1×run | ATM hack | 1 hack + pranie | **~15 min** | **200–700$** po praniu | >875$ | | | kwota za run |
| D4 | P1 | 1×run | Kasetka | 1× kasetka solo | **~10 min** | **250–600$** brutto dirty | >720$ | | | |
| D5 | P1 | 1×run | Sejf sklepu | 1× sejf, bez Sandy bonus | **~15 min** | **800–2200$** brutto | >2640$ | | | |
| D6 | P0 | 1×run | Fleeca duo | 2 os., 1 pełny napad | **~20 min** | **2250–6000$/os.** po praniu | >7200$/os. | | | nie mierz $/h |
| D7 | P0 | CD | CD Fleeca | 2. start przed CD | **~40 min** | **2. zablokowany** | 2 wypłaty | | | głównie oczekiwanie CD |
| D8 | P1 | 1×run | Tracker | 1× tier średni | **~25 min** | **500–1200$** black | >1500$ | | | |
| D9 | P1 | 1×run | Boosting | 1× tier średni | **~20 min** | **350–900$** clean | >1080$ | | | |
| D10 | P2 | 1×run | Trucker crime | 1× dostawa | **~25 min** | **3500–8000$** brutto | >9600$ | | | |
| D11 | P1 | 1×run | Pacific | 1× grupa 4+ | **~45 min** | **15–35k$/os.** po praniu | >42k$/os. | | | |
| D12 | P0 | 15m | Rotacja CD | spam 1 typu napadu | **15 min** | **max 1 wypłata** / typ | >1 | | | |

---

## E. Narkotyki

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| E1 | P1 | 15m | Ulica weed | joint / `weed_packed` | **15 min** | **800–1200$/h** brutto | >1500$/h | | | |
| E2 | P1 | 15m | Ulica hard | coke / meth | **15 min** | **1000–1500$/h** brutto | >1800$/h | | | |
| E3 | P0 | 15m | Ulica aktywna | max tempo solo | **15 min** | **≤2200$/h** brutto | >2500$/h | | | |
| E4 | P1 | 15m | Tempo dealów | spam dealów | **15 min** | **≥20 s** między (jeśli wdrożone) | <10 s | | | |
| E5 | P0 | spot | `copsRequired` | sprzedaż przy <2 PD | **2 min** | **zablokowane** | sprzedaż OK | | | |
| E6 | P1 | 1×run | KQ weed | 1 zebrany cykl / odmiana niska | **~35 min** | **320–500$** za cykl OG | >600$ | | | lub T6 60m |
| E7 | P0 | 20m | Weed + ulica | produkcja z zapasu + deal | **20 min** | **≤2200$/h** łącznie | >2500$/h | | | |
| E8 | P1 | spot | Rico brick | 1 skup + próba 2. | **5 min** | **50–85k$**; 2. blocked 24h | >100k | | | |
| E9 | P2 | spot | Jakość | 1 deal 100% vs 50% | **5 min** | wyższa **≤2×** | >2,5× | | | |

---

## F. Pralnia

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| F1 | P1 | spot | Automatyczna | 10k black | **3 min** | zwrot **55–65%** | >70% | | | |
| F2 | P1 | 1×run | Ręczna | 1 kurs (max punktów v2) | **~20 min** | **≤6750$** czystego; strata 20–30% | >8000$ | | | |
| F3 | P1 | CD | CD ręczna | 2. kurs zaraz po 1. | **~25 min** | **2. zablokowany** <45 min | 2 kursy | | | |
| F4 | P0 | spot | Bez lootu | start 0 black | **2 min** | brak generowania $ | dodatnie $ | | | |

---

## G. Hazard i sinki

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| G1 | P0 | spot | `MaxWager` | max stawka UI | **1 min** | **≤10 000$** | >15k | | | |
| G2 | P0 | spot | Zdrapka Deluxe | 5× (nie 20×) | **5 min** | max **≤8000$**; brak >10k | >10k | | | |
| G3 | P1 | 15m | Kasyno | 20 spinów po 5k | **15 min** | strata **10–30%** obrótu | zysk >0 | | | |
| G4 | P1 | spot | Horse racing | 30 zakładów ×1k (log) | **~15 min** | **EV ≤ −8%** | EV ≥ 0 | | | lub skrypt offline |
| G5 | P2 | spot | Koło fortuny | 5 spinów | **5 min** | nagroda **≤50k** | >50k | | | |
| G6 | P2 | spot | Kręgle | 1 gra z opłatą | **5 min** | **~40$** sink; brak wypłaty | dodatni $ | | | |

---

## H. Koszty życia i progresja

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| H1 | P1 | 15m | Profil Minimum | loadout Minimum w 15m | **15 min** | koszt **~11–12$** / 15m → **~45$/h** | >55$/h | | | |
| H2 | P1 | 15m | Koszty łącznie | jedzenie + paliwo w 15m | **15 min** | **20–29$** / 15m → **80–115$/h** | >38$/15m | | | |
| H3 | P2 | 15m | Pierwsze auto | side job 15m od startu v2 | **15 min** | extrap. **≥375$** → **1,5k$** w ~60m | <250$ w 15m | | | pełny H3-T6: 60m opcja |
| H4 | P2 | spot | Odsprzedaż | kup 10k → sprzedaj | **3 min** | **6500$** (65%) | >7000$ | | | |
| H5 | P1 | spot | Housing kredyt | 1 cykl CreditTime (admin / wait) | **1× cykl** | **6%** salda | >10% | | | nie wymaga 45m jeśli admin trigger |
| H6 | P2 | RL | Czynsz 14d | najemca na willi 700k | **14 dni** | **7000$** | >10k | | | post-launch |
| H7 | P2 | RL | Cap pasywny | wynajem wielu domów | **30 dni** | **≤150$/h** ekwiw. | >200$/h | | | post-launch |

---

## I. Crime — sesja mieszana

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| I1 | P0 | 20m | Typowy mix | NPC + ATM + kasetka + 3 deale | **20 min** | **700–1200$/h** po praniu | >1440$/h | | | |
| I2 | P0 | 30m | Aktywny mix | sejf + hack + deal + boosting | **30 min** | **1800–2800$/h** po praniu | >3360$/h | | | |
| I3 | P1 | spot | Dominacja źródła | z logu I2 | **0 min** | **<50%** z 1 źródła | ≥60% | | | analiza I2 |
| I4 | P0 | 30m | Ekipa Fleeca | 4 os.: 1 Fleeca + inne w 30m | **30 min** | **≤2800$/h**/os. | >3500$/h | | | nie 2h |
| I5 | P2 | 20m | Wejście crime | 15m side job + zakup narzędzi + 1 napad entry | **~20 min** | łup entry **≥ koszt narzędzi** | narzędzia nieosiągalne | | | |

---

## J. Zadania Moris

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| J1 | P1 | 1×run | Fernando zad.2 | pralnia tutorial | **~15 min** | **1500–2500$** clean + 4–6k dirty | >5k clean | | | |
| J2 | P1 | 1×run | Carlos boosting | tier niski | **~20 min** | **350–900$** clean | >1080$ | | | |
| J3 | P1 | 1×run | Carlos tracker | tier średni | **~25 min** | **500–1200$** black | >1500$ | | | |
| J4 | P2 | 1×run | Moris haracz | zad.1 flow | **~25 min** | bonus **300–600$** + kasetka | bonus >1000$ | | | |
| J5 | P1 | spot | Po linii questów | 1 próba powtórki Carla | **5 min** | brak lub ≤ tier napadu | >1500$ quest only | | | |

---

## K. Anti-exploit i regresja

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Uwagi |
|---|---|---|---|---|---|---|---|---|---|
| K1 | P0 | spot | Pełny deploy | audyt §0 wdrożenie | **30 min** | 13/13 blokerów | częściowy deploy | | | |
| K2 | P0 | spot | Duplikat wypłaty | reconnect przy heist | **5 min** | **1×** wypłata | 2× | | |
| K3 | P0 | spot | Society exploit | deposit bez itemów | **2 min** | odrzucone | society ↑ | | |
| K4 | P1 | spot | Stack wyroków | 10 drobnych w 1 wyroku | **5 min** | cap **750$** | >750$ | | |
| K5 | P0 | 30m | Paycheck off duty | `c_*` off duty | **30 min** | **0$** | >0$ | | |
| K6 | P1 | 1×run | Pacific loot | ukończ napad | **~45 min** | wypłata wg v2 | brak lootu | | |
| K7 | P1 | 1×run | Sejf Sandy | napad Sandy | **~15 min** | bez **+5000$** | +5000$ | | |
| K8 | P0 | CD | NPC CD | 2 napady <3 min | **5 min** | 2. zablokowany | 2 wypłaty | | |

---

## T6 — sesje opcjonalne (tylko przy wątpliwym FAIL)

Uruchamiać **wyłącznie** gdy próbka 15–20m da wynik w strefie **110–120%** górnej granicy lub tester zgłosi anomalię.

| ID bazowy | Czas | Cel |
|---|---:|---|
| A1–A9 | 60 min | potwierdzenie $/h side job |
| B1, B5 | 60 min | potwierdzenie frakcji |
| E6 | 60 min | pełny KQ weed $/h |
| H3 | 60 min | czas do pierwszego auta |
| I2 | 60 min | potwierdzenie aktywnego crime |

---

## Podsumowanie przebiegu testów

| Sekcja | Testów | Typowy czas | PASS | FAIL | SKIP |
|---|:---:|:---:|---|---|---|
| A — Side joby | 10 | ~2,5 h | | | |
| B — Paycheck / frakcje | 12 | ~2,5 h | | | |
| C — Firmy | 6 | ~1 h | | | |
| D — Napady | 12 | ~3 h | | | |
| E — Narkotyki | 9 | ~1,5 h | | | |
| F — Pralnia | 4 | ~30 min | | | |
| G — Hazard | 6 | ~30 min | | | |
| H — Koszty / progresja | 7 | ~45 min (+RL) | | | |
| I — Crime mix | 5 | ~1,5 h | | | |
| J — Questy Moris | 5 | ~1,5 h | | | |
| K — Anti-exploit | 8 | ~1,5 h | | | |
| **Razem** | **84** | **~12–16 h** | | | |

**Kryterium GO:** wszystkie **P0** = PASS · **P1** = PASS lub hotfix 48h · **T6** nie wymagane do GO.

| Pole | Wartość |
|---|---|
| Data QA | |
| Commit | |
| Testerzy | |
| Decyzja | `GO` / `NO-GO` |

---

## Changelog

| Data | Zmiana |
|---|---|
| 2026-06-22 | Utworzenie checklisty (84 testy) |
| 2026-06-22 | Skrócenie metodologii: typy spot/15m/30m/1×run/CD; ~12–16h zamiast ~60h+ |
