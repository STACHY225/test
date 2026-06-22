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
| **cykl** | zależy od protokołu | Side job, KQ — **liczba akcji / serie** (§A) | `$/h` z stopera po protokole |
| **20m** | ~20 min | Aktywna frakcja, crime mix skrócony | `$/h` z stopera |
| **30m** | **30 min** | Paycheck, AFK (2× interwał 15 min) | suma wypłat ÷ 0,5 h |
| **1×run** | 1 ukończona akcja | Napad, quest, kurs firmowy | kwota **za run** (nie $/h) |
| **CD** | do upływu CD | Tylko czy **blokada** działa | PASS/FAIL bez $/h |
| **T6** | 60 min | **Opcjonalny** — granica FAIL w teście **cykl** | pełna sesja referencyjna |
| **RL** | dni RL | Housing czynsz, pasywny wynajem | poza blokiem pre-launch |

### Zasady pomiaru (liczba akcji, nie „wyczucie czasu”)

Testy **$/h** opierają się na **protokułach** — tester wykonuje określoną liczbę akcji lub zapełnia ekwipunek do progu, **bez** zgadywania ile stać na stacji.

| Zasada | Opis |
|---|---|
| **Start stopera** | pierwsza akcja produkcyjna protokołu (nie branie duty, nie dojazd) |
| **Stop stopera** | ostatnia sprzedaż / wypłata / dostawa w protokole |
| **Serie** | większość side jobów = **2 serie** pełnego cyklu (poniżej) |
| **SKIP** | jeśli nie ukończono całego protokołu (np. przerwany w połowie) |
| **Paycheck / AFK** | wyjątek: czekaj **2× interwał paycheck** (30 min), nie licz akcji |
| **1×run** | napad / quest — mierz **kwotę za 1 ukończoną akcję**, bez $/h |
| **FAIL** | wynik **> 120%** górnej granicy widełki v2 |

#### Co to jest „seria” / „cykl”

**Cykl** = przejście całego łańcucha joba od zbierania do sprzedaży (lub jedna dostawa KQ od startu do wypłaty).

Dwa warianty zbierania (wybierz **jeden** per test — domyślnie **A**):

| Wariant | Zasada zbierania | Reszta łańcucha |
|---|---|---|
| **A — próg** | Zbieraj aż item wejściowy osiągnie **próg z tabeli** (np. 30× `stone`) | **Przetwórz całość** + **sprzedaj całość** |
| **B — limit** | Zbieraj aż **pełny limit** itemu z configu joba (np. 80× `stone`) | j.w. |

Po zakończeniu cyklu ekwipunek produktów joba powinien być **pusty** (wszystko sprzedane). **Powtórz 2×** (2 serie). Stoper liczy obie serie.

### Pomiar

