# **Analiza dopasowania wahadłowego**

## Założenia

Na wstępie zakładam, że poza próbką 20 meczów z udziałem rozpatrywanego zawodnika, mam również dostęp do wszystkich meczów Pogoni, żeby zbudować sobie punkt odniesienia.

W pierwszej kolejności, zanim w ogóle wziąłbym pod uwagę dane dot. kandydata na wahadłowego, chciałbym ustalić kwestie związane ze stylem gry Pogoni, w szczególności profile obecnych wahadłowych.

Ważne jest oddzielenie metryk dostarczających informacji o jakości gry od tych mówiących o stylu (np. proporcja wygranych/przegranych pojedynków w defensywie vs. rejon boiska, gdzie te pojedynki najczęściej mają miejsce)

#

Jeden z eventów w danych StatsBomb to `Tactical Shift`. Należałoby wziąć pod uwagę wyłącznie fragmenty meczów, gdzie gra toczy się w ustawieniu 3-5-2. 

Ocenę stylu gry rozbiłbym na dwie części - analizę całego zespołu kolektywnie oraz oddzielnie wahadłowych. Podwójne spojrzenie zabezpiecza na wypadek, gdyby obecni wahadłowi nie spełniali założeń taktycznych i ich dane nie dawałyby pełnego obrazu oczekiwań stawianych wobec nich.

## Analiza gry całego zespołu 

Dzięki fladze `is_transition` można oszacować, jakie są proporcje pomiędzy szybkimi atakami po odbiorze a udziałem gry pozycyjnej w grze Pogoni. W ramach dopełnienia model On-Ball Value StatsBomb pozwoliłby oszacować które fazy gry są dla drużyny bardziej efektywne.

Kilka pozostałych kwestii do rozpatrzenia to 

* wysokość odzyskiwania piłki	(rozkład pozycji na boisku dla `Ball Recovery` / `Interception` / `Pressure`)
* intensywność pressingu, np. licząc liczbę eventów `Pressure` w stosunku do liczby dotknięć piłki przeciwnika lub licząc PPDA
* waga stałych fragmentów, mierzona np. poprzez udział `set_piece_phase` w akcjach kończących się strzałem
* dominujące typy dośrodkowań (`pass_cross` vs `pass_cut_back`) i strefy boiska z których dośrodkowywana jest piłka w pole karne
* waga poszczególnych sektorów boiska przy budowaniu akcji (przez które sektory najczęściej przechodzi piłka, z których sektorów Pogoń wchodzi w pole karne)

## Analiza wahadłowych

Gra całego zespołu dostarcza już częściowo informacji, co mniej więcej oczekiwane jest od wahadłowych, ale to wciąż niepełny obraz.

Przy bezpośredniej analizie wahadłowych, chciałbym zdefiniować, jakie są ich główne zadania. Czy często schodzą do półprzestrzeni, czy grają bliżej linii bocznych, jak często angażują się w defensywie, z których sektorów boiska dośrodkowują najczęściej.

### Pokrycie placu gry

Dane eventowe nie uwzględniają momentów, gdzie zawodnik nie jest zaangażowany bezpośrednio w grę, ale eventy z jego udziałem są wystarczająco informatywne żeby wiedzieć gdzie jest aktywny.

Przy analizie pozycji na boisku zwróciłbym uwagę na pokrycie pionowe boiska po `location_x` z wyłączeniem kilku kwantyli najwyżej i najniżej odnotowanych eventów żeby nie zaszumiać danych, rozkład `location_y` + udział dotknięć w finalnej tercji i przy liniach bocznych.

### Podania / prowadzenie piłki

W dalszej kolejności żeby ocenić jak wygląda zazwyczaj prowadzenie piłki, chciałbym przyjrzeć się częstotliwości eventów z flagą `Carries`, gdzie piłka prowadzona jest wzdłuż boiska przynajmniej 5-6 metrów, częstotliwość wejść w pole karne i w ostatnią tercję boiska z piłką, dodatkowo możnaby wziąć pod uwagę model On-Ball Value  i przyjrzeć się jak wygląda bilans OBV z `Carries` w porównaniu do OBV z podań, żeby uzupełnić obraz.

Analogicznie do analizy całej drużyny, osobno należałoby uwzględnić rodzaje dośrodkowań i obszary skąd dośrodkowywana jest piłka (StatsBomb pozwala oddzielnie uwzględnić `pass_cross` i `pass_cut_back`), liczbę podań zakończonych strzałem i przełożeń z jednej flanki na drugą.

Wszystkie podania są pogrupowane w danych eventowych w zależności od ich charakterystyki (rejonu, wysokości, długości kierunku), więc opierając się bezpośrednio na ID poszczególnych klastrów da się zdefiniować tendencje podającego.

### Defensywa

Żeby ocenić, jak w jaki sposób defensywnie zaangażowani są obecni wahadłowi, przede wszystkim uwzględniłbym lokalizację i częstotliwość eventów takich jak `Defensive Duel`, `Ball Recovery`, `Block` wraz z danymi dot. pressingu. Poza standardowym `Pressure` dot. presji na zawodnika z piłką StatsBomb oznacza też pojedynki o wolną piłkę - również do uwzględnienia.

## Ocena jakości 

Dzięki wbudowanym modelom StatsBomb, zamykając profilowanie bieżących wahadłowych warto uwzględnić ich impakt na rezultaty. Dałoby to również informację, jakie ewentualne niedociągnięcia w ich grze powinien być w stanie naprawic nowy nabytek.

Kilka z nich do uwzględnienia:

* Najbardziej uogólniony model On-Ball Value (pamiętając, na ile ofensywna jest rola danego wahadłowego)

* Wartość dodana przy podaniach (over/undeperformance względem oczekiwanej liczby udanych podań z użyciem `pass_success_probability`) oraz liczba podań o wysokim ryzyku powyżej ustalonego progu zakończonych sukcesem

*  miara odporności na pressing poprzez porównanie skuteczność podań z flagą `under_pressure` do tych, gdzie zagrywający ma więcej swobody

* Bilans przegranych/wygranych pojedynków, dodatkowo do kontekstu można dodać `defensive_responsibility_probability` by zważyć istotność przegranych pojedynków. 

# Dopasowanie kandydata 

Ponownie, przy analizie kandydata trzeba spojrzeć oddzielnie na styl jego gry oraz jakość. Należy również zastosować wszystkie poprzednie założenia (jak odfiltrowanie wyłącznie czasu, gdy jego zespoł grał w systemie z wahadłowymi) oraz porównując, ustandaryzować dane z uwzględnieniem posiadania piłki (żeby np. nie zawyżać sztucznie liczby akcji defensywnych) oraz ustandaryzować per 90 minut gry.

### Styl

Zależnie od rezultatów analizy z poprzedniego kroku, należy wyselekcjonować odpowiednie metryki reprezentujące pożądany styl, a z niektórych zrezygnować lub przypisać im mniejsze wagi (np. rozliczając wahadłowego, który często schodzi do środka przypisanie dużej wagi dośrodkowaniom z okolic linii bocznej zaszumiłoby analizę). 

Wyniki analizy dot. obecnych wahadłowych dobrze jest przedstawić jako ważony wektor w przestrzeni wielowymiarowej (czyli statystyczną reprezentację profilu gracza), a następnie zmierzyć podobieństwo analogicznego wektora zbudowanego dla naszego kandydata. 

Żeby nie zmarnować możliwości danych eventowych, analizę wsparłbym testem oka opartym o heatmapy pokrycia boiska, stref z których wychodzą podania itd.

Dla danych, które same w sobie nie zawierają się w jednej liczbie (takiej jak np. liczba wejść w pole karne) a zebrane z danych eventowych stanowią cały rozkład (w szczególności dane dot. pozycjonowania się na boisku) można dodatkowo wykonać porównanie z pomocą testów statystycznych służących do oceny podobieństwa rozkładów danych - innymi słowy statystycznie dowieść czy dwie chmury punktów (np. miejsca odbiorów piłki) pochodzą z tego samego rozkładu.

### Jakość

Żeby skutecznie ocenić jakość kandydata, idealnie umieścić go w kontekście wszystkich wahadłowych w lidze i standaryzując uprzednio dane (najprościej z użyciem z-score, tworzącego ustandaryzowaną skalę na tle ligi) stworzyć zważony wskaźnik jakości, który zawierałby bilans OBV, bilansu sukcesu podań, wygranych pojedynków itd. Pełny dobór metryk częściowo również powinien być zależny od rozpatrywanego profilu wahadłowego. Jeżeli wykorzystanie danych na temat całej ligi jest niemożliwe, pozostaje bezpośrednie porównanie liczbowe z wahadłowymi Pogoni

Poszczególne metryki powinny również być rozpatrzone indywidualnie, żeby zidentyfikować braki w jego grze, które mogłyby się okazać dyskwalifikujące poniżej pewnego progu (np. wybitnie słaby performance podań, gdy rywal pressuje).

# Braki 

Jak wspomniałem wyżej, dane eventowe nie uwzględniają tego, gdzie znajduje się i jak porusza się zawodnik niezaangażowany bezpośrednio w akcję, np. tworząc wolne przestrzenie dla pozostałych zawodników. Potrzebne byłyby pełne dane freeze frame/trackingowe ze stopklatek, żeby zamodelować grę lepiej.

Ponadto, niemożliwa jest na ich podstawie dokładna ocena parametrów fizycznych i wydajnościowych, przez co obraz danego zawodnika nigdy nie będzie pełny. W przypadku wahadłowego brak pomiarów jego zdolności sprinterskich i wydolności stanowi istotny problem.
