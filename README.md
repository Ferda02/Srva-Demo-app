# Brixa – Rezervace zasedacích místností

Webový systém pro správu a rezervaci 6 zasedacích místností firmy Brixa (cca 60 zaměstnanců). Řeší problém blokovaných prázdných místností („duchů“) automatickým uvolňováním rezervací při neúčasti a monitoruje kvalitu vzduchu (CO2) pomocí IoT čidel.

## Jak aplikaci spustit (krok za krokem)

1. Příprava prostředí (Python 3.10+):
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r requirements.txt   # (plánované)
   ```
2. Spuštění aplikace (jedním příkazem):
   ```bash
   python run.py                     # (plánované)
   ```
3. Otevření v prohlížeči: `http://localhost:5000` *(plánované)*

## Struktura složek

* `web/` – prezentační vrstva: Flask routes, šablony a statické soubory *(plánované)*
* `logika/` – aplikační vrstva: chytrá pravidla (15min limit, CO2), validace bez překryvu *(plánované)*
* `uloziste/` – datová vrstva: přístup k SQLite přes vestavěný modul `sqlite3` *(plánované)*
* `docs/` – projektová dokumentace a funkční specifikace (`NAVRH.md`)
* `GEMINI.md` – závazná pravidla vývoje projektu

## Rozhodnutí klienta

* **20. 10. 2026**:
  * Lhůta pro uvolnění rezervace při neúčasti („duchové“) je stanovena přesně na **15 minut**.
  * Zavedení **tabletů u dveří zasedaček** (kiosk displej) pro check-in a uvolnění v součinnosti s čidly přítomnosti.
  * Zbývající provozní parametry (notifikace do MS Teams, role asistentek, kompromis v reportech a CO2 varování) svěřeny k řešení dle schváleného návrhu.

## Otevřené otázky

* Fyzická specifikace tabletů u dveří (typ zařízení, OS a způsob zajištění kiosk režimu).
* Konkrétní protokol a formát pro příjem telemetrie z čidel v ostrém provozu (REST endpoint vs. MQTT).
* Způsob autentizace uživatelů (napojení na firemní Microsoft 365 / SSO vs. interní správa účtů).

## Použité nástroje AI

* **Antigravity IDE** (Google DeepMind): asistent pro agentní vývoj, model Gemini 3.8 Flash.
* Využití: rozbor zadání klienta, identifikace sporných bodů, návrh třívrstvé architektury a dokumentace.