```
zarobek_netto = (konto_koniec − konto_start) − koszty_bez_zwrotu
czas_aktywny  = stop − start (min)
$/h           = zarobek_netto ÷ czas_aktywny × 60
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
| **Typ** | spot / cykl / 20m / 30m / 1×run / CD / T6 / RL |
| **Protokół** | odwołanie do § z liczbą akcji (nie „graj 15 min”) |
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
3. **A7, A8** (cykl) — ~45 min  
4. **C2, C3** (CD + spot) — ~20 min  
5. **D6, D7, D12, I2** (1×run + 20m) — ~2 h  
6. **E3, E7** (15m) — ~30 min  
7. **G1–G3** (spot + 15m) — ~20 min  
8. Reszta P1/P2 — równolegle  

---

## A. Kotwica — prace dorywcze

### Protokoły pętli (szczegóły per job)

Wspólny start: **weź duty** + pojazd / strój joba · **zapisz $** · **start stopera** przy pierwszej akcji zbierania.

#### A1 — Górnik (`miner`)

| Krok | Stacja (blip / task) | Akcja |
|---:|---|---|
| 1 | **Złup kamień** (kopalnia) | Wariant **A:** zbieraj do **30× `stone`** · Wariant **B:** do limitu **80×** |
| 2 | **Przesiewanie kamienia** | Przetwórz **cały** `stone` → `washed_stone` |
| 3 | **Wydobywanie surowców** | Przetwórz **cały** `washed_stone` → rudy |
| 4 | **Skup** (miedź / żelazo / złoto / diament) | **Sprzedaj wszystkie** rudy (4 pedów) |
| 5 | | **Powtórz kroki 1–4** = **2 serie** łącznie |

#### A2 — Drwal (`lumberjack`)

| Krok | Stacja | Akcja |
|---:|---|---|
| 1 | **Zbierz drewno** (las) | Wariant **A:** **30× `wood`** · **B:** limit **100×** |
| 2 | **Przeróbka drewna** | Przetwórz **całe** `wood` → `cutted_wood` |
| 3 | **Pakowanie desek** | Zapakuj **max** możliwe paczki (`cutted_wood` → `packaged_plank`, 16 desek / paczka) |
| 4 | **Skup / dostawa desek** | **Sprzedaj wszystkie** `packaged_plank` |
| 5 | | **2 serie** |

#### A3 — Krawiec (`tailor`)

| Krok | Stacja | Akcja |
|---:|---|---|
| 1 | **Zbiórka wełny** | Wariant **A:** **30× `wool`** · **B:** limit **100×** |
| 2 | **Tworzenie tkaniny** (baza) | Przetwórz **całą** wełnę → `fabric` |
| 3 | **Szycie ubrań** | Uszyj **max** `clothe` z tkaniny (2× `fabric` / ubranie) |
| 4 | **Skup ubrań** | **Sprzedaj wszystkie** `clothe` |
| 5 | | **2 serie** |

#### A4 — Rzeźnik (`slaughterer`)

| Krok | Stacja | Akcja |
|---:|---|---|
| 1 | **Zabierz kurczaki** | Wariant **A:** **30× `raw_chicken`** · **B:** limit **100×** |
| 2 | **Przygotuj kurczaka** | Przetwórz **cały** surowy kurczak (2 szt. → 1 `processed_chicken`) |
| 3 | **Zapakuj kurczaki** | Zapakuj **cały** `processed_chicken` |
| 4 | **Skup** | **Sprzedaj wszystkie** `packed_chicken` |
| 5 | | **2 serie** |

#### A5 — Myśliwy (`hunter`)

| Krok | Akcja |
|---:|---|
| 1 | Weź duty + broń · jedź na **kurs myśliwski** (NPC Mike Meat) |
| 2 | **Zabij i opróżnij łup** z **5 zwierząt** (mix dzik / jeleń) |
| 3 | W **skupie myśliwego** → **Sprzedaj wszystko** (`sellAllMeat`) |
| 4 | **Powtórz kroki 2–3** = **2 serie** (łącznie **10 zwierząt**) |

#### A6 — Plażowy sprzedawca (`beach_vendor`)

| Krok | Akcja |
|---:|---|
| 1 | Weź duty · ustaw stragan · wejdź w **kurs sprzedaży** |
| 2 | Wykonaj **10 udanych sprzedaży** do NPC (dowolne produkty) |
| 3 | **Powtórz** drugą serię: kolejne **10 sprzedaży** (łącznie **20**) |

#### A7 — KQ Deliveries

| Krok | Akcja |
|---:|---|
| 1 | Solo · job tier środkowy · **bez** team bonus abuse |
| 2 | **Ukończ 2 pełne kontrakty** (od przyjęcia do wypłaty) |
| 3 | Stoper: start 1. kontraktu → stop po wypłacie 2. |

#### A8 — KQ Powerwashing

| Krok | Akcja |
|---:|---|
| 1 | **Ukończ 2 pełne kontrakty** (od startu mycia do wypłaty) |

#### A9 — KQ Trucker (legalny)

| Krok | Akcja |
|---:|---|
| 1 | **Ukończ 1 pełną trasę** RS Haul (od przyjęcia zlecenia do wypłaty) |
| 2 | **Rozpocznij 2. trasę** — stop stoper po **połowie** trasy lub po pierwszym punkcie załadunku (ekstrapolacja $/h) |
| 3 | Alternatywa: **2 pełne trasy** jeśli czas pozwala |

#### A10 — AFK side job

| Krok | Akcja |
|---:|---|
| 1 | on duty · **bez ruchu** · odczekaj **2× paycheck** (2× 15 min) |
| 2 | Powtórz off duty: **2× interwał** bez duty |

---

### Tabela wyników — sekcja A

| ID | P | Typ | Test | Protokół | Szac. czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---:|---|---|---:|---|
| A1 | P1 | cykl | Miner | §A1 · wariant A · **2 serie** | ~25–40 min | **500–700$/h** | >840$/h | | | wariant B jeśli A <15 min |
| A2 | P1 | cykl | Lumberjack | §A2 · **2 serie** | ~30–45 min | **500–650$/h** | >780$/h | | | |
| A3 | P2 | cykl | Tailor | §A3 · **2 serie** | ~25–35 min | **450–550$/h** | >660$/h | | | |
| A4 | P2 | cykl | Slaughterer | §A4 · **2 serie** | ~25–35 min | **450–600$/h** | >720$/h | | | |
| A5 | P1 | cykl | Hunter | §A5 · **10 zwierząt** | ~25–40 min | **450–600$/h** | >720$/h | | | |
| A6 | P2 | cykl | Beach vendor | §A6 · **20 sprzedaży** | ~20–30 min | **350–500$/h** | >600$/h | | | |
| A7 | P0 | cykl | KQ Deliveries | §A7 · **2 kontrakty** | ~20–35 min | **450–650$/h** | >780$/h | | | |
| A8 | P0 | cykl | KQ Powerwashing | §A8 · **2 kontrakty** | ~25–40 min | **500–750$/h** | >900$/h | | | |
| A9 | P1 | cykl | KQ Trucker | §A9 · 1 trasa + część 2. | ~25–35 min | **600–850$/h** | >1020$/h | | | |
| A10 | P0 | 30m | AFK side job | §A10 · 2× paycheck | **30 min** | **≤60$/h** lub 0 off duty | >100$/h | | | |

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

**Pomiar $/h (D1, D2):** stoper od 1. akcji do ostatniej wypłaty · liczba akcji wg tabeli.

| ID | P | Typ | Test | Protokół | Szac. czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---:|---|---|---:|---|
| D1 | P1 | cykl | NPC solo | **5×** udany napad (CD 3 min) + skup całej biżuterii po każdej serii · **1 seria** | ~20 min | **400–700$/h** po skupie | >900$/h | | | |
| D2 | P2 | cykl | ATM karta | **5×** wypłata kartą (różne bankomaty jeśli CD) | ~15–25 min | **600–1200$/h** | >1500$/h | | | |
| D3 | P2 | 1×run | ATM hack | 1 hack + pranie | ~15 min | **200–700$** po praniu | >875$ | | | |
| D4 | P1 | 1×run | Kasetka | 1× kasetka solo | ~10 min | **250–600$** brutto dirty | >720$ | | | |
| D5 | P1 | 1×run | Sejf sklepu | 1× sejf, bez Sandy bonus | ~15 min | **800–2200$** brutto | >2640$ | | | |
| D6 | P0 | 1×run | Fleeca duo | 2 os., 1 pełny napad | ~20 min | **2250–6000$/os.** po praniu | >7200$/os. | | | |
| D7 | P0 | CD | CD Fleeca | 2. start przed CD | ~40 min | **2. zablokowany** | 2 wypłaty | | | |
| D8 | P1 | 1×run | Tracker | 1× tier średni | ~25 min | **500–1200$** black | >1500$ | | | |
| D9 | P1 | 1×run | Boosting | 1× tier średni | ~20 min | **350–900$** clean | >1080$ | | | |
| D10 | P2 | 1×run | Trucker crime | 1× dostawa | ~25 min | **3500–8000$** brutto | >9600$ | | | |
| D11 | P1 | 1×run | Pacific | 1× grupa 4+ | ~45 min | **15–35k$/os.** po praniu | >42k$/os. | | | |
| D12 | P0 | CD | Rotacja CD | **3×** próba tego samego napadu bez CD | ~15 min | **max 1 wypłata** | >1 | | | |

---

## E. Narkotyki

| ID | P | Typ | Test | Protokół | Szac. czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---:|---|---|---:|---|
| E1 | P1 | cykl | Ulica weed | **8×** udany deal (`joint` lub `weed_packed`) | ~15–25 min | **800–1200$/h** brutto | >1500$/h | | | |
| E2 | P1 | cykl | Ulica hard | **8×** udany deal (coke / meth) | ~15–25 min | **1000–1500$/h** brutto | >1800$/h | | | |
| E3 | P0 | cykl | Ulica aktywna | **12×** deal max tempo | ~20–30 min | **≤2200$/h** brutto | >2500$/h | | | |
| E4 | P1 | cykl | Tempo dealów | **10×** próba dealów pod rząd | ~10 min | **≥20 s** między udanymi (jeśli wdrożone) | <10 s | | | |
| E5 | P0 | spot | `copsRequired` | sprzedaż przy <2 PD | 2 min | **zablokowane** | sprzedaż OK | | | |
| E6 | P1 | 1×run | KQ weed | 1 pełny cykl OG Kush (sadzonka → sprzedaż bag) | ~35 min | **320–500$** za cykl | >600$ | | | |
| E7 | P0 | cykl | Weed + ulica | 1 cykl KQ (zapas) + **6×** deal uliczny | ~25–35 min | **≤2200$/h** łącznie | >2500$/h | | | |
| E8 | P1 | spot | Rico brick | 1 skup + próba 2. | 5 min | **50–85k$**; 2. blocked 24h | >100k | | | |
| E9 | P2 | spot | Jakość | 1 deal 100% vs 50% | 5 min | wyższa **≤2×** | >2,5× | | | |

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
| G3 | P1 | cykl | Kasyno | **20 spinów** po 5k (licznik spinów, nie czas) | ~10–15 min | strata **10–30%** obrótu | zysk >0 | | | |
| G4 | P1 | spot | Horse racing | 30 zakładów ×1k (log) | **~15 min** | **EV ≤ −8%** | EV ≥ 0 | | | lub skrypt offline |
| G5 | P2 | spot | Koło fortuny | 5 spinów | **5 min** | nagroda **≤50k** | >50k | | | |
| G6 | P2 | spot | Kręgle | 1 gra z opłatą | **5 min** | **~40$** sink; brak wypłaty | dodatni $ | | | |

---

## H. Koszty życia i progresja

| ID | P | Typ | Test | Kroki | Czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---|---|---|---:|---|
| H1 | P1 | cykl | Profil Minimum | zużyj loadout Minimum: **4× woda + 2× kanapka + 1× chips** (1 „h” głodu/pragnienia w skali HUD) | ~10 min | koszt **~45$/h** ekwiw. | >55$/h | | | przelicz z faktycznego zużycia |
| H2 | P1 | cykl | Koszty łącznie | loadout H1 + **~5 km** jazdy + 1 myjnia | ~15 min | **80–115$/h** ekwiw. @ 500$/h | >150$/h ekwiw. | | | |
| H3 | P2 | cykl | Pierwsze auto | **2 serie** side job (np. §A1) od startu v2 | ~50 min | zarobek **≥1500$** łącznie | <1200$ | | | T6: 4 serie |
| H4 | P2 | spot | Odsprzedaż | kup 10k → sprzedaj | **3 min** | **6500$** (65%) | >7000$ | | | |
| H5 | P1 | spot | Housing kredyt | 1 cykl CreditTime (admin / wait) | **1× cykl** | **6%** salda | >10% | | | nie wymaga 45m jeśli admin trigger |
| H6 | P2 | RL | Czynsz 14d | najemca na willi 700k | **14 dni** | **7000$** | >10k | | | post-launch |
| H7 | P2 | RL | Cap pasywny | wynajem wielu domów | **30 dni** | **≤150$/h** ekwiw. | >200$/h | | | post-launch |

---

## I. Crime — sesja mieszana

| ID | P | Typ | Test | Protokół | Szac. czas | Oczekiwany rezultat | FAIL jeśli | Wynik | Zmierzone | Uwagi |
|---|---|---|---|---|---:|---|---|---:|---|
| I1 | P0 | cykl | Typowy mix | **3×** NPC + **2×** ATM karta + **1×** kasetka + **4×** deal + pranie | ~25–35 min | **700–1200$/h** po praniu | >1440$/h | | | |
| I2 | P0 | cykl | Aktywny mix | **1×** sejf + **1×** hack ATM + **6×** deal + **1×** boosting + pranie | ~35–50 min | **1800–2800$/h** po praniu | >3360$/h | | | |
| I3 | P1 | spot | Dominacja źródła | analiza logu $ z I2 | 0 min | **<50%** z 1 źródła | ≥60% | | | |
| I4 | P0 | cykl | Ekipa Fleeca | 4 os.: **1×** Fleeca + **3×** NPC + **4×** deal / os. | ~35–45 min | **≤2800$/h**/os. | >3500$/h | | | |
| I5 | P2 | cykl | Wejście crime | **1 seria** §A1 (lub inny side job) → kup lockpick + hack → **1×** kasetka | ~40–55 min | łup kasetki **≥** koszt narzędzi | narzędzia nieosiągalne | | | |

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
| A1–A9 | 4 serie lub wariant B (limit) | potwierdzenie $/h side job |
| B1, B5 | 60 min | potwierdzenie frakcji |
| E6 | 60 min | pełny KQ weed $/h |
| H3 | 60 min | czas do pierwszego auta |
| I2 | 60 min | potwierdzenie aktywnego crime |

---

## Podsumowanie przebiegu testów

| Sekcja | Testów | Typowy czas | PASS | FAIL | SKIP |
|---|:---:|:---:|---|---|---|
| A — Side joby | 10 | ~3–5 h | | | |
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
| 2026-06-22 | Protokoły pętli (liczba akcji / serie) zamiast „graj X minut” |
