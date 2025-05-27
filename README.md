# Uruchamianie i budowanie dokumentacji MkDocs

## Wymagania wstępne
- Python 3.7+
- Menedżer pakietów `pip`

## Instalacja

1. Zainstaluj MkDocs:
    ```bash
    pip install mkdocs
    ```

2. Zainstaluj motyw Material:
    ```bash
    pip install mkdocs-material
    ```

## Uruchamianie dokumentacji lokalnie

Uruchom serwer deweloperski:
```bash
mkdocs serve
```
Odwiedź [http://127.0.0.1:8000](http://127.0.0.1:8000) w przeglądarce.

## Budowanie dokumentacji

Wygeneruj statyczne pliki strony:
```bash
mkdocs build
```
Wynik znajdziesz w katalogu `site/`.

## Dodatkowe polecenia

- Wyczyść katalog site:
  ```bash
  mkdocs clean
  ```