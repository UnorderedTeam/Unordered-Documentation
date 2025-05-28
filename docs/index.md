# Unordered – dokumentacja aplikacji

## Opis aplikacji

**Unordered** to nowoczesna aplikacja mobilna, która pomaga użytkownikom w szybkim i trafnym wyborze prezentów na różne okazje,
takie jak urodziny, rocznice czy święta. Głównym celem aplikacji jest automatyzacja procesu doboru prezentów oraz
przypominanie o ważnych datach, eliminując problem braku pomysłów, czasu czy zapominania o okazjach.

Użytkownik może tworzyć profile osób, dla których planuje prezenty, określając ich zainteresowania i preferencje.
Następnie, dzięki integracji z AI (OpenAI GPT), aplikacja generuje spersonalizowane propozycje prezentów (tzw. boxy),
dostosowane do profilu, okazji oraz budżetu. Możliwe jest także ustawienie cyklicznych boxów, które automatycznie
przypominają i generują prezenty na wybrane daty.

Unordered wyróżnia się pełną automatyzacją - od analizy profilu, przez dobór produktów z wielu platform zakupowych,
aż po przypomnienia o nadchodzących okazjach. Dzięki temu użytkownik nie musi samodzielnie przeszukiwać sklepów ani
pamiętać o terminach - aplikacja zrobi to za niego, zapewniając jednocześnie element zaskoczenia i personalizacji.

Aplikacja jest skierowana zarówno do osób prywatnych, jak i firm, które chcą zadbać o swoich pracowników
czy bliskich, oferując im wyjątkowe, dopasowane prezenty bez zbędnego wysiłku.

---

## Wyjaśnienie pojęć

- **Box**  
    Spersonalizowany zestaw prezentowy, generowany automatycznie na podstawie profilu osoby obdarowywanej, okazji, budżetu lub wyboru losowego. Box zawiera propozycje konkretnych produktów, które razem tworzą gotowy do wręczenia prezent. Każdy box jest unikalny i dopasowany do wybranych przez użytkownika kryteriów.

- **AI**  
    Sztuczna inteligencja, odnosi się do wykorzystania modeli językowych OpenAI (np. o4-mini) do generowania spersonalizowanych propozycji prezentów na podstawie kryteriów podanych przez użytkownika.

---

## Architektura systemu

- **Frontend:** Flutter (Android/iOS)
- **Backend:** Firebase (Cloud Functions, Firestore, Authentication)
- **AI:** OpenAI GPT (generowanie propozycji prezentów)
- **Automatyzacja:** Cloud Functions Scheduler (cykliczne generowanie boxów)

---

## Dokumentacja szczegółowa

- [Dokumentacja Frontend (Flutter)](flutter/overview.md)
- [Dokumentacja Backend (Firebase)](gcloud/overview.md)
- [Dokumentacja bazy danych (Firestore)](firestore/overview.md)

---

## Znane ograniczenia

- Limity i koszty związane z wykorzystaniem API OpenAI (np. limity tokenów, opłaty za zapytania)
- Limity Firestore (np. liczba dokumentów, limity odczytów/zapisów)
- Brak pełnej automatyzacji zamówień fizycznych – aplikacja generuje propozycje, ale nie realizuje zamówień
- Zależność od dostępności zewnętrznych platform zakupowych
- Brak implementacji manualnego wyboru produktów przez użytkownika (na obecnym etapie)

---

## Roadmap i rozwój

- Integracja z dodatkowymi platformami zakupowymi (szerszy wybór produktów)
- Współpraca z markami i sklepami (dedykowane boxy, oferty specjalne)
- System subskrypcji (cykliczne boxy, automatyczne przypomnienia)
- Integracja z kalendarzami zewnętrznymi (np. Google Calendar)
- Usprawnienie procesu zamawiania produktów (np. automatyczne przekierowanie do sklepu)
- Rozszerzenie funkcjonalności AI (lepsze dopasowanie prezentów do profilu)
- Możliwość współpracy z influencerami i partnerami
- Rozbudowa panelu administracyjnego i statystyk

---

## Autorzy

- Adam Dybcio
- Łukasz Czapski
- Igor Ciżewski
- Denis Jabłoński
- Mateusz Sztankiewicz