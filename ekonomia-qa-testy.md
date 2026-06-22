# RageCity — QA ekonomii v2

Instrukcja dla testerów. Serwer testowy, pełny deploy v2 (bez częściowych zmian).

---

## Zanim zaczniesz

1. Postać startowa v2: **2500$ bank + 1500$ gotówka**. Bez admin giveaway.
2. Zapisz **$ na start** przed testem.
3. **Stoper** od pierwszej akcji w instrukcji (nie od dojazdu / brania duty).
4. **Kaucje** pojazdów jobów — zwrot nie licz się do zarobku.
5. Nie dokończony test → zgłoś **SKIP**, nie PASS/FAIL.

**$/h** licz tak: `(konto_koniec − konto_start − koszty) ÷ minuty × 60`

---

## Co wysłać po teście

Skopiuj i uzupełnij:

```
ID:
Tester:
PASS / FAIL / SKIP:
Czas (min):
$ start → koniec (netto):
$/h:                    (jeśli dotyczy)
$ za akcję:             (napady / questy — jeśli dotyczy)
Uwagi:
```

**PASS** = wynik w normie · **FAIL** = poza normą · **SKIP** = nie ukończono

---

## Kolejność (ważne najpierw)

`K1` `K2` `K3` `K5` `K8` · `B8` `B9` `A10` · `A7` `A8` · `C2` `C3` · `D6` `D7` `D12` · `E3` `E5` `E10` `E12` · `G1` `G2` · reszta

---

## A — Prace dorywcze

**Wspólna zasada (A1–A4):** 2 serie. W każdej serii: zbierz **30 szt.** surowca → przetwórz **wszystko** → sprzedaj **wszystko**. Stoper na obie serie.

### A1 — Górnik [P1]

**Zrób:** Kamień (30) → przesiewanie → wydobycie rud → skup wszystkich rud. ×2 serie.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **500–700 $/h**

---

### A2 — Drwal [P1]

**Zrób:** Drewno (30) → przeróbka → pakowanie (max paczek) → sprzedaż desek. ×2 serie.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **500–650 $/h**

---

### A3 — Krawiec [P2]

**Zrób:** Wełna (30) → tkanina → ubrania (max) → sprzedaż. ×2 serie.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **450–550 $/h**

---

### A4 — Rzeźnik [P2]

**Zrób:** Kurczaki (30) → przeróbka → pakowanie → sprzedaż. ×2 serie.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **450–600 $/h**

---

### A5 — Myśliwy [P1]

**Zrób:** Kurs u Mike Meat · **10 zwierząt** (łup) · po każdej 5 sprzedaj wszystko w skupie (2 serie po 5).

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **450–600 $/h**

---

### A6 — Plażowy sprzedawca [P2]

**Zrób:** Duty · stragan · **20 udanych sprzedaży** do NPC.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **350–500 $/h**

---

### A7 — KQ Deliveries [P0]

**Zrób:** Solo · **2 ukończone kontrakty** (od przyjęcia do wypłaty).

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **450–650 $/h**

---

### A8 — KQ Powerwashing [P0]

**Zrób:** **2 ukończone kontrakty** (od startu do wypłaty).

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **500–750 $/h**

---

### A9 — KQ Trucker [P1]

**Zrób:** **1 pełna trasa** + druga trasa do połowy (albo 2 pełne trasy).

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **600–850 $/h**

---

### A10 — AFK side job [P0]

**Zrób:** On duty · stój w miejscu **30 min** (2× wypłata). Potem off duty · **30 min**.

**Raport:** ile $ dostałeś na duty / off duty · PASS/FAIL

**Norma:** on duty **≤60 $/h** · off duty **0$** (lub sam zasiłek)

---

## B — Paycheck i frakcje

### B1 — LSPD spokojna [P1]

**Zrób:** On duty · **30 min** · bez mandatów i dispatch.

**Raport:** min · $ netto · $/h · PASS/FAIL — **Norma: 650–750 $/h**

### B2 — LSPD aktywna [P1]

**Zrób:** On duty · **20 min** · reaguj na dispatch.

**Raport:** min · $ netto · $/h · PASS/FAIL — **Norma: 780–980 $/h**

### B3 — Mandaty LSPD [P2]

**Zrób:** Wystaw **3 drobne mandaty** · policz ile $ trafiło do Ciebie vs minutówka.

**Raport:** % z mandatów · PASS/FAIL — **Norma: mandaty ≤30% łącznego $**

### B4 — EMS spokojna [P1]

**Zrób:** On duty · **30 min** · bez akcji.

**Raport:** $/h — **Norma: 460–500 $/h**

### B5 — EMS aktywna [P1]

**Zrób:** On duty · **20 min** · max revive + kursy NPC.

**Raport:** $/h — **Norma: 1000–1500 $/h**

### B6 — DOJ spokojna [P2]

**Zrób:** On duty · **30 min** · bez spraw.

**Raport:** $/h — **Norma: 580–700 $/h**

### B7 — Mechanik LSC [P1]

**Zrób:** On duty · **30 min** · bez usług MDT.

**Raport:** $/h — **Norma: 650–700 $/h**

### B8 — AFK frakcja [P0]

**Zrób:** On duty · AFK · **30 min**.

**Raport:** PASS/FAIL — **Norma: 0$ z duty**

### B9 — Off duty frakcja [P0]

**Zrób:** Off duty · **30 min**.

**Raport:** PASS/FAIL — **Norma: zasiłek ~60 $/h lub 0**

### B10 — Firma c_* [P1]

**Zrób:** On duty w firmie · **30 min** · bez kursów.

**Raport:** $/h — **Norma: 40–60 $/h**

### B11 — Cap mandatu FP [P1]

**Zrób:** Jeden wyrok **fine 5000$+** · sprawdź ile dostał wystawiający.

**Raport:** kwota udziału — **Norma: ≤750$**

### B12 — Ulgi MDT [P0]

**Zrób:** W MDT szukaj ulg **−5000 / −10000 / −25000**.

**Raport:** PASS/FAIL — **Norma: takich pozycji nie ma**

---

## C — Firmy

### C1 — Kurs Burger Shot [P1]

**Zrób:** **1 kurs** · 10 paczek · zapisz wypłatę pracownika.

**Raport:** $ pracownika — **Norma: 800–1500$**

### C2 — CD kursu [P0]

**Zrób:** Zrób kurs · od razu drugi.

**Raport:** PASS/FAIL — **Norma: 2. kurs zablokowany (30–45 min)**

### C3 — Deposit produktów [P0]

**Zrób:** Craft 10× cheeseburger · `depositProducts` · sprawdź wpływ na society.

**Raport:** $ society — **Norma: ~29$ / szt. (34×0,85), nie ×1,5 menu**

### C4 — Sprzedaż klientowi [P2]

**Zrób:** **3 sprzedaże** z menu BS.

**Raport:** PASS/FAIL — **Norma: firma nie traci na każdej sprzedaży**

### C5 — Właściciel [P1]

**Zrób:** 2 pracowników + właściciel · **20 min** aktywnej zmiany.

**Raport:** $/h właściciela — **Norma: 900–1400 $/h**

### C6 — Cap premii [P1]

**Zrób:** Bossmenu · **2× premia 10k** w tym samym tygodniu.

**Raport:** PASS/FAIL — **Norma: 2. odrzucona**

---

## D — Napady

### D1 — NPC [P1]

**Zrób:** **5 napadów** na NPC (CD 3 min) · skup biżuterii.

**Raport:** min · $ netto · $/h — **Norma: 400–700 $/h**

### D2 — ATM karta [P2]

**Zrób:** **5 wypłat** kartą z bankomatów.

**Raport:** $/h — **Norma: 600–1200 $/h**

### D3 — ATM hack [P2]

**Zrób:** **1 hack** · pranie.

**Raport:** $ po praniu — **Norma: 200–700$**

### D4 — Kasetka [P1]

**Zrób:** **1 kasetka** solo.

**Raport:** $ brutto dirty — **Norma: 250–600$**

### D5 — Sejf sklepu [P1]

**Zrób:** **1 sejf** (lokalizacja bez bonusu Sandy).

**Raport:** $ brutto — **Norma: 800–2200$**

### D6 — Fleeca [P0]

**Zrób:** **2 osoby** · **1 pełny napad** · pranie · $ na osobę.

**Raport:** $/os. po praniu — **Norma: 2250–6000$**

### D7 — CD Fleeca [P0]

**Zrób:** Drugi napad **przed końcem CD**.

**Raport:** PASS/FAIL — **Norma: zablokowany, 1 wypłata**

### D8 — Tracker [P1]

**Zrób:** **1 tracker** (tier średni).

**Raport:** $ black — **Norma: 500–1200$**

### D9 — Boosting [P1]

**Zrób:** **1 boosting** (tier średni).

**Raport:** $ clean — **Norma: 350–900$**

### D10 — Trucker crime [P2]

**Zrób:** **1 dostawa** ciężarówki.

**Raport:** $ brutto — **Norma: 3500–8000$**

### D11 — Pacific [P1]

**Zrób:** **4+ osób** · **1 napad** · pranie.

**Raport:** $/os. — **Norma: 15–35k$**

### D12 — CD napadu [P0]

**Zrób:** Ten sam typ napadu **3× pod rząd** bez czekania.

**Raport:** PASS/FAIL — **Norma: tylko 1 wypłata**

---

## E — Narkotyki

**Sprzedaż uliczna** (E1–E5, E9) — gotowy towar w ekwipunku.  
**Produkcja** (E6, E10–E12) — stoper od pierwszego zbierania / sadzenia; w raporcie podaj też **czas samej produkcji** (bez dojazdów).

### E1 — Weed ulica [P1]

**Zrób:** **8 udanych dealów** (joint / weed_packed).

**Raport:** $/h brutto — **Norma: 800–1200 $/h**

### E2 — Hard ulica [P1]

**Zrób:** **8 dealów** (coke / meth).

**Raport:** $/h brutto — **Norma: 1000–1500 $/h**

### E3 — Ulica max tempo [P0]

**Zrób:** **12 dealów** jak najszybciej.

**Raport:** $/h brutto — **Norma: ≤2200 $/h**

### E4 — Tempo dealów [P1]

**Zrób:** **10 prób** dealów pod rząd · mierz odstępy.

**Raport:** PASS/FAIL — **Norma: ≥20 s między udanymi** (jeśli wdrożone)

### E5 — Wymóg PD [P0]

**Zrób:** Spróbuj sprzedać przy **<2 PD** online.

**Raport:** PASS/FAIL — **Norma: zablokowane**

### E6 — KQ weed produkcja [P1]

**Zrób:** **1 pełny cykl** OG Kush (sadzonka → zbiór → bag/joint) → sprzedaj cały towar ulicą.

**Raport:** min całkowite · min produkcji · $ netto · $/h · PASS/FAIL

**Norma:** cykl produkcji **≥28 min** · **700–1000 $/h** (wyżej niż side job ~500–700)

### E7 — Weed + ulica [P0]

**Zrób:** Zapas z KQ + **6 dealów** ulicznych.

**Raport:** $/h łącznie — **Norma: ≤2200 $/h**

### E8 — Rico brick [P1]

**Zrób:** **1 skup** brick · próba drugiego w 24h.

**Raport:** kwoty — **Norma: 50–85k$ · 2. zablokowany**

### E9 — Jakość [P2]

**Zrób:** Ten sam item · deal **100%** vs **50%** jakości.

**Raport:** stosunek cen — **Norma: wyższa ≤2×**

### E10 — Weed pole [P1]

