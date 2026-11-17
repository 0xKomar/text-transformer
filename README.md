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

Operacja odwraca kolejność znaków, w tym spacji, cyfr i interpunkcji. Każda litera przenosi się na nową pozycję ze swoją dotychczasową wielkością. Operacja obsługuje tekst w UTF-8, w tym polskie znaki.

- `MirEk` po wykonaniu `inverse` → `kEriM`
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

Pełna lista skrótów jest do uzgodnienia. Dotyczy to również pary `i tym podobne` ↔ `itd.`.

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