# Návrh systému rezervace zasedaček (Brixa)

Tento dokument představuje ucelený funkční a datový návrh aplikace na základě zadání klientky Kláry Benešové a schválených upřesnění (limit 15 minut, tablety u zasedaček, vyřešení sporných bodů).

---

## 1. Entity a vazby mezi nimi

### Datové entity
* **Místnost (`Room`)**
  * `id`: unikátní identifikátor (6 zasedaček v budově, od budky pro 2 po sál pro 20 lidí)
  * `nazev`: název místnosti
  * `patro`: číslo podlaží
  * `kapacita`: maximální počet osob (2 až 20)
  * `vybaveni`: seznam dostupného vybavení (projektor, obrazovka, tabule)
* **Uživatel (`User`)**
  * `id`: unikátní identifikátor zaměstnance (~60 lidí)
  * `jmeno`: jméno a příjmení
  * `email_teams_id`: firemní e-mail / identifikátor pro MS Teams
  * `role`: `zamestnanec`, `asistentka`, `office_manager` (Klára), `reditel`
* **Rezervace (`Reservation`)**
  * `id`: unikátní identifikátor
  * `mistnost_id`: vazba na rezervovanou místnost
  * `organizator_id`: vazba na uživatele
  * `cas_od`, `cas_do`: plánovaný časový interval
  * `ucel`: předmět jednání
  * `cas_vytvoreni`: automatické systémové časové razítko (needitovatelné)
  * `stav`: `planovana`, `potvrzena_checkin`, `probihajici`, `uvolnena_neucast`, `zrusena_uzivatelem`, `zrusena_spravcem`, `dokoncena`
  * `checkin_cas`: timestamp potvrzení příchodu (čidlem nebo odkliknutím na tabletu)
  * `pravidlo_opakovani_id`: volitelná vazba pro opakované porady
* **Záznam čidla (`SensorReading`)**
  * `id`: unikátní identifikátor odečtu
  * `mistnost_id`: vazba na místnost
  * `cas_mereni`: timestamp odečtu (interval 1× za minutu)
  * `pritomnost`: boolean (detekována přítomnost osob: ano/ne)
  * `odhad_osob`: odhad počtu přítomných osob
  * `co2_ppm`: naměřená koncentrace CO2 v ppm
* **Auditní záznam (`AuditLog`)**
  * `id`: unikátní identifikátor záznamu
  * `rezervace_id`: vazba na rezervaci (záznamy se fyzicky nemažou, auditovatelná historie)
  * `cas_udalosti`: timestamp provedení akce
  * `autor_akce`: identifikátor uživatele nebo `SYSTEM`
  * `typ_udalosti`: `vytvoreno`, `checkin_tablet`, `checkin_cidlo`, `uvolneno_neucast`, `zruseno_autorem`, `zruseno_asistentkou`
  * `duvod`: textové odůvodnění změny pro zpětnou kontrolu

### Vazby
* **Místnost 1 : N Rezervace** (striktní zákaz překryvu aktivních rezervací ani o 1 minutu).
* **Místnost 1 : N Záznam čidla** (minutová telemetrie pro každou zasedačku).
* **Uživatel 1 : N Rezervace** (zaměstnanci tvoří rezervace; asistentky/Klára mají rozšířená práva).
* **Rezervace 1 : N Auditní záznam** (nemazatelná historie pro zpětnou kontrolu Klárou).

---

## 2. Obrazovky systému a akce uživatele

### Obrazovka 1: Přehled dne (Dashboard zasedaček)
* **Co uživatel vidí:**
  * Karty všech 6 zasedaček s aktuálním stavem v reálném čase: **Volno** (zelená), **Obsazeno** (červená), **Čeká na příchod** (oranžová).
  * Informaci, do kdy je místnost volná, nebo kdy končí probíhající schůzka a místnost se uvolní.
  * Aktuální data z čidel: reálná přítomnost, odhad počtu lidí, CO2 (ppm) s barevnou indikací kvality vzduchu.
  * Časovou osu dne s rezervacemi.
* **Co uživatel udělá:**
  * Kliknutím na volnou místnost vytvoří rychlou okamžitou rezervaci (ad-hoc).
  * Filtruje zobrazení podle času a proklikává se na detail zasedačky.

### Obrazovka 2: Vyhledání a nová rezervace
* **Co uživatel vidí:**
  * Filtry: datum, čas od–do, požadovaná kapacita (počet osob), vybavení (projektor, obrazovka, tabule).
  * Seznam vyhovujících volných místností bez kolize.
  * Formulář: výběr místnosti, název/účel porady, volba případného opakování (např. každé pondělí).
* **Co uživatel udělá:**
  * Zadá parametry schůzky, vybere volnou místnost a potvrdí rezervaci (čas vytvoření se uloží automaticky).

### Obrazovka 3: Moje rezervace a auditní správa
* **Co uživatel vidí:**
  * Běžný zaměstnanec: své aktivní a minulé rezervace včetně stavu (potvrzeno, uvolněno pro neúčast).
  * Asistentky / Klára: kompletní přehled všech schůzek v budově včetně autora a historie úprav.
* **Co uživatel udělá:**
  * Zaměstnanec zruší svou vlastní rezervaci, pokud porada odpadá.
  * Asistentka / Klára může zrušit cizí rezervaci (s povinným zadáním důvodu do auditního logu).

### Obrazovka 4: Tablet u zasedačky (Kiosk mód)
* **Co uživatel vidí:**
  * Zjednodušený displej na zdi u dveří konkrétní zasedačky.
  * Velký stav místnosti: VOLNO / OBSAZENO / ČEKÁ NA PŘÍCHOD.
  * Název aktuální schůzky, organizátor a odpočet 15 minut pro check-in.
  * Výstrahu při špatném vzduchu (> 1 000 ppm CO2: „Prosím vyvětrejte“).
* **Co uživatel udělá:**
  * Tlačítko **„Potvrdit příchod (Check-in)“**: potvrdí zahájení schůzky.
  * Tlačítko **„Ukončit poradu dříve“**: okamžitě uvolní zasedačku pro ostatní.
  * Tlačítko **„Rychlá rezervace na 15/30 min“**, pokud je zasedačka volná.

### Obrazovka 5: Statistiky a vytížení (Report pro vedení)
* **Co uživatel vidí:**
  * Měsíční souhrn vytížení místností a podíl uvolněných rezervací („duchové“).
  * Týmové přehledy blokování místností (prevence neoprávněného blokování).
* **Co uživatel udělá:**
  * Klára a ředitel filtrují data po měsících a exportují podklady pro vyhodnocení.

---

## 3. Chytrá pravidla (Událost – Podmínka – Akce)

### Pravidlo 1: Uvolnění při neúčasti („duchové“ po 15 minutách)
* **Událost:** Uplynutí 15 minut od plánovaného začátku rezervace ($T_{od} + 15\text{ min}$).
* **Podmínka:** Rezervace nebyla potvrzena tlačítkem na tabletu **A ZÁROVEŇ** čidlo v místnosti nezaznamenalo žádnou přítomnost osob po celých 15 minut.
* **Akce:** Přepnout stav rezervace na `uvolnena_neucast`, zkrátit blokaci k času uvolnění (místnost se ihned stává volnou v systému i na tabletu), zapsat změnu do `AuditLog` a odeslat notifikaci organizátorovi a do kanálu MS Teams (`#zasedacky-volne`).

### Pravidlo 2: Automatický check-in čidlem (ochrana před zapomnětlivostí)
* **Událost:** Přijetí telemetrie z čidla v intervalu $T_{od}$ až $T_{od} + 15\text{ min}$.
* **Podmínka:** Rezervace je ve stavu `planovana` a čidlo detekuje přítomnost osob v místnosti.
* **Akce:** Automaticky potvrdit příchod (stav `potvrzena_checkin`), nastavit `checkin_cas`, zapsat do `AuditLog` a na tabletu přepnout zobrazení na probíhající schůzku.

### Pravidlo 3: Varování před vydýchaným vzduchem (CO2 > 1 000 ppm)
* **Událost:** Přijetí telemetrie z čidla (každou minutu).
* **Podmínka:** Koncentrace CO2 překročí 1 000 ppm **A ZÁROVEŇ** je v místnosti detekována přítomnost osob (nebo schůzka trvá déle než 2 hodiny).
* **Akce:** Zobrazit žlutou/červenou výstrahu na tabletu u dveří („Vysoká hladina CO2 – prosím vyvětrejte“) a odeslat upozornění organizátorovi schůzky přes MS Teams webhook.

### Pravidlo 4: Detekce výpadku čidla (Fail-safe ochrana)
* **Událost:** Pravidelná systémová kontrola telemetrie (každých 5 minut).
* **Podmínka:** Z čidla dané místnosti nedorazila žádná zpráva déle než 5 minut.
* **Akce:** Nastavit stav senzoru na `offline`, odeslat technickou notifikaci Kláře a **pozastavit** automatické rušení rezervací dle čidla (rezervace zůstává v platnosti, aby lidé nebyli vyhozeni kvůli závadě hardware).

### Pravidlo 5: Striktní zákaz překryvu rezervací
* **Událost:** Odeslání požadavku na novou nebo změněnou rezervaci.
* **Podmínka:** Existuje jiná platná rezervace stejné místnosti s překryvem $\ge 1\text{ minuta}$.
* **Akce:** Transakci zamítnout, zobrazit kolidující čas a nabídnout alternativní místnost/čas.

---

## 4. Vyřešené body zadání a architektonická rozhodnutí

Na základě zadání klienta a schválených parametrů byly otevřené body rozhodnuty takto:

1. **Lhůta neúčasti:** Stanovena přesně na **15 minut** od plánovaného začátku.
2. **Kombinace tablet + čidlo:** Primární je pohodlí uživatelů. Příchod lze odkliknout na tabletu, ale pokud lidé zapomenou a vejdou, **čidlo přítomnosti provede check-in automaticky**. Pokud po 15 min není odkliknuto a čidlo nikoho nevidí, místnost se uvolní.
3. **Notifikace a MS Teams:** Při uvolnění pro neúčast dostane organizátor zprávu přes Teams webhook. Zároveň se pošle echo do kanálu `#zasedacky-volne`, aby lidé čekající v kuchyňce věděli o volném prostoru.
4. **Oprávnění k rušení:** Běžný zaměstnanec ruší pouze své rezervace. Asistentky ředitele a Klára mají právo zrušit i cizí rezervaci (např. prioritní jednání vedení); každé takové zrušení vyžaduje zadání důvodu a trvale se zapíše do auditního logu.
5. **Statistiky blokování vs. soukromí:** Kompromis mezi ředitelem a Klárou: Statistiky primárně sledují celofiremní a týmové metriky (podíl propadlých schůzek a nevyužitých hodin). Jmenný přehled „hříšníků“ je neveřejný, přístupný pouze Kláře a řediteli v rámci auditního dohledu pro řešení chronických blokací.
6. **Opakované porady:** Podpora týdenních cyklů. Pokud v sérii vznikne kolize v některém týdnu, systém zarezervuje všechny volné termíny a uživateli nabídne náhradní řešení pro kolizní dny.
7. **Výpadek čidla:** Při výpadku se rezervace neruší – systém preferuje ochranu probíhajícího jednání před technickým omylem.
