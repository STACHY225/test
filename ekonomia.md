# RageCity — docelowa ekonomia serwera (v2)

Dokument do analizy przez zespół. Po zatwierdzeniu powstanie plik `ekonomia-wdrozenie.md` z checklistą zmian per skrypt.

---

## Założenia ogólne

Ekonomia RageCity ma być wolniejsza, czytelna i oparta o progres. Praca dorywcza jest punktem odniesienia. Frakcje, firmy i crime pozwalają zarabiać szybciej dzięki aktywnej grze, ryzyku lub współpracy — ale **endgame wymaga tygodni/miesięcy**, nie kilkunastu dni grindu.

| Obszar | Docelowy zarobek |
|---|---:|
| Zasiłek / przeżycie | 60$/h |
| Praca dorywcza (zasiłek + aktywność) | ok. 500$/h |
| Pracownik firmy prywatnej | 800–1500$/h |
| LSPD / LSSD / EMS | 1800–2500$/h |
| DOJ | 1500–2200$/h |
| Crime (po praniu, z cooldownami) | 2000–3500$/h |
| Właściciel aktywnej firmy | 1800–3500$/h+ (może wyjść ponad frakcje) |

**Założenie sesji referencyjnej:** 6 h/dzień, 6 dni/tydzień = **~36 h/tydzień** (typowy aktywny gracz).

**Tempo progresu (orientacyjne):**

| Cel | Przy 500$/h (side job) | Przy 2200$/h (frakcja) | Przy 2800$/h (crime) |
|---|---:|---:|---:|
| Pierwsze auto (1.5k) | 3 h | 1 h | 1 h |
| Sensowne auto (10k) | 20 h | 5 h | 4 h |
| Dobre auto (75k) | 150 h | 34 h | 27 h |
| Endgame auto (1.2 mln) | **2400 h** | **545 h** | **429 h** |

Przy 6 h/dzień endgame auto to ok. **72 dni** pracy dorywczej, **~15 tygodni** aktywnej frakcji lub **~12 tygodni** crime — nie 10 dni. Obecna inflacja (20k+/h) daje endgame w ~60–100 h — stąd przeskalowanie.

---

## Paycheck i zasiłek

Obecnie `es_extended/server/paycheck.lua` wypłaca **125$/15 min** wszystkim i **250$/15 min** frakcjom — bez warunku on duty. To trzeba zmienić.

### Jedna tabela stawek

| Grupa | Co 15 min | $/h | Warunek |
|---|---:|---:|---|
| Bezrobotny (zasiłek) | 15$ | **60$** | brak pracy / off-duty |
| Praca dorywcza | 15$ | **60$** | on duty + zarobek z aktywności |
| Firma prywatna `c_*` | 10–15$ | 40–60$ | on duty |
| LSPD / LSSD / EMS | 20–22$ | 80–88$ | on duty |
| DOJ | 25–30$ | **100–120$** | on duty |

**Zasady:**

- Wypłata co 15 min **bezpośrednio na konto gracza**.
- Side job: zasiłek 60$/h + aktywność (~440$/h) = ok. 500$/h łącznie.
- Paycheck frakcji to **podłoga**, nie główne źródło — szczególnie dla LSPD/LSSD zależnych od mandatów.
- DOJ dostaje **wyższą minutówkę**, bo nie ma kursów ani regularnych interakcji jak EMS czy LSPD.

---

## Frakcje publiczne

### Model wypłat (zatwierdzony)

Trzy niezależne źródła. Gracz zarabia **na bieżąco**; premia tygodniowa to **dodatek**.

| Źródło | Kiedy | Rola |
|---|---|---|
| Paycheck | co 15 min, on duty | podłoga 80–120$/h |
| Zarobek aktywny | natychmiast po akcji | 200–1200$/h w zależności od aktywności |
| Premia tygodniowa | bossmenu, raz/tydz. | +50–150$/h średnio (przy 36 h/tydz.) |

Realistycznie większość frakcjonariuszy **nie osiągnie** górnej granicy 2500$/h każdej godziny — i tak ma być. Cel 1800–2500$/h to **aktywna sesja + premia rozłożona**, nie stała stawka.

### Zarobek aktywny — realistyczne widełki

