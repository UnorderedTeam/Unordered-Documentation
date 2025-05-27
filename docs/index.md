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

- [Frontend (Flutter)](flutter/overview.md)
- [Backend (Cloud Functions)](gcloud/overview.md)

---

## Roadmap i rozwój

- Integracja z dodatkowymi platformami zakupowymi
- Współpraca z markami i sklepami (dedykowane boxy)
- System subskrypcji (cykliczne boxy)
- Integracja z kalendarzami zewnętrznymi
- Usprawnienie procesu zamawiania produktów
- Rozszerzenie funkcjonalności AI

---

## Autorzy

- Adam Dybcio
- Łukasz Czapski
- Igor Ciżewski
- Denis Jabłoński
- Mateusz Sztankiewicz