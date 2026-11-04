# ⚡ VENA 3 PLC — Przemysłowy Sterownik Programowalny (C++ & Embedded)

[![Language](https://img.shields.io/badge/Language-C%2B%2B%20%2F%20Arduino-00599C?logo=c%2B%2B)](Pilot3.ino)
[![MCU](https://img.shields.io/badge/Hardware-ATmega328%20%7C%20ESP8266-E7352C?logo=arduino)](Dokumentacja_Techniczna_Vena_PLC.pdf)
[![Architecture](https://img.shields.io/badge/Architecture-PLC%20Ladder%20Logic%20in%20C%2B%2B-blue)]()
[![Status](https://img.shields.io/badge/Status-Completed%20%2F%20Archive-success)]()

Dedykowany sterownik automatyki przemysłowej (**PLC** — *Programmable Logic Controller*) zaprojektowany i zbudowany od podstaw na mikrokontrolerach **ATmega328** oraz **ESP8266MOD**. Projekt łączy fizyczny układ elektroniczny (płytki PCB, sterowanie przekaźnikami, wejścia optoizolowane, komunikację radiową) z własną, obiektową biblioteką w **C++**, przenoszącą klasyczne paradygmaty programowania PLC (drabinki logiczne, markery, timery nieblokujące, liczniki cykli) do świata nowoczesnego programowania embedded.

---

## 📸 Zdjęcia i Realizacja Sprzętowa

<p align="center">
  <img src="Zdjecia/IMG20240407170413.jpeg" width="31%" alt="Płytka sterownika VENA PLC" />
  <img src="Zdjecia/IMG20240407170450.jpeg" width="31%" alt="Okablowanie i moduł wykonawczy" />
  <img src="Zdjecia/IMG20240407170546.jpeg" width="31%" alt="Stanowisko testowe i pilot" />
</p>

*Więcej zdjęć z etapów montażu i testów znajduje się w katalogu [`Zdjecia/`](Zdjecia/).*

---

## 🎯 Główne Założenia i Cechy Systemu

* **Podwójny rdzeń obliczeniowo-komunikacyjny:**
  * **ATmega328** odpowiada za deterministyczne operacje czasu rzeczywistego (pętla sterowania maszyną, obsługa wejść/wyjść, generowanie impulsów, liczniki).
  * **ESP8266MOD** zapewnia warstwę łączności bezprzewodowej i integracji ze zdalnymi interfejsami.
* **Obiektowa Biblioteka Sterownika w C++ (`venaPilot3.h`):**
  * **Nieblokująca pętla sterowania:** Zastąpienie blokujących opóźnień (`delay`) wbudowanymi timerami opartymi o `millis()`, co eliminuje ryzyko zawieszenia procesu.
  * **Wykrywanie zboczy i debouncing:** Automatyczna filtracja drgań styków oraz detekcja impulsów na wejściach cyfrowych.
  * **Pamięć nieulotna EEPROM:** Trwałe przechowywanie parametrów kalibracyjnych i stanów markerów po odcięciu zasilania.
  * **Komunikacja UART / SoftwareSerial:** Asynchroniczna wymiana rozkazów i statusów pomiędzy modułami wykonawczymi.
* **Bezpieczeństwo (E-STOP):**
  * Obsługa zatrzymania awaryjnego na dedykowanym **przerwaniu sprzętowym (`attachInterrupt`)** – natychmiastowa reakcja i zrzucenie wyjść bez oczekiwania na zakończenie cyklu pętli `loop()`.

---

## 📊 Specyfikacja Techniczna

| Parametr | Opis |
|---|---|
| **Jednostka główna** | Microchip ATmega328P (16 MHz) |
| **Moduł komunikacyjny** | Espressif ESP8266MOD |
| **Język programowania** | C++11 / Arduino Core |
| **Interfejsy I/O** | Wejścia cyfrowe z filtrem programowym, wyjścia przekaźnikowe/tranzystorowe |
| **Magistrale** | Hardware UART, SoftwareSerial (piny TX/RX), EEPROM 512B |
| **Wspierane operacje PLC** | Bramki logiczne (AND, OR, NAND, NOR, XOR, XNOR, NOT), Markery (M), Liczniki (C), Timery (T), Impulsatory |

---

## 💻 Przykładowy Kod Użycia (`Pilot3.ino`)

Przykładowy fragment implementujący cykliczną sekwencję pracy siłowników pneumatycznych (Y1, Y2, Y3) z wykorzystaniem timerów i liczników:

```cpp
#include "venaPilot3.h"

Vena vena;
int x = 0, y = 0;

void stopHandler() {
    vena.sendMsg("9");  // Rozkaz zatrzymania awaryjnego
    vena.reset();       // Bezpieczny reset sterownika
}

void setup() {
    vena.initialize();
    vena.setStopFunction(stopHandler); // Przerwanie bezpieczeństwa E-STOP
    
    vena.setMarker("M001", LOW);
    vena.setCounter("C001", 3);        // Limit 3 cykli sekwencji
    vena.setCounter("C002", 0);
}

void loop() {
    // Warunek startu sekwencji: sygnał na wejściu 0 LUB aktywny licznik
    if (vena.l_or(vena.check(0), vena.getCounter("C002") != 0)) {
        vena.sendMsg("3"); // Sygnalizacja LED

        // Faza 1: Rozpoczęcie wysuwu i uruchomienie nieblokującego timera na 5000 ms
        if (vena.getCounter("C002") == 1) {
            vena.sendMsg("1");
            vena.startTimer("T001", 5000);
        }

        // Faza 2: Po odliczeniu czasu przejście do kolejnych etapów sekwencji
        if (vena.checkTimer("T001", 0) == 1) {
            vena.sendMsg("5"); // Wysuw siłownika Y1
            // ... dalsze etapy sekwencji
        }
    }
}
```

---

## 📂 Struktura Repozytorium

```text
├── venaPilot3.h                        # Główna biblioteka C++ klasy Vena (API PLC)
├── Pilot3.ino                          # Program sterujący sekwencjami wykonawczymi
├── venaPilot2.h / Pilot2.ino           # Wersja archiwalna biblioteki (v2)
├── DOKUMENTACJA.md                     # Pełne zestawienie metod, parametrów i typów zwracanych
├── Dokumentacja_Techniczna_Vena_PLC.pdf# Oryginalna dokumentacja techniczna projektu
└── Zdjecia/                            # Zdjęcia gotowego układu, PCB i stanowiska
```

---

## 📖 Pełna Dokumentacja API

Kompletny opis wszystkich funkcji logicznych, obsługi timerów, liczników i markerów znajduje się w pliku:
👉 **[DOKUMENTACJA.md](DOKUMENTACJA.md)**

---

<p align="center">
  <sub>Projekt zrealizowany w ramach praktycznej automatyzacji układów wykonawczych i sterowania maszynowego.</sub>
</p>
