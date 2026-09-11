# Bramka jakości kartoteki towarów — walidacja kodów i nazw przy zakładaniu karty (enova365)

> Element większej całości: **[Kartoteka, sprzedaż i fakturowanie w enova365 — mapa rozwiązania](https://github.com/trynityeu/enova365-obieg-sprzedazy-i-kartoteki)**

Dodatek do systemu ERP **enova365** (Soneta sp. z o.o.), który nadaje karcie
towaru identyfikator i opis — ale **tylko wtedy, gdy karta jest zgodna
z zasadami kodów i nazw**. Kontrola jakości kartoteki przeniesiona z raportu
zbiorczego wykonywanego po fakcie na moment zakładania karty.

To repozytorium zawiera wyłącznie **opis funkcjonalny** — bez kodu źródłowego,
bez danych klienta, bez treści reguł specyficznych dla wdrożenia.

## Problem, który rozwiązuje

Kartoteka towarowa licząca ponad dziesięć tysięcy pozycji jest budowana według
**ścisłych zasad kodów i nazw**: prefiks kodu musi odpowiadać nazwie towaru,
wymiary zapisane w kodzie muszą zgadzać się z jednostkami i opakowaniami, numer
katalogowy w kodzie musi odpowiadać polu numeru katalogowego, produkty muszą
mieć technologię.

Zasady były spisane i sprawdzane **zapytaniem raportowym uruchamianym
zbiorczo** — czyli już po fakcie. Skutek: błędy trafiały do kartoteki i były
wykrywane dopiero po tygodniach, gdy towar był już w obrocie, a poprawienie
kodu oznaczało ruszanie dokumentów, które się do niego odwołują.

Osobno istniał wymóg techniczny: pole identyfikatora wewnętrznego wyprowadza
się z numeru rekordu karty, a opis — z nazwy według ustalonej reguły. Obie
rzeczy robiono ręcznie albo prostym przyciskiem, który niczego nie sprawdzał.

## Pomysł, na którym opiera się całość

Kontrola została **doczepiona do czynności obowiązkowej**.

Nadanie identyfikatora jest krokiem, którego nie da się pominąć — bez niego
karta nie funkcjonuje w obiegu. Przycisk robi więc dwie rzeczy naraz: nadaje
identyfikator i opis **oraz** sprawdza zgodność karty z zasadami. Karta nie
dostanie identyfikatora, dopóki nie jest poprawna.

Dzięki temu **kontrola stała się nieomijalna**, choć nikt jej nie wymusza
administracyjnie. To istotna różnica wobec walidacji doczepionej do zapisu
karty: tamtą da się obejść, tej — nie, bo obejście oznacza kartę bez
identyfikatora.

## Miejsce zastosowania

Przycisk stoi na **formularzu karty towaru** i obsługiwany jest skrótem
klawiszowym — czynność wykonuje się wielokrotnie dziennie, więc sięganie
myszą do paska byłoby realnym kosztem.

Istnieje również **wariant seryjny** działający na zaznaczeniu z listy
towarów, z osobną pozycją w drzewie uprawnień. Opis poniżej dotyczy wariantu
pojedynczego.

## Przebieg krok po kroku

| # | Krok | Co się dzieje |
|---|---|---|
| 1 | **Przepisanie wartości z edytorów** | dane z otwartego formularza trafiają do karty **bez uruchamiania weryfikatorów zapisu** — celowo, patrz niżej |
| 2 | **Walidacja twarda** | komplet reguł; naruszenia → komunikat z **listą wszystkich** i zatrzymanie. Nic nie zostaje zapisane, karta pozostaje nietknięta |
| 3 | **Ostrzeżenie: produkt bez technologii** | komunikat z samym „OK" — potwierdzenie kontynuuje |
| 4 | **Ostrzeżenie: prefiks kodu wskazuje innego producenta** | komunikat z wyborem: „Tak" zapisuje mimo rozbieżności, „Nie" anuluje bez zapisu |
| 5 | **Zapis karty** | dopiero tutaj pojawiają się systemowe ostrzeżenia zapisu, jeśli są |
| 6 | **Ustawienie pól** | identyfikator i opis wg reguł niżej |

Kroki 3 i 4 wykonują się **wyłącznie wtedy, gdy walidacja twarda przeszła** —
ostrzeżenie o rzeczy drugorzędnej nie ma sensu przy karcie, która i tak nie
przejdzie.

## Co dokładnie ustawia

| Pole | Reguła |
|---|---|
| **Identyfikator wewnętrzny** | budowany z numeru rekordu karty |
| **Opis — karta usługi** | opis = nazwa, bez zmian |
| **Opis — pozostałe karty** | opis = **oczyszczona nazwa** + identyfikator |

Czyszczenie nazwy polega na obcięciu jej **od pierwszej cyfry** oraz usunięciu
typowych końcówek wymiarowych i katalogowych. Cel jest praktyczny: opis ma być
czytelny i **nie powtarzać wymiarów, które są już zapisane w kodzie**.

## Dwa rodzaje zastrzeżeń

Podział jest zasadniczy dla użyteczności narzędzia — decyduje o tym, co
zatrzymuje pracę, a co tylko ją komentuje:

| | **Błąd** | **Ostrzeżenie** |
|---|---|---|
| Skutek | **blokuje bezwarunkowo** | pozostawia decyzję operatorowi |
| Kiedy się pojawia | zawsze, gdy reguła naruszona | dopiero **gdy nie ma żadnego błędu** |
| Czy karta zostaje zapisana | nie — nic nie jest ruszane | tak, po potwierdzeniu |
| Rodzaj sprawy | rozstrzygalna maszynowo | wymagająca oceny człowieka |

**Wszystkie błędy wypisywane są naraz**, nie pierwszy napotkany. Raport
zbiorczy pokazywał tylko pierwszy, przez co poprawianie karty było serią
iteracji: popraw, uruchom, dowiedz się o kolejnym, popraw. Pełna lista zamienia
to w jedno przejście.

### Grupy reguł twardych

Kilkanaście grup, ujętych tematycznie — pełne brzmienie operator dostaje
w komunikacie:

| Grupa | Czego dotyczy |
|---|---|
| **Zapis kodu** | spacje w kodzie, podwójne podkreślenia, sąsiadujące separatory, niedomknięte nawiasy, dopuszczalna pozycja kropki, znak przed nawiasem |
| **Wielkość liter i skróty** | wyłącznie wielkie litery, ograniczenie liczby liter obok siebie, pełne formy skrótów zamiast skróconych |
| **Zakazane człony** | człon oznaczający kopię karty, oznaczenia powłok w niewłaściwym typie karty, oznaczenia koloru i średnicy w kodzie |
| **Nazwa** | podwójne spacje, wymóg pełnego zapisu oznaczenia koloru |
| **Zgodność kodu z wymiarami** | wymiary zapisane w kodzie muszą zgadzać się z zakładką jednostek i opakowań — z tolerancją ułamka milimetra, z obsługą sum i wariantów zapisu |
| **Sens wymiarów** | bardzo mała wysokość rozpoznawana jako grubość, z wyjątkami dla wybranych grup |
| **Jednostka miary** | jednostka długości wymaga podanego wymiaru |
| **Numer katalogowy** | numer wewnątrz nawiasu w kodzie musi odpowiadać **pierwszemu** numerowi w polu numeru katalogowego |
| **Prefiks kodu** | trzyznakowy prefiks musi istnieć w słowniku, a nazwa karty musi zaczynać się od jednej z nazw dozwolonych dla tego prefiksu |
| **Dostawca** | dostawca domyślny nie może mieć cechy wykluczającej |
| **Produkt** | istniejąca technologia o kodzie równym docelowemu identyfikatorowi musi zgadzać się nazwą i opisem oraz być zatwierdzona i niezablokowana |

**Karty zablokowane też podlegają kontroli.** Raport zbiorczy je pomijał,
przycisk — nie. Wycofanie towaru z obrotu nie zwalnia z zasad: wadliwy kod
zostaje w bazie i nadal myli.

### Dwa ostrzeżenia i dlaczego akurat one

**Produkt bez technologii** — komunikat z samym potwierdzeniem. Nie blokuje,
i jest ku temu konkretny powód: **technologię zakłada się osobno i nazywa
identyfikatorem, który dopiero ten przycisk nadaje**. Zablokowanie karty do
czasu założenia technologii byłoby zapętleniem — nie dałoby się zrobić ani
jednego, ani drugiego.

**Prefiks kodu wskazuje innego producenta niż cecha producenta** — komunikat
z wyborem. Pojawia się, gdy prefiks kodu jest w słowniku kodem producenta,
a cecha producenta na karcie wskazuje inną firmę. Rozstrzygnięcie wymaga
wiedzy, której system nie ma: która z dwóch informacji jest prawdziwa.
Komunikat **podaje obie nazwy**, żeby dało się to ocenić bez otwierania
słownika.

## Komunikaty — jak je czytać

| Komunikat | Znaczenie | Co zrobić |
|---|---|---|
| **Lista naruszeń kodu lub nazwy** | walidacja twarda nie przeszła; **nic nie zostało zapisane** | poprawić kartę i uruchomić przycisk ponownie — lista jest kompletna, więc jedno podejście wystarczy |
| **Wskazanie dozwolonych nazw dla prefiksu** | nazwa karty nie zaczyna się od żadnej z nazw przypisanych temu prefiksowi | rozstrzygnąć, co jest poprawne: nazwa karty czy wpis w słowniku. Prefiks może mieć **kilka** dozwolonych nazw — każda jest akceptowana |
| **Prefiks nie istnieje w słowniku** | kod używa prefiksu, którego nikt nie zdefiniował | uzupełnić słownik albo poprawić kod karty |
| **Brak technologii przy produkcie** | ostrzeżenie, nie błąd | potwierdzić; technologię założyć osobno i nazwać nadanym identyfikatorem |
| **Niezgodność producenta** | ostrzeżenie z wyborem | ustalić, które dane są poprawne — komunikat podaje obie nazwy |

Najważniejsza wskazówka diagnostyczna: **gdy karta wygląda poprawnie, a mimo
to jest odrzucana, problem najczęściej leży w słowniku prefiksów, nie
w karcie**. Dlatego komunikat nie poprzestaje na „błędny kod", tylko wypisuje,
jakie nazwy są dla tego prefiksu dozwolone — operator dostaje od razu materiał
do rozstrzygnięcia, zamiast musieć szukać go w konfiguracji.

## Rozwiązania warte odnotowania

### Kolejność komunikatów

Przy wcześniejszych wydaniach operator po kliknięciu dostawał najpierw
**systemowe ostrzeżenie zapisu** („zmieniono wartość ważnego pola", Tak/Nie),
a dopiero za nim listę błędów — albo odwrotnie. Efekt: właściwy komunikat ginął
w tłumie.

Stąd bierze się krok 1 przebiegu: akcja **przepisuje wartości z edytorów bez
uruchamiania weryfikatorów zapisu**, sprawdza reguły i — jeżeli są błędy —
kończy samą listą naruszeń. Systemowe ostrzeżenia pojawiają się dopiero wtedy,
gdy walidacja przeszła i faktycznie dochodzi do zapisu.

### Walidacja na danych niezapisanych

Reguły sprawdzane są na **bieżących danych formularza**, także tych jeszcze
niezapisanych. Operator dostaje odpowiedź o karcie, którą właśnie wypełnia, a
nie o jej poprzedniej wersji z bazy.

### Odczyt słowników z pominięciem pamięci podręcznej

Reguły prefiksów i producentów czytane są **bezpośrednio ze słowników**,
z pominięciem wewnętrznej pamięci podręcznej programu — inaczej poprawka
w słowniku nie działałaby do czasu ponownego uruchomienia programu, co przy
diagnostyce „poprawiłem słownik, a nadal odrzuca" byłoby mylące.

## Ograniczenia, które trzeba znać

- **Zasady walidacji są wbudowane w dodatek** — ich zmiana wymaga nowej wersji
  pliku. To świadomy wybór: reguły dotyczą kształtu kodu i są na tyle złożone
  (tolerancje, warianty zapisu, wyjątki dla grup), że wyrażenie ich jako danych
  konfiguracyjnych byłoby budową drugiego języka.
- **Działanie zależy od zawartości dwóch słowników** — prefiksów kodów wraz
  z dozwolonymi nazwami oraz kodów producentów. Błąd w słowniku **blokuje
  poprawne karty**, więc słowniki są częścią kontraktu, nie tłem.
- **Wariant seryjny nie pokazuje ostrzeżeń** — komunikat z wyborem nie pasuje
  do pętli po zaznaczeniu, bo zamieniłby przebieg w serię okien. Karta
  z produktem bez technologii albo z niezgodnym producentem zostaje tam
  przetworzona bez pytania. Kontrola tych dwóch spraw istnieje **wyłącznie
  w wariancie pojedynczym**.
- **Dwie niezależne pozycje uprawnień** dla dwóch wariantów. Warto sprawdzać je
  razem: łatwo doprowadzić do stanu, w którym ktoś ma odebrany przycisk
  pojedynczy, a zachował seryjny działający na całej liście — czyli uprawnienie
  szersze zamiast węższego.

## Miejsce w rodzinie

Bramka stoi na samym początku obiegu: **kod towaru jest identyfikatorem, do
którego odwołują się wszystkie dokumenty**. Błąd wpuszczony do kartoteki
rozchodzi się na zamówienia, dokumenty magazynowe, faktury i opisy analityczne
— a poprawienie go po fakcie oznacza ruszanie tych dokumentów.

Powiązane elementy fundamentu:

- **dokumentacja techniczna z kart** dołączana wprost do dokumentów handlowych
  → [Zapytanie ofertowe do dostawcy](https://github.com/trynityeu/enova365-zapytanie-ofertowe-do-dostawcy)

## Pochodzenie

Pierwotna wersja przycisku pochodziła od innego dostawcy i wykonywała samo
nadanie identyfikatora i opisu — bez żadnej kontroli. Obecne wydanie zachowuje
tę funkcję i rozbudowuje ją o walidator, dzięki czemu zmiana nie wymagała
przyzwyczajania użytkowników do nowego narzędzia: **ten sam skrót klawiszowy,
ten sam efekt, dodatkowo zatrzymanie przy błędzie**.

## Co jest do tego potrzebne

- enova365 z modułem Handel (kartoteka towarów) oraz — dla reguły produktowej —
  technologiami.
- **Dwa słowniki**: prefiksów kodów wraz z dozwolonymi nazwami oraz kodów
  producentów.
- Cecha wykluczająca na kartotece kontrahentów (reguła dostawcy domyślnego).
- Uprawnienie do akcji, nadane świadomie — z uwzględnieniem drugiej pozycji dla
  wariantu seryjnego.

## Zgodność

| | |
|---|---|
| System | enova365 (Soneta sp. z o.o.), klient desktop |
| Postać | dodatek (worker) — przycisk na formularzu karty towaru, ze skrótem klawiszowym |
| Zapisuje | identyfikator wewnętrzny i opis karty towaru |
| Rola dodatkowa | **bramka jakości kartoteki** — blokuje kartę niezgodną z zasadami |
| Idempotentny | **tak** — powtórzenie daje ten sam wynik |
| Integracje zewnętrzne | **brak** |

## Czego tu nie ma

Kod źródłowy, dokumentacja wdrożeniowa, pełne brzmienie reguł, zawartość
słowników, dane handlowe oraz konfiguracja specyficzna dla wdrożenia pozostają
w repozytorium prywatnym. To repo służy wyłącznie jako publiczny opis funkcji
dodatku.