**Zrób:** **2 serie.** W każdej: zbierz **10× weed_plant** → przetwórz na stole → spakuj **weed_packed** (5 weed + woreczek) → sprzedaj całość ulicą.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **700–1000 $/h** · nie szybciej niż **~20 min / seria** produkcji

### E11 — Koka pole [P1]

**Zrób:** **1 seria:** zbierz **10× coke_leaf** → przetwórz (woreczki) → **8 dealów** coke_packed.

**Raport:** min · $ netto · $/h · PASS/FAIL

**Norma:** **850–1200 $/h** · cykl zbierania+craft **≥35 min**

### E12 — Meta lab [P1]

**Zrób:** **1 pełny cykl:** zbierz aceton (pole) + lit + methlab → **1× cook** w journey → sprzedaj całą metę ulicą.

**Raport:** min cyklu · szt. meth_packed · $ netto · $/h · PASS/FAIL

**Norma:** cykl **≥25 min** · **900–1400 $/h** · yield **3–8** szt. / cook

---

## F — Pralnia

### F1 — Automat [P1]

**Zrób:** Włóż **10k** black_money.

**Raport:** ile dostałeś — **Norma: 55–65%**

### F2 — Ręczna [P1]

**Zrób:** **1 pełny kurs** (max punktów wg v2).

**Raport:** $ czyste po stracie — **Norma: ≤6750$ · strata 20–30%**

### F3 — CD ręczna [P1]

**Zrób:** Drugi kurs zaraz po pierwszym.

**Raport:** PASS/FAIL — **Norma: 2. zablokowany**

### F4 — Bez black [P0]

**Zrób:** Start bez black_money · uruchom pralnię.

**Raport:** PASS/FAIL — **Norma: brak $ z niczego**

---

## G — Hazard

### G1 — Max stawka [P0]

**Zrób:** Ustaw **max stawkę** w ruletce / slocie.

**Raport:** kwota — **Norma: ≤10 000$**

### G2 — Zdrapka Deluxe [P0]

**Zrób:** **5 zdrapek** Deluxe.

**Raport:** max wygrana — **Norma: ≤8000$**

### G3 — Kasyno [P1]

**Zrób:** **20 spinów** po 5k · zapisz obrót i wynik.

**Raport:** PASS/FAIL — **Norma: strata 10–30% obrotu**

### G4 — Konie [P1]

**Zrób:** **30 zakładów** po 1k (lub log z serwera).

**Raport:** PASS/FAIL — **Norma: kasyno na plus (gracz traci)**

### G5 — Koło fortuny [P2]

**Zrób:** **5 spinów**.

**Raport:** max nagroda — **Norma: ≤50k$**

### G6 — Kręgle [P2]

**Zrób:** **1 gra** z opłatą.

**Raport:** PASS/FAIL — **Norma: ~40$ opłaty, brak wypłaty**

---

## H — Koszty życia

### H1 — Jedzenie minimum [P1]

**Zrób:** Kup i zużyj: **4 wody + 2 kanapki + 1 chips** (jak w grze 1h).

**Raport:** koszt $ — **Norma: ~45$ / h ekwiwalent**

### H2 — Koszty łącznie [P1]

**Zrób:** Loadout H1 + **~5 km** jazdy + **1 myjnia**.

**Raport:** koszt $ — **Norma: ~80–115$ / h przy 500$/h dochodu**

### H3 — Pierwsze auto [P2]

**Zrób:** Od startu v2 · **2 serie** side jobu (np. A1).

**Raport:** $ zarobione łącznie — **Norma: ≥1500$**

### H4 — Odsprzedaż [P2]

**Zrób:** Kup auto za **10k** · sprzedaj dealerowi.

**Raport:** kwota zwrotu — **Norma: 6500$ (65%)**

### H5 — Kredyt mieszkania [P1]

**Zrób:** **1 cykl** spłaty kredytu (admin może przyspieszyć).

**Raport:** % spłaty — **Norma: 6% salda**

### H6 — Czynsz [P2] · po starcie

**Zrób:** Najemca na willi 700k · **14 dni** RL.

