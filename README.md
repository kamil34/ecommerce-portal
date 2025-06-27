# ecommerce-portal

Frontend aplikacji e-commerce zbudowany z wykorzystaniem Vue 3, Vuetify, Axios i Pinia.

## Stos technologiczny

- **Vue.js 3** — nowoczesny framework frontendowy, szybki i modularny.
- **Vuetify 3** — gotowy framework front-endowy. Wybrałem ze względu na Vue.
- **Pinia** — prosty, nowoczesny i typowany menedżer stanu dla Vue. Skorzystałem, by wyświetlić ładowanie.
- **Axios** — klient HTTP do komunikacji z backendem.
- **Vite** — ultraszybki bundler do projektów frontendowych.
- **Vue Router** — oficjalny router do Vue. Używany do tworzenia widoków i nawigacji po aplikacji.

## Instrukcja uruchomienia

1. **Zainstaluj zależności**  
   ```bash
   npm install
   ```

2. **Uruchom projekt w trybie deweloperskim**  
   ```bash
   npm run dev
   ```

3. Aplikacja będzie dostępna pod adresem:  
   [http://localhost:3000](http://localhost:3000)

> Upewnij się, że backend (`ecommerce-backend`) działa pod adresem `http://localhost:3001`.

## Struktura danych (frontend)

Frontend komunikuje się z backendem za pomocą API i prezentuje dane takie jak:

- **Produkty** (lista, szczegóły)
- **Zamówienia użytkownika**
- **Panel dostawcy**

Poniżej krótki opis najważniejszych tabel w bazie danych, które są używane w aplikacji.

---

### orders – Zamówienia

Tutaj zapisujemy każde zamówienie.

- `orderID` – unikalny identyfikator zamówienia (klucz główny)
- `orderDate` – data złożenia zamówienia
- `completed` – czy zamówienie zostało ukończone (0 lub 1)

---

### order_items – Pozycje zamówienia

Tutaj przechowujemy, jakie produkty i w jakiej ilości zostały zamówione.

- `orderID` – ID zamówienia
- `productID` – ID produktu
- `quantity` – ilość zamówionego produktu

---

### shipments – Dostawy

Rejestrujemy tu każdą wysyłkę produktu w ramach zamówienia.

- `shipmentID` – unikalne ID wysyłki
- `orderID` – ID zamówienia
- `productID` – ID produktu
- `quantity` – ilość wysłana
- `confirmed` – czy klient potwierdził dostawę (0 lub 1)
- `confirmed_at` – kiedy klient potwierdził dostawę
- `created_at` – kiedy zgłoszono wysyłkę

---

### order_changes – Historia zmian

Tu zapisujemy wszystkie zmiany w zamówieniu (np. edycje, komentarze).

- `changeID` – ID zmiany
- `orderID` – ID zamówienia
- `changeUser` – kto wprowadził zmianę
- `changeText` – opis zmiany
- `changeDate` – data zmiany

---

### products – Produkty

Lista produktów dostępnych do zamówienia.

- `id` – ID produktu
- `title` – nazwa produktu
- `description` – opis
- `image` – ścieżka do obrazka (opcjonalnie)
- `unit` – jednostka miary (powiązana z tabelą `units`)
- `available` – ile sztuk jest dostępnych

---

### units – Jednostki miary

Tutaj trzymamy jednostki typu szt, kg, l itd.

- `id` – ID jednostki
- `symbol` – skrót jednostki (np. "szt")

> Połączone są odpowiednimi relacjami.

## Założenia i uproszczenia

- Zakładamy, że użytkownik i pracownik jest już zalogowany (brak systemu autoryzacji na froncie i w endpointach na backendzie). W dalszych etapach wprowadziłbym system logowania wraz z tokenami, na przykład JWT z Authorization Bearer.
- Zamówienia wyświetlono wszystkie. Z poziomu konta dostawcy jak i klienta ze względu na brak funkcji logowania.
- Wdrożyłbym obsługę Web Socket dla dynamicznego pobierania danych z API, które klient by widział na bieżąco po tym zabiegu.
- Znalazłbym opcję na zmienne kierujące do serwera Backend, typu '.env', bądź w oddzielnym komponencie na FrontEndzie, by uniknąć ciągłego wypisywania linku do API.
- Wszystkie dane pobierane są z lokalnego API (brak zewnętrznych źródeł danych).
- Koszyk dla zapełnienia nawigacji i dla przyszłego rozbudowania strony.
- Footer nie jest używany. Jest on domyślnie wygenerowany przez framework Vuetify.

### Pytania do menedżera produktu

- Czy produkty będą miały inne warianty do obsługi? Nad jakimi zmianami będziemy pracować?
- Jakie metody płatności mają być obsługiwane?
- Czy będziemy wdrażać obsługę zwrotów?
