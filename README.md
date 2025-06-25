# ecommerce-portal

Frontend aplikacji e-commerce zbudowany z wykorzystaniem Vue 3, Vuetify, Axios i Pinia.

## Stos technologiczny

- **Vue.js 3** — nowoczesny framework frontendowy, szybki i modularny.
- **Vuetify 3** — gotowy framework front-endowy. Wybrałem ze względu na Vue.
- **Pinia** — prosty, nowoczesny i typowany menedżer stanu dla Vue. Skorzystałem, by wyświetlić ładowanie.
- **Axios** — klient HTTP do komunikacji z backendem.
- **Vite** — ultraszybki bundler do projektów frontendowych.

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

## Założenia i pytania

- Zakładamy, że użytkownik i pracownik jest już zalogowany (brak systemu autoryzacji na froncie).
- Wszystkie dane pobierane są z lokalnego API (brak zewnętrznych źródeł danych).

### Pytania do menedżera produktu

- Czy produkty mają różne inne warianty do obsługi? Nad jakimi zmianami będziemy pracować?
- Jakie metody płatności mają być obsługiwane?
- Czy konieczne jest wdrożenie obsługi zwrotów?
- Jakie raporty powinny być dostępne dla administratorów?
