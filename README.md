# Internetowe Kasyno

> **Motto projektu:** *„Dzisiaj w Betlejem, jutro Bentleyem”* 🚗💨  
> Platforma rozrywkowa online oferująca proste gry losowe i zręcznościowe dla pełnoletnich użytkowników.  
> Każdy znajdzie coś dla siebie — od klasycznej ruletki po dynamiczny coinflip!

---

<p align="center">
  <img src="./design/logo_prototype.jpeg" alt="kasyno logo" width="300">
</p>




## Opis projektu

**Internetowe Kasyno** to aplikacja webowa umożliwiająca użytkownikom grę w różne mini-gry oparte na losowości i zręczności.  
Celem projektu jest stworzenie bezpiecznego, intuicyjnego i atrakcyjnego środowiska do rozgrywki online z zachowaniem zasad fair play.

Projekt ma charakter **edukacyjny i demonstracyjny** – nie służy do prawdziwego hazardu.

---

## Zakres MVP (Minimum Viable Product)


### Rejestracja i logowanie
- Tworzenie nowego konta użytkownika  
- Logowanie z wykorzystaniem hasła (opcjonalnie 2FA)  
- Weryfikacja pełnoletności

### Zarządzanie kontem użytkownika
- Edycja danych profilu  
- Zmiana hasła  
- Podgląd historii gier i transakcji  

### Portfel i saldo użytkownika
- Wirtualna waluta (MufaCOIN)  
- Doładowanie i wypłata środków (symulacja)  
- Historia transakcji  

### Rozgrywka – proste gry
Dostępne mini-gry w wersji MVP:
- 🎲 **Ruletka** – klasyczna europejska ruletka z jednym zerem  
- 💣 **Saper** – gra logiczno-losowa z możliwością wygrania punktów  
- 🎰 **Slot** – automat z symbolami (trzy bębny, system nagród)  
- 🎯 **Suwak** – gra refleksu i precyzji  
- 🪙 **Coinflip** – rzut monetą: orzeł czy reszka?

### Bezpieczeństwo
- Szyfrowanie haseł
- Walidacja danych użytkownika  
- Sesje z tokenami JWT  

### System rankingowy
- Tabela najlepszych graczy (tygodnia?)
- Punkty rankingowe zależne od aktywności i wygranych  
- Odznaki i poziomy prestiżu  

---

## Technologia

| Warstwa | Technologia |
|----------|--------------|
| Frontend | Vue |
| Backend  | Node.js (Express) |
| Baza danych |  PostgreSQL |
| ORM | Prisma |
| Dokumentacja | Swagger UI |


---

# Backend

## Struktura projektu

### Główne katalogi
```
backend/
├── prisma/                  # Konfiguracja bazy danych
│   ├── schema.prisma       # Schema modeli i relacji
│   ├── prismaSingleton.ts  # Singleton klienta Prisma
│   ├── seed.ts            # Podstawowe dane testowe
│   └── seedRich.ts        # Rozszerzone dane testowe
└── src/
    ├── Controllers/        # Logika biznesowa endpointów
    │   ├── authController.ts      # Rejestracja, logowanie
    │   ├── walletController.ts    # Operacje portfela
    │   ├── coinflipController.ts  # Gra Coinflip
    │   ├── rouleteController.ts   # Gra Ruletka
    │   ├── slotsController.ts     # Gra Slots
    │   ├── sliderController.ts    # Gra Slider
    │   ├── sapperController.ts    # Gra Saper
    │   ├── rankingController.ts   # Rankingi graczy
    │   ├── passwordResetController.ts # Reset haseł
    │   └── adminController.ts     # Panel administratora
    ├── Middleware/         # Middleware warstwy zabezpieczeń
    │   ├── authMiddleware.ts      # Weryfikacja JWT tokenów
    │   ├── adminMiddleware.ts     # Sprawdzanie uprawnień admin
    │   └── balanceMiddleware.ts   # Walidacja salda przed grą
    ├── Routes/            # Definicje tras API
    │   ├── userRoutes.ts         # Trasy użytkowników
    │   ├── walletRoutes.ts       # Trasy portfela
    │   ├── gamesRoutes.ts        # Trasy gier
    │   ├── sapperRoutes.ts       # Dedykowane trasy Sapera
    │   ├── rankingRoutes.ts      # Trasy rankingów
    │   └── adminRoutes.ts        # Trasy administratora
    └── index.ts           # Punkt wejściowy serwera
```

