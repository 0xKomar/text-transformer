# Text Transformer

## Wprowadzenie

Text Transformer to aplikacja do przekształcania tekstu. Użytkownik może wybrać jedną z dostępnych transformacji albo wykonać kilka operacji w dowolnej kolejności. Ten sam zestaw operacji jest dostępny w interfejsie graficznym i przez REST API, z którego mogą korzystać inne aplikacje i skrypty.

## Proces biznesowy

### BC1. Przekształcenie tekstu

**Aktorzy** – [Użytkownik](#ac1), [Klient API](#ac2)

**Scenariusz główny**

1. Użytkownik wpisuje lub wkleja tekst w interfejsie graficznym. ([UC1](#uc1))
2. Użytkownik wybiera co najmniej jedną operację. Przy kilku operacjach ustala ich kolejność.
3. System wykonuje wybrane operacje po kolei.
4. System wyświetla przekształcony tekst.

**Scenariusz alternatywny**

1.A. Klient API przesyła tekst i listę operacji w żądaniu HTTP. ([UC2](#uc2))

- 1.A.1. System sprawdza dane i wykonuje operacje w podanej kolejności.
- 1.A.2. System zwraca przekształcony tekst w odpowiedzi JSON.

## Aktorzy

<a id="ac1"></a>

### AC1. Użytkownik

Osoba, która przekształca tekst za pomocą interfejsu graficznego.

<a id="ac2"></a>

### AC2. Klient API

Aplikacja lub skrypt, który wywołuje transformację przez REST API i korzysta z otrzymanego wyniku.

## Przypadki użycia

<a id="uc1"></a>

### UC1. Przekształcenie tekstu w interfejsie graficznym

1. Użytkownik wpisuje lub wkleja tekst do pola wejściowego.
2. Użytkownik wybiera operacje z listy dostępnych transformacji i ustala kolejność wykonania.
3. Użytkownik uruchamia transformację.
4. System wyświetla przekształcony tekst w polu wyniku.

<a id="uc2"></a>

### UC2. Przekształcenie tekstu przez REST API

1. Klient wysyła żądanie JSON zawierające tekst i listę z co najmniej jedną nazwą operacji.
2. System sprawdza typy danych i nazwy operacji.
3. System wykonuje operacje w kolejności podanej na liście.
4. System zwraca `200 OK` i wynik w formacie JSON.

**Scenariusz alternatywny**

2.A. Brakuje wymaganego pola, dane mają nieprawidłowy typ lub lista operacji jest pusta albo zawiera nieznaną nazwę.

- 2.A.1. System zwraca `400 Bad Request` z opisem błędu.

## Obiekty biznesowe

| ID | Obiekt | Opis |
| --- | --- | --- |
| BO1 | Tekst wejściowy | Tekst wpisany w interfejsie graficznym lub przesłany do API. |
| BO2 | Sekwencja transformacji | Lista wybranych operacji w kolejności wykonania. |
| BO3 | Tekst wynikowy | Tekst uzyskany po wykonaniu wszystkich wybranych operacji. |

## Reguły biznesowe

### BR1. Kolejność operacji

Pierwsza operacja przetwarza tekst wejściowy. Każda następna operacja przetwarza wynik poprzedniej. Dla `mirek` sekwencja `upper`, `lower` zwraca `mirek`, a sekwencja `lower`, `upper` zwraca `MIREK`.

### BR2. Zmiana wielkości liter

| Operacja | Działanie | Przykład |
| --- | --- | --- |
| `upper` | Zamienia wszystkie litery na wielkie. | `żółć` → `ŻÓŁĆ` |
| `lower` | Zamienia wszystkie litery na małe. | `MIREK` → `mirek` |
| `capitalize` | Zamienia pierwszą literę każdego wyrazu na wielką. | `ala ma kota` → `Ala Ma Kota` |

### BR3. Odwracanie tekstu (`inverse`)

Operacja odwraca kolejność znaków, w tym spacji, cyfr i interpunkcji. Wielkość liter zostaje na swoich pozycjach: jeśli przed odwróceniem na danej pozycji była wielka litera, po odwróceniu na tej samej pozycji też jest wielka litera. Operacja obsługuje tekst w UTF-8, w tym polskie znaki.

- `MirEk` po wykonaniu `inverse` → `KerIm` (wielkie litery zostają na pozycjach 1 i 4)
- `żółć` po wykonaniu `inverse` → `ćłóż`
- `mirek` po wykonaniu `inverse`, a następnie `upper` → `KERIM`

### BR4. Zamiana liczb na słowa (`numbersToWords`)

Operacja zastępuje liczby zapisane cyframi ich słownymi odpowiednikami w języku polskim. Zakres do potwierdzenia obejmuje liczby całkowite do 1000 i liczby dziesiętne z dokładnością do setnych.

- `Wpłać 100 złotych` → `Wpłać sto złotych`
- `Wpłać 100 złotych` po wykonaniu `numbersToWords`, a następnie `lower` → `wpłać sto złotych`

### BR5. Skracanie wyrażeń (`abbreviate`)

Operacja rozpoznaje wyrażenia ze słownika niezależnie od wielkości liter i zastępuje je przypisanymi skrótami.

- `Pieczywo to na przykład chleb i bułki` → `Pieczywo to np. chleb i bułki`
- `między innymi` → `m.in.`

- `i tym podobne` → `itd.`

Pełna lista skrótów jest do uzgodnienia z Product Ownerem. Para `i tym podobne` ↔ `itd.` pochodzi z kryteriów akceptacji, ale w polszczyźnie `i tym podobne` skraca się do `itp.`, a `itd.` oznacza `i tak dalej`. Trzeba to potwierdzić przed implementacją.

### BR6. Rozwijanie skrótów (`expandAbbreviations`)

Operacja zastępuje rozpoznane skróty pełnymi wyrażeniami. Pierwsza litera rozwinięcia ma taką samą wielkość jak pierwsza litera skrótu.

- `Pan Prof. spóźnił się na zajęcia` → `Pan Profesor spóźnił się na zajęcia`
- `dr` → `doktor`
- `np.` → `na przykład`
- `Np. chleb i bułki` → `Na przykład chleb i bułki`

### BR7. Przygotowanie tekstu do LaTeX-a (`escapeLatex`)

Operacja zapisuje obsługiwane znaki specjalne w postaci, która pozwala wyświetlić je jako zwykły tekst w LaTeX-u. Znak `&` zamienia na `\&`, a znak `$` na `\$`.

`John Smith & Sons` → `John Smith \& Sons`

### BR8. Usuwanie sąsiadujących powtórzeń (`removeAdjacentDuplicates`)

Operacja zastępuje każdy ciąg dwóch lub więcej identycznych, sąsiadujących wyrazów jednym wystąpieniem danego wyrazu.

- `Wyślij do do mnie wiadomość` → `Wyślij do mnie wiadomość`
- `Wyślij do do do mnie wiadomość` → `Wyślij do mnie wiadomość`
- `Idę do domu i do sklepu` → `Idę do domu i do sklepu`

## Macierz CRUDL

| Przypadek użycia | BO1. Tekst wejściowy | BO2. Sekwencja transformacji | BO3. Tekst wynikowy |
| --- | --- | --- | --- |
| [UC1](#uc1) | R | R | C |
| [UC2](#uc2) | R | R | C |

`R` oznacza odczyt danych, a `C` utworzenie wyniku.

## REST API

Endpoint `POST /transform` przyjmuje żądania JSON w kodowaniu UTF-8.

| Pole | Typ | Opis |
| --- | --- | --- |
| `text` | Ciąg znaków | Tekst wejściowy. |
| `transforms` | Tablica ciągów znaków | Co najmniej jedna nazwa operacji. Kolejność elementów określa kolejność wykonania. |

### Przykładowe żądanie

```json
{
  "text": "Wpłać 100 złotych",
  "transforms": ["numbersToWords", "lower"]
}
```

### Odpowiedź `200 OK`

```json
{
  "result": "wpłać sto złotych"
}
```

Żeby wykonać pojedynczą operację, klient przekazuje listę z jedną nazwą, np. `["capitalize"]`.

## Architektura

Interfejs graficzny korzysta z tego samego REST API co aplikacje zewnętrzne. Kontroler sprawdza żądanie i przekazuje tekst oraz listę operacji do modułu transformacji. Serwer będzie napisany w Javie z użyciem Spring Boot i budowany za pomocą Mavena.

Moduł transformacji wykorzystuje wzorzec Dekorator. Dla każdego żądania składa łańcuch z wybranych operacji w podanej kolejności. Każdy dekorator wykonuje swoją operację na wyniku poprzedniego kroku.

## Budowanie i uruchomienie

[![Build](https://github.com/0xKomar/text-transformer/actions/workflows/build.yml/badge.svg)](https://github.com/0xKomar/text-transformer/actions/workflows/build.yml)

Wymagania: JDK 11 i Maven.

```bash
mvn package
java -jar target/text-transformer-0.1.0.jar
```

Aplikacja nasłuchuje na porcie `8080`. GitHub Actions buduje projekt po każdym pushu i w każdym pull requeście, a gotowy plik `.jar` zapisuje jako artefakt builda.

## Zespół

| Osoba | GitHub | Rola |
| --- | --- | --- |
| Igor Komar | [0xKomar](https://github.com/0xKomar) | Scrum Master, developer |
| Oliwier Gołembiewski | [Ponioli](https://github.com/Ponioli) | Proxy Product Owner, developer |
| Bartosz Preus | [BartoszPreus](https://github.com/BartoszPreus) | Developer |
| Wiktor Sinkiewicz | [WiktorSinkiewicz](https://github.com/WiktorSinkiewicz) | Developer |

Rolę Product Ownera pełni prowadzący zajęcia laboratoryjne.

## Organizacja pracy

- Elementy rejestru produktu mają etykietę [`PB Item`](https://github.com/0xKomar/text-transformer/issues?q=label%3A%22PB+Item%22). Są uszeregowane według ważności, a w opisie każdego elementu są ważność, pracochłonność, Business Value i kryteria akceptacji.
- Zadania mają etykietę [`Task`](https://github.com/0xKomar/text-transformer/issues?q=label%3ATask) i są podpięte jako sub-issues pod swój element rejestru. Zadania z Definition of Done mają dodatkowo etykietę [`DoD`](https://github.com/0xKomar/text-transformer/issues?q=label%3ADoD).
- Zakres sprintu wyznacza Milestone, np. [Sprint 1](https://github.com/0xKomar/text-transformer/milestone/1).
- Status zadań jest na tablicy w zakładce [Projects](https://github.com/0xKomar/text-transformer/projects) w kolumnach Todo, In Progress i Done.
- Każde zadanie ma przypisaną osobę. Kto zaczyna pracę, przenosi zadanie do In Progress.
- Zmiany trafiają do `main` przez pull request z gałęzi zadania. Zadanie zamyka się po wypełnieniu DoD, najlepiej przez `Closes #<numer>` w opisie pull requesta.

## Sprint 1

**Termin:** 17.11.2026 – 15.12.2026. Sprint Review odbywa się na laboratorium we wtorek 15.12.2026 o 13:30.

**Cel sprintu:** dostarczyć działające REST API, które przyjmuje tekst i sekwencję transformacji w JSON i wykonuje zmianę wielkości liter (`upper`, `lower`, `capitalize`), `inverse`, zamianę liczb na tekst oraz zamianę wyrażeń na skróty i rozwijanie skrótów. Przyrost ma spełniać całe Definition of Done.

**Pojemność:** 4 developerów × 10 h = 40 h.

| Element rejestru produktu | Pracochłonność | Business Value |
| --- | --- | --- |
| [#1 REST API do wywoływania transformacji](https://github.com/0xKomar/text-transformer/issues/1) | 8 h | 1 |
| [#2 Zmiana wielkości liter (upper, lower, capitalize)](https://github.com/0xKomar/text-transformer/issues/2) | 3 h | 0,5 |
| [#3 Odwracanie tekstu z zachowaniem wielkości liter (inverse)](https://github.com/0xKomar/text-transformer/issues/3) | 3 h | 0,5 |
| [#4 Zamiana liczb na tekst w języku polskim](https://github.com/0xKomar/text-transformer/issues/4) | 8 h | 1 |
| [#5 Zamiana wyrażeń na skróty](https://github.com/0xKomar/text-transformer/issues/5) | 4 h | 0,5 |
| [#6 Rozwijanie skrótów](https://github.com/0xKomar/text-transformer/issues/6) | 4 h | 0,5 |
| Definition of Done i zadania punktowane ([#32](https://github.com/0xKomar/text-transformer/issues/32)–[#39](https://github.com/0xKomar/text-transformer/issues/39)) | 10 h | – |
| **Razem** | **40 h** | **4** |

Elementy #1–#6 dają razem 4 BV. Pełne punkty za wartość biznesową wymagają ukończenia wszystkich sześciu.

Poza sprintem w rejestrze produktu zostają [#7 LaTeX](https://github.com/0xKomar/text-transformer/issues/7), [#8 usuwanie powtórzeń](https://github.com/0xKomar/text-transformer/issues/8) i [#9 GUI](https://github.com/0xKomar/text-transformer/issues/9).

### Podział pracy

| Osoba | Zadania | Pracochłonność |
| --- | --- | --- |
| Wiktor Sinkiewicz | REST API i wzorzec Dekorator ([#10](https://github.com/0xKomar/text-transformer/issues/10)–[#14](https://github.com/0xKomar/text-transformer/issues/14)), rozwijanie skrótów ([#29](https://github.com/0xKomar/text-transformer/issues/29)–[#31](https://github.com/0xKomar/text-transformer/issues/31)) | 12 h |
| Bartosz Preus | Zmiana wielkości liter ([#15](https://github.com/0xKomar/text-transformer/issues/15)–[#17](https://github.com/0xKomar/text-transformer/issues/17)), odwracanie tekstu ([#18](https://github.com/0xKomar/text-transformer/issues/18)–[#20](https://github.com/0xKomar/text-transformer/issues/20)), zamiana wyrażeń na skróty ([#26](https://github.com/0xKomar/text-transformer/issues/26)–[#28](https://github.com/0xKomar/text-transformer/issues/28)), logowanie SLF4J ([#37](https://github.com/0xKomar/text-transformer/issues/37)) | 11 h |
| Oliwier Gołembiewski | Zamiana liczb na tekst ([#21](https://github.com/0xKomar/text-transformer/issues/21)–[#24](https://github.com/0xKomar/text-transformer/issues/24)), ciągła integracja w GitHub Actions ([#33](https://github.com/0xKomar/text-transformer/issues/33)), generowanie dokumentacji ([#34](https://github.com/0xKomar/text-transformer/issues/34)) | 9 h |
| Igor Komar | Testy zamiany liczb na tekst ([#25](https://github.com/0xKomar/text-transformer/issues/25)), repozytorium ([#32](https://github.com/0xKomar/text-transformer/issues/32)), dokumentacja Javadoc ([#35](https://github.com/0xKomar/text-transformer/issues/35)), tablica GitHub Projects ([#36](https://github.com/0xKomar/text-transformer/issues/36)), diagram klas UML ([#38](https://github.com/0xKomar/text-transformer/issues/38)), weryfikacja DoD ([#39](https://github.com/0xKomar/text-transformer/issues/39)) | 8 h |
| **Razem** | | **40 h** |

Scrum Master i Proxy Product Owner mają mniej zadań programistycznych ze względu na obowiązki związane z pełnioną rolą.

Proxy Product Owner uzgadnia z Product Ownerem listę skrótów (BR5, BR6) i zakres liczb (BR4).

### Harmonogram

| Termin | Co ma być gotowe |
| --- | --- |
| nd 29.11 | Interfejs dekoratora i składanie sekwencji ([#12](https://github.com/0xKomar/text-transformer/issues/12)) oraz DTO ([#10](https://github.com/0xKomar/text-transformer/issues/10)) w `main`. Tablica GitHub Projects i logowanie SLF4J. |
| wt 8.12 | Wszystkie transformacje z testami w `main`. Endpoint wykonuje każdą sekwencję operacji. |
| nd 13.12 | Javadoc, generowanie dokumentacji przy budowie i diagram klas UML. |
| pn 14.12 | Weryfikacja całego DoD, tag `v0.1.0` zgodny z wersją w `pom.xml`. |
| wt 15.12 | Sprint Review. |

Do 29.11 pozostałe osoby piszą logikę transformacji i testy jako zwykłe klasy, a potem podpinają je pod interfejs dekoratora.

### Daily Scrum

Daily Scrum trwa 15 minut i odbywa się w każdy dzień pracy nad projektem. Każdy mówi, co zrobił od ostatniego spotkania, co zrobi do następnego i co go blokuje. W piątki i soboty nie planujemy pracy nad projektem.

| Dzień | Godzina | Miejsce |
| --- | --- | --- |
| Poniedziałek | 20:15–20:30 | Online, Microsoft Teams |
| Wtorek | 18:30–18:45 | Online, Microsoft Teams |
| Środa | 17:00–17:15 | Online, Microsoft Teams |
| Czwartek | 20:15–20:30 | Online, Microsoft Teams |
| Niedziela | 19:00–19:15 | Online, Microsoft Teams |

Terminy w Sprincie 1: 24.11, 25.11, 26.11, 29.11, 30.11, 1.12, 2.12, 3.12, 6.12, 7.12, 8.12, 9.12, 10.12, 13.12 i 14.12.

## Definition of Done

Wymagania i punktacja są w dokumencie [Zasady punktowania sprintów](https://ekursy.put.poznan.pl/mod/url/view.php?id=1001442) na e-Kursach. Zespół realizuje wszystkie punkty przewidziane dla zespołu do 4 osób.

| Punkt | Wymaganie | Punkty | Zadanie |
| --- | --- | --- | --- |
| 1 | Kod w GitHubie, prowadzący ma dostęp | 20 | [#32](https://github.com/0xKomar/text-transformer/issues/32) |
| 2 | Tag z numerem wersji i ta sama wersja w `pom.xml` | 20 | [#32](https://github.com/0xKomar/text-transformer/issues/32), [#39](https://github.com/0xKomar/text-transformer/issues/39) |
| 3 | Automatyczne budowanie (kompilacja i paczka) | 30 | [#33](https://github.com/0xKomar/text-transformer/issues/33) |
| 4.1 | Ciągła integracja, prowadzący ma dostęp | 30 | [#33](https://github.com/0xKomar/text-transformer/issues/33) |
| 5.1 | Dokumentacja generuje się podczas budowy | 10 | [#34](https://github.com/0xKomar/text-transformer/issues/34) |
| 5.2, 5.3 | Pełny Javadoc dla co najmniej 3 klas lub interfejsów | 30 + 10 | [#35](https://github.com/0xKomar/text-transformer/issues/35) |
| 6 | Rejestr produktu i sprintu w narzędziu, prowadzący ma dostęp | 20 | [#36](https://github.com/0xKomar/text-transformer/issues/36) |
| 7.1 | Logowanie SLF4J na poziomach DEBUG i INFO | 10 | [#37](https://github.com/0xKomar/text-transformer/issues/37) |
| 8.1 | Diagram klas UML w repozytorium lub wiki | 40 | [#38](https://github.com/0xKomar/text-transformer/issues/38) |
| 9.1 | Wzorzec Dekorator w module transformacji | 40 | [#12](https://github.com/0xKomar/text-transformer/issues/12) |

Za prowadzenie projektu (rejestr produktu w GitHub Issues, elementy w Milestone, zadania pod elementami) jest 60 punktów, a za wartość biznesową 80 punktów. Razem z DoD daje to 400 punktów.

Szacunki i rozbicie na zadania są też w arkuszu [`TextTransformer.xlsx`](TextTransformer.xlsx).
