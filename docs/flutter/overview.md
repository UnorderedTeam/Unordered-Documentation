# Dokumentacja Frontend (Flutter)

## Wprowadzenie

Unordered-Mobile to aplikacja mobilna napisana w Flutterze, spełniająca funkcję frontendu dla aplikacji Unordered. Umożliwia użytkownikom tworzenie i zarządzanie boxami, przeglądanie historii zamówień, zarządzanie profilami odbiorców oraz korzystanie z powiadomień.

## Wymagania systemowe

- Flutter SDK
- Dart (zalecana najnowsza wersja)
- Android Studio lub Xcode (dla Android/iOS)
- Konto Firebase z odpowiednią konfiguracją projektu

## Instalacja i uruchomienie

1. Przejdź do katalogu projektu.
2. Zainstaluj zależności:  
   `flutter pub get`
3. Skonfiguruj pliki środowiskowe Firebase komendą:
   `flutterfire configure`
4. Uruchom aplikację na emulatorze lub urządzeniu fizycznym:  
   `flutter run`

## Struktura projektu

Główne katalogi i pliki:

- `lib/` – kod źródłowy aplikacji Flutter:
  - `core/` – podstawowe klasy i funkcje
  - `features/` – moduły funkcjonalne (np. boxy, profile, historia)
    - `mystery_box_creator/` – kreator boxów
    - `mystery_box_discover/` – przeglądanie boxów
    - `user_center/` – zarządzanie profilami użytkowników
    - `welcome_new_users/` – ekran powitalny dla nowych użytkowników
  - `custom_navigation.dart` – niestandardowa nawigacja
  - `main.dart` – punkt wejścia aplikacji
  - `firebase_options.dart` – konfiguracja Firebase

## Główne zależności

- **cupertino_icons** – zestaw ikon w stylu iOS
- **decimal** – precyzyjna arytmetyka dziesiętna
- **font_awesome_flutter** – ikony FontAwesome
- **animate_do** – animacje UI
- **google_maps_flutter** – integracja z Mapami Google
- **intl** – obsługa dat i czasu, lokalizacja
- **get** – zarządzanie stanem, routing
- **get_storage** – lokalna baza danych
- **get_it** – wstrzykiwanie zależności (dependency injection)
- **dio** – zapytania HTTP
- **firebase_core** – inicjalizacja Firebase
- **firebase_auth** – uwierzytelnianie użytkowników
- **cloud_firestore** – baza danych Firestore
- **cloud_functions** – wywoływanie funkcji chmurowych Firebase
- **http** – zapytania HTTP
- **rxdart** – programowanie reaktywne, obsługa strumieni
- **lottie** – animacje Lottie (gify)
- **smooth_page_indicator** – wskaźniki stron (np. onboarding)
- **google_fonts** – czcionki Google Fonts
- **timeline_tile** – oś czasu w UI
- **readmore** – rozwijane teksty
- **table_calendar** – kalendarz w UI
- **shimmer** – efekt ładowania shimmer
- **photo_view** – powiększanie zdjęć
- **google_nav_bar** – dolna nawigacja w stylu Google

## Opis funkcjonalności (interfejsy użytkownika)

- **Tworzenie mystery boxów (kreator):**  
  Interfejs umożliwia użytkownikowi utworzenie boxa na podstawie profilu, budżetu i okazji lub wygenerowanie boxa losowego. Kreator prowadzi przez kolejne kroki wyboru.
- **Zarządzanie profilami odbiorców:**  
  Interfejs pozwala na tworzenie, edycję i usuwanie profili odbiorców boxów (np. rodzina, znajomi).
- **Historia zamówień i boxów:**  
  Interfejs umożliwia przeglądanie historii zamówień, szczegółów boxów oraz statusów realizacji.
- **Powiadomienia i promocje:**  
  Interfejs prezentuje powiadomienia o statusie zamówień, promocjach i nowościach.
- **Zarządzanie kontem użytkownika:**  
  Interfejs pozwala użytkownikowi na zarządzanie swoim kontem.

Funkcjonalność oraz widoki poszczególnych modułów są opisane na stronie [Dokumentacja Widgetów Flutter](widgets.md).

---
