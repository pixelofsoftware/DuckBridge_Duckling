# DuckBridge: sndfile - wrapper libsndfile dla Ducklinga

![Napisane w Duckling](https://img.shields.io/badge/napisane%20w-Duckling-f2c94c)

Zespół: Pixel of Software (Marcin Kowalczyk). Temat: P02 DuckBridge, biblioteka libsndfile (poziom 2). Skład: 1 osoba.

Moduł `sndfile` (`app/sndfile.dk`) udostępnia w Ducklingu odczyt i zapis plików dźwiękowych, w szczególności WAV PCM16, przez bibliotekę C libsndfile. API przyjmuje i zwraca zwykłe typy Ducklinga. Użytkownik nie używa wskaźników, nie dopisuje `\0` i nie zwalnia pamięci. Zasoby zwalnia wrapper: bufory w każdym wywołaniu, a plik w `zamknij`.

Całe minimum karty libsndfile działa i jest sprawdzone testami. Do tego są dwa przykłady:
* podstawowy zapis i odczyt;
* praktyczny normalizator głośności plików WAV, np. dźwięków do gry.

Z rozszerzeń jest odczyt innych formatów obsługiwanych przez libsndfile, z przeliczeniem na i16; testy sprawdzają to na WAV 24-bit.

## Uruchomienie

Wymagane są:
* Python 3;
* Duckling (`duckc` w `PATH`, instalacja według tutoriala);
* biblioteka libsndfile.

| System | Instalacja libsndfile |
| --- | --- |
| Ubuntu | `sudo apt install libsndfile1` (nagłówki `libsndfile1-dev` nie są potrzebne) |
| macOS | `brew install libsndfile` |

Z katalogu projektu:

```bash
python3 build.py --target all --output build      # aplikacja, przykład i testy (wariant natywny)
python3 run.py --build build --target tests       # testy: PASS/FAIL, kod = liczba błędów
python3 run.py --build build --target przyklad    # przykład podstawowy (tworzy build/ton-stereo.wav)
printf 'build/ton-stereo.wav\nbuild/glosniej.wav\n\n' | python3 run.py --build build --workdir .
```

Ostatnia komenda uruchamia normalizator. Czyta on trzy wiersze ze standardowego wejścia:
1. plik wejściowy;
2. plik wyjściowy;
3. docelowy szczyt w procentach pełnej skali; pusty wiersz oznacza 89%, czyli około −1 dBFS.

Testy i przykład zapisują pliki WAV w katalogu budowy, w którym domyślnie uruchamia je `run.py`. `build.py` szuka biblioteki w tej kolejności:
1. ścieżka z opcji `--sndfile /sciezka/libsndfile.so.1`;
2. `pkg-config sndfile`;
3. typowe miejsca: `/usr/lib/x86_64-linux-gnu/libsndfile.so.1` oraz `/opt/homebrew/lib/libsndfile.dylib`;
4. `ctypes.util.find_library`.

Wybraną ścieżkę zapisuje w `build/budowa.json`. `build.py` i `run.py` powstały z szablonu konkursu. Doszła obsługa biblioteki i cel `przyklad`, a został tylko wariant natywny.

## Funkcje i demonstracja

| Funkcja z minimum karty | Sposób uruchomienia | Oczekiwany rezultat |
| --- | --- | --- |
| Zapis i odczyt WAV PCM16; częstotliwość, kanały, format, ramki | przykład: `--target przyklad` | `WAV PCM_16, 44100 Hz, 2 kanaly, 44100 ramek (1 s), 88200 probek` |
| Wielokanałowe próbki w jednoznacznym układzie; ramki a próbki | `czytajRamki(&plik, 10)` w stereo | 10 ramek = 20 próbek, przeplot L0, P0, L1, P1…; `probka(&probki, kanaly, ramka, kanal)` |
| Odczyt porcjami, koniec danych, przewijanie w ramkach | przykład: przewinięcie na ramkę 11111 i odczyt reszty po 10000 ramek | reszta 32987 ramek w 4 porcjach, `koniec` po ostatniej, pozycja 44100 |
| Zapis sprawdza liczbę zapisanych ramek i wynik zamknięcia | `zapiszRamki`, `zamknij` | `ok` tylko gdy zapisano wszystkie ramki i zamknięcie się powiodło |
| Sprawdzanie formatu, rozmiarów, kanałów; błąd brakującego lub złego pliku | przykład: brakujący plik, 3 próbki do pliku stereo | `nie mozna otworzyc '…' do odczytu: System error : No such file or directory.`; `3 probek to nie pelne ramki (2 kanaly) - nic nie zapisano` |
| Zgodność próbek po zapisie i odczycie (≥2 kanały, wartości ujemne) | `--target tests` | 1, 2 i 6 kanałów, wartości −32768, −1, 0 i 32767, 100 000 ramek: wszystkie próbki zgodne |

**Demonstracja:**
1. Zbuduj projekt według instrukcji i uruchom przykład. Pokazuje on informacje o pliku, pierwsze ramki, przewijanie, odczyt do końca i dwa kontrolowane błędy.
2. Uruchom normalizator na wygenerowanym pliku. Szczyt lewego kanału rośnie z 16383 (−6.0 dBFS) do 29163 (−1.0 dBFS), a program sam sprawdza wynik ponownym odczytem.
3. Pokaż kontrolowany błąd, np. brakujący plik wejściowy: kod 3 i komunikat libsndfile.

Kto odpowiada za zwolnienie wyniku: plik zamyka ten, kto go otworzył, wywołując `zamknij`. Listy próbek należą do wywołującego i są ważne także po zamknięciu pliku.

## API modułu `sndfile`

| Funkcja | Zwraca | Błąd |
| --- | --- | --- |
| `otworzDoOdczytu(sciezka: str) -> Plik` | otwarty plik z `info` | `otwarty(&p) == false`, opis w `p.blad` (np. brak pliku, `Format not recognised.`) |
| `utworzWav16(sciezka: str, czestotliwosc: i64, kanaly: i64) -> Plik` | nowy plik WAV PCM16 do zapisu (zastępuje istniejący) | częstotliwość poza 1..768000, kanały poza 1..1024, odrzucenie przez libsndfile |
| `czytajRamki(p: ref Plik, ileRamek: i64) -> Odczyt` | `probki: List[i16]` (przeplatane), `ramki`, `koniec` | `ok == false`: plik zamknięty lub do zapisu, `ileRamek < 1`, porcja ponad 16 777 216 próbek, błąd odczytu |
| `zapiszRamki(p: ref Plik, probki: ref List[i16]) -> Wynik` | `ok` | niepełne ramki (nic nie zapisano), plik do odczytu lub zamknięty, mniej zapisanych ramek (opis podaje ile) |
| `przewin(p: ref Plik, ramka: i64) -> Wynik` | `ok`, `p.pozycja = ramka` | ramka poza 0..`info.ramki`, plik nie do odczytu |
| `zamknij(p: ref Plik) -> Wynik` | `ok`; nagłówek WAV uzupełniony | kod `sf_close` różny od 0; ponowne zamknięcie: „plik nie jest otwarty” |
| `otwarty(p: ref Plik) -> bool` | czy plik jest otwarty | - |
| `typFormatu(&info)`, `kodowanie(&info)`, `jestWavPcm16(&info)` | np. „WAV”, „PCM_16” | - |
| `formatPoprawny(czestotliwosc, kanaly) -> bool` | czy libsndfile przyjmie WAV PCM16 (bez tworzenia pliku) | - |
| `probka(&probki, kanaly, ramka, kanal) -> i16` | próbka z przeplatanego bufora | indeks spoza listy to błąd programisty, jak przy `List.at` |
| `wersjaBiblioteki() -> String` | np. „libsndfile-1.2.2” | - |

`Info` ma pola `czestotliwosc`, `kanaly`, `ramki`, `format` (kod libsndfile) i `przewijalny`. `Plik.pozycja` to numer następnej ramki do odczytu albo liczba zapisanych ramek. Pole `Plik.uchwyt` jest wewnętrzne. Żadna funkcja wrappera nie przerywa programu: błędy wracają jako `ok == false` i opis po polsku, z komunikatem libsndfile, gdy biblioteka go podaje.

## Środowisko i zależności

| Element | Wartość |
| --- | --- |
| Kompilator | `duckc 0.3-alpha (83882805a 2026-11-20)`, `commit-hash: 83882805a` |
| Uruchomienie | natywnie |
| System operacyjny | macOS 27.2, Apple Silicon (arm64) |
| Zależności | libsndfile 1.2.2 (Homebrew `libsndfile 1.2.2_1`, `/opt/homebrew/lib/libsndfile.dylib`), Python 3 |

Wersja libsndfile, na której sprawdzono całe minimum, to 1.2.2, czyli wersja odniesienia z karty. Na Linuksie x86_64 (Ubuntu 24.04, `libsndfile.so.1`) projektu nie sprawdzałem. Deklaracje C nie zależą od systemu: struktura `SF_INFO` i uchwyt jako u64 są takie same w ABI LP64 Linuksa x86_64 i macOS arm64.

## Struktura i działanie

| Plik | Odpowiedzialność |
| --- | --- |
| `app/sndfile.dk` | wrapper: deklaracje C, struktura `SF_INFO`, publiczne API, sprawdzanie granic, bufory, błędy |
| `app/app.dk` | praktyczne zastosowanie: normalizator głośności WAV (dwa przebiegi porcjami, przewinięcie, sprawdzenie wyniku) |
| `przyklady/podstawowy.dk` | przykład podstawowy: zapis tonu stereo porcjami, odczyt, przewijanie, kontrolowane błędy |
| `tests/testy.dk` | testy zwykłych danych, granic i błędów |
| `build.py`, `run.py` | budowa i uruchomienie (szablon konkursu + libsndfile) |

**Deklaracje C** są pisane ręcznie (Duckling nie ma generatora) według `sndfile.h` 1.2.2. Wrapper używa minimalnego API z karty:
* struktura `SF_INFO`;
* funkcje `sf_format_check`, `sf_open`, `sf_readf_short`, `sf_writef_short`, `sf_seek`, `sf_strerror`, `sf_close`;
* do opisów: `sf_version_string` i `strlen`.

Stałe (`SFM_READ = 0x10`, `SFM_WRITE = 0x20`, `SF_FORMAT_WAV = 0x010000`, `SF_FORMAT_PCM_16 = 0x0002`, maski typu i kodowania) są przepisane z nagłówka.

**Uchwyt i NULL.** `sf_open` zwraca `SNDFILE*`, a przy błędzie NULL. Wrapper deklaruje ten wynik jako `u64`, więc NULL to po prostu 0, a do `sf_strerror(NULL)` można przekazać 0. Nie trzeba adaptera w C. Wskaźnik i `u64` przekazuje się identycznie w ABI 64-bitowych systemów (LP64). Na DVM wymagałoby to adaptera C, dlatego wariant DVM nie jest obsługiwany.

**Pamięć.** Każde `czytajRamki` i `zapiszRamki` przydziela bufor `malloc` na dokładną liczbę próbek, kopiuje dane z listy `List[i16]` albo do niej i zwalnia bufor przed powrotem. Bufor nie wycieka ani przy sukcesie, ani przy błędzie. Pojedyncza porcja to najwyżej 16 777 216 próbek, czyli 32 MiB, co chroni przed przepełnieniem i przypadkową ogromną alokacją.

**Granice i jednostki:**
* liczba ramek i pozycje są w ramkach, a liczba próbek to ramki × kanały;
* częstotliwość: 1-768000 Hz;
* kanały: 1-1024 (`SF_MAX_CHANNELS` w libsndfile);
* niepełna ramka w zapisie jest odrzucana w całości, zanim cokolwiek trafi do pliku.

**Koniec danych:** `koniec == true`, gdy odczytano mniej ramek, niż żądano. Kolejny odczyt zwraca 0 ramek bez błędu.

**Ograniczenia:**
* zapis tylko WAV PCM16;
* odczyt innych formatów zwraca próbki przeliczone przez libsndfile na i16;
* brak odczytu i zapisu metadanych (tytuł, znaczniki) oraz brak urządzeń audio, czego minimum nie wymaga.

## Sprawdzenie projektu

```bash
python3 run.py --build build --target tests      # 37 sprawdzeń
python3 run.py --build build --target przyklad   # kod 0 = wszystkie oczekiwania przykładu spełnione
```

Testy sprawdzają:
* zgodność próbek po zapisie i odczycie dla 1, 2 i 6 kanałów, z wartościami granicznymi −32768 i 32767 i ujemnymi, porcjami niebędącymi dzielnikami długości;
* surowe bajty zapisanego pliku, odczytane osobno przez `fopen`: nagłówek RIFF/WAVE, PCM, 2 kanały, 44100 Hz, 16 bitów, próbki little-endian w kolejności L0, P0, L1;
* ręcznie zbudowany plik WAV 24-bit: kodowanie PCM_24 i przeliczenie na i16;
* przewijanie na koniec i poza zakres;
* odczyt za końcem i pusty plik;
* błędy: brakujący, tekstowy i ucięty plik, zły tryb, niepełne ramki, operacje po zamknięciu, ponowne zamknięcie, porcje 0 i ponad limit.

Scenariusze są w [tests/scenariusze.md](tests/scenariusze.md).

## Autorstwo i materiały

Kod i dokumentacja: Pixel of Software (Marcin Kowalczyk), z pomocą asystenta AI. Szczegóły są w [ZRODLA.md](ZRODLA.md).

Licencja własnego kodu: MIT (plik `LICENSE`). Biblioteka libsndfile ma licencję LGPL-2.1-or-later i nie jest dołączona do repozytorium. Wrapper łączy się z nią dynamicznie jako z biblioteką systemową.
