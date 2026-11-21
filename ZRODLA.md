# Autorstwo i źródła

## Wkład autorów

| Autor | Wykonane elementy | Lokalizacja w projekcie |
| --- | --- | --- |
| Marcin Kowalczyk (Pixel of Software) | wybór biblioteki, założenia, przegląd i akceptacja kodu, publikacja | całość |
| Asystent AI Claude (Anthropic) | wrapper, przykłady, testy, skrypty budowania, dokumentacja | `app/`, `przyklady/`, `tests/`, `build.py`, `run.py`, `README.md` |

## Wykorzystane materiały

| Materiał | Autor i źródło | Wersja lub data | Licencja | Sposób wykorzystania |
| --- | --- | --- | --- | --- |
| libsndfile | Erik de Castro Lopo i współtwórcy, https://github.com/libsndfile/libsndfile | 1.2.2 | LGPL-2.1-or-later (https://github.com/libsndfile/libsndfile/blob/1.2.2/COPYING) | biblioteka łączona dynamicznie, nie jest dołączona; deklaracje funkcji, struktury `SF_INFO` i stałych przepisane ręcznie z `sndfile.h` 1.2.2 |
| `build.py`, `run.py`, `LICENSE` | Fundacja Duckling, szablon projektu z materiałów na start DuckQuest | 2026 | MIT (plik `LICENSE`) | skrypty rozbudowane o dołączanie libsndfile i cel `przyklad`; licencja z dopisanym autorem |
| Przykład SDL2 z materiałów na start (`budowanie.py`) | Fundacja Duckling | 2026 | MIT | wzór opcji `--additional-link-options` przy łączeniu biblioteki C |
| Duckling (kompilator i biblioteka standardowa) | ducktype-org/duckling | 0.3-alpha, commit 83882805a | licencja Ducklinga | zależność budowania |

Pliki dźwiękowe w testach i przykładzie powstają w czasie działania programów (ton sinusoidalny, wzorce próbek, ręcznie zbudowany nagłówek WAV 24-bit); repozytorium nie zawiera cudzych nagrań.

## Narzędzia AI

Kod wrappera, aplikacji, przykładu i testów, zmiany skryptów budowania oraz dokumentację przygotowano z pomocą asystenta AI Claude (Anthropic). Autor ustalił zakres i wybór biblioteki, przejrzał wyniki, sprawdził działanie (budowa, 37 testów, przykład, normalizator) i odpowiada za oddaną wersję.

## Licencja własnego projektu

MIT, treść w pliku `LICENSE` (szablon Fundacji Duckling z dopisaną linią `Copyright (c) 2026 Pixel of Software (Marcin Kowalczyk)`). Licencja libsndfile (LGPL-2.1-or-later) dotyczy samej biblioteki, instalowanej osobno.
