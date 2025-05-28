# Dokumentacja bazy danych (Firestore)

## Wprowadzenie

W tej sekcji opisano strukturę bazy danych Firestore dla aplikacji Unordered. Baza danych jest zorganizowana w kolekcje i dokumenty, które przechowują informacje o boxach, zamówieniach, produktach, użytkownikach i innych istotnych danych.

---

## Kolekcje główne

### `daily`
Kolekcja przechowująca codzienne boxy.

- **Dokument:**
    - `budget` – budżet boxa
    - `createdInMillis` – data utworzenia (timestamp w ms)
    - `price` – cena boxa
    - `products` – lista dokumentów [`products`](#products)
    - `theme` – motyw boxa

---

### `orders`
Kolekcja zamówień.

- **Dokument:**
    - `address` – adres dostawy
    - `addressMethod` – sposób dostawy/adresowania
    - `lastUpdate` – data ostatniej aktualizacji
    - `orderedAt` – data złożenia zamówienia
    - `orderedBy` – identyfikator zamawiającego
    - `paymentMethod` – metoda płatności
    - `products` – lista zamówionych produktów, dokumenty [`products`](#products)
    - `status` – status zamówienia
    - `totalPrice` – łączna cena zamówienia

---

### `products`
Kolekcja produktów.

- **Dokument:**
    - `link` – link do produktu
    - `name` – nazwa produktu
    - `price` – cena produktu
    - `prodDesc` – opis produktu
    - `storeId` – identyfikator sklepu

---

### `shared`
Kolekcja współdzielonych boxów.

- **Dokument:**
    - `author` – autor boxa
    - `budget` – budżet boxa
    - `createdInMillis` – data utworzenia (timestamp w ms)
    - `cycleFreq` – częstotliwość cyklu (np. miesięczna)
    - `isPeriodical` – czy box jest cykliczny
    - `occasion` – okazja
    - `plannedDate` – planowana data
    - `price` – cena boxa
    - `products` – lista dokumentów [`products`](#products)

---

### `stores`
Kolekcja sklepów.

- **Dokument:**
    - `link` – link do sklepu
    - `logo` – logo sklepu
    - `name` – nazwa sklepu

---

###  `users`
Kolekcja użytkowników.

- **Dokument:**
  - **Podkolekcja: `boxes`**
    - **Dokument:**
        - `budget` – budżet boxa
        - `createdInMillis` – data utworzenia
        - `cycleFreq` – częstotliwość cyklu
        - `isPeriodical` – czy box jest cykliczny
        - `occasion` – okazja
        - `plannedDate` – planowana data
        - `price` – cena boxa
        - `products` – lista dokumentów [`products`](#products)
        - `profileId` – powiązany dokument [`profiles`](#users)

  - **Podkolekcja: `liked`**
    - **Dokument:**
        - `likeDate` – data polubienia

  - **Podkolekcja: `orders`**
        - lista dokumentów [`orders`](#orders)

  - **Podkolekcja: `profiles`**
    - **Dokument:**
        - `createdInMillis` – data utworzenia profilu
        - `description` – opis profilu
        - `interests` – zainteresowania
        - `name` – nazwa profilu

---

## Uwagi

- Każda kolekcja i dokument może być rozszerzana o dodatkowe pola w zależności od wymagań projektu.
- Struktura pozwala na łatwe zarządzanie boxami, zamówieniami, produktami oraz profilami użytkowników.
- Podkolekcje w `users` umożliwiają przechowywanie danych powiązanych bezpośrednio z użytkownikiem.

---