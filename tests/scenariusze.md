# Scenariusze sprawdzenia

Próby z 2026-11-21 na macOS 27.2 (Apple Silicon), `duckc 0.3-alpha (83882805a 2026-11-20)`, libsndfile 1.2.2 (Homebrew), wariant natywny, po `python3 build.py --target all --output build`.

| Scenariusz | Dane lub czynności | Oczekiwany wynik | Komenda lub procedura | Otrzymany wynik |
| --- | --- | --- | --- | --- |
| Zwykłe działanie: zapis i odczyt stereo | 1000 ramek, 44100 Hz, zapis porcjami po 300, odczyt po 333 | info 44100 Hz / 2 kan. / 1000 ramek / WAV PCM_16; wszystkie 2000 próbek zgodne | `run.py --target tests` | zgodny - OK |
| Ramki a próbki | odczyt 10 ramek stereo | 10 ramek, 20 próbek, przeplot L, P | testy | zgodny - OK |
| Wiele kanałów i wartości ujemne | 1, 2 i 6 kanałów, −32768, −1, 0, 32767 | próbki identyczne po zapisie i odczycie | testy | zgodny - OK |
| Duży plik porcjami | 100 000 ramek stereo, zapis po 4096, odczyt po 8192 | zgodność wszystkich próbek | testy | zgodny - OK |
| Zgodność z formatem WAV | surowe bajty zapisanego pliku | RIFF/WAVE, PCM (1), 2 kan., 44100 Hz, 16 bit, 4000 B danych, little-endian L0, P0, L1 | testy (`fopen`/`fread`) | zgodny - OK |
| Inny format na wejściu | ręczny WAV 24-bit mono | kodowanie PCM_24, nie PCM16; próbki 32767, −32768, 16384, 1 | testy | zgodny - OK |
| Przewijanie i koniec danych | przewinięcie na 500, 1000, 1001, −1; odczyt za końcem | 500 i 1000 dozwolone; 1001 i −1 odrzucone; za końcem 0 ramek bez błędu | testy; przykład (ramka 11111, reszta 32987 ramek w 4 porcjach) | zgodny - OK |
| Niepoprawne dane: brak pliku | `nie-ma-takiego-pliku.wav` | opis z nazwą pliku i komunikatem libsndfile | testy, przykład, normalizator (kod 3) | „System error : No such file or directory.” - OK |
| Niepoprawne dane: zły plik | plik tekstowy `tekst.wav`, ucięty nagłówek `RIFF` | odrzucenie z opisem | testy | „Format not recognised.” - OK |
| Niepoprawne dane: niepełne ramki | 3 próbki do pliku stereo | odrzucenie, nic nie zapisane, liczba ramek bez zmian | testy, przykład | zgodny - OK |
| Niepoprawne użycie | odczyt pliku do zapisu, zapis do pliku do odczytu, operacje po zamknięciu, ponowne zamknięcie | błędy z opisem, bez awarii | testy | zgodny - OK |
| Granice parametrów | 0 kanałów, 1025 kanałów, 0 Hz, 800 000 Hz, porcja 0 i ponad 16 777 216 próbek | odrzucenie | testy | zgodny - OK |
| Praktyczne zastosowanie | normalizacja `ton-stereo.wav` do 89% | szczyt 16383 (−6.0 dBFS) → 29163 (−1.0 dBFS); sprawdzenie ramek, formatu i szczytu | `printf 'build/ton-stereo.wav\nbuild/glosniej.wav\n\n' \| python3 run.py --build build --workdir .` | zgodny, kod 0 - OK |
| Normalizator: złe parametry | ten sam plik wejścia i wyjścia; szczyt 250 | kod 2 z opisem | jak wyżej | zgodny - OK |
| Testy | 37 sprawdzeń | `Bledy: 0`, kod 0 | `python3 run.py --build build --target tests` | `Bledy: 0` - OK |
| Odtworzenie w nowym katalogu | świeży klon repozytorium | budowa, przykład, normalizator i testy zgodne z README | `git clone`, komendy z README | macOS: OK; Linux x86_64 (Ubuntu 24.04, libsndfile1): nie sprawdzono |
