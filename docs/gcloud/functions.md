# Dokumentacja funkcji chmurowych (Cloud Functions)

## Wprowadzenie

W tej sekcji opisano funkcje chmurowe wykorzystywane w backendzie aplikacji Unordered. Funkcje te realizują logikę biznesową, komunikację z zewnętrznymi API oraz obsługę żądań z aplikacji mobilnej.

## Inżynieria promptów
Przy inżynierii promptów uwaga była skupiona na wykorzystaniu najmniejszej liczby tokenów przy zachowaniu jak najlepszej jakości odpowiedzi.  
Struktura wszystkich promptów to:  

**ZASADA** → **OPIS ZASADY**

Wiele opisów dotyczących jednej zasady jest oddzielonych przecinkiem.
Koniec zasady jest oznaczony średnikiem.

## Wybór modelu OpenAI

Do generowania rekomendacji produktów wybrano model **OpenAI o4-mini**. Model ten działa w trybie „reasoning”, co pozwala na uwzględnienie wszystkich parametrów przekazanych przez użytkownika podczas generowania odpowiedzi. Dodatkowo, **OpenAI o4-mini** jest rozwiązaniem bardziej ekonomicznym w porównaniu do modeli **GPT-4.1** oraz **OpenAI o3**, zapewniając jednocześnie wysoką jakość generowanych wyników.

## Moduł `matching.js`

Moduł `matching.js` odpowiada za logikę tworzenia boxów dla użytkowników, zarówno na podstawie profilu, jak i w trybie losowym. Zawiera także funkcje do generowania codziennych boxów.

### Funkcja `createBox`

Tworzy box na podstawie wybranego profilu użytkownika, budżetu i okazji. Wykorzystuje OpenAI do doboru produktów.

Sprawdza, czy użytkownik jest zalogowany i posiada odpowiednie uprawnienia do wywołania funkcji.
```javascript
if (!request.auth) {
  throw new functions.https.HttpsError(
      "unauthenticated",
      "The function must be called while authenticated.",
  );
}
// ...pobranie danych profilu i produktów...
```
Tworzy prompt na podstawie pobranych danych, który zostanie przesłany do modelu OpenAI.
```javascript
const prompt = "Profile: " + JSON.stringify(profileData) +
              "\nBudget: " + JSON.stringify(budget) +
              "\nOccasion: " + JSON.stringify(occasion) +
              "\nProducts: "+ JSON.stringify(productsData);
```
Wysyła prompt i token autoryzacyjny użytkownika do [funkcji openAIResponse](#funkcja-openairesponse), która generuje rekomendacje produktów.
```javascript
const openaiResponse = await axios.post(
    "<link do cloud funkcji openAIResponse>",
    { prompt },
    { headers: { Authorization: `Bearer ${request.auth.token}` } }
);
```
Przetwarza odpowiedź z OpenAI, zapisuje nowy box w bazie danych i zwraca identyfikator boxa oraz status operacji.
```javascript
// ...obsługa odpowiedzi...
await boxRef.set({
  products: selectedProducts,
  createdInMillis: createdInMillis,
  isPeriodical,
  plannedDate: plannedDate || null,
  price,
  cycleFreq: cycleFreq || null,
  budget: budget || null,
  occasion: occasion || null,
  profileId: profileRef.id,
});

return {boxId: boxRef.id, success: true};
});
```

---

### Funkcja `createRandomBox`

Tworzy box z losowo dobranymi produktami (bez profilu i okazji).

Sprawdza, czy użytkownik jest zalogowany.
```javascript
if (!request.auth) {
  throw new functions.https.HttpsError(
      "unauthenticated",
      "The function must be called while authenticated.",
  );
}
// ...pobranie produktów...
```
Tworzy prompt zawierający tylko listę produktów.
```javascript
const prompt = "Products: "+ JSON.stringify(productsData);
```
Wysyła prompt i token autoryzacyjny użytkownika do [funkcji randomOpenAIResponse](#funkcja-randomopenairesponse), która generuje losowe rekomendacje produktów.
```javascript
const openaiResponse = await axios.post(
    "<link do cloud funkcji randomOpenAIResponse>",
    { prompt },
    { headers: { Authorization: `Bearer ${request.auth.token}` } }
);
```
Przetwarza odpowiedź i zapisuje nowy box z losowymi produktami.
```javascript
// ...obsługa odpowiedzi...
await boxRef.set({
  products: selectedProducts,
  createdInMillis: createdInMillis,
  isPeriodical: false,
  plannedDate: null,
  price,
  cycleFreq: null,
  budget: null,
  occasion: null,
  profileId: null,
});

return {boxId: boxRef.id, success: true};
});
```

---

### Funkcja `createDailyBoxes`

Tworzy codziennie zestaw boxów tematycznych (wykonywana cyklicznie przez scheduler).

Funkcja uruchamiana automatycznie każdego dnia o 02:00 (UTC).
```javascript
exports.createDailyBoxes = onSchedule("every day 02:00", async (event) => {
  // ...losowanie tematów i budżetów...
```
Dla każdego z 5 tematów generuje prompt i wywołuje [funkcję dailyOpenAIResponse](#funkcja-dailyopenairesponse) do wygenerowania rekomendacji produktów.
```javascript
for (let i=0; i<5; i++) {
  // ...generowanie promptu i wywołanie OpenAI...
  await dailyRef.add(boxData);
}
});
```

---

## Moduł `openai.js`

Moduł `openai.js` odpowiada za komunikację z API OpenAI oraz generowanie rekomendacji produktów na podstawie promptów przesyłanych z aplikacji lub innych funkcji backendu. Zawiera funkcje obsługujące różne scenariusze generowania boxów.

### Funkcja `openAIResponse`

Wywołuje model OpenAI na podstawie promptu z danymi profilu, budżetu i okazji. Zwraca listę wybranych produktów w formacie JSON.

Sprawdza autoryzację oraz obecność promptu w żądaniu.
```javascript
// ...autoryzacja...
const {prompt} = req.body;
if (!prompt) {
  res.status(400).send("Prompt is required");
  return;
}
```
Tworzy zapytanie do modelu OpenAI z określonymi zasadami doboru produktów.

Struktura promptu jest opisana w sekcji [Inżynieria promptów](#inzynieria-promptow).

Szczegóły związane z wyborem modelu OpenAI i jego właściwościami są opisane w sekcji [Wybór modelu OpenAI](#wybor-modelu-openai).

```javascript
const completion = await openai.chat.completions.create({
  model: "o4-mini-2025-04-16",
  messages: [
    {
      role: "system",
      content: `Zasady:
ANALIZA->Uwzględnij zainteresowania, okazję, i budżet,
    Ignoruj niepowiązane kategorie,
    jeśli brak pasujących produktów, wybierz 1 uniwersalny;

RÓŻNORODNOŚĆ->Unikaj powtórzeń, dobieraj różne kategorie;

BUDŻET->Nigdy nie przekraczaj budżetu!, Nie schodź znacznie poniżej,
    Dany budżet dotyczy SUMY produktów, nie pojedynczego!;

ODPOWIEDŹ->Zwróć TYLKO JSON array z ID produktów,
    bez komentarzy, lista niepusta;`,
    },
    {
      role: "user",
      content: `Zwróć TYLKO JSON array z wybranymi ID produktów.
      Dane wejściowe: ${prompt}`,
    },
  ],
  response_format: {type: "json_object"},
});
```
Zwraca odpowiedź z modelu OpenAI do klienta.
```javascript
const responseText = completion.choices[0].message.content;
res.status(200).json({response: responseText});
// ...obsługa błędów...
```

---

### Funkcja `dailyOpenAIResponse`

Generuje rekomendacje produktów dla codziennych boxów tematycznych.

Sprawdza autoryzację oraz obecność promptu w żądaniu.
```javascript
try {
  const {prompt} = req.body;
  if (!prompt) {
    res.status(400).send("Prompt is required");
    return;
  }
```
Tworzy zapytanie do modelu OpenAI z zasadami doboru produktów uwzględniającymi tematykę boxa.

Struktura promptu jest opisana w sekcji [Inżynieria promptów](#inzynieria-promptow).

Szczegóły związane z wyborem modelu OpenAI i jego właściwościami są opisane w sekcji [Wybór modelu OpenAI](#wybor-modelu-openai).

```javascript
const completion = await openai.chat.completions.create({
  model: "o4-mini-2025-04-16",
  messages: [
    {
      role: "system",
      content: `Zasady:
DOBÓR->Uwzględnij temat, unikaj innych kategorii,
    Wybierz 1 uniwersalny, jeśli brak pasujących produktów;

RÓŻNORODNOŚĆ->Unikaj powtórzeń!, dobieraj różne kategorie;

BUDŻET->Nigdy nie przekraczaj budżetu!, Nie schodź znacznie poniżej,
    Dany budżet dotyczy SUMY produktów, nie pojedynczego!;

ODPOWIEDŹ->TYLKO JSON array z ID produktów,
    bez komentarzy, lista niepusta;`,
    },
    {
      role: "user",
      content: `Zwróć TYLKO JSON array z wybranymi ID produktów.
      Dane wejściowe: ${prompt}`,
    },
  ],
  response_format: {type: "json_object"},
});
```
Zwraca odpowiedź z modelu OpenAI do klienta.
```javascript
const responseText = completion.choices[0].message.content;
res.status(200).json({response: responseText});
} catch (error) {
  // ...obsługa błędów...
}
});
```

---

### Funkcja `randomOpenAIResponse`

Generuje losowe rekomendacje produktów do boxa bez profilu i okazji.

Sprawdza autoryzację oraz obecność promptu w żądaniu.
```javascript
try {
  // ...autoryzacja...
  const {prompt} = req.body;
  if (!prompt) {
    res.status(400).send("Prompt is required");
    return;
  }
```
Tworzy zapytanie do modelu OpenAI z zasadami doboru produktów w trybie losowym.

Struktura promptu jest opisana w sekcji [Inżynieria promptów](#inzynieria-promptow).

Szczegóły związane z wyborem modelu OpenAI i jego właściwościami są opisane w sekcji [Wybór modelu OpenAI](#wybor-modelu-openai).

```javascript
const completion = await openai.chat.completions.create({
  model: "o4-mini-2025-04-16",
  messages: [
    {
      role: "system",
      content: `Zasady:
LOSOWOŚĆ->Wybierz losowe przedmioty, maks. 8,
    Unikaj powtórzeń, lista niepusta;

RÓŻNORODNOŚĆ->Dobieraj różne kategorie;

BUDŻET->Utrzymaj realistyczny budżet;

ODPOWIEDŹ->TYLKO JSON array z ID produktów, bez komentarzy,
    lista niepusta;`,
    },
    {
      role: "user",
      content: `Zwróć TYLKO JSON array z wybranymi ID produktów.
      Dane wejściowe: ${prompt}`,
    },
  ],
  response_format: {type: "json_object"},
});
```
Zwraca odpowiedź z modelu OpenAI do klienta.
```javascript
const responseText = completion.choices[0].message.content;
res.status(200).json({response: responseText});
} catch (error) {
  // ...obsługa błędów...
}
});
```

---

## Moduł `products.js`

Moduł `products.js` odpowiada za zarządzanie produktami w bazie danych Firestore. Umożliwia pobieranie listy produktów, pobieranie produktu po ID oraz dodawanie nowych produktów (w tym losowych).

### Funkcja `getProducts`

Zwraca listę wszystkich produktów zapisanych w kolekcji `products`.

Pobiera referencję do kolekcji produktów.
```javascript
const productsRef = admin.firestore().collection("products");
```
Pobiera wszystkie dokumenty z kolekcji, mapuje je na tablicę obiektów z polem `id` i danymi produktu.
```javascript
productsRef.get()
    .then((snapshot) => {
      const products = [];
      snapshot.forEach((doc) => {
        products.push({
          id: doc.id,
          ...doc.data(),
        });
      });
      res.status(200).json(products);
    })
```
Obsługuje błędy podczas pobierania produktów.
```javascript
.catch((error) => {
  console.error("Error fetching products:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `getProductById`

Zwraca pojedynczy produkt na podstawie przekazanego ID.

Pobiera identyfikator produktu z zapytania i sprawdza jego obecność.
```javascript
const productId = req.query.id;

if (!productId) {
  res.status(400).send("Product ID is required");
  return;
}
```
Pobiera referencję do dokumentu produktu o podanym ID.
```javascript
const productRef = admin.firestore().collection("products").doc(productId);
```
Pobiera dokument produktu i zwraca go w odpowiedzi, jeśli istnieje.
```javascript
productRef.get()
    .then((doc) => {
      if (!doc.exists) {
        res.status(404).send("Product not found");
      } else {
        res.status(200).json({
          id: doc.id,
          ...doc.data(),
        });
      }
    })
```
Obsługuje błędy podczas pobierania produktu.
```javascript
.catch((error) => {
  console.error("Error fetching product:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `createProduct`

Dodaje nowy produkt do kolekcji `products` na podstawie danych przesłanych w body żądania.

Pobiera dane produktu z ciała żądania i sprawdza ich obecność.
```javascript
const product = req.body;

if (!product) {
  res.status(400).send("Product is required");
  return;
}
```
Dodaje nowy dokument do kolekcji produktów.
```javascript
const productsRef = admin.firestore().collection("products");

productsRef.add(product)
    .then((doc) => {
      res.status(201).json({
        id: doc.id,
        ...product,
      });
    })
```
Obsługuje błędy podczas dodawania produktu.
```javascript
.catch((error) => {
  console.error("Error creating product:", error);
  res.status(500).send("Internal Server Error");
});
```

---

## Moduł `profiles.js`

Moduł `profiles.js` odpowiada za zarządzanie profilami użytkowników w bazie danych Firestore. Umożliwia pobieranie wszystkich profili użytkownika oraz pobieranie pojedynczego profilu po ID.

### Funkcja `getProfiles`

Zwraca listę wszystkich profili użytkownika na podstawie przekazanego `userId`.

Pobiera identyfikator użytkownika z zapytania i sprawdza jego obecność.
```javascript
const usersRef = admin.firestore().collection("users");
const userId = req.query.userId;

if (!userId) {
  res.status(400).send("User ID is required");
  return;
}
```
Pobiera wszystkie profile użytkownika z podkolekcji `profiles`.
```javascript
usersRef.doc(userId).collection("profiles").get()
    .then((snapshot) => {
      const profiles = [];
      snapshot.forEach((doc) => {
        profiles.push({
          id: doc.id,
          ...doc.data(),
        });
      });
      res.status(200).json(profiles);
    })
```
Obsługuje błędy podczas pobierania profili.
```javascript
.catch((error) => {
  console.error("Error fetching profiles:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `getProfileById`

Zwraca pojedynczy profil użytkownika na podstawie przekazanych `userId` i `profileId`.

Pobiera identyfikatory użytkownika i profilu z zapytania oraz sprawdza ich obecność.
```javascript
const userId = req.query.userId;
const profileId = req.query.profileId;

if (!userId) {
  res.status(400).send("User ID is required");
  return;
}

if (!profileId) {
  res.status(400).send("Profile ID is required");
  return;
}
```
Pobiera referencję do dokumentu profilu użytkownika.
```javascript
const profileRef = admin.firestore().collection("users")
    .doc(userId).collection("profiles").doc(profileId);
```
Pobiera dokument profilu i zwraca go w odpowiedzi, jeśli istnieje.
```javascript
profileRef.get()
    .then((doc) => {
      if (!doc.exists) {
        res.status(404).send("Profile not found");
      } else {
        res.status(200).json({
          id: doc.id,
          ...doc.data(),
        });
      }
    })
```
Obsługuje błędy podczas pobierania profilu.
```javascript
.catch((error) => {
  console.error("Error fetching profile:", error);
  res.status(500).send("Internal Server Error");
});
```

---

## Moduł `stores.js`

Moduł `stores.js` odpowiada za zarządzanie sklepami w bazie danych Firestore. Umożliwia pobieranie listy sklepów, pobieranie sklepu po ID lub nazwie oraz dodawanie nowych sklepów (w tym losowych).

### Funkcja `getStores`

Zwraca listę wszystkich sklepów zapisanych w kolekcji `stores`.

Pobiera referencję do kolekcji sklepów.
```javascript
const storesRef = admin.firestore().collection("stores");
```
Pobiera wszystkie dokumenty z kolekcji, mapuje je na tablicę obiektów z polem `id` i danymi sklepu.
```javascript
storesRef.get()
    .then((snapshot) => {
      const stores = [];
      snapshot.forEach((doc) => {
        stores.push({
          id: doc.id,
          ...doc.data(),
        });
      });
      res.status(200).json(stores);
    })
```
Obsługuje błędy podczas pobierania sklepów.
```javascript
.catch((error) => {
  console.error("Error fetching stores:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `getStoreById`

Zwraca pojedynczy sklep na podstawie przekazanego ID.

Pobiera identyfikator sklepu z zapytania i sprawdza jego obecność.
```javascript
const storeId = req.query.id;

if (!storeId) {
  res.status(400).send("Store ID is required");
  return;
}
```
Pobiera referencję do dokumentu sklepu o podanym ID.
```javascript
const storesRef = admin.firestore().collection("stores").doc(storeId);
```
Pobiera dokument sklepu i zwraca go w odpowiedzi, jeśli istnieje.
```javascript
storesRef.get()
    .then((doc) => {
      if (!doc.exists) {
        res.status(404).send("Store not found");
      } else {
        res.status(200).json({
          id: doc.id,
          ...doc.data(),
        });
      }
    })
```
Obsługuje błędy podczas pobierania sklepu.
```javascript
.catch((error) => {
  console.error("Error fetching store:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `getStoreByName`

Zwraca sklep lub sklepy na podstawie przekazanej nazwy.

Pobiera nazwę sklepu z zapytania i sprawdza jej obecność.
```javascript
const storeName = req.query.name;
if (!storeName) {
  res.status(400).send("Store name is required");
  return;
}
```
Wyszukuje sklepy o podanej nazwie w kolekcji.
```javascript
const storesRef = admin.firestore().collection("stores");
storesRef.where("name", "==", storeName).get()
    .then((snapshot) => {
      const stores = [];
      snapshot.forEach((doc) => {
        stores.push({
          id: doc.id,
          ...doc.data(),
        });
      });
      res.status(200).json(stores);
    })
```
Obsługuje błędy podczas pobierania sklepów.
```javascript
.catch((error) => {
  console.error("Error fetching stores:", error);
  res.status(500).send("Internal Server Error");
});
```

---

### Funkcja `createStore`

Dodaje nowy sklep do kolekcji `stores` na podstawie danych przesłanych w body żądania.

Pobiera dane sklepu z ciała żądania i sprawdza ich obecność.
```javascript
const store = req.body;

if (!store) {
  res.status(400).send("Store is required");
  return;
}
```
Dodaje nowy dokument do kolekcji sklepów.
```javascript
const storeRef = admin.firestore().collection("stores");

storeRef.add(store)
    .then((doc) => {
      res.status(201).json({
        id: doc.id,
        ...store,
      });
    })
```
Obsługuje błędy podczas dodawania sklepu.
```javascript
.catch((error) => {
  console.error("Error creating product:", error);
  res.status(500).send("Internal Server Error");
});
```

---

## Bezpieczeństwo i autoryzacja

Wszystkie funkcje chmurowe w projekcie Unordered zostały zaprojektowane z uwzględnieniem bezpieczeństwa i kontroli dostępu. Poniżej opisano główne mechanizmy zabezpieczeń stosowane w funkcjach backendowych:

- **Weryfikacja autoryzacji użytkownika:**  
  Funkcje, które wykonują operacje na danych użytkownika (np. tworzenie boxów, pobieranie profili, dodawanie produktów), wymagają obecności ważnego tokena autoryzacyjnego. Przed wykonaniem logiki funkcji sprawdzana jest autoryzacja użytkownika, a w przypadku jej braku zwracany jest odpowiedni błąd (np. `unauthenticated` lub kod HTTP 400/401).

- **Walidacja danych wejściowych:**  
  Każda funkcja sprawdza obecność wymaganych parametrów wejściowych (np. `userId`, `profileId`, `productId`, `storeId`). W przypadku braku wymaganych danych funkcja natychmiast przerywa działanie i zwraca błąd.

- **Ograniczenie dostępu do danych:**  
  Funkcje umożliwiają dostęp wyłącznie do danych powiązanych z zalogowanym użytkownikiem (np. profile, boxy). Nie jest możliwe pobranie lub modyfikacja danych innych użytkowników bez odpowiednich uprawnień.

- **Obsługa błędów i logowanie:**  
  Wszystkie funkcje posiadają obsługę błędów oraz logowanie nieudanych operacji, co pozwala na monitorowanie potencjalnych nadużyć i szybkie reagowanie na incydenty.

- **Bezpieczna komunikacja z zewnętrznymi API:**  
  Wywołania do zewnętrznych usług (np. OpenAI) są realizowane z użyciem tokenów autoryzacyjnych użytkownika oraz bezpiecznych protokołów komunikacyjnych.

- **Dodatkowe zabezpieczenia (założenie):**  
  Zakłada się, że w środowisku produkcyjnym funkcje są dodatkowo chronione przez reguły bezpieczeństwa Firestore, ograniczenia sieciowe oraz mechanizmy rate limiting, co uniemożliwia nieautoryzowany dostęp i nadużycia.

---