**Raport:** $ właściciela — **Norma: 7000$**

### H7 — Pasywny wynajem [P2] · po starcie

**Zrób:** Wiele mieszkań na wynajmie · **30 dni**.

**Raport:** $/h ekwiwalent — **Norma: ≤150 $/h**

---

## I — Crime mix

### I1 — Typowy [P0]

**Zrób:** **3× NPC** + **2× ATM karta** + **1× kasetka** + **4× deal** · pranie.

**Raport:** $/h po praniu — **Norma: 700–1200 $/h**

### I2 — Aktywny [P0]

**Zrób:** **1× sejf** + **1× hack ATM** + **6× deal** + **1× boosting** · pranie.

**Raport:** $/h — **Norma: 1800–2800 $/h**

### I3 — Dominacja źródła [P1]

**Zrób:** Z raportu **I2** — z którego źródła było najwięcej $?

**Raport:** % — **Norma: jedno źródło <50%**

### I4 — Ekipa Fleeca [P0]

**Zrób:** **4 osoby:** **1× Fleeca** + **3× NPC** + **4× deal** na osobę.

**Raport:** $/h/os. — **Norma: ≤2800 $/h**

### I5 — Wejście w crime [P2]

**Zrób:** **1 seria** side jobu → kup lockpick + hack → **1× kasetka**.

**Raport:** PASS/FAIL — **Norma: łup kasetki ≥ koszt narzędzi**

---

## J — Questy Moris

### J1 — Fernando zad. 2 [P1]

**Zrób:** Tutorial pralni do końca.

**Raport:** $ clean + dirty — **Norma: 1500–2500 clean + 4–6k dirty**

### J2 — Carlos boosting [P1]

**Zrób:** **1 boosting** niski tier.

**Raport:** $ clean — **Norma: 350–900$**

### J3 — Carlos tracker [P1]

**Zrób:** **1 tracker** średni tier.

**Raport:** $ black — **Norma: 500–1200$**

### J4 — Moris haracz [P2]

**Zrób:** Zadanie 1 pełny flow.

**Raport:** bonus quest — **Norma: 300–600$** (+ loot kasetki osobno)

### J5 — Po questach [P1]

**Zrób:** Po ukończeniu linii · spróbuj powtórzyć Carla boosting.

**Raport:** PASS/FAIL — **Norma: brak lub kwota jak napad, nie >1500$ sam quest**

---

## K — Anti-exploit

### K1 — Deploy [P0]

**Zrób:** Lead potwierdza pełny deploy §0.

**Raport:** PASS/FAIL

### K2 — Duplikat wypłaty [P0]

**Zrób:** Reconnect w trakcie wypłaty z napadu.

**Raport:** PASS/FAIL — **Norma: tylko 1 wypłata**

### K3 — Deposit cheat [P0]

**Zrób:** `depositProducts` **bez itemów** w ekwipunku.

**Raport:** PASS/FAIL — **Norma: odrzucone**

### K4 — Stack wyroków [P1]

**Zrób:** **10 drobnych** pozycji w **1 wyroku**.

**Raport:** udział FP — **Norma: ≤750$**

### K5 — Paycheck off duty [P0]

**Zrób:** Firma `c_*` · off duty · **30 min**.

**Raport:** PASS/FAIL — **Norma: 0$**

### K6 — Pacific loot [P1]

**Zrób:** Ukończ Pacific.

**Raport:** PASS/FAIL — **Norma: jest wypłata lootu**

### K7 — Sandy sejf [P1]

**Zrób:** Napad sejf **Sandy**.

**Raport:** PASS/FAIL — **Norma: bez +5000$ bonusu**

### K8 — NPC CD [P0]

**Zrób:** **2 napady NPC** w ciągu **<3 min**.

**Raport:** PASS/FAIL — **Norma: 2. zablokowany**

---

*Szczegóły dla leada: [`ekonomia-qa-testy-dev.md`](ekonomia-qa-testy-dev.md)*
