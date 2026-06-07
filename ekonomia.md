# RageCity — docelowa ekonomia serwera (v2)

Dokument do analizy przez zespół. Opisuje docelowy stan ekonomii po przeskalowaniu. Po zatwierdzeniu wszystkich decyzji powstanie osobny plik `ekonomia-wdrozenie.md` z checklistą zmian per skrypt (np. `paycheck.lua`, `job_grades`, `rage_market`).

---

## Założenia ogólne

Ekonomia RageCity ma być wolniejsza, czytelna i oparta o progres. Praca dorywcza jest punktem odniesienia dla podstawowego zarobku. Frakcje, firmy i crime pozwalają zarabiać szybciej dzięki aktywnej grze, odpowiedzialności, ryzyku lub współpracy z innymi graczami.

| Obszar | Docelowy zarobek |
|---|---:|
| Praca dorywcza | ok. 500$/h |
| Pracownik firmy prywatnej | 1000–2000$/h |
| LSPD / LSSD / EMS / DOJ | 2500–3500$/h |
| Crime (łącznie z praniem) | 3000–5000$/h |
| Właściciel aktywnej firmy | 2000–4000$/h+ (może wyjść ponad frakcje) |

Kwoty frakcyjne i crime są wartościami docelowymi dla **aktywnej gry**. Nie powinny wynikać wyłącznie z pasywnej wypłaty za bycie online.

---

## Paycheck — zasady ogólne

Obecnie `es_extended/server/paycheck.lua` wypłaca **125$/15 min** wszystkim cywilom i **250$/15 min** frakcjom/firmom — niezależnie od tego, czy gracz faktycznie pracuje. To trzeba zmienić.

### Docelowy model

| Grupa | Paycheck co 15 min | Warunek | Efektywnie $/h |
|---|---:|---|---:|
| Bezrobotny / off-duty | 0$ | brak aktywnej pracy | 0$/h |
| Praca dorywcza (on duty) | 0$ | gracz zarabia z aktywności jobu | 0$/h + praca |
| Frakcja publiczna (on duty) | 75–100$ | tylko podczas służby | 300–400$/h |
| Firma prywatna `c_*` (on duty) | 25–50$ | tylko podczas służby | 100–200$/h |

**Zasady:**

- Paycheck idzie **bezpośrednio na konto gracza** co 15 minut, ale **tylko gdy jest on duty**.
- Side joby **nie dostają paychecka** — cały zarobek pochodzi z pracy (dostawy, craft, sprzedaż itd.).
- Paycheck frakcji to **podłoga dochodu**, nie główne źródło. Przy spokojnej służbie bez interakcji z graczami policjant/DOJ nadal dostaje ~300–400$/h, ale nie docelowych 2500–3500$/h.

---

## Skala cen pojazdów

Przy średnim zarobku 500$/h z pracy dorywczej:

| Poziom | Cena | Czas pracy dorywczej |
|---|---:|---:|
| Najtańsze auto | 1000–1500$ | 2–3 h |
| Pierwsze sensowne auto | 5000–10000$ | 10–20 h |
| Dobre auto | 25000–75000$ | 50–150 h |
| Bardzo dobre auto | 150000–400000$ | 300–800 h |
| Endgame auto | 600000–1200000$ | 1200–2400 h |

### Segmenty aut osobowych

| Segment | Cena |
|---|---:|
| Najtańsze auta / gruzy | 1000–1500$ |
| Słabe auta przejściowe | 1500–5000$ |
| Pierwsze sensowne auta | 5000–10000$ |
| Niższa klasa średnia | 10000–25000$ |
| Dobre auta codzienne | 25000–75000$ |
| Wyróżniające się auta | 75000–150000$ |
| Bardzo dobre auta | 150000–400000$ |
| Premium / top | 400000–600000$ |
| Endgame / super / import | 600000–1200000$ |

### Motocykle

| Segment | Cena |
|---|---:|
| Tanie / miejskie | 1500–8000$ |
| Przejściowe | 8000–25000$ |
| Sport / custom | 25000–150000$ |
| Top / limitowane | 150000–600000$ |
| Endgame moto | 600000–800000$ |

### Łodzie i skutery wodne

| Segment | Cena |
|---|---:|
| Skuter wodny / jet ski | ok. 800000–1200000$ |
| Łodzie rekreacyjne | 1000000–1800000$ |
| Łodzie premium / jachty | 1800000–2500000$ |

Helikoptery i samoloty to zakup **firmowy / grupowy**, nie solo endgame dla zwykłego gracza.

| Typ | Cena |
|---|---:|
| Helikopter (start frakcji/firmy) | 2000000–4000000$ |
| Samolot | 6000000–8000000$ |

### Widełki pojazdów według typu (config)

```lua
types = {
    automobile = { minPrice = 1000,   maxPrice = 1200000, roundTo = 500   },
    bike       = { minPrice = 1500,   maxPrice = 800000,  roundTo = 500   },
    bicycle    = { minPrice = 100,    maxPrice = 1000,    roundTo = 10    },
    boat       = { minPrice = 1000000, maxPrice = 2500000, roundTo = 10000 },
    submarine  = { minPrice = 500000, maxPrice = 1500000, roundTo = 10000 },
    heli       = { minPrice = 2000000, maxPrice = 4000000, roundTo = 10000 },
    plane      = { minPrice = 6000000, maxPrice = 8000000, roundTo = 10000 },
    trailer    = { minPrice = 2500,   maxPrice = 100000,  roundTo = 500   },
    train      = { minPrice = 0,      maxPrice = 0,       roundTo = 1     },
}
```

**Decyzja:** auta kończą się na **1.2 mln**. Motory max **800k**. Skutery wodne ok. **1 mln**, pozostałe łodzie **1–2.5 mln**. Helikoptery **2–4 mln**, samoloty **6–8 mln**.

---

## Prace dorywcze

| Praca | Docelowy zarobek | Uwagi |
|---|---:|---|
| Beach vendor / lekka praca RP | 350–500$/h | sprzedaż NPC, niski skill ceiling |
| Tailor | 450–550$/h | craft + sprzedaż |
| Slaughterer | 450–600$/h | craft + sprzedaż |
| Lumberjack | 500–650$/h | craft + sprzedaż |
| Miner | 500–700$/h | craft + sprzedaż |
| Deliveries (KQ) | 450–650$/h | wymaga drastycznego obniżenia `payPerMinute` |
| Powerwashing (KQ) | 500–750$/h | wymaga przeskalowania kontraktów |
| Trucker | 600–850$/h | najwyższa praca dorywcza |

Średnia dla zwykłych prac powinna oscylować wokół **500$/h**. Najlepsze prace dorywcze mogą być wyższe, ale nie powinny stale konkurować z aktywną grą frakcji lub crime.

**Priorytet wdrożenia:** KQ Deliveries i Powerwashing — obecnie dają **20k–30k+/h** i zniszczą kotwicę 500$/h, jeśli zostaną bez zmian.

---

## Frakcje publiczne

Docelowy łączny zarobek aktywnego gracza LSPD / LSSD / EMS / DOJ: **2500–3500$/h**.

### Model wypłat — rekomendacja

Trzy niezależne źródła. Gracz **sam zarabia na bieżąco**; premia tygodniowa to **dodatek**, nie zastępuje reszty.

| Źródło | Kiedy wypłacane | Co obejmuje | Rola w $/h |
|---|---|---|---|
| Paycheck (minutówka) | co 15 min, na koncie gracza | podstawa za bycie on duty | 300–400$/h (podłoga) |
| Zarobek aktywny | natychmiast po akcji | mandaty, revive, faktury, kursy EMS, wyroki DOJ | 1200–2500$/h (główna część) |
| Premia tygodniowa | raz w tygodniu, bossmenu | godziny służby, jakość, odpowiedzialność | +100–1500$/h średnio tygodniowo |

**Nie rekomendujemy** modelu, w którym cały zarobek frakcyjny jest wypłacany dopiero w piątek. Powody:

- LSPD / LSSD / DOJ zarabiają głównie z interakcji z graczami — mogą mieć godziny bez mandatów.
- EMS ma dodatkowo kursy do lokalnych medyków — to też powinno iść od razu na konto.
- Paycheck zapewnia podłogę (~300–400$/h) przy spokojnej służbie.
- Aktywność daje główny dochód i motywuje do RP.

**Przykład policjanta przy spokojnej służbie (1 h, zero mandatów):**

- Paycheck: ~350$/h
- Aktywność: 0–200$/h (patrol, obecność)
- **Razem: ~350–550$/h** — poniżej celu, ale fair za brak interakcji

**Przykład policjanta przy normalnej służbie (1 h, 3–4 mandaty po ~300$):**

- Paycheck: ~350$/h
- Mandaty (25% × ~1200$): ~300$/h
- **Razem: ~650$/h** + ewentualne większe sprawy

Przy pełnej aktywności (mandaty, zatrzymania, eventy) + premia tygodniowa rozłożona na godziny docelowy **2500–3500$/h** jest osiągalny.

### Minutówka / paycheck

| Grupa | Propozycja |
|---|---:|
| LSPD / LSSD / EMS / DOJ | 75–100$ co 15 minut (on duty) |
| Firmy prywatne `c_*` | 25–50$ co 15 minut (on duty) |
| Prace dorywcze | 0$ |
| Bezrobotny | 0$ |

### Procenty od pracy (natychmiast na konto gracza)

Obecny model splitów zostaje (25% oficer, reszta frakcja + DOJ), ale **kwoty mandatów i usług** trzeba przeskalować.

| Poziom aktywności | Dodatkowy zarobek gracza |
|---|---:|
| Spokojna służba | 0–500$/h |
| Normalna aktywna służba | 800–1800$/h |
| Bardzo aktywna / eventy / duże sprawy | 1800–2800$/h |

| Frakcja | Źródła zarobku gracza |
|---|---|
| LSPD / LSSD | mandaty, faktury, konwoje, sprawy, akcje kryminalne |
| EMS | revive, leczenie, transport, kursy do lokalnych medyków, zabezpieczenia eventów |
| DOJ | wyroki, ugody, licencje, sprawy sądowe, dokumenty |

### Docelowe kwoty mandatów i usług (MDT)

Aby procenty dawały sensowny zarobek bez inflacji:

| Kategoria | Przykładowa kwota mandatu/faktury | Udział oficera (25%) |
|---|---:|---:|
| Drobne wykroczenie | 50–150$ | 12–37$ |
| Standardowe wykroczenie | 150–400$ | 37–100$ |
| Poważne wykroczenie | 400–1000$ | 100–250$ |
| Przestępstwo | 1000–2500$ | 250–625$ |
| Ciężkie przestępstwo | 2500–5000$ | 625–1250$ |

| Usługa EMS | Cena dla klienta | Udział medyka (~25%) |
|---|---:|---:|
| Revive podstawowy | 400–800$ | 100–200$ |
| Leczenie / transport | 200–600$ | 50–150$ |
| Kurs do lokalnego medyka | 300–700$ | 75–175$ |

| Usługa DOJ | Cena | Udział pracownika (~25%) |
|---|---:|---:|
| Licencja / dokument | 200–800$ | 50–200$ |
| Ugoda / sprawa | 500–2000$ | 125–500$ |
| Wyrok / większa sprawa | 1500–5000$ | 375–1250$ |

*Szczegółowa tabela per mandat w MDT powstanie w pliku wdrożeniowym.*

### Premie tygodniowe (bossmenu)

Premia jest **dodatkiem** do paychecka i zarobku aktywnego. Gracz zgłasza się / odbiera ją w bossmenu po tygodniu.

| Aktywność tygodniowa | Premia | Limit |
|---|---:|---|
| Niska, ale obecna (3–5 h) | 1000–3000$ | max 25000$/os./tydz. |
| Regularna (8–15 h) | 4000–8000$ | max 25000$/os./tydz. |
| Wysoka (15–25 h) | 9000–15000$ | max 25000$/os./tydz. |
| Lider / wysoka odpowiedzialność | 15000–25000$ | max 25000$/os./tydz. |

Przykład: premia 10000$ przy 10 h służby w tygodniu = średnio **+1000$/h** w skali tygodnia.

**Decyzja:** limit tygodniowy **25000$/osobę**.

---

## Firmy prywatne

| Rola | Docelowy zarobek |
|---|---:|
| Pracownik firmy | 1000–2000$/h |
| Manager | 1500–2500$/h |
| Właściciel aktywnej firmy | 2000–4000$/h+ |

**Decyzja:** właściciel dobrze prowadzonej firmy **może zarabiać więcej niż frakcjonariusz** — kosztem kosztów operacyjnych, zarządzania personelem i aktywnej sprzedaży.

Firmy zarabiają przez klientów, usługi, produkcję lub sprzedaż. Dochód właściciela powinien uwzględniać:

- magazyn i surowce,
- pojazdy służbowe,
- wypłaty pracowników,
- podatki / wpłaty DOJ (obecnie 10% od sprzedaży).

Helikopter (2–4 mln) i większe łodzie (1–2.5 mln) są celowo drogie — zakładamy, że firma lub grupa graczy składa się na taki zakup.

---

## Crime

Docelowy zarobek crime: **3000–5000$/h** (po praniu, z cooldownami).

Crime **nie powinno** opierać się na jednym napadzie co godzinę. Cel 3000–5000$/h zakłada **mix aktywności**:

- drobne napady (NPC, ATM, kasetki) jako filler,
- sejfy / Fleeca jako główny dochód sesji,
- Pacific jako event tygodniowy.

| Element | Założenie |
|---|---|
| Koszt wejścia | lockpicki, hackingdevice, thermite, broń |
| Ryzyko | policja, utrata itemów, więzienie |
| Cooldowny | ograniczenie powtarzalności |
| Brudna gotówka | realny zysk liczony po praniu |
| Organizacja | większe akcje wymagają grupy |

---

## Napady — pełna rozpiska

### Tabela zbiorcza

| Napad | Min osób | Max osób | Łup min | Łup max | Typ nagrody | Uwagi |
|---|---:|---:|---:|---:|---|---|
| Napad na NPC | 1 | 1 | 25$ | 150$ | gotówka + loot | solo, niski skill |
| Bankomat — karta | 1 | 1 | 100$ | 350$ | gotówka | wymaga karty |
| Bankomat — hack | 1 | 1 | 200$ | 700$ | black_money | wymaga narzędzia |
| Kasetka sklepowa | 1 | 2 | 250$ | 600$ | 50–70% dirty + reszta gotówka | krótki cooldown (~8 min) |
| Sejf sklepu | 1 | 3 | 800$ | 2200$ | black_money | dłuższy cooldown (~10 min+) |
| Bank Fleeca | 2 | 4 | 6000$ | 16000$ | black_money | min. 2 napastników |
| Bank Pacific | 4 | 6 | 40000$ | 120000$ | black_money / loot / itemy | event crime, długi CD |

### Mnożniki nagrody (łączny loot)

Mnożniki zwiększają **łączny** loot grupy. Przy większej liczbie osób loot na osobę **nie rośnie proporcjonalnie**.

```lua
Config.AttackerMultipliers = {
    shop_cashregister = {
        [1] = 1.0,
        [2] = 1.15,
    },
    shop_vault = {
        [1] = 1.0,
        [2] = 1.2,
        [3] = 1.35,
    },
    bank = {
        [2] = 1.0,
        [3] = 1.2,
        [4] = 1.35,
    },
    bank_pacific = {
        [4] = 1.0,
        [5] = 1.2,
        [6] = 1.35,
    },
}
```

### Loot łączny i na osobę — kasetka sklepowa

Bazowy łup: **250–600$**

| Napastnicy | Mnożnik | Loot łączny | Loot na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 250–600$ | 250–600$ |
| 2 | ×1.15 | 287–690$ | 143–345$ |

### Loot łączny i na osobę — sejf sklepu

Bazowy łup: **800–2200$**

| Napastnicy | Mnożnik | Loot łączny | Loot na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 800–2200$ | 800–2200$ |
| 2 | ×1.2 | 960–2640$ | 480–1320$ |
| 3 | ×1.35 | 1080–2970$ | 360–990$ |

### Loot łączny i na osobę — bank Fleeca

Bazowy łup: **6000–16000$** | Min. napastników: **2**

| Napastnicy | Mnożnik | Loot łączny | Loot na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 6000–16000$ | 3000–8000$ |
| 3 | ×1.2 | 7200–19200$ | 2400–6400$ |
| 4 | ×1.35 | 8100–21600$ | 2025–5400$ |

**Decyzja:** Fleeca wymaga **minimum 2 napastników**.

### Loot łączny i na osobę — bank Pacific

Bazowy łup: **40000–120000$** | Min. napastników: **4**

| Napastnicy | Mnożnik | Loot łączny | Loot na osobę |
|---:|---:|---:|---:|
| 4 | ×1.0 | 40000–120000$ | 10000–30000$ |
| 5 | ×1.2 | 48000–144000$ | 9600–28800$ |
| 6 | ×1.35 | 54000–162000$ | 9000–27000$ |

### Napady solo (bez mnożników)

**Napad na NPC**

| Nagroda | Zakres |
|---|---:|
| Gotówka | 25–150$ |
| Drobny loot | według lombardu |
| Rzadkie itemy crime | niska szansa |

**Bankomat — karta:** 100–350$ gotówki  
**Bankomat — hack:** 200–700$ black_money

### Przyszłe napady (nieaktywne w kodzie)

W `rage_heists/Config.lua` są placeholder mnożniki dla jubilera, jachtu i Humane Labs. Po aktywacji powinny wpasować się między Fleeca a Pacific pod względem lootu i wymagań grupy. Do ustalenia w osobnej iteracji.

---

## Pralnia pieniędzy

| Typ prania | Strata | Uwagi |
|---|---:|---|
| Automatyczna pralnia | 35–45% | wygodna, mniej opłacalna |
| Ręczna pralnia (trasa) | 20–30% | bardziej opłacalna, wymaga trasy i ryzyka |

Prędkość automatycznej pralni: **25–75$/s** (obecnie 175$/s — za szybko).

| Kwota black_money | Metoda | Czysta gotówka |
|---:|---|---:|
| 10000$ | automatyczna, 40% straty | 6000$ |
| 10000$ | ręczna, 25% straty | 7500$ |

Przykład Fleeca (2 os., 8000$/os. dirty, pranie ręczne 25%): **6000$ czystego na osobę**.

---

## Narkotyki

Osobny loop crime — obecnie w `docs/drugs-ekonomia.md`. Nie jest uwzględniony w tabelach napadów, ale wpływa na balans crime.

### Docelowe założenia

| Element | Założenie |
|---|---|
| Docelowy zarobek | 2500–4000$/h (po kosztach produkcji) |
| Pozycja w hierarchii | porównywalne z napadami, ale wolniejsze i bardziej powtarzalne |
| Koszt wejścia | surowce, czas produkcji, ryzyko policji |
| Sprzedaż | wymaga posiadania towaru i aktywnego szukania klientów |

### Docelowe widełki cen sprzedaży NPC (orientacyjne)

| Tier | Przykładowe itemy | Cena za szt. (obecnie) | Docelowo |
|---|---|---:|---:|
| Niski | joint, grzyby | 75–150$ | 40–80$ |
| Średni | weed_packed, większość woreczków | 240–580$ | 80–200$ |
| Wysoki | coke, coke_bag, heroin | 430–1050$ | 150–350$ |

*Szczegółowa tabela per item powstanie w pliku wdrożeniowym na bazie `Config.Drugs`.*

---

## Koszty życia i usługi

| Rzecz | Cena docelowa | Obecnie (orientacyjnie) |
|---|---:|---:|
| Woda | 5–10$ | 20$ |
| Proste jedzenie | 8–20$ | 36$ |
| Lepsze jedzenie / restauracja | 25–80$ | — |
| Telefon | 500–1000$ | 1250–1500$ |
| Radio | 300–700$ | 850$ |
| Lockpick | 100–250$ | 1200$ |
| Fixkit | 250–600$ | 5000$ |
| Odholowanie zwykłe | 300–700$ | 800$ |
| Impound specjalny (helikopter/łódź) | 1500–3000$ | 5000$ |
| Lokalny medyk bez EMS | 500–1500$ | 50–10000$ (dynamiczny) |
| Lokalny mechanik | 500–1500$ | 1000–5000$ |
| Naprawa pojazdu z garażu | 500–1500$ | 1500$ (hardcoded) |
| Odsprzedaż pojazdu | 65% ceny zakupu | 65% (bez zmian) |
| Startowy bank | 2500$ | 2500$ (OK — starczy na pierwsze auto) |

---

## Bronie i narzędzia crime

| Item | Cena docelowa | Obecnie (orientacyjnie) |
|---|---:|---:|
| Broń biała | 500–2500$ | — |
| Słaby pistolet | 8000–15000$ | 90000–160000$ |
| Dobry pistolet | 15000–30000$ | — |
| SMG | 50000–120000$ | — |
| Karabin | 120000–300000$ | — |
| Hackingdevice | 1000–2500$ | 4000–5000$ |
| Hackingtablet | 2500–6000$ | — |
| Thermite | 500–1500$ | 2000$ |

---

## Mieszkania (qs-housing)

Moduł mieszkaniowy nie jest jeszcze w pełni uwzględniony w v2. Po przeskalowaniu głównej ekonomii trzeba osobno dopasować:

| Element | Uwaga |
|---|---|
| Ceny domów | DB-driven — wymaga audytu i przeskalowania |
| Kredyt / hipoteka | enabled — raty muszą być spójne z 500$/h bazą |
| Czynsz | co 5 min — może być zbyt agresywny po deflacji |
| Meble | obecnie 61–1350$ — prawdopodobnie OK |
| Opłaty przy zakupie | bank 10%, broker 5%, podatki 5% |

*Szczegóły w pliku wdrożeniowym po zatwierdzeniu głównego planu.*

---

## Kolejność wdrożenia

1. Zatwierdzić stawki bazowe i model wypłat frakcji.
2. Ustalić końcową tabelę segmentów pojazdów (w tym decyzje helikopter/samolot).
3. Przeskalować paycheck (`es_extended`) — tylko on duty, nowe stawki.
4. Przeskalować UI job center (wyświetlane salary ≠ real pay).
5. Przeskalować prace dorywcze — **priorytet: KQ Deliveries, Powerwashing, rage_jobs**.
6. Przeskalować frakcje:
   - mandaty MDT (tabela per wykroczenie),
   - usługi EMS / DOJ,
   - premie tygodniowe bossmenu (nowa implementacja).
7. Przeskalować napady i mnożniki napastników.
8. Przeskalować pralnię pieniędzy.
9. Przeskalować sklepy, bronie, narzędzia crime, koszty życia.
10. Przeskalować ceny pojazdów (`vehicles.json` + `VehiclePriceConfig`).
11. Przeskalować narkotyki (`Config.Drugs`).
12. Audyt i przeskalowanie qs-housing.

---

## Decyzje — podsumowanie

| Temat | Status | Decyzja |
|---|---|---|
| Właściciel firmy vs frakcja | ✅ Ustalone | Właściciel aktywnej firmy może zarabiać więcej (2000–4000$/h+) |
| Endgame auta | ✅ Ustalone | Max 1.2 mln |
| Motory | ✅ Ustalone | Max 600–800k |
| Skutery wodne | ✅ Ustalone | ok. 1 mln |
| Łodzie | ✅ Ustalone | 1–2.5 mln |
| Helikoptery | ✅ Ustalone | 2–4 mln (Zakup firmowy/grupowy) |
| Samoloty | ✅ Ustalone | 6–8 mln |
| Fleeca min. napastników | ✅ Ustalone | 2 |
| Kasetki sklepowe | ✅ Ustalone | 250–600$, krótki cooldown |
| Premie frakcyjne — limit | ✅ Ustalone | 25000$/os./tydz. |
| Model wypłat frakcji | ✅ Rekomendacja | Paycheck + zarobek aktywny na bieżąco + premia tygodniowa jako dodatek |
| Paycheck side jobów | ✅ Ustalone | 0$ — zarobek tylko z aktywności |
| Paycheck off-duty | ✅ Ustalone | 0$ |
| Narkotyki — docelowe ceny per item | ⏳ Do ustalenia | Widełki ogólne w dokumencie, szczegóły w wdrożeniu |
| Mieszkania qs-housing | ⏳ Do ustalenia | Osobna iteracja po głównym przeskalowaniu |
| Mandaty MDT — tabela per wykroczenie | ⏳ Do ustalenia | Widełki ogólne w dokumencie, szczegóły w wdrożeniu |
| Przyszłe napady (jubiler, jacht, Humane) | ⏳ Do ustalenia | Po aktywacji w kodzie |

---

## Następny krok

Po przeanalizowaniu przez zespół i zatwierdzeniu pozycji oznaczonych ⏳ powstanie plik **`ekonomia-wdrozenie.md`** — checklista zmian per plik/skrypt, np.:

- `[core]/es_extended/server/paycheck.lua` — stawki, warunek on duty
- `[rage]/rage_jobcenter/Config.lua` — wyświetlane salary
- `[tebex]/kq_deliveries/config.lua` — payPerMinute
- `[rage]/rage_market/Config.lua` — ceny itemów
- `[rage]/rage_heists/**` — nagrody, mnożniki, min/max grup
- `[rage]/rage_vehicleshop/vehicles.json` — ceny per model
- SQL `job_grades` — grade_salary (jeśli zostanie reaktywowane)