| Poziom aktywności | Dodatkowy zarobek gracza | Kiedy |
|---|---:|---|
| Spokojna służba | 0–200$/h | patrol, brak graczy, brak spraw |
| Normalna służba | 200–600$/h | kilka mandatów, revive, drobne sprawy |
| Wysoka aktywność | 600–1200$/h | dużo interakcji, eventy, poważne sprawy |

**Przykład LSPD — spokojna godzina (zero mandatów):** paycheck ~85$/h → **~85$/h**  
**Przykład LSPD — normalna godzina (4 mandaty × ~120$, 25%):** ~85 + ~120 = **~205$/h**  
**Przykład EMS — aktywna godzina (6 revive × ~80$ udziału):** ~85 + ~480 = **~565$/h**  
**Przykład DOJ — spokojna godzina:** paycheck ~110$/h → **~110$/h** (minutówka nadrabia brak kursów)  
**Przykład DOJ — normalna godzina (2 sprawy × ~400$, 25%):** ~110 + ~200 = **~310$/h**

### LSPD / LSSD

| Źródło | Opis |
|---|---|
| Paycheck | 80–88$/h |
| Mandaty / faktury | 25% kwoty — główne źródło aktywne |
| Konwoje, sprawy | okazjonalne bonusy |

### EMS

| Źródło | Opis |
|---|---|
| Paycheck | 80–88$/h |
| Revive / leczenie | 25% kwoty usługi |
| Kursy do lokalnych medyków | stałe, powtarzalne źródło — główny dochód EMS |

### DOJ — model osobny

DOJ **nie ma kursów** ani stałego strumienia interakcji. Dlatego:

| Element | Założenie |
|---|---|
| Paycheck gracza | **100–120$/h** — wyższy niż LSPD/EMS |
| Zarobek aktywny | wyroki, ugody, licencje — nieregularny, 100–600$/h gdy są sprawy |
| Konto DOJ (society) | **10% od podatków** — wpływy z mandatów, sprzedaży firm, salonu pojazdów, usług EMS itd. |
| Premia tygodniowa | jak pozostałe frakcje |

Konto DOJ nie trafia bezpośrednio do kieszeni gracza — finansuje frakcję, premie, eventy. Pracownik DOJ żyje głównie z **wyższej minutówki + sporadycznych spraw + premii**.

### Docelowe kwoty mandatów (obniżone)

Poprzednie widełki były za wysokie — przy 25% udziale gracza dawały zawyżony zarobek. Nowe:

| Kategoria | Kwota mandatu | Udział LSPD/LSSD (25%) |
|---|---:|---:|
| Drobne wykroczenie | 30–80$ | 8–20$ |
| Standardowe wykroczenie | 80–200$ | 20–50$ |
| Poważne wykroczenie | 200–500$ | 50–125$ |
| Przestępstwo | 500–1200$ | 125–300$ |
| Ciężkie przestępstwo | 1200–3000$ | 300–750$ |

Przy 4 mandatach/h × 120$ × 25% = **~120$/h** z mandatów — realistyczne uzupełnienie paychecka.

### Docelowe usługi EMS / DOJ

| Usługa | Cena dla klienta | Udział pracownika (~25%) |
|---|---:|---:|
| Revive podstawowy | 200–500$ | 50–125$ |
| Leczenie / transport | 150–400$ | 37–100$ |
| Kurs EMS → lokalny medyk | 200–500$ | 50–125$ |
| Licencja / dokument DOJ | 150–500$ | 37–125$ |
| Ugoda / sprawa DOJ | 300–1000$ | 75–250$ |
| Wyrok / większa sprawa | 800–2500$ | 200–625$ |

### Premie tygodniowe (bossmenu)

Założenie: **~36 h/tydzień** (6 h × 6 dni). Premia to dodatek — przy 36 h premia 3600$ = **+100$/h** średnio.

| Godziny w tygodniu | Premia | Średnio $/h (przy 36 h) |
|---:|---:|---:|
| 5–15 h (obecność) | 500–1500$ | 14–42$/h |
| 20–30 h | 1500–2500$ | 50–83$/h |
| 30–40 h (typowy grind) | 2500–4000$ | 69–111$/h |
| 40–50 h | 4000–6000$ | 80–120$/h |
| Lider / odpowiedzialność | 6000–10000$ | 120–200$/h |
| **Limit** | **max 10000$/os./tydz.** | — |

---

## Firmy prywatne

| Rola | Docelowy zarobek |
|---|---:|
| Pracownik | 800–1500$/h |
| Manager | 1200–2000$/h |
| Właściciel aktywnej firmy | 1800–3500$/h+ |

Właściciel dobrze prowadzonej firmy **może zarabiać więcej niż frakcjonariusz** — kosztem kosztów operacyjnych, personelem i aktywną sprzedażą. DOJ dostaje 10% od obrotu firmy na konto society.

---

## Skala cen pojazdów

| Poziom | Cena | Czas @ 500$/h |
|---|---:|---:|
| Najtańszy pojazd | 1000–1500$ | 2–3 h |
| Pierwszy sensowny | 5000–10000$ | 10–20 h |
| Dobry | 25000–75000$ | 50–150 h |
| Bardzo dobry | 150000–400000$ | 300–800 h |
| Endgame auto | 600000–1200000$ | 1200–2400 h |

**Auta** max **1.2 mln** | **Motory** max **800k** | **Skutery wodne** ~**1 mln** | **Łodzie** **1–2.5 mln** | **Helikoptery** **2–4 mln** | **Samoloty** **6–8 mln**

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

---

## Prace dorywcze

| Praca | Aktywność $/h | + zasiłek 60$/h | Razem |
|---|---:|---:|---:|
| Beach vendor | 290–440$ | 60$ | 350–500$ |
| Tailor | 390–490$ | 60$ | 450–550$ |
| Slaughterer | 390–540$ | 60$ | 450–600$ |
| Lumberjack | 440–590$ | 60$ | 500–650$ |
| Miner | 440–640$ | 60$ | 500–700$ |
| Deliveries (KQ) | 390–590$ | 60$ | 450–650$ |
| Powerwashing (KQ) | 440–690$ | 60$ | 500–750$ |
| Trucker | 540–790$ | 60$ | 600–850$ |

**Priorytet wdrożenia:** KQ Deliveries i Powerwashing — obecnie **20k–30k+/h**, wymagają ~40× obniżki.

---

## Crime — założenia

Docelowy zarobek: **2000–3500$/h** po praniu, z cooldownami. Crime to **mix** aktywności, nie jeden napad co godzinę.

| Element | Założenie |
|---|---|
| Koszt wejścia | narzędzia tanie na start, droższe na wyższe napady |
| Ryzyko | policja, więzienie, utrata itemów |
| Cooldowny | ograniczenie farmienia |
| Biżuteria | sprzedaż w lombardzie / dark_lombardzie |
| Brudna gotówka | realny zysk po praniu (20–45% straty) |

---

## Napady — progresja i pełna rozpiska

### Kolejność (od najłatwiejszego)

1. Napad na NPC  
2. Bankomat — karta  
3. Boosting  
4. Bankomat — hack  
5. Kasetka sklepowa  
6. Tracker  
7. Sejf sklepu  
8. Jubiler  
9. Bank Fleeca  
10. Napad na lombard  
11. Napad na jacht  
12. Bank Pacific  
13. Humane Labs  
14. Cayo Perico (endgame)

### Tabela zbiorcza

