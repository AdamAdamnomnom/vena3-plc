# Dokumentacja API Biblioteki VENA PLC (`venaPilot3.h`)

Biblioteka `Vena` implementuje koncepcje znane z przemysłowych sterowników programowalnych (PLC) w języku **C++** dla środowiska Arduino / AVR (ATmega328) oraz modułów ESP8266.

---

## Spis Treści
1. [Inicjalizacja i Bezpieczeństwo](#inicjalizacja-i-bezpieczeństwo)
2. [Obsługa Wejść / Wyjść (I/O)](#obsługa-wejść--wyjść-io)
3. [Operacje Logiczne](#operacje-logiczne)
4. [Markery i Pamięć](#markery-i-pamięć)
5. [Liczniki (Counters)](#liczniki-counters)
6. [Timery i Generatory Impulsów](#timery-i-generatory-impulsów)
7. [Komunikacja Międzysterownikowa](#komunikacja-międzysterownikowa)

---

## Inicjalizacja i Bezpieczeństwo

### `void initialize()`
* **Opis:** Konfiguruje piny I/O mikrokontrolera, inicjalizuje pamięć EEPROM, uruchamia magistralę szeregową oraz konfiguruje przerwanie sprzętowe przycisku bezpieczeństwa.
* **Wymagania:** Musi zostać wywołana w sekcji `setup()` programu.

### `static void setStopFunction(void (*stop)())`
* **Parametry:** `void (*stop)()` – wskaźnik na funkcję obsługującą zatrzymanie awaryjne.
* **Opis:** Rejestruje procedurę bezpieczeństwa wywoływaną natychmiast po wyzwoleniu przerwania sprzętowego (przycisk E-STOP).

### `void reset()`
* **Opis:** Wykonuje programowy restart/reset sterownika do stanu początkowego.

---

## Obsługa Wejść / Wyjść (I/O)

### `int check(int index, String impuls = "")`
* **Parametry:** 
  * `int index` — numer/indeks pinu wejściowego.
  * `String impuls` *(opcjonalne)* — przekazanie `"impuls"` aktywuje wykrywanie nowego zbocza sygnału.
* **Zwraca:** `1` jeśli wykryto stan wysoki/nowy impuls, `0` w przeciwnym razie, `-1` w przypadku błędu indeksu.

### `void out(int index, bool val, unsigned long duration = 0)`
* **Parametry:**
  * `int index` — numer pinu wyjściowego.
  * `bool val` — stan (`HIGH`/`LOW`, `true`/`false`).
  * `unsigned long duration` *(opcjonalne)* — czas w ms, przez który stan ma być utrzymywany.

### `void showInOut(bool inputs, bool outputs)`
* **Opis:** Wypisuje aktualny stan pinów wejściowych i/lub wyjściowych na porcie szeregowym (ułatwia diagnostykę i debugowanie maszyny).

---

## Operacje Logiczne

Biblioteka oferuje komplet funkcji logicznych do budowania drabinek/sekwencji sterowania:

| Funkcja | Argumenty | Opis |
|---|---|---|
| `l_and(x, y)` | `bool, bool` | Iloczyn logiczny (AND) |
| `l_or(x, y)` | `bool, bool` | Suma logiczna (OR) |
| `l_nand(x, y)` | `bool, bool` | Negacja iloczynu (NAND) |
| `l_nor(x, y)` | `bool, bool` | Negacja sumy (NOR) |
| `l_xor(x, y)` | `bool, bool` | Różnica symetryczna (XOR) |
| `l_xnor(x, y)` | `bool, bool` | Równoważność logiczna (XNOR) |
| `l_not(x)` | `bool` | Negacja logiczna (NOT) |

---

## Markery i Pamięć

Markery odpowiadają pamięci wewnętrznej PLC (flagi stanu).

### `void setMarker(String name, int value)`
* **Opis:** Zapisuje wartość markera o podanej nazwie (np. `"M001"`).

### `int getMarker(const char* name)`
* **Opis:** Zwraca bieżącą wartość przypisaną do markera.

---

## Liczniki (Counters)

Moduł liczników do zliczania cykli pracy, powtórzeń sekwencji i elementów produkcyjnych.

| Funkcja | Parametry | Zwraca | Opis |
|---|---|---|---|
| `setCounter` | `String name, int value` | `void` | Ustawia wartość początkową/docelową licznika |
| `getCounter` | `const char* name` | `long int` | Odczytuje bieżącą wartość licznika |
| `addToCounter` | `const char* name, int value` | `void` | Inkrementuje licznik o zadaną wartość |
| `checkCounter` | `const char* name` | `bool` | Sprawdza zgodność stanu licznika z limitem |
| `resetCounter` | `const char* name` | `void` | Zeruje licznik |

---

## Timery i Generatory Impulsów

Nieblokujące odmierzanie czasu w pętli sterowania bez zatrzymywania wykonywania programu (unikanie funkcji `delay()`).

### `void startTimer(String name, unsigned long duration)`
* **Opis:** Rozpoczyna odliczanie czasu `duration` (w milisekundach) dla timera o podanej nazwie (np. `"T001"`).

### `bool checkTimer(String name, unsigned long duration = 0)`
* **Opis:** Zwraca `true`, gdy zdefiniowany czas timera upłynął.

### `bool generateImpulse(String name, unsigned long czas1, unsigned long czas2, bool warunek)`
* **Opis:** Generator fali prostokątnej/impulsów o zadanym czasie trwania stanu wysokiego (`czas1`) i niskiego (`czas2`), aktywny gdy spełniony jest `warunek`.

---

## Komunikacja Międzysterownikowa

Obsługa wymiany ramek danych i kodów rozkazowych pomiędzy dwoma jednostkami sterującymi po magistrali UART (SoftwareSerial):

* `void sendMsg(const char* message)` — wysyła kod rozkazu lub meldunek do drugiego modułu.
* `String readMsg()` — odbiera i przetwarza nadesłaną ramkę.
