---
title: "R Intro für Biotechnologen"
author: "U. Rieger"
output: 
  html_document: 
    fig_caption: yes
    toc: yes
---



# Laden benötigter Pakete

In diesem Intro werden wir mit dem Paket Biotech arbeiten, dieses installieren wir mit:

```r
# devtools::install_github("https://github.com/Utzi1/Biotech")
devtools::load_all("~/Bachelor/Biotech")
```

```
## Loading Biotech
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✔ ggplot2 3.3.2     ✔ purrr   0.3.4
## ✔ tibble  3.0.1     ✔ dplyr   1.0.0
## ✔ tidyr   1.1.0     ✔ stringr 1.4.0
## ✔ readr   1.3.1     ✔ forcats 0.5.0
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```
Installieren müssen wir es nur ein Mal, danach jedoch müssen wir in jedem Skript, in welchem es zum Einsatz kommt, auf seine Existenz verweisen, wir müssen es "laden":

```r
library(Biotech)

# Wir laden auch tidverse, dieses Paket enthält eine Menge
# nützlicher Funktionen
library(tidyverse)
```
# Teaser

## Rohdatenverarbeitung

Ihr habt im Labor den Bradford-Assay durchgeführt und mit Rinderserumalbumin eine Standardreihe aufgenommen, dazu wurden zu jeder Verdünnung vier technische Replikate angefertigt und deren Absorption gemäß der Vorschrift gemessen.
Die Ergebnisse der Messung lauten:

--------------------------------------
 mes.1   mes.2   mes.3   mes.4   conc 
------- ------- ------- ------- ------
 0.031   0.04    0.009   0.007    0   

 0.092   0.103   0.077   0.084    20  

 0.191   0.201   0.166   0.166    40  

 0.278   0.279   0.205   0.205    60  

 0.332   0.348   0.25    0.353    80  

 0.363   0.36    0.397   0.371   100  
--------------------------------------

Diese fassen wir gleich als tibble zusammen:

```r
Std <- tibble(
  mes.1 = c(0.031,	0.092,	0.191,	0.278,	0.332, 0.363),
  mes.2 = c(0.04, 0.103, 0.201, 0.279, 0.348, 0.36),
  mes.3 = c(0.009, 0.077, 0.166, 0.205, 0.25, 0.397), 
  mes.4 = c(0.007, 0.084, 0.166, 0.205, 0.353, 0.371),
  )
```
Zusätzlich dazu brauchen wir eine Vector, welcher Information über die Konzentration des BSA in den angesetzten Standards (in $\frac{\mu g}{ml}$) bereithält:

```r
conc <- seq(
            # die geringste Konzentration
            from = 0,
            # die Höchste Konzentration
            to = 100,
            # die Länge
            length.out = 
                # diese muss der Anzahl der Messungen entsprechen
                length(Std$mes.1)
            )
```

Die Länge (also Anzahl der einzelnen Werte muss der Anzahl der Standards entsprechen) wird über das Argument length.out gesteuert.
Als Nächstes berechnen wir das arithmetische Mittel der technischen Replikate:

```r
Std.mean <- rowMeans(Std)
```

## Errechnen eines Linearen Modell

Damit haben wir nun alles um ein lineares Modell vom Typ $y = m \cdot x + b$ zu errechnen und zu plotten, dies geht recht leicht:

```r
# Berechnen des Modells
LinMod(abs = Std.mean, conc = conc) %>%
    # Überführen des Modells in eine schöne Tabelle
  pander::pander()
```


---------------------------------------------------------------
     &nbsp;        Estimate   Std. Error   t value   Pr(>|t|)  
----------------- ---------- ------------ --------- -----------
 **(Intercept)**    -6.74       2.277       -2.96     0.04157  

     **abs**        277.5       9.547       29.06    8.345e-06 
---------------------------------------------------------------


--------------------------------------------------------------
 Observations   Residual Std. Error   $R^2$    Adjusted $R^2$ 
-------------- --------------------- -------- ----------------
      6                2.872          0.9953       0.9941     
--------------------------------------------------------------

Table: Fitting linear model: conc ~ abs

```r
# Plotten des Modells
plot_regression(abs = Std.mean, conc = conc)
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png)

## Konzentrationsbestimmung der Proben

Nach dem selben Verfahren wie die Standardreihe wurde auch mit Proben unbekannter Konzentration verfahren, für diese wurden folgende Absorptionswerte gemessen:

```r
# die gemessenen Werte:
mes.1 <- c(0.185, 0.245, 0.399, 0.429, 0.448, 0.431)

# definieren des Verdünnungsfaktor
FV <- c(80, 80, 40, 40, 20, 20)

# Hieraus berechnen wir dann direkt die Konzentrationen:
conc.mes <- 
    conc_eval(abs_P = mes.1, abs_std = Std.mean, conc_std = conc)
```

```
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##       1       2       3       4       5       6 
##  0.7057  2.0466 -3.4797 -0.3354 -2.2546  3.3175 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   -6.740      2.277   -2.96   0.0416 *  
## abs_std      277.459      9.547   29.06 8.35e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.872 on 4 degrees of freedom
## Multiple R-squared:  0.9953,	Adjusted R-squared:  0.9941 
## F-statistic: 844.6 on 1 and 4 DF,  p-value: 8.345e-06
```

```r
conc.rea <- 
    conc.mes * FV

# Erzeugen einer Ausgabe
print(conc.rea)
```

```
## [1] 3567.163 4898.968 4158.634 4491.585 2351.227 2256.891
```
Bei genauerer Betrachtung fällt auf, dass die Werte, welche zu den geringeren Verdünnungen gehören viel geringere Konzentrationen aufweisen, als jene der schwachen Verdünnungen.
Dies lässt sich damit erklären, dass der Bradford Assay nur innerhalb eines bestimmten linearen Bereichs zuverlässige Ergebnisse liefert.
Dieser liegt in der Regel unterhalb einer Konzentration von $120 \frac{\mu g}{ml}$, ein Vergleich der Messwerte mit den theoretischen Konzentrationen ist dabei hilfreich.
 
Eine Einführung in R
====================

Skript, Konsole & Interpreter
-----------------------------

R ist eine interpretierte Sprache, das bedeutet, dass ein Interpreter
die Statements, welche an ihn gegeben werden, interpretiert. Wenn man
nun R in seiner ursprünglichsten Form haben will, so bekommt man es als
Kommandozeilen-Interpreter. Dazu muss man `R` lediglich installieren und
in einer [TE]{acronym-label="TE" acronym-form="singular+short"} `R`
eingeben und Enter drücken. Dadurch wird der Interpreter *erweckt* und
steht zur Verfügung, wie das aussehen kann, soll durch
[\[app::Kommandozeile\]](#app::Kommandozeile){reference-type="ref"
reference="app::Kommandozeile"} nachvollziehbar gemacht werden.
Innerhalb dieser Umgebung gilt nun R, allerdings soll gesagt sein, dass
diese Umgebung nichts für tippfaule ist, sie ist als Taschenrechner
ebenso verlässlich wie für etwas umfangreichere Aufgaben. Sollte man
jedoch eine Auswertung häufiger durchführen oder eine umfangreiche
Untersuchung durchführen, so bietet sich das schreiben von Skripten an.
Ein R-Skript wird durch zwei grundlegende Informationen geprägt:

Dateiendung

:   Das Skript selbst ist eine Textdatei, diese endet mit `.R`, also wie
    eine Excel-Datei auf `.xsxl` endet. Die Datei mit Namen
    `irgendwas.R` kann zwar in jedem beliebigen Editor bearbeitet werden
    aber wird nur erkannt, wenn die Endung korrekt ist.

Inhalt

:   In der Datei, welche auf `.R`, endet finden wir einen von R
    interpretierbaren Text. Der Inhalt muss also konform zur
    R-Language-Definition [@R_gen] sein.

Wenn man dann das Skript geschrieben hat, kann es zum Beispiel über den
Kommandozeilen-Interpreter interpretiert werden, bewerkstelligt wird das
dann über `source("Pfad/zur/Datei")`. Wenn man sich in einem
[IDE]{acronym-label="IDE" acronym-form="singular+short"} befindet, wie
zum Beispiel R-Studio, erfolgt das bearbeiten und interpretieren von
Skripten interaktiv.

Grundlegende Begriffe
---------------------

Die recht gut nachvollziehbare Syntax kann gerade auch fachfremden
Anwendern den Beginn enorm erleichtern, bereits ein paar Kenntnisse der
englischen Sprache reichen Teilweise aus um viele der Befehle zu
verstehen, ein paar besonders wichtige Zeichen (Operatoren) und
Grundfunktionen wollen wir uns jedoch explizit ansehen:

`+ - * /  ̂%% %/%`

:   Sind die arithmetischen Operatoren *plus*, *minus*, *mal*,
    *geteilt*, *hoch*, *Modulus* (Teilen mit Rest) und die *ganzzahlige
    Division*.

`<-`

:   Ordnet innerhalb einer Umgebung zu, er ist der gebräuchlichste
    `assignment operator`.

`< > <= >= == !=`

:   Sind relationale Operatoren für *kleiner als*, *größer als*,
    *kleiner gleich*, *größer gleich*, *exakt gleich* und *ungleich*.

`& | !`

:   Sind logische Operatoren für *und* , *oder* und *nicht*.

`[] $`

:   Sind subsetting-Operatoren.

`function()`

:   Erzeugt eine Funktion.

`if() else()`

:   Sind Bedingungen.

`for()`

:   Erzeugt eine `for`-Schleife.

`while()`

:   Erzeugt eine `while`-Schleife.

`#`

:   Markiert den Beginn eines Kommentars.

Alle gelisteten Begriffe sind *reserviert*, das bedeutet, dass sie durch
zum Beispiel eine Zuordnung durch einen Pfeil nicht überschrieben werden
können. Es gibt weitere *Reserved Words in R*, diese können durch
`?reserved` ausgegeben werden.

### Zuweisungen

Ein weiterer reservierter Begriff ist `NA`, wenn wir Versuchen `NA` mit
einer neuen Variable zu überschreiben bekommen wir eine Fehlermeldung:

    > NA <- 1
    Fehler in NA <- 1 : ungültige (do_set) linke Seite in Zuweisung

Das Selbe wird auch bei den anderen reservierten Begriffen passieren.
Eine Zuweisung kann auch in eine andere Richtung erfolgen:

    > c(1, 2, 3) -> x
    > x
    [1] 1 2 3
    # das geht nicht:
    > y -> c(1, 2, 3)
    Fehler: Objekt 'y' nicht gefunden

Das 2. Statement hingegen weist schlichtweg in die falsche Richtung.

    # auch das gleich ist in der Lage eine Zuweisung vorzunehmen
    > x = c(1, 2, 3)
    # x wird nun auf y übertragen
    > x -> y
    > y
    [1] 1 2 3
    # x bleibt bestehen
    > x
    [1] 1 2 3
    # sind die beiden exakt gleich?
    > x == y
    [1] TRUE TRUE TRUE
    # Ja!

Es ist empfehlenswert sich, gerade zu Beginn, mit einfachen Operationen
ein Bild von der Funktion der Sprache zu machen und die Operatoren zu
nutzen, um sie kennen zu lernen.

### Vektoren und deren Subsetting

Unter Subsetting versteht man das bilden kleiner Untereinheiten
(subsets) aus größeren Sets (Datensets), im einfachsten Fall ein Vektor:

    > x <- c(1:6)
    # Subsets nach Position
    > x[2:3]
    [1] 2 3
    # ein Sunbset als Zuweisung
    > y <- x[1:4]
    > y
    [1] 1 2 3 4

Den Elementen eines Vektors können auch Namen gegeben werden, das
geschieht über `names()`:

    > names(x) = c("eins", "zwei", "drei", "vier", "fünf", "sechs")
    > x
     eins  zwei  drei  vier  fünf sechs 
        1     2     3     4     5     6 
    # nun lassen wir uns den Wert zum Namen ausgeben
    > x["zwei"]
    zwei 
       2 
    # !!ACHTUNG!! der Name wird nur als String erkannt 
    > x[zwei]
    Fehler: Objekt 'zwei' nicht gefunden
    # die Namen werden auf neue Objekte übertragen
    > y <- x[1:4]
    > y
    eins zwei drei vier 
       1    2    3    4

**Namen** = `string`?! Ja, korrekt geraten, der Begriff `string`
bedeutet, dass es sich um eine Reihe von Buchstaben handelt. Für eine
Reihe von Buchstaben gelten andere Regeln als für `numeric`'s (Zahlen,
diese werden auch als `double`) bezeichnet. Grundsätzlich gilt für
`strings`, dass sie eben in Anführungszeichen weiter gegeben werden:

    > Satz <- "Ich bin ein String"
    > Satz
    [1] "Ich bin ein String"
    # wobei der String auch anders entstehen kann:
    > Satz.auch <- c("Auch ", "ich ", "bin ein String")
    > Satz.auch
    [1] "Auch ""ich ""bin ein String"
    # der Vektor aus strings kann auch numerics enthalen
    > Satz.auch[2] <- 2
    > Satz.auch
    [1] "Auch " "2" "bin ein String"
    # allrdings ist er erst jetzt ein "echter" Satz
    > paste(Satz.auch, collapse=" ")
    [1] "Auch  ich  bin ein String"
    # man kann den String mit grep durchsuchen
    > grep("2", Satz.auch)
    [1] 2

Höhere Datenstrukturen
----------------------

### Matrices

Eine Matrix ist primär eine zweidimensionale Anordnung von Werten in
Reihen und Spalten. Wenn wir eine solche Matrix entstehen lassen wollen
können wir dazu mit einem Vektor starten:

    > x <- 1:9
    # eine Matrix mit 3 Spalten und 3 Reihen
    > mat.x = matrix(x, nrow = 3, ncol = 3)
    > mat.x
         [,1] [,2] [,3]
    [1,]    1    4    7
    [2,]    2    5    8
    [3,]    3    6    9
    # Transponieren der Matrix
    > t(mat.x)
         [,1] [,2] [,3]
    [1,]    1    2    3
    [2,]    4    5    6
    [3,]    7    8    9
    # Subset, jetz in zwei Dimensionen
    > mat.x[1:2, 2:3]
         [,1] [,2]
    [1,]    4    7
    [2,]    5    8
    # und noch eins
    > mat.x[3,3]
    [1] 9

### Listen

Der Datentyp `list` ist recht flexibel, er kann salopp gesagt alles
enthalten beispielsweise Matrices, Vektoren, Funktionen und mehr:

    > Liste <- list(
      Funktion = function(x, y){x+y},
      Matrix = matrix(1:9, nrow = 3, ncol = 3),
      Vektor = c(runif(10)),
      )
    > Liste
    $Funktion
    function(x, y){x+y}

    $Matrix
         [,1] [,2] [,3]
    [1,]    1    4    7
    [2,]    2    5    8
    [3,]    3    6    9

    $Vektor
     [1] 0.63860417 0.25105358 0.64089547 0.49214046 0.48232077 0.30024825
     [7] 0.92896802 0.62018010 0.02472022 0.77797613

    # Listen können über das Dollar Zeichen subsettet werden
    > Liste$Funktion
    function(x, y){x+y}
    # oder über die eckigen Klammern
    > Liste[2]
    $Matrix
         [,1] [,2] [,3]
    [1,]    1    4    7
    [2,]    2    5    8
    [3,]    3    6    9

### Dataframes

Der Dataframe ist eine Sonderform der Liste, alle in ihm enthaltenen
Elemente haben die selbe Länge, er taucht gerade deswegen sehr oft auf.
Sowohl das Subsetting, welches wir bei den Listen kennen gelernt haben,
als auch das Subsetting welches bei Matrices angewandt wird kann auch
auf den Typ Dataframe angewandt werden.

Tibble
------

Der `tibble` wird auf der offiziellen Website
`https://tibble.tidyverse.org/index.html` wie folgt beschrieben:

A `tibble`, or tbl\_df, is a modern reimagining of the data.frame,
keeping what time has proven to be effective, and throwing out what is
not. Tibbles are data.frames that are lazy and surly: they do less (...)
and complain more (...).\
[@KirillMueller2020]

Der `tibble` kann also als moderne  Überarbeitung des `data.frame`
verstanden werden. Diese Modernisierung beinhaltete vor allem das faul
machen  der Datenstruktur welche dazu führt, dass sie öfter Fehler
aufwirft. Der `tibble` ist eine sehr gebräuchliche Datenstruktur im
`tidyverse` [@Wickham2019], eine moderne Paketsammlung welcher später
noch mehr Aufmerksamkeit geschenkt wird. Die Regeln für das Subsetting
bei `tibble`'s sind die selben wie beim `data.frame`. Um einen `tibble`
aus, zum Beispiel, Messwerten enstehen zu lassen gilt:

    > tibble(
    + Namen = c("Gustav", "Anna", "Roman", "Veronika"),
    + Alter = c(55, 17, 26, 19))
    # A tibble: 4 x 2
      Namen    Alter
      <chr>    <dbl>
    1 Gustav      55
    2 Anna        17
    3 Roman       26
    4 Veronika    19

Datenimport
-----------

In der Regel liegen Messdaten als [csv]{acronym-label="csv"
acronym-form="singular+short"}-File (.csv), [xslx]{acronym-label="xslx"
acronym-form="singular+short"} oder als handgeschriebenes Manuskript
vor. Das handgeschriebene Manuskript kann gut direkt in die R-Umgebung
übertragen werden, in Form eines `tibble`'s zum Beispiel. Um Daten aus
[csv]{acronym-label="csv" acronym-form="singular+short"}-Dateien zu
importieren steht `read_csv` aus dem Paket `readr` zur Verfügung. So
kann die Tabelle
[\[app::CSV\_Dat1\]](#app::CSV_Dat1){reference-type="ref"
reference="app::CSV_Dat1"}, welche als [csv]{acronym-label="csv"
acronym-form="singular+short"} dort vorliegt wie folgt importiert
werden:

    tab.1 <- read_csv("bsp.csv")
    Parsed with column specification:
    cols(
      V1 = col_double(),
      V2 = col_double(),
      V3 = col_double()
    )
    > tab.1
    # A tibble: 3 x 3
         V1    V2    V3
      <dbl> <dbl> <dbl>
    1     1     4     7
    2     2     5     8
    3     3     6     9
    # umkonvertieren in eine Matrix
    > mat <- as.matrix(tab.1)
    > mat
         V1 V2 V3
    [1,]  1  4  7
    [2,]  2  5  8
    [3,]  3  6  9
    # Dimenstionsnamen einführen
    > dimnames(mat) <- list( c("x", "y", "z"), c("a", "b", "c"))
    > mat
      a b c
    x 1 4 7
    y 2 5 8
    z 3 6 9

Für das Einlesen von [xslx]{acronym-label="xslx"
acronym-form="singular+short"}-Datensätzen steht das Paket `readxl` zur
Verfügung, die dort implementierten Funktionen importieren nach einem
ähnlichen Schema wie `read_csv` Excel-Datensätze. Unter Verwendung von
R-Studio wird der Datenimport stark vereinfacht da dort ein
[GUI]{acronym-label="GUI" acronym-form="singular+short"} den Datenimport
begleitet.

Funktionen und Kontrollstrukturen
---------------------------------

### `if`

`if` untersucht eine Aussage und abhängig vom Ergebnis der Aussage
erzeugt `if` dann eine Ausgabe, ähnlich wie in
[Aschenputtel]{.smallcaps} beim Linsen sortieren zu ihren Täubchen:

...die schlechten ins Kröpfchen, die guten ins Töpfchen.\
[@Grimm2008]

Das hierbei beschriebene Problem könnte in R so gelöst werden:

    > x = "schlecht"
    > if (x == "gut") {
    +   print("ab in's Töpfchen")
    + } else if (x == "schlecht") {
    +   print("ab in's Kröpfchen")
    + }
    [1] "ab in's Kröpfchen"

### `function`

Das `function`-Statement kann [Aschenputtel]{.smallcaps} das Leben
zusätzlich leichter machen, wenn sie tippfaul ist, kann die es in einer
Funktion verpacken, welche immer wieder aufgerufen werden kann. Es ist
wichtig zu bedenken, dass später von der Funktion nur der Name sichtbar
ist und aus diesem Grund ein sinnvoller Name wärmstens zu empfehlen ist.
Die Definition einer Funktion erfolgt immer nach dem selben Schema:

    > Name <- function(Argumente){Funktion (body)}
    # oder:
    > Add <- function(a, b) a + b
    > Add(1, 2)
    [1] 3

Im 2. Fall kann auf die geschweifte Klammer verzichtet werden da die
ganze Definition auf einer Zeile platz hat und der Sinn durchaus
erhalten bleibt. Für [Aschenputtel]{.smallcaps} könnten wir annehmen:

    > Taube <- function(Linse) {
    +         if (Linse == "gut") {
    +           print("ab in's Töpfchen")
    +         } else if (Linse == "schlecht") {
    +           print("ab in's Kröpfchen")
    +         }
    + }
    > x = "schlecht"
    > Taube(x)
    [1] "ab in's Kröpfchen"
    > y = "gut"
    > Taube(y)
    [1] "ab in's Töpfchen"

### `for`

Das `for`-Statement ermöglicht es für mehrere Werte wiederholend die
selbe Aufgabe durchzuführen, denn [Aschenputtel]{.smallcaps} hat ja
nicht nur eine Linse sondern eine ganze Menge. Hierbei soll aber darauf
verwiesen werden, dass der Gebrauch von Schleifen in R eher untypisch
ist da eine Vielzahl von Funktionen zur Verfügung steht welche sie
leicht ersetzen können. Dennoch macht es Sinn sich diese Technik
anzueignen, wenn man sie beherrscht kann man sie häufig sinnvoll
einsetzen. Eine weitere wichtige Information zum Gebrauch von
`for`-Loops bezieht sich auf ihre Ausgabe. Der `for`-Loop ist langsam.
Wenn seine Ausgabe nicht zuvor definiert wurde, das Bedeutet im
Umkehrschluss, dass man sich über die Ausgabe Gedanken machen sollte.
Für [Aschenputtel]{.smallcaps} wollen wir nun die `Taube` automatisieren
damit sie in der Lage ist einen Vektor aus `Linsen` zu klassifizieren
und diese entweder ins `Kröpfchen` oder `Töpfchen` zu werfen:

    1> y <- c("gut", "schlecht", "schlecht", "gut", "gut")
    2> Ausgabe <- vector(mode = "list", length = length(y)) 
    3> for (i in y) {
    4>    Ausgabe[[i]] <- Taube(i) 
    4>}
    [1] "Ab in's Töpfchen"
    [1] "Ab in's Kröpfchen"
    # und so weiter

1.  Zu Beginn erstellen wir den Vektor aus `Linsen` welche dann eben
    `schlecht` oder `gut` sind

2.  der Vektor `y` aus `Linsen` bestimmt die Länge `length` der Ausgabe
    da entsprechend der Anzahl an Linsen auch Klassifizierungen
    vorgenommen werden.

3.  Nun wird in `for` jedes Element `i` (also jede `Linse`) in `y`
    untersucht.

4.  Hier wird definiert mit was untersucht wird, nämlich mit `Taube`.

### `while`

Die `while`-Schleife führt so lange eine Anweisung aus bis eine
Bedingung erfüllt ist, dabei gilt:

    while(Bedingung){Anweisung}

Diese Bedingung kann [Aschenputtel]{.smallcaps} zum Beispiel dabei
helfen so lange Linsen zu sortieren, biss sie quantitativ genug davon
hat um mit diesen einen Eintopf zu kochen. Wir können schreiben:

    1> x = 20
    2> Linsen = 100
    3> while( x != Linsen ){
    4+     print("Esst mehr Linsen")
    5+     x = x + 20
    6+     print(x)
     + }
    [1] "Esst mehr Linsen"
    [1] 40
     usw
    [1] "Esst mehr Linsen"
    [1] 100

Hierbei ordnen wir im ersten Schritt `x` den Wert 20 zu, dabei handelt
es dann um $20$ fiktive Linsen, wir benötigen allerdings $100$ fiktive
Linsen, diese Zuordnung erfolgt in Schritt `2`. Um diese $100$ fiktive
Linsen zu bekommen definieren wir die Bedingung für den `while`-Loop,
diese bedeutet in Worten: So lange `x` nicht `Linsen` ist  läuft der
Loop. Was der Loop dann macht ist in den Zeilen $4$ bis $6$ definiert.
In Zeile $4$ wird zu `x` 20 addiert und daraus das neue `x` definiert,
dieses wird dann in $6$ ausgegeben. Dieser Vorgang geschieht so lange,
bis `x != Linsen` nicht mehr stimmt, also sobald $x = 100$ gilt.

Pakete und das `tidyverse`
--------------------------

Wie bereits angesprochen verwendet man in R Pakete als Erweiterung des
Funktions-Repertoires. Diese Erweiterung erfolgt abhängig von den
Ansprüchen des Anwenders und muss dementsprechend vom Anwender
individuell angepasst werden, eine sehr populäre Paketesammlung, das
`tidyverse` soll jedoch nicht unbeachtet bleiben. Unter einer
Paketsammlung versteht man den Zusammenschluss mehrere Pakete zu einem
neuen und größerem Paket. Im Fall des `tidyverse` ist diese Sammlung auf
das *saubere* Verarbeiten von Daten ausgelegt, der Fokus liegt auf einer
verständlichen Syntax und einem Anwenderfreundlichen *Code*. Das
`tidyverse` beinhaltet folgende Kern-Komponenten [@Wickham2020]:

`ggplot2`

:   Im Namen versteckt sich bereits ein Hinweis auf das Buch *The Grammr
    of Graphics* von [Leland Wilkinson]{.smallcaps} [@LW2005]. Auf Basis
    dieses Buchs werden in `ggplot2` die Weichen für eine verbesserte
    Datenvisualisierung gelegt.

`dplyr`

:   Das Paket `dplyr` enthält eine Vielzahl verschiedener Funktionen zur
    Datenmanipulation.

`readr`

:   Das Paket `readr` enthält Funktionen zum Import von Datensätzen.

`tidyr`

:   Das Paket `tidyr` hilft bei der Ordnung von Daten.

`tibble`

:   Der `tibble` wird in [1.4](#tibble){reference-type="ref"
    reference="tibble"} genauer erklärt.

`purr`

:   Das Paket `purr` ermöglicht weitere Schritte in Richtung
    funktionales Programmieren.

`stringr`

:   Der Name verrät es bereits: es geht um `string`'s, um das
    Programmieren mit diesen zu vereinfachen, wird dieses Paket zur
    Verfügung gestellt.

`forcats`

:   Das Paket `forcats` stellt Tools zur Arbeit mit `factor`'s zur
    Verfügung.

Die Pakete `ggplot2` und `dplyr` finden in `Biotech` viel Anwendung,
dazu später mehr.


Zu Beginn wollen wir uns den verschiedenen Operatoren widmen.
Operatoren gehören zu den reservierten Begri


```r
x = 1
y = 2

x != y
```

```
## [1] TRUE
```

```r
# [1] TRUE
x == y
```

```
## [1] FALSE
```

```r
# [1] FALSE
x < y
```

```
## [1] TRUE
```

```r
# [1] TRUE
x > y
```

```
## [1] FALSE
```

```r
# [1] FALSE

z = 1

x >= z
```

```
## [1] TRUE
```

```r
# [1] TRUE

x <= z
```

```
## [1] TRUE
```

```r
# [1] TRUE

vec1 <- trunc(runif(n = 10, min = 3, max = 7))
vec2 <- trunc(runif(n = 10, min = 3.5, max = 7.7))
vec3 <- trunc(runif(n = 10, min = 3, max = 8))

inf1 <- c(vec1 == vec2)
inf2 <- c(vec2 == vec3)
inf3 <- c(vec1 == vec3)

inf1 &  inf2
```

```
##  [1] FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
```

```r
inf1 && inf2
```

```
## [1] FALSE
```

```r
inf1 |  inf2
```

```
##  [1] FALSE  TRUE  TRUE  TRUE  TRUE FALSE FALSE  TRUE FALSE  TRUE
```

```r
inf1 || inf2
```

```
## [1] FALSE
```

# Auswertung einer 96-Well-Platte

Das Paket Biotech enthält Datensätze welche zum Testen und Experimentieren angefügt wurden, zwei davon sind ELISA-Assays wie sie im Immunologischen Praktikum zum Nachweis vom Humanserumalbumin in Urin entstehen.
Um uns zu verdeutlichen wie groß die Absorptionen in welchen Wells waren können wir durch das anfertigen einer Heatmap schnell die gemessenen Daten visualisiert vor Augen führen.
Eine Heatmap ordnet einem Wert eine Farbintensität zu, hierbei kann die räumliche Anordnung leicht berücksichtigt werden.

```r
str(HSA1)
```

```
## tibble [8 × 14] (S3: tbl_df/tbl/data.frame)
##  $ clara: chr [1:8] "A" "B" "C" "D" ...
##  $ s1   : num [1:8] -42.1 -42 -42 -42 -41.7 ...
##  $ s2   : num [1:8] -4 -1.46 12.95 18.74 28.12 ...
##  $ s3   : num [1:8] -5.088 0.187 10.266 19.309 29.059 ...
##  $ s4   : num [1:8] -7.02 2.07 10.08 17.14 29.86 ...
##  $ s5   : num [1:8] 4.85 -8.15 -8.2 19.12 8.71 ...
##  $ s6   : num [1:8] 4.43 -4.85 13.89 19.07 4.71 ...
##  $ s7   : num [1:8] -6.5 17.61 1.65 21.66 -8.34 ...
##  $ s8   : num [1:8] -5.983 13.987 0.234 18.603 -10.787 ...
##  $ s9   : num [1:8] -6.69 3.3 -10.65 -8.71 -9.47 ...
##  $ s10  : num [1:8] -9.23 4.76 -6.93 -10.88 -4.15 ...
##  $ s11  : num [1:8] 13.94 19.45 -21.71 -4.76 -2.12 ...
##  $ s12  : num [1:8] 17.1426 15.3057 -19.5948 -3.6282 0.0455 ...
##  $ Std  : num [1:8] -5.371 0.265 11.098 18.399 29.012 ...
```

## Rohdatenaufarbeitung und Plotten

Um nur die Informationen zu bekommen, die wir benötigen werden wir ein paar Transformationsschritte durchführen:

```r
daten <- HSA1[,2:13] %>% 
    as.matrix() %>% 
    as.vector()
```
Nun liegen die Messwerte als Vektor in lineare Form vor, diese können wir in einem Grid "fangen", dazu müssen wir dieses Grid erst mal "aufspannen":

```r
# y wird durchnummeriert, als charcter
y  <- paste0(seq(1,12))

# x bekommt die erstest acht Buchstaben des Alphabets
x <- LETTERS[1:8]

# erstellen des Grid
grid.1 <- expand.grid(X = x, Y = y)

# füllen des Grid mit den Daten als Abs
grid.1$Abs <- daten

# plotten des Grid
ggplot( data = grid.1, mapping = aes( X, Y, fill = Abs ) )+
    geom_tile()
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)

## Definition einer Funktion

Um in Zukunft schneller das selbe machen zu können schreiben wir eine Funktion welche diese Aufgabe übernimmt.
R ist eine funktionale Programmsprache, das bedeutet, dass es für viele Probleme bereits Funktionen gibt.
Dies erleichtert gerade nicht-Programmierern das schreiben von Skripten.
Sollte nun jedoch, wie in unserem Fall, keine Funktion für das Problem zur Hand sein wird diese einfach geschrieben:


```r
    # Name:
vis96 <- 
    # die function-Funktion
    function(
             # Die Argumente der Funktion vis96
             HSA.assay) {
    # die eigentliche Funktion
    daten <- HSA.assay[,2:13] %>% 
    as.matrix() %>% 
    as.vector()
    y  <- paste0(seq(1,12))
    # x bekommt die erstest acht Buchstaben des Alphabets
    x <- LETTERS[1:8]
    # erstellen des Grid
    grid.1 <- expand.grid(X = x, Y = y)
    # füllen des Grid mit den Daten als Abs
    grid.1$Abs <- daten
    # plotten des Grid
    vis96 <- ggplot( data = grid.1, mapping = aes( X, Y, fill = Abs ) )+
        geom_tile()
    # Definition der Ausgabe
    return(vis96)
}
```
Wir können diese Funktion nun testen, dazu verwenden wir den zweiten HSA-Assay (HSA2):

```r
vis96(HSA.assay = HSA2)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png)

Die Vorteil des Verwendens einer Funktion werden im Beispiel klar:

* die Funktion bedeutet weniger Tippen
* sie macht den Code übersichtlicher

Ein weiterer Vorteil ist, dass die Funktion leicht geändert werden kann, wenn nun zum Beispiel eine Änderung in der Methode (Format der Ausgabe) des Assays eingeführt wird kann die Funktion einfach geändert werden.

## Vergleich mit den zu erwartenden Werten 

Das Originalpipettierschema, in hellgrün die Standardreihe, in gelb und hellgelb die Studentenurinproben und die tieferen grün töne markieren die Auftragungspunkten des Patientenurin, gibt uns eine Idee, welche Messwerte wo zu erwarten sind:
![hellgrün die Standardreihe, in gelb und hellgelb die Studentenurinproben und die tieferen Grüntöne markieren die Auftragungspunkten des Patientenurin](pipettierschema.jpg)

So wird klar, dass beim ersten Test (HSA1) dieses Schema richtig angewandt wurde, abhängig von der Konzentration variiert auch der die Intensität der jeweils gemessenen Zelle.
Besonders klar deutlich wird das anhand der Standardreihen sowie den Patientenproben.
Ins Auge fallen die Werte F5 und F6.
Für den 2. Assay können wir einen ähnlichen Zusammenhang feststellen, auch auf diesem ist vor allem die Standardreihe sichtbar.

# Konzentrationsbestimmung für den HSA-Assay

# Rohdatenverarbeitung

Vor Beginn der Berechnung der HSA-Konzentration in den Proben müssen wir den Mittelwert der Standardreihen berechnen, die Herausforderung besteht hierbei darin die passenden Spalten zu selektieren.
Anhand des Pipettierschemas kann nachvollzogen werden, dass die entsprechenden Auftragungen in den Spalten 2 bis 4 liegen.

```r
# wo liegen die Werte der Standards
stdr <- HSA1[3:5]  %>% 
    rowMeans()
```
Aus dieser Standardreihe können wir dann wieder, wie bereits vorher geschehen, eine Regression errechnen:

```r
# Zuordnung der Konzentrationen
conc <- c(0, 5, 10, 15, 20, 30, 40, 50)

# welche Konzentrationen gibt das Modell für die Standards aus
conc_eval(abs_P = stdr, abs_std = stdr, conc_std = conc)
```

```
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.3909 -2.3392 -0.0641  3.1028  7.7572 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.6994     3.0164   0.563 0.593584    
## abs_std       0.9200     0.1107   8.308 0.000165 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.338 on 6 degrees of freedom
## Multiple R-squared:   0.92,	Adjusted R-squared:  0.9067 
## F-statistic: 69.03 on 1 and 6 DF,  p-value: 0.0001649
```

```
## [1] -3.241947  1.943520 11.910019 18.626572 28.390852 31.424134 38.704011
## [8] 42.242840
```

```r
# Definition einer Funktion
HSA.conc.eval <- function(HSA,
         mes) {
  stdr <- HSA[3:5]  %>%
    rowMeans()
  conc <- c(0, 5, 10, 15, 20, 30, 40, 50)
  eval <- conc_eval(abs_P = mes,
            abs_std = stdr,
            conc_std = conc)
  return(eval)
}

# Testen der Funktion
HSA.conc.eval(HSA1, HSA1[3:5])
```

```
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.3909 -2.3392 -0.0641  3.1028  7.7572 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.6994     3.0164   0.563 0.593584    
## abs_std       0.9200     0.1107   8.308 0.000165 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.338 on 6 degrees of freedom
## Multiple R-squared:   0.92,	Adjusted R-squared:  0.9067 
## F-statistic: 69.03 on 1 and 6 DF,  p-value: 0.0001649
```

```
##           s2        s3        s4
## 1 -1.9853020 -2.981952 -4.758589
## 2  0.3546584  1.871299  3.604604
## 3 13.6144345 11.144476 10.971146
## 4 18.9443445 19.464336 17.471036
## 5 27.5675322 28.434184 29.170838
## 6 32.1607880 32.724112 29.387501
## 7 39.7006606 41.173969 35.237403
## 8 40.8273083 39.354000 46.547212
```
Wenn wir uns dieses Ergebnis ansehe, fällt auf, dass die Leerwerte negativ sind, das kann aber (real) nicht sein.

## Berücksichtigung des Verdünnungsfaktor

Die Verdünnungsfaktoren sind Bekannt, sie müssen lediglich mit-einberechnet werden.
Man könnte die Funktion zusätzlich erweitern:

```r
HSA.conc.eval <- function(HSA,
         mes,
         FV = 1) {
  stdr <- HSA[3:5]  %>%
    rowMeans()
  conc <- c(0, 5, 10, 15, 20, 30, 40, 50)
  eval <- conc_eval(abs_P = mes,
            abs_std = stdr,
            conc_std = conc)
  return(eval * FV)
}

# Testen mit dem zweiten Datensatz
HSA.conc.eval(HSA2, HSA2[3:5])
```

```
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -4.455 -3.896 -0.661  1.751  6.856 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  -39.560      6.824  -5.797  0.00115 ** 
## abs_std       37.805      4.106   9.207 9.26e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.853 on 6 degrees of freedom
## Multiple R-squared:  0.9339,	Adjusted R-squared:  0.9229 
## F-statistic: 84.76 on 1 and 6 DF,  p-value: 9.261e-05
```

```
##         s2       s3        s4
## 1 -6.63252 -7.69105 -5.384965
## 2  5.99424  5.99424  5.275952
## 3 13.59298 15.59663 12.496644
## 4 18.99905 18.58320 19.112461
## 5 25.38804 23.68683 24.291701
## 6 31.09654 31.13435 29.470941
## 7 38.88430 41.15258 39.526983
## 8 44.63061 43.38306 41.417216
```
Da nach dem Auftragsschema die Verdünnungsfaktoren auch in den Spalten variieren können müssen wir uns diese noch ein mal genauer ansehen.

```r
dat.1 <- tibble(
 "FV1000" = c(rowMeans(HSA2[,6:7]),
              rowMeans(HSA2[1:4, 8:9])
              ),
 "FV5000" = c(rowMeans(HSA2[5:8, 8:9]),
              rowMeans(HSA2[,10:11])
              )
) 
conc.1 <- tibble(
  "aus FV1000" = HSA.conc.eval(HSA2, dat.1$"FV1000", 1000),
  "aus FV5000" =  HSA.conc.eval(HSA2, dat.1$FV5000, 5000)
)
```

```
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -4.455 -3.896 -0.661  1.751  6.856 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  -39.560      6.824  -5.797  0.00115 ** 
## abs_std       37.805      4.106   9.207 9.26e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.853 on 6 degrees of freedom
## Multiple R-squared:  0.9339,	Adjusted R-squared:  0.9229 
## F-statistic: 84.76 on 1 and 6 DF,  p-value: 9.261e-05
## 
## 
## Call:
## stats::lm(formula = conc_std ~ abs_std)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -4.455 -3.896 -0.661  1.751  6.856 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  -39.560      6.824  -5.797  0.00115 ** 
## abs_std       37.805      4.106   9.207 9.26e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.853 on 6 degrees of freedom
## Multiple R-squared:  0.9339,	Adjusted R-squared:  0.9229 
## F-statistic: 84.76 on 1 and 6 DF,  p-value: 9.261e-05
```

```r
conc.1
```

```
## # A tibble: 12 x 2
##    `aus FV1000` `aus FV5000`
##           <dbl>        <dbl>
##  1      -10640.      -97431.
##  2      -12719.      -65202.
##  3       -1945.      -79662.
##  4       24802.      -59909.
##  5       -5839.      -95918.
##  6      -15687.     -135330.
##  7      -16235.      -91854.
##  8       -4875.      -52727.
##  9       -2531.      -89870.
## 10      -20337.      -93650.
## 11        2743.      -81175.
## 12      -19467.      -60855.
```