| # | Napad | Min os. | Max os. | Łup min | Łup max | Typ nagrody | Status w kodzie |
|---:|---|---:|---:|---:|---:|---|---|
| 1 | NPC | 1 | 1 | 25$ | 150$ | gotówka + biżuteria (niska) | ✅ aktywny |
| 2 | Bankomat — karta | 1 | 1 | 100$ | 350$ | gotówka | ✅ aktywny |
| 3 | Boosting | 1 | 1 | 350$ | 900$ | gotówka / dirty | 🔧 do wdrożenia |
| 4 | Bankomat — hack | 1 | 1 | 200$ | 700$ | black_money | ✅ aktywny |
| 5 | Kasetka sklepowa | 1 | 2 | 250$ | 600$ | 50–70% dirty + gotówka | ✅ aktywny |
| 6 | Tracker | 1 | 2 | 500$ | 1400$ | black_money | 🔧 do wdrożenia |
| 7 | Sejf sklepu | 1 | 3 | 800$ | 2200$ | black_money | ✅ aktywny |
| 8 | Jubiler | 2 | 5 | 1200$ | 4500$ | biżuteria → lombard | 🔧 do wdrożenia |
| 9 | Bank Fleeca | 2 | 4 | 6000$ | 16000$ | black_money | ✅ aktywny |
| 10 | Lombard | 1 | 3 | 600$ | 1800$ | gotówka + biżuteria | 🔧 do wdrożenia |
| 11 | Jacht | 3 | 5 | 10000$ | 25000$ | black_money / loot | 🔧 do wdrożenia |
| 12 | Bank Pacific | 4 | 6 | 35000$ | 90000$ | black_money / loot | ✅ aktywny |
| 13 | Humane Labs | 3 | 5 | 22000$ | 55000$ | black_money / loot | 🔧 do wdrożenia |
| 14 | Cayo Perico | 5 | 8 | 70000$ | 130000$ | black_money / loot | 🔧 do wdrożenia |

*Łup jubilera/lombardu/NPC w biżuterii = łączna wartość skupu w lombardzie (patrz tabela biżuterii).*

### Mnożniki nagrody (łączny loot)

```lua
Config.AttackerMultipliers = {
    shop_cashregister = { [1] = 1.0,  [2] = 1.15 },
    boosting          = { [1] = 1.0 },
    tracker           = { [1] = 1.0,  [2] = 1.10 },
    shop_vault        = { [1] = 1.0,  [2] = 1.2,  [3] = 1.35 },
    jeweler           = { [2] = 1.0,  [3] = 1.15, [4] = 1.25, [5] = 1.35 },
    bank              = { [2] = 1.0,  [3] = 1.2,  [4] = 1.35 },
    pawnshop          = { [1] = 1.0,  [2] = 1.15, [3] = 1.25 },
    yacht             = { [3] = 1.0,  [4] = 1.15, [5] = 1.30 },
    bank_pacific      = { [4] = 1.0,  [5] = 1.2,  [6] = 1.35 },
    humane_labs       = { [3] = 1.0,  [4] = 1.15, [5] = 1.30 },
    cayo_perico       = { [5] = 1.0,  [6] = 1.15, [7] = 1.25, [8] = 1.35 },
}
```

### Szczegóły per napad

**1. NPC** — gotówka 25–150$ + biżuteria niskiej jakości (srebro, kolczyki: 15–45$/szt. w lombardzie). Szansa na telefon/bankcard.

**3. Boosting** — znajdź wskazane auto w mieście, dostarcz do punktu. Czas ~10–20 min, cooldown ~15 min.

**6. Tracker** — ucieczka przed LSPD autem z nadajnikiem GPS. Wymaga jazdy bez zatrzymania przez X min. 1–2 osoby.

**8. Jubiler** — loot to biżuteria średniej/wysokiej jakości (złoto, diamenty). Wartość łączna 1200–4500$ w lombardzie. Sprzedaż w lombardzie lub dark_lombardzie (×0.95).

**10. Lombard** — gotówka 600–1800$ + biżuteria 800–2500$ wartości skupu. Część itemów sprzedaje się tylko w dark_lombardzie.

**11. Jacht** — między Fleeca a Pacific. Wymaga łodzi/transportu. Długi cooldown.

**13. Humane Labs** — między Pacific a Cayo pod wymaganiami; wymaga thermite + hacking.

**14. Cayo Perico** — endgame. Wysoki koszt wejścia, długi cooldown, 5–8 osób, wymaga pełnego wyposażenia.

### Loot na osobę — przykłady grupowe

**Kasetka (250–600$):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 250–600$ | 250–600$ |
| 2 | ×1.15 | 287–690$ | 143–345$ |

**Sejf (800–2200$):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 1 | ×1.0 | 800–2200$ | 800–2200$ |
| 2 | ×1.2 | 960–2640$ | 480–1320$ |
| 3 | ×1.35 | 1080–2970$ | 360–990$ |

**Fleeca (6000–16000$, min. 2 os.):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 2 | ×1.0 | 6000–16000$ | 3000–8000$ |
| 3 | ×1.2 | 7200–19200$ | 2400–6400$ |
| 4 | ×1.35 | 8100–21600$ | 2025–5400$ |

**Pacific (35000–90000$, min. 4 os.):**

| Os. | Mnożnik | Łącznie | Na osobę |
|---:|---:|---:|---:|
| 4 | ×1.0 | 35000–90000$ | 8750–22500$ |
| 5 | ×1.2 | 42000–108000$ | 8400–21600$ |
| 6 | ×1.35 | 47250–121500$ | 7875–20250$ |

**Cayo Perico (70000–130000$, min. 5 os.):**

| Os. | Mnożnik | Łącznie | Na osobę (brutto) |
|---:|---:|---:|---:|
| 5 | ×1.0 | 70000–130000$ | 14000–26000$ |
| 6 | ×1.15 | 80500–149500$ | 13416–24916$ |
| 8 | ×1.35 | 94500–175500$ | 11812–21937$ |

---

## Biżuteria i lombard

Napady na NPC, jubilera i lombard dają biżuterię do sprzedaży. Ceny skupu docelowe (obecne wartości są ~8–10× za wysokie):

| Item | Skup docelowy | Jakość / źródło |
|---|---:|---|
| silver_ring | 15–25$ | NPC |
| earrings | 12–20$ | NPC |
| bracelet | 18–30$ | NPC |
| wallet | 15–25$ | NPC |
| watch | 30–55$ | NPC / lombard |
| gold_ring | 45–75$ | jubiler / lombard |
| gold_chain | 55–90$ | jubiler |
| necklace_silver | 35–60$ | jubiler |
| necklace_gold | 55–90$ | jubiler |
| earrings_gold | 40–70$ | jubiler |
| earrings_diamond | 80–130$ | jubiler |
| necklace_emerald | 90–150$ | jubiler |
| diamond | 100–160$ | jubiler (rzadki) |
| designer_bag | 70–120$ | jubiler / lombard |

Dark lombard (Lester): **×0.95** wartości skupu. Telefony i elektronika z NPC idą do dark_lombardu po obniżonych stawkach (telefon 40–70$, smartwatch 35–60$).

---

## Narzędzia do napadów

Obecne ceny hackingdevice (4000–5000$) są za wysokie względem zarobków. Docelowo narzędzia mają być dostępne po kilku godzinach side joba, nie po tygodniu.

| Item | Cena docelowa | Używane w | Obecnie |
|---|---:|---|---:|
| Lockpick | 80–200$ | kasetka, NPC | 1200$ |
| Hackingdevice | **400–800$** | ATM hack, Fleeca | 4000–5000$ |
| Hackingtablet | **800–1500$** | Pacific, Humane | — |
| Thermite | **400–600$** | Pacific, Humane, Cayo | 2000$ |
| Wiertło / inne | 300–700$ | Fleeca | — |

Przykład progresji: 3 h side joba (1500$) → lockpick + hackingdevice → pierwsze napady ATM/kasetka.

---

## Pralnia pieniędzy

| Typ | Strata | Prędkość / uwagi |
|---|---:|---|
| Automatyczna | 35–45% | 25–75$/s (obecnie 175$/s) |
| Ręczna trasa | 20–30% | 3000–7500$/punkt, wypłata po kursie |

Przykład: Fleeca 6000$ dirty → ręczne pranie 25% straty → **4500$ czystego**.

---

## Narkotyki — pełna tabela

Źródła produkcji: `rage_drugs`, KQ weed, inne. Docelowy zarobek: **1500–2500$/h** (wolniejsze, powtarzalne crime — poniżej napadów).

| Item | Ilość / trans. | Szansa | Cena obecna | **Cena docelowa** |
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

Szacunek: ~8–12 udanych transakcji/h × ~100$ średnio = **800–1200$/h** czystej sprzedaży + koszt produkcji i czasu → realnie **1500–2500$/h** przy pełnym loopie.

---

## Koszty życia i usługi

