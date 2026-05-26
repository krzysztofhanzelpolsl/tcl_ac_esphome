tl;dr 
Working on TCL TAC-12CHSD/XA71I; added buzer, eco, display managment

Below is an AI essay

---

# Opis projektu: Integracja ESPHome dla klimatyzatorów TCL

## Czym jest ten projekt i do czego służy?
Projekt stanowi dedykowany komponent niestandardowy (*custom component*) dla środowiska **ESPHome**, napisany w języku C++. Jego głównym celem jest umożliwienie **w pełni lokalnego, stabilnego i niezależnego od chmury producenta sterowania klimatyzatorem marki TCL** za pomocą mikrokontrolera ESP32 (np. ESP32-C3) połączonego bezpośrednio z płytą główną klimatyzatora przez interfejs szeregowy UART. 

Dzięki tej integracji klimatyzator staje się natywną encją typu `climate` w systemie Home Assistant, co pozwala na tworzenie zaawansowanych automatyzacji domowych, harmonogramów oraz pełny monitoring urządzenia w czasie rzeczywistym.

## Specyfika sprzętowa (Dedykowana modyfikacja)
Ta konkretna wersja kodu została zmodyfikowana i zoptymalizowana pod kątem współpracy z klimatyzatorem **TCL TAC-12CHSD/XA71I**. Jest to model wyposażony **wyłącznie w jednowymiarowy, pionowy swing** (ruch żaluzji góra-dół) i pozbawiony fizycznego silnika do sterowania żaluzjami poziomymi (lewo-prawo). Zrozumienie tej architektury pozwoliło na wyeliminowanie błędów uniwersalnych sterowników, które próbowały wymuszać na płycie głównej obsługę nieistniejących podzespołów.

---

## Kluczowe modyfikacje i funkcje (Ujęcie funkcjonalne)

W stosunku do bazowych, ogólnodostępnych w sieci bibliotek, w projekcie wprowadzono oraz naprawiono następujące funkcjonalności:

### 1. Całkowita naprawa i stabilizacja trybu Swing (Pionowego)
* **Problem bazowy:** W oryginalnym kodzie zmiana jakiegokolwiek parametru w aplikacji Home Assistant (np. korekta temperatury docelowej lub zmiana siły nawiewu) powodowała natychmiastowe, samoczynne zatrzymanie ruchu żaluzji pionowej. Wynikało to z faktu, że protokół TCL używa odwróconej logiki bitowej przy odczycie stanu w stosunku do wysyłania komend.
* **Rozwiązanie:** Logika wysyłania ramek została napisana na nowo. Wprowadzono rygorystyczne sterowanie rejestrami ruchu (`vswing_mv`). Podczas aktywacji swingu z poziomu GUI komponent wysyła teraz precyzyjne polecenie startu pełnego ruchu (`0x07`), a przy wyłączaniu – poprawną komendę zatrzymania w aktualnej pozycji (`0x06`). Zapobiega to odrzucaniu paczek danych przez mikrokontroler klimatyzatora i trwale separuje sterowanie temperaturą od sterowania żaluzją.

### 2. Dynamiczne wyciszenie buzzera (Funkcja Mute)
* **Opis:** Standardowo klimatyzator TCL potwierdza każdą, nawet najmniejszą zmianę ustawień przesłaną z Home Assistant głośnym i irytującym pisknięciem (beeperem) z jednostki wewnętrznej.
* **Modyfikacja:** Do kodu dodano zmienną stanową oraz powiązaną funkcję, która nadpisuje hardkodowany dotychczas bit sygnału dźwiękowego (`m_set_cmd.data.beep`). Wprowadzenie dedykowanego przełącznika w Home Assistant pozwala na pełne wyciszenie dźwięków klimatyzatora, dzięki czemu nocne automatyzacje (np. korygujące temperaturę lub bieg wentylatora podczas snu) odbywają się w całkowitej ciszy.

### 3. Sprzętowy Tryb ECO (Energooszczędny)
* **Opis:** Klimatyzatory TCL posiadają wbudowany algorytm oszczędzania energii, który optymalizuje krzywą pracy kompresora i ogranicza pobór prądu. W fabrycznym kodzie integracji funkcja ta była na stałe zablokowana na zero.
* **Modyfikacja:** Odblokowano rejestr odpowiedzialny za tryb ekonomiczny (`m_set_cmd.data.eco`). Od teraz, za pomocą przełącznika w automatyce domowej, można bezpośrednio wymusić na urządzeniu przejście w ekologiczny tryb pracy kompresora, co ułatwia zarządzanie bilansem energetycznym domu.

### 4. Zdalne zarządzanie ekranem panelu (Funkcja Display)
* **Opis:** Fabrycznie wyświetlacz LED na obudowie klimatyzatora świeci intensywnym światłem, pokazując aktualną temperaturę. W nocy, w sypialni, bywa to uciążliwe.
* **Modyfikacja:** Wprowadzono kontrolę nad bitem wyświetlacza (`m_set_cmd.data.disp`). Nowy przełącznik pozwala na całkowite gaszenie i zapalanie diod LED na przednim panelu klimatyzacji bezpośrednio z poziomu telefonu lub automatyzacji uzależnionej od pory dnia.

### 5. Blokada rejestru swingu poziomego (Horizontal Swing Lockout)
* **Opis:** Aby ramka sterująca wysyłana do jednostki **TAC-12CHSD/XA71I** była w 100% poprawna strukturalnie, wyczyszczono błędy interpretacji swingu poziomego.
* **Modyfikacja:** W sekwencji wyjściowej na sztywno zaimplementowano zerowanie parametrów nawiewu lewo-prawo (`hswing`). Zapobiega to powstawaniu tzw. "stanów duchów" w interfejsie oraz gwarantuje, że klimatyzator nigdy nie uzna ramki UART za uszkodzoną z powodu prób wysterowania nieistniejących klap poziomych.

---

## Efekt końcowy
Dzięki powyższym zmianom projekt przekształcił się ze zwykłego, kapryśnego sterownika temperatury w **kompletne centrum kontroli klimatyzacji**, które pozwala dostosować zachowanie urządzenia (zarówno pod kątem komfortu mechanicznego, jak i akustycznego czy wizualnego) do rzeczywistych potrzeb domowników.
