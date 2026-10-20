# Pravidla vývoje projektu

1. **Stack**: Python 3, Flask, SQLite přes vestavěný modul `sqlite3`; jiné knihovny jen po schválení.
2. **Architektura**: 3 vrstvy ve složkách `web/`, `logika/`, `uloziste/`; vrstva `web/` nevolá SQL přímo.
3. **Spouštění**: Aplikace se spouští jedním příkazem, který je popsaný v `README.md`.
4. **Data a bezpečnost**: Používat pouze syntetická data; databáze a `.env` patří do `.gitignore`.
5. **Plánování**: Před jakoukoliv změnou souborů vždy předlož plán a počkej na schválení.
6. **Dokumentace**: Po každé provedené změně aktualizuj `README.md`.
7. **Git**: Commit prováděj vždy s `-m`, žádné interaktivní příkazy, `push` pouze na výslovný pokyn.