## Struktura bazy danych

### Główne modele

#### User
Użytkownicy aplikacji z podstawowymi danymi osobowymi, rolą (NORMAL/ADMIN) i systemem banów.
- **Klucze:** email (unikalny), username
- **Relacje:** 1:1 z Wallet, 1:N z SapperMap i PasswordReset

#### Wallet  
Portfel użytkownika przechowujący saldo w formacie Decimal(10,2).
- **Relacje:** 1:1 z User, 1:N z Transaction

#### Transaction
Historia operacji finansowych (wpłaty, wypłaty, zakłady, wygrane).
- **Typy:** DEPOSIT, WITHDRAWAL, LOST, WIN, BET

#### SapperMap
Aktywna gra Saper użytkownika z planszą zapisaną jako string.
- **Ograniczenie:** jeden aktywny Saper na użytkownika

#### Game
Katalog dostępnych gier w kasynie z możliwością włączania/wyłączania.

#### PasswordReset
Tokeny resetowania haseł z datą wygaśnięcia i indeksami wydajnościowymi.

### Kluczowe cechy
- **UUID** jako ID we wszystkich tabelach
- **Cascade delete** dla powiązanych danych użytkownika
- **Indeksy** na często wyszukiwanych polach
- **Enumy** dla typów transakcji i ról użytkowników

**Uwaga:** Tabela `spatial_ref_sys` to systemowa tabela PostgreSQL.

## Logika gier

### Mechanika i wypłaty

| Gra | RNG | Mechanika | Wypłaty | House Edge |
|-----|-----|-----------|---------|------------|
| **Ruletka** | `Math.random() * 100 % 37` | Liczby 0-36 + kolory (parz./nieparz.) | Liczba: 5x, Kolor: 2x (kumulatywne) | ~13.5% |
| **Coinflip** | `Math.random() * 100 % 2` | 50/50 (parz./nieparz.) | 2x przy wygranej | 0% |
| **Slots** | Weighted random | 9 symboli, 3 bębny, tylko 3x | 🍒🍋: 2x, 🍊: 3x, 🍇: 4-5x, 🍉: 8x, 🔔: 10x, ⭐: 15x, 7️⃣: 50x | ~25% |
| **Slider** | `getRandomInt(0, 100)` | Gracz wybiera zakres [min,max] | `bet * (100/zakres) * 0.98` | 2% |
| **Saper** | - | NxN plansza, progresywny mnożnik | Rośnie z odkrytymi polami | Zmienny |

### Slot Machine - symbole i wagi
| Symbol | Waga | Wypłata (3x) | 
|--------|------|--------------|
| 🍒 CHERRY | 100 | 2x |
| 🍋 LEMON | 100 | 2x |
| 🍊 ORANGE | 90 | 3x |
| 🍇 PLUM | 80 | 4x |
| 🍇 GRAPE | 70 | 5x |
| 🍉 WATERMELON | 60 | 8x |
| 🔔 BELL | 40 | 10x |
| ⭐ STAR | 20 | 15x |
| 7️⃣ SEVEN | 10 | 50x |

### Zabezpieczenia
- **Walidacja:** Wszystkie kontrolery sprawdzają użytkownika
- **Transakcje:** Każda gra → wpis w `Transaction` (WIN/LOST)  
- **Atomowość:** Operacje Prisma w pojedynczych transakcjach
- **Error handling:** Pełne obsługa błędów z kodami HTTP

## Instrukcja uruchomienia
Aby poprawnie uruchomić serwer lokelnie, wykonaj poniższe kroki w głównym katalogu projektu.

### 1. Wymagania systemowe
- Node.js v16.0 lub nowszy
- npm v8.0 lub nowszy  
- PostgreSQL v12 lub nowszy

### 2. Instalacja zależności
    npm install

### 3. Konfiguracja zmiennych środowiskowych (.env)
Utwórz plik .env w głównym katalogu (tam gdzie package.json) i uzupełnij go według wzoru:

    DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public"
    JWT_SECRET="twoj_tajny_klucz_jwt"
    PORT=8000

### 4. Baza danych (Prisma)
Synchronizacja schematu bazy danych (tworzenie tabel) oraz generowanie klienta Prisma.
**Uwaga:** Plik schematu znajduje się w folderze backend/prisma/

Wpisz następujące komendy do terminala:

  #### Generowanie klienta (wymagane przy pierwszym uruchomieniu)
    npx prisma generate --schema ./backend/prisma/schema.prisma

  #### Wypchnięcie zmian do bazy danych
    npx prisma db push --schema ./backend/prisma/schema.prisma

### 5.Uruchomienie serwera
Uruchom backend w trybie developerskim (z automatycznym restartem nodemon):

    npm run backend


## Dokumentacja API
Projekt posiada automatycznie generowaną dokumentację endpointów (Swagger).
Po uruchomieniu serwera wejdź na adres z dopiskiem `/docs` w adresie, np.:

http://localhost:8000/docs

### Opis endpointów

#### Roulette
- `POST /api/games/play-roulette` – Gra w ruletkę

#### Coin Flip
- `POST /api/games/play-coin-flip` – Gra w orzeł czy reszka

#### Slots
- `POST /api/games/play-slots` – Gra na automacie

#### Slider
- `POST /api/games/play-slider` – Gra na suwaku

#### Rankings
- `GET /api/ranking/{period}` – Ranking graczy (wg wygranych)

#### Sapper Game
- `POST /api/Sapper/start-sapper` – Rozpocznij nową grę Saper
- `POST /api/Sapper/play-sapper` – Wykonaj ruch w Saperze
- `POST /api/Sapper/resign-sapper` – Zakończ grę Saper

#### Users
- `GET /api/users/profile` – Pobierz profil użytkownika
- `POST /api/users/register` – Rejestracja
- `POST /api/users/login` – Logowanie
- `GET /api/users/is-authenticated` – Sprawdź autoryzację
- `PATCH /api/users/update-username` – Zmień nazwę użytkownika

#### Wallet
- `GET /api/wallet/get-wallet` – Pobierz portfel
- `POST /api/wallet/deposit` – Dodaj środki do portfela
- `POST /api/wallet/withdraw` – Wypłać środki z portfela

#### General
- `GET /` – Endpoint powitalny

## Bezpieczeństwo

### Autoryzacja JWT
- Tokeny są generowane przy logowaniu
- Middleware `authMiddleware.js` weryfikuje tokeny
- Tokeny zawierają ID użytkownika i czas wygaśnięcia

### Walidacja danych
- Wszystkie dane wejściowe są walidowane
- Użycie `express-validator` do sprawdzania formatów
- Sanityzacja danych przed zapisem do bazy

### Hashowanie haseł
- Hasła są hashowane za pomocą bcrypt
- Salt rounds: 10 (domyślnie)

---

# Frontend

## Architektura aplikacji

Frontend zbudowany w **Vue 3** z **Composition API**, wykorzystujący nowoczesne narzędzia i biblioteki dla optymalnego user experience.


## Struktura projektu

### Główne katalogi
```
frontend/src/
├── components/           # Komponenty wielokrotnego użytku
│   ├── games/           # Modale gier (Slots, Roulette, itp.)
│   ├── layout/          # Layout (Header, Footer)
│   ├── modals/          # Modale systemowe (Login, Wallet)
│   ├── sections/        # Sekcje strony (Hero, Games, Leaderboard)
│   └── ui/             # Komponenty UI (GameCard, LeaderboardRow)
├── views/               # Widoki główne aplikacji
│   └── user/           # Panel użytkownika
├── router/             # Konfiguracja routingu
└── stores/            # Magazyny stanu (Pinia)
```

### Routing i nawigacja
System routingu obsługuje:
- **Publiczne trasy** (`/home`, `/games`, `/leaderboard`)
- **Chronione trasy** (`/panel/*` - wymaga autoryzacji)
- **Trasy administracyjne** (`/admin` - tylko dla adminów)
- **Strony pomocnicze** (`/legal/*`, `/support/*`)

### State Management

#### Auth Store
Centralny magazyn autoryzacji obsługujący:
- **Autentyfikację JWT** - logowanie, wylogowanie, odświeżanie tokenów
- **Dane użytkownika** - profil, rola, uprawnienia
- **Saldo portfela** - aktualne środki, historia transakcji
- **Persystencję** - zapis do `localStorage`

```typescript
// Kluczowe metody auth store
loginSuccess(token, user)    // Logowanie użytkownika
fetchBalance()               // Pobieranie salda z API
updateUsername(newUsername)  // Aktualizacja nazwy użytkownika
logout()                    // Wylogowanie i czyszczenie danych
```

## Komponenty gier

### Modale gier
Każda gra ma dedykowany modal z ujednoliconą strukturą:
- 🎰 **SlotGameModal** - automat z 3 bębnami i 9 symbolami
- 🎲 **RouletteGameModal** - ruletka europejska 0-36
- 🪙 **CoinflipGameModal** - rzut monetą z animacjami
- 🎯 **SliderGameModal** - gra precyzyjna z suwakiem
- 💣 **MinesweeperGameModal** - saper z progresywnym mnożnikiem

### Wspólne cechy modali gier:
- **Walidacja zakładów** - sprawdzanie salda przed grą
- **Animacje rezultatów** - confetti przy wygranych
- **Real-time updates** - natychmiastowa aktualizacja salda
- **Error handling** - obsługa błędów API i sieci

## Interfejs użytkownika

### Design System
- **Motyw** - ciemny z gradientowymi akcentami (fiolet/cyan)
- **Typografia** - Space Grotesk (Google Fonts)
- **Ikony** - Material Symbols Outlined
- **Kolory** - gradientowe tła, transparentne elementy
- **Responsywność** - mobile-first, adaptacyjny layout

### Komponenty UI
- **GameCard** - karty gier z hover effects
- **LeaderboardRow** - wiersze rankingu z kolorowym tłem
- **HeaderComponent** - różne wersje dla auth/unauth
- **FooterComponent** - linki prawne i wsparcie

### Animacje i UX
- **Smooth scrolling** - płynne przewijanie do sekcji
- **Backdrop blur** - rozmycie tła modali
- **Hover states** - interaktywne stany elementów
- **Loading states** - wskaźniki ładowania
- **Canvas confetti** - fajerwerki przy wygranych

## Panel użytkownika

Chroniona sekcja `/panel` z podstronami:
- **Dashboard** - podsumowanie konta i aktywności
- **Profil** - edycja danych osobowych
- **Bezpieczeństwo** - ustawienia zabezpieczeń
- **Hasło** - zmiana hasła
- **Powiadomienia** - centrum powiadomień

## Zabezpieczenia frontend

### Ochrona tras
- **Route guards** - middleware sprawdzający autoryzację
- **Role-based access** - różne uprawnienia (USER/ADMIN)
- **Token validation** - weryfikacja JWT w każdym żądaniu
- **Auto-logout** - wylogowanie po wygaśnięciu tokena

### Walidacja po stronie klienta
- **Input validation** - sprawdzanie formularzy przed wysłaniem
- **Sanityzacja danych** - oczyszczanie danych wejściowych  
- **XSS protection** - ochrona przed atakami skryptowymi
- **CSRF tokens** - (planowane) tokeny antyfałszywościowe

## Instrukcja uruchomienia Frontend

### 1. Wymagania systemowe
- Node.js v20.19+ lub v22.12+
- npm v8.0 lub nowszy
- Nowoczesna przeglądarka (Chrome 90+, Firefox 88+)

### 2. Instalacja zależności
```bash
cd frontend
npm install
```

### 3. Uruchomienie w trybie deweloperskim
```bash
npm run dev
```

### 4. Build produkcyjny
```bash
npm run build      # Budowanie
npm run preview    # Podgląd buildu
```