| Rzecz | Docelowo | Obecnie |
|---|---:|---:|
| Woda | 5–10$ | 20$ |
| Proste jedzenie | 8–20$ | 36$ |
| Lepsze jedzenie / restauracja | 25–80$ | — |
| Telefon | 500–1000$ | 1250–1500$ |
| Radio | 300–700$ | 850$ |
| Lockpick | 80–200$ | 1200$ |
| Fixkit | 250–600$ | 5000$ |
| Odholowanie | 300–700$ | 800$ |
| Impound specjalny | 1500–3000$ | 5000$ |
| Lokalny medyk | 500–1500$ | 50–10000$ |
| Lokalny mechanik | 500–1500$ | 1000–5000$ |
| Naprawa z garażu | 500–1500$ | 1500$ |
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

## Mieszkania (qs-housing)

Osobna iteracja po głównym przeskalowaniu. Raty i czynsze muszą być spójne z bazą 500$/h — inaczej mieszkania staną się tańsze niż dobre auto.

---

## Kolejność wdrożenia

1. Paycheck + zasiłek 60$/h + warunek on duty  
2. Prace dorywcze (KQ priorytet)  
3. Mandaty MDT + usługi EMS/DOJ  
4. Premie tygodniowe bossmenu  
5. Napady aktywne + narzędzia crime  
6. Pralnia  
7. Narkotyki  
8. Sklepy, usługi, bronie  
9. Pojazdy  
10. Nowe napady (boosting, tracker, jubiler, lombard, jacht, humane, cayo)  
11. qs-housing  

---

## Decyzje — podsumowanie

| Temat | Status |
|---|---|
| Zasiłek 60$/h | ✅ |
| Model wypłat: aktywny + premia tygodniowa | ✅ |
| DOJ wyższa minutówka + podatki na konto society | ✅ |
| Obniżone mandaty i usługi | ✅ |
| Obniżone premie tygodniowe (~36 h/tydz.) | ✅ |
| Pełna progresja napadów (14 typów) | ✅ |
| Wszystkie narkotyki w tabeli | ✅ |
| Tańsze narzędzia napadów | ✅ |
| Endgame auta / helikoptery / samoloty | ✅ |
| Mandaty MDT per wykroczenie (szczegóły) | ⏳ wdrożenie |
| qs-housing | ⏳ osobna iteracja |
| Nowe napady (boosting, tracker itd.) | ⏳ implementacja |

---

## Analiza balansu

### Co jest spójne

- **Kotwica 500$/h** (60$ zasiłek + 440$ praca) daje sensowny progres do pierwszego auta w kilka godzin i endgame w **~2400 h** side joba.
- **Hierarchia zarobków** ma sens: zasiłek < side job < firma < frakcja < crime < właściciel firmy.
- **Obniżone mandaty i aktywny zarobek frakcji** (200–600$/h normalnie) eliminują ryzyko 1.2 mln w 10 dni — przy 6 h/d frakcjonariusz zarabia ~200–700$/h realnie, czyli **~43k–151k/tydzień**, nie 1.2 mln.
- **DOJ** ma sensowny model: wyższa minutówka (100–120$/h) + podatki na society + sporadyczne sprawy.
- **Progresja napadów** (14 stopni) daje długi crime endgame — Cayo wymaga grupy 5–8 osób i pełnego wyposażenia.
- **Narzędzia 400–800$** są osiągalne po kilku godzinach, thermite 500$ OK.
- **Narkotyki 1500–2500$/h** — poniżej napadów, nie dominują nad resztą crime.

### Na co uważać

| Ryzyko | Opis | Mitygacja |
|---|---|---|
| KQ nie przeskalowane | Side job nadal 20k+/h | priorytet #2 wdrożenia |
| Gracze 10 h/d | Przy crime 2800$/h → endgame auto w ~43 dni | akceptowalne; Cayo/samolot wymagają grupy |
| Biżuteria + napady chain | NPC → jubiler → lombard w pętli | cooldowny + malejące szanse |
| EMS kursy | Powtarzalne, stabilne — mogą dominować nad LSPD | monitorować; ewentualnie cap dzienny |
| Stare ceny w kodzie | Wszystko jest 5–40× za drogie | wdrożenie musi być **globalne i synchroniczne** |
| qs-housing | Nieprzeskalowane mieszkania mogą być tańsze niż auta | osobna iteracja |