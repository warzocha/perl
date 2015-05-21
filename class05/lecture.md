# Warsztat 05
## Wyrażenia regularne - ciąg dalszy
### Alternatywa
Alternatywa pozwala nam dopasować dwa różne wzorce w jednym miescu. Metaznak
**|**, oznacza, że w danym miescu może pasować i wzorzec po prawej
i wzorzec po lewej.
```perl
m/abc|def/;
```
Wzorzec powyżej będzie pasował do napisów:
```perl
'abc'
'def'
```
Po ludzku będzie znaczył _"napis 'abc' lub napis 'def'"_.

**UWAGA:** Alternatywa zawiera całość wzorca od znaku **|** do brzegu grupy lub
końca wyrażenia regularnego. Wyrażenia podanego w powyższym przykładzie nie
należy interpretować jako _"napis 'ab', litera 'c' lub litera 'd', napis
'ef'"_.

Alternatywy można łączyć w dopasowania więcej niż dwóch wzorców:
```perl
m/\AToday I ate (?:pasta|potatoes|rice)\Z/;
```

Powyższe wyrażenie dopasuje:
```
'Today I ate pasta'
'Today I ate potatoes'
'Today I ate rice'
```

### Chciwe i skąpe dopasowania
Kwantyfikatory **+** i __\*__ są _chciwe_. To znaczy, że dopasowują
największą część napisu, jaką się da. Nie zawsze jest to pożądane
zachowanie.

Załóżmy, że chcemy wyciągnąć z napisu tekst znajdujący się pomiędzy dwoma
cudzysłowami. Użycie dopasowania **.+** może wymknąć się spod kontroli
w następujym przypadku:
```perl
my $string = q(test "napis", "inny napis" i coś jeszcze);
my ($quote) = $string =~ /"(.+)"/;
say $quote;
```
Takie wyrażenie dopasuje najdłuższy możliwy ciąg znaków otoczony
cudzysłowami. Wynikiem powyższego programu jest:
```
napis", "inny napis
```

Dodanie znaku **?** po chciwym kwantyfikatorze zmieni jego tryb
dopasowania na skąpy. Dopasowanie skąpe dopasuje najkrótszy ciąg znaków
pasujący do wyrażenia. Skąpa wersja poprzedniego przykładu:
```perl
my $string = q(test "napis", "inny napis" i coś jeszcze);
my ($quote) = $string =~ /"(.+?)"/;
say $quote;
```
Wynik:
```
napis
```

### Modyfikatory dopasowań
Na końcu wyrażenia regularnego można dodawać modyfikatory, które zmieniają
zachowanie wyrażenia. Kilka bardziej przydatnych:

#### /i
Modyfikator **/i** (case-**i**nsensitive) sprawia, że wielkość liter jest
ignorowana przy dopasowaniu.
```perl
print '1 Match' if 'TesT' =~ /test/;
print '2 Match' if 'TesT' =~ /test/i;
```
Wynik:
```
2 Match
```

#### /s
Pod wpływem modyfikatora **/s** (**s**ingle-line) wyrażenie traktuje napis
jako pojedynczą linię, metaznak **.** dopasowuje wtedy również _"\n"_.
```perl
print '1 Match' if "Te\nst" =~ /.{5}/;
print '2 Match' if "Te\nst" =~ /.{5}/s;
```
Wynik:
```
2 Match
```

#### /x
Modyfikator **/x** sprawia, że większość białych znaków wewnątrz wyrażenia
regularnego zostaje zignorowana. Pozwala to na rozbicie wyrażenia na mniejsze
części dla zwiększenia czytelności.
```perl
say '1 Match' if "Test" =~ /Test/;
say '2 Match' if "Test" =~ /Te      st/x;
```
Wynik:
```
1 Match
2 Match
```

### Quotemeta
Kiedy przyjmujemy wyrażenie regularne lub jego część jako wejście z
niekontrolowanego źródła (np. od użyszkodnika), należy uważać na znaki
specjalne. Do dosłownej interpretacji napisów wewnątrz wyrażeń służy funkcja
**quotemeta** i para metaznaków **\Q**, **\E**.

**quotemeta** przyjmuje napis jako argument i poprzedza wszystkie wystąpienia 
znaków niealfanumerycznych znakiem **\**.
```perl
my $string = 'abc;-*_*a()[]';
say quotemeta $string;
```
Wynik:
```
abc\;\-\*_\*a\(\)\[\]
```

Interpolacja napisu przetworzonego przez **quotemeta** wewnątrz wyrażenia
regularnego spowoduje dopasowanie dosłownego napisu bez interpretacji znaków
specjalnych.

Metaznaki **\Q** i **\E** pozwalają kontrolować interpretację znaków
specjalnych wewnątrz wyrażeń regularnych. Od znaku **\Q** aż do znaku **\E**
wszytkie znaki wyrażenia będą interpretowane dosłownie.
```perl
my $string = '---[\d.+]---';
say 'Match 1' if $string =~ /-+[\d.+]-+/;
say 'Match 2' if $string =~ /-+\[\\d\.\+\]-+/;
say 'Match 3' if $string =~ /-+\Q[\d.+]\E-+/;
```
Wynik:
```
Match 2
Match 3
```

### Zamiana dopasowań
Składnia zamiany wygląda następująco:
```perl
s/old/new/
```
Znak przed **s/** wzorcem oznacza tryb zamiany. Po nim następuje wzorzec do
dopasowania i ciąg znaków, którym będzie zastąpiony. Na końcu mogą zostać
dodane modyfikatory, np. **/g**.
```perl
my $text = 'Lorem ipsum 617 if any 12 dwarves go home at 11:34.';
$text =~ s/\d+/<NUMBERS>/g;
say $text;
```
Wynik:
```
Lorem ipsum <NUMBERS> if any <NUMBERS> dwarves go home at <NUMBERS>:<NUMBERS>.
```

Wyrażenie zamiany zwraca ilość zamienionych dopasowań.
```perl
my $text = 'Lorem ipsum 617 if any 12 dwarves go home at 11:34.';
my $count = $text =~ s/\d+/<NUMBERS>/g;
say $count;
```
Wynik:
```
4
```

#### Modyfikatory zamiany
Oprócz **/g**, istnieje kilka innych przydatnych modyfikatorów, które pozwalają
zmienić zachowanie wyrażenia regularnego.

##### /r
Modyfikator **/r** zmienia wartość zwracaną przez wyrażenie zamiany z liczby 
dopasowań na zmodyfikowaną kopię napisu. Pozwala to zachować początkowy
napis bez zmian.
```perl
my $text = '[name]: hue hue hue';
foreach my $name (qw(John Michael Tiffany Kate)) {
    say $text =~ s/\Q[name]\E/$name/r;
}
say $text;
```
Wynik:
```
John: hue hue hue
Michael: hue hue hue
Tiffany: hue hue hue
Kate: hue hue hue
[name]: hue hue hue
```

##### /e
Modyfikator **/e** pozwala podmieniać dopasowania na wynik działania funkcji
zamiast konkretnego napisu. Jeśli wykonujemy podmianę z tym modyfikatorem,
drugi operand operatora podmiany jest anonimową funkcją, podobnie jak
w przypadku **map**, **sort** i **grep**. Przy każdym dopasowaniu funkcja ta
zostanie wywołana, a dopasowanie zamienione na jej wynik. Dzięki temu możemy,
na przykład, w prosty sposób ponumerować wystąpienia wzorca:
```perl
my $text = 'test bla test bla Test';
my $counter = 0;
$text =~ s/(test)/"${1}[" . ++$counter . ']'/gei;
say $text;
```
Wynik:
```
test[1] bla test[2] bla Test[3]
```

### Nazwane grupy
Odwoływanie się do grup przez zmienne liczbowe (**$1**, **$2**...) może
być niewygodne w bardziej rozbudowanych wyrażeniach
(zwłaszcza zawierających alternatywy), dlatego Perl pozwala nam na 
tworzenie nazwanych grup, do których odwołujemy się po nazwie, a nie
numerze.

Nazwane grupy tworzy się przez dodanie na początku grupy konstrukcji
**?<nazwa>**. Zawartość takiej grupy nie trafi do zmiennej liczbowej,
za to będzie można się do niej odwołać przez specjalny hasz **%+**.
```perl
my $paren = "tekst [w nawiasie] i poza nawiasem";
if ($paren =~ /\[(?<paren>.+?)\]/) {
    print $+{paren};
}
```
Wynik:
```
w nawiasie
```

Wykorzystując nazwane grupy można znacznie zwiększyć czytelność kodu
odwołującego się do dopasowań.