# Mole2090

Whac-a-Mole für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4x4-Tastenfeld.

- 243 Programmschritte, 13 frei
- Datei: [mole.mic](mole.mic)

## Spiel

- **Felder:** Die Matrix hat 4x4 Felder zu je 2x2 LEDs. Feld 0 ist unten links, Feld 3 unten rechts, Feld F oben rechts.
- **Maulwürfe:** Bis zu 3 leuchten gleichzeitig. Jeder hat eine Lebensdauer, die mit der Zeit abläuft.
- **Treffer:** Die passende Taste 0-F vor Ablauf. Das Feld erlischt.
- **Miss:** Die Lebensdauer läuft ab. Eine Taste auf ein leeres Feld bleibt folgenlos.
- **Spielende:** Bei 3 Misses. Die Anzeige zeigt die Treffer dezimal.
- **Schwierigkeit:** Je 16 Treffer steigt die Stufe S (bis 5). Lebensdauer am Start F − 2·S, Chance auf einen weiteren Maulwurf je Tick (2·S + 2)/16. Ohne aktiven Maulwurf erscheint sofort einer.

## Bedienung

Tastenfeld: Reihe r (0 = unten) liegt an Ausgang r+1, Spalte c (0 = links) an Eingang c+1. Taste = 4·Reihe + Spalte. Im Emulator die Tasten 0-9 und a-f.

## Start

RESET, Matrix einschalten (siehe unten), `HALT` `NEXT` `00` `RUN`. Am Start sendet das Programm ein leeres Bild.

## Geschichte

- Der Urahn ist das japanische Automatenspiel *Mogura Tataki* („Maulwurf-Klopfen“, auch *Mogura Taiji*), 1975 von **Kazuo Yamada** bei der Firma TOGO entwickelt. Bandai brachte es 1977 in Serie.
- Den Namen *Whac-A-Mole* gab ihm **Geoffrey Fechter**, der in Nordamerika die Version baute und vertrieb. Er arbeitete dafür mit der Firma Bob's Space Racers.
- Der Ausdruck hat sich im Englischen zum Sprichwort entwickelt: *Whac-A-Mole* nennt man jede Aufgabe, bei der kaum ein Problem gelöst ist, bevor das nächste hochkommt.
- Das Programm hier hat keinen Hammer, sondern eine Taste je Loch, und das Spiel läuft auf der Matrix des ESP2090.

Quellen: Wikipedia „Whac-A-Mole“. Die Rolle der Bob's Space Racers ist nur dort belegt.

## Technik

**Schreibweise in diesem README:** Ein Register heißt `R0` bis `RF` (Speicher: `M0` bis `MF`), ein Wert beginnt mit `#`, zum Beispiel `#3` oder `#C`. `RC` ist also ein Register, `#C` die Zahl zwölf. Die Programmzeilen sind hier mit `R` und `#` geschrieben, in der Datei `mole.mic` stehen die Nummern ohne Zusatz: `MOVI #C,RE` (Wert `#C` in das Register `RE`) heißt dort `MOVI C,E`.

- Die Matrix im Speicher `M0`-`MF` ist die Wahrheit. Maulwürfe werden per Addition gesetzt (`#3` / `#C`) und gelöscht (`#D` / `#4`). Setzen und Löschen nutzen denselben Code (`PLACE`).
- Fenster statt `MAS`: untere Bank mit `EXRL`, obere Bank mit `EXRA, EXRM, EXRA`.
- Treffer: 3 Hex-Stellen in `R5`-`R7`, am Ende über `EXRA` und `HXDZ` dezimal.
- Tastenabfrage (`SCAN`): vier Strobes, die höchste Spalte zählt. Ein Tick alle 3 Abfragen.

| Register | Bedeutung |
|---|---|
| `R0` / `R1`, `R2` / `R3`, `R4` / `R9` | Feld / Lebensdauer der 3 Maulwurf-Plätze |
| `R5` `R6` `R7` | Treffer hexadezimal |
| `RA` | Tick-Zähler |
| `RB` | Misses |
| `R8`, `RC`-`RF` | Scratch |

## Tempo und Schwierigkeit einstellen

Alle Werte stehen als `MOVI`/`CMPI`-Befehle im Programm. In der Spalte „Befehl“ steht die Zeile so, wie sie in `mole.mic` steht, also ohne `R` und ohne `#`: `MOVI F,E` lädt den Wert `#F` in das Register `RE`. Nach Änderungen am Programm verschieben sich die Adressen.

| Was | Adresse | Befehl | Wirkung |
|---|---|---|---|
| Tick | 06 | `CMPI 2,A` | Vergleich von `RA` mit dem Wert `#2`: ein Tick alle n + 1 Tastenabfragen (n = 2: alle 3) |
| Startwert der Lebensdauer | 4D | `MOVI F,E` | `RE` = `#F`: Lebensdauer ohne Stufe |
| Abzug je Stufe | 4C | `ADD C,C` | `RC` + `RC`: Faktor 2 je Stufe, ohne den Befehl 1 je Stufe |
| Treffer je Stufe | 75 | `MOV 6,C` | Die Stufe ist `R6` = Treffer / 16 |
| Höchste Stufe | 78 und 7B | `CMPI 5,C`, `MOVI 5,C` | Wert `#5`; höchstens `#7`, sonst wird die Lebensdauer 0 |
| Chance auf weiteren Maulwurf | 3E und 3F | `ADD C,C`, `ADDI 2,C` | (2·S + 2) von 16, höchstens `#F` |

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt den Wert `#F` an und hält ihn, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Eingang 1 und die Ausgänge sind frei für das Tastenfeld.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP).

Bekannt: Wird beim Wecken eine Taste der linken Spalte (Eingang 1, Felder 0, 4, 8, C) gehalten, kann das Bild zu früh gesendet werden und verloren gehen. Das nächste Bild stimmt wieder.

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung und nur eine Rücksprungadresse. Ein paar Kniffe, mit denen Mole in 243 Schritte passt. Schreibweise wie oben: `RC` ist ein Register, `#C` ein Wert.

### Fenster: Logik und Bild tauschen die Plätze

Das Bild liegt in `M0`-`MF`, die Logik in `R0`-`RF`. `EXRL` tauscht `R0`-`R7` mit `M0`-`M7`, `EXRM` tauscht `R8`-`RF` mit `M8`-`MF` und `EXRA` tauscht `R0`-`R7` mit `R8`-`RF`. Wer die untere Bildhälfte bearbeiten will, öffnet sie mit `EXRL`, rechnet in `R0`-`R7` und schließt sie mit demselben `EXRL` wieder. Die Folge `EXRA, EXRM, EXRA` öffnet die obere Hälfte genauso. Jede dieser Folgen ist ihre eigene Umkehrung, und die Logik-Register kehren nach dem zweiten Tausch unverändert zurück. Deshalb kommt das Programm ganz ohne `MAS` aus, und die Maulwurf-Register überleben jeden Zugriff auf das Bild.

```
EXRL        ; untere Bank: R0-R7 = M0-M7
  ...       ; hier mit R0-R7 rechnen
EXRL        ; zurück: die Bank steht wieder in M0-M7

EXRA        ; obere Bank: Logik nach R0-R7 ...
EXRM        ; ... R8-RF = M8-MF ...
EXRA        ; ... und M8-MF steht nun in R0-R7
  ...       ; hier mit R0-R7 rechnen
EXRA        ; dieselbe Folge noch einmal schließt das Fenster
EXRM
EXRA
```

### Setzen und Löschen: dieselbe Addition

Ein Maulwurf ist ein Feld aus 2x2 LEDs. Ein Matrixregister hat 4 LEDs, also zwei Felder: Das linke Feld ist der Wert `#3`, das rechte `#C`. Das Programm addiert diesen Wert in den zwei Registern, in denen die beiden LED-Zeilen des Feldes liegen. Löschen ist dieselbe Addition mit dem Negativ (`#D` = −3, `#4` = −12, beides modulo 16). Die Bits zweier Maulwürfe überlappen sich nie, deshalb gibt es keinen Übertrag. Voraussetzung ist, dass Setzen nur auf ein freies und Löschen nur auf ein aktives Feld geht. Das Programm prüft das vorher.

Die Masken stehen in `RE` (P) und `RF` (Q). Statt sie jedes Mal zu laden, hat `PLACE` zwei Einsprünge: `PSET` lädt `RE` = `#3` und `RF` = `#C`, `PCLR` lädt `RE` = `#D` und `RF` = `#4`, und beide laufen in dieselbe Routine hinein. Der Prolog ist der einzige Unterschied, der Rest ist gemeinsamer Code.

```
PSET:  MOVI #3,RE   ; RE = #3   linke Hälfte  +3
       MOVI #C,RF   ; RF = #C   rechte Hälfte +12
       GOTO  PLACE
PCLR:  MOVI #D,RE   ; RE = #D   linke Hälfte  -3
       MOVI #4,RF   ; RF = #4   rechte Hälfte -12
PLACE: ...          ; gemeinsamer Code, addiert RE bzw. RF
```

### Das Register aus k mod 8

Der Maulwurf k liegt in zwei Registern der unteren (k < 8) oder oberen Bildhälfte. Das Muster ist in beiden Hälften gleich, mit j = k mod 8: j = 0 und 1 betreffen `R0` und `R2`, j = 2 und 3 betreffen `R1` und `R3`, j = 4 und 5 betreffen `R4` und `R6`, j = 6 und 7 betreffen `R5` und `R7`. Bei geradem j wird `RE` addiert, bei ungeradem `RF`. Eine Kette `CMPI #j,R8` / `BRZ` wählt den Fall, j = 7 läuft durch. Weil das Programm nur einen CALL hat, steht alles in einem Stück: Fenster öffnen, Verzweigung, addieren, `GOTO TAIL`, Fenster schließen, `RET`.

```
DISP: CMPI #0,R8    ; ist R8 = #0 ?
      BRZ  M0
      CMPI #1,R8    ; ist R8 = #1 ?
      BRZ  M1
      ...           ; so weiter bis #6
M7:   ADD  RF,R5    ; j = 7 läuft durch: R5 = R5 + RF ...
      ADD  RF,R7    ; ... und R7 = R7 + RF
      GOTO TAIL
M0:   ADD  RE,R0    ; j = 0: R0 = R0 + RE ...
      ADD  RE,R2    ; ... und R2 = R2 + RE
      GOTO TAIL
```

### Treffer: hexadezimal zählen, dezimal anzeigen

Die Treffer zählen in `R5`, `R6` und `R7` hexadezimal: `ADDI #1,R5` addiert den Wert `#1` zu `R5`, dann tragen `ADC R6` und `ADC R7` den Übertrag weiter. Am Ende tauscht `EXRA` die drei Register nach `RD`, `RE` und `RF`. Genau dort erwartet `HXDZ` seine Zahl (`RF` ist die höchste Stelle) und liefert sie dezimal zurück, `DISP #3,RD` zeigt sie an.

```
ADDI #1,R5   ; R5 = R5 + #1, Übertrag in den Carry
ADC  R6       ; R6 = R6 + Carry
ADC  R7       ; R7 = R7 + Carry      (#FFF + 1 -> #000)
...
EXRA          ; Ende: R5-R7 stehen jetzt in RD-RF
HXDZ          ; hexadezimal -> dezimal
DISP #3,RD    ; Anzeige RD, RE, RF: Hunderter Zehner Einer
```

Die mittlere Stelle `R6` ist außerdem die Stufe: Sie zählt Treffer geteilt durch 16, ganz ohne Rechnung. Die Stufe S ist `R6`, höchstens 5, und ab 256 Treffern (`R7` > 0) immer 5.

### Zeit: Tick, Lebensdauer und Wahrscheinlichkeit

Das Programm hat keine Uhr. Jede Tastenabfrage (`SCAN`) kostet etwa gleich viel Zeit, also zählt es Abfragen (die Bilder verlängern den Tick etwas): Alle 3 Abfragen ist ein Tick. Bei jedem Tick sinkt die Lebensdauer jedes aktiven Maulwurfs um 1. `SUBI #1,R1` setzt den Zero-Flag, wenn der Wert 0 erreicht, dann läuft er ab. Eine Lebensdauer von 0 bedeutet zugleich „Platz frei“, ein alter Feldwert im Platz bleibt deshalb stehen und wird nie gelöscht. Ein Treffer zählt nur, wenn die Lebensdauer größer als 0 ist.

```
TICK:   ADDI #1,RA   ; RA = RA + #1  Tick-Zähler
        CMPI #2,RA   ; Carry = 1, wenn RA > #2, also bei RA = #3
        BRC  DOTICK  ; Tick fällig
        GOTO LOOP
DOTICK: MOVI #0,RA   ; RA = #0
TK0:    CMPI #0,R1   ; R1 = #0 ? (Platz 0 frei)
        BRZ  TK1
        SUBI #1,R1   ; R1 = R1 - #1  (Lebensdauer)
        BRZ  EX0     ; auf #0 gesunken: Maulwurf läuft ab
```

Der Zufall kommt aus einem einzigen `RND`: `RD` ist der Würfel für die Chance, `RE` das neue Feld. Ob ein weiterer Maulwurf kommt, ist ein Vergleich `CMP RD,RC` mit der Schwelle `RC` = 2·S + 2 (Carry = Würfel < Schwelle): Die Wahrscheinlichkeit ist (2·S + 2)/16, also 12,5 % bis 75 %. Sind alle Plätze frei, kommt sofort einer. Die Start-Lebensdauer ist `#F` − 2·S.

### Die Tastenabfrage in einer Schleife

**Warum nicht die normale Tastatur?** Das Gerät liest seine eigene Hex-Tastatur mit `KIN`. Der Befehl hält das Programm an, bis eine Taste gedrückt ist. Das passt zu Spielen, die auf den Spieler warten (Springer, Kniffel). Bei Mole darf das Programm aber nie warten: Die Maulwürfe laufen ab, neue erscheinen, und die Zeit zählt weiter, auch wenn niemand eine Taste drückt. Mit `KIN` stünde das Spiel bis zum nächsten Tastendruck still. Eine Unterbrechung, die das Programm bei einer Taste aus der Arbeit holt, kennt der 2090 nicht. Also muss das Programm selbst nachsehen, und das so oft, dass ihm keine Taste entgeht.

**Das Keypad.** Dafür dient ein zusätzliches 4x4-Tastenfeld, das an die Ein- und Ausgänge des Geräts angeschlossen wird. Es ist eine Matrix: Jede der 16 Tasten verbindet, wenn sie gedrückt ist, eine Zeilenleitung mit einer Spaltenleitung. Die vier Zeilen hängen an den Ausgängen 1-4, die vier Spalten an den Eingängen 1-4. Das Programm schaltet mit `DOT` immer nur eine Zeile ein (den „Strobe“) und liest mit `DIN` die Spalten. Ein gesetztes Bit im gelesenen Wert heißt: In dieser Zeile ist die Taste dieser Spalte gedrückt. Weil nur eine Zeile aktiv ist, weiß das Programm, welche. So entstehen aus 8 Leitungen 16 Tasten. Dafür muss die Abfrage alle vier Zeilen nacheinander durchlaufen, und das geht nur im Programm, nie von selbst. `DIN` kehrt sofort zurück, auch ohne Taste, und das Spiel läuft weiter.

**Die Schleife.** `SCAN` aktiviert eine Keypad-Zeile nach der anderen und liest die Spalten. Der Strobe ist der Wert in `RC`: `#1`, `#2`, `#4`, `#8`. `DOT RC` legt den Inhalt von `RC` an die Ausgänge. Der Strobe wird jedes Mal mit `ADD RC,RC` verdoppelt, und der Überlauf nach `#8` (Carry) beendet die Abfrage. `DIN R8` schreibt die gelesenen Spaltenbits nach `R8` und setzt den Zero-Flag genau dann, wenn dabei 0 gelesen wurde, also fällt der Test „keine Taste in dieser Zeile“ direkt in ein `BRZ`. Die Zeilen werden mit einem Sprung rückwärts durchlaufen, das spart ein `GOTO` je Zeile. Bei mehreren Tasten in einer Zeile zählt die höchste Spalte (Vergleiche der Spaltenbits mit `#7`, `#3` und `#1`).

```
SCAN: MOVI #1,RC   ; RC = #1   Strobe für Zeile 0
      MOVI #0,RD   ; RD = #0   4 * Zeile
      GOTO SC1
SCN:  ADDI #4,RD   ; RD = RD + #4   nächste Zeile
      ADD  RC,RC   ; RC = RC + RC   Strobe verdoppeln: #1 -> #2 -> #4 -> #8 -> Überlauf
      BRC  NOK     ; Carry: alle 4 Zeilen ohne Taste
SC1:  DOT  RC      ; Ausgänge = RC   diese Zeile einschalten
      DIN  R8      ; R8 = Eingänge   Spaltenbits; Zero, wenn #0
      BRZ  SCN     ; keine Taste: nächste Zeile
FND:  CMPI #7,R8   ; Carry, wenn R8 > #7: Bit 4, Spalte 3
      BRC  F3
      CMPI #3,R8   ; Carry, wenn R8 > #3: Bit 3, Spalte 2
      BRC  F2
      CMPI #1,R8   ; Carry, wenn R8 > #1: Bit 2, Spalte 1
      BRC  F1
NOK:  RET          ; Spalte 0
```

### Eine gehaltene Taste blockiert nichts

Das Programm merkt sich nicht, ob eine Taste schon bearbeitet wurde. Es braucht das auch nicht: Eine Taste auf ein leeres Feld hat keine Folgen, und ein Treffer entfernt den Maulwurf, sodass dieselbe Taste beim nächsten Durchlauf ins Leere geht. Nur ein Maulwurf, der genau auf diesem Feld neu erscheint, wird dann getroffen. Die Hauptschleife kehrt nach jeder Abfrage zurück, gehalten oder nicht.

### Spielende ohne Rücksprung

Nach dem dritten Miss springt `MISS` direkt nach `END`, ohne `RET`. Das macht nichts: Das Programm bleibt danach in einer Endlosschleife stehen, und es gibt nur eine Rücksprungadresse, die ohnehin überschrieben würde.

```
MISS: ADDI #1,RB   ; RB = RB + #1   Misses
      CMPI #3,RB   ; RB = #3 ?
      BRZ  END     ; ja: Spielende, kein RET
      RET
```

### Ein Frame, der nie anders aussieht

Die Matrix verlangt die 16 Register im Takt von 9,2 ms. Der Frame `SHOWS` ist deshalb immer derselbe Block ohne eingeschobene Befehle: `DOT R0` bis `DOT RF` geben die Register `R0` bis `RF` aus. Das Programm ruft ihn nach jeder Änderung des Bildes auf (Treffer, Ablauf, neuer Maulwurf) und am Start einmal mit leerer Matrix.

```
DOT R0     ; Ausgänge = R0   Zeile 0 links
DOT R1     ; Ausgänge = R1   Zeile 0 rechts
...        ; ohne Zwischenbefehl, 9,3 ms je Befehl
DOT RF     ; Ausgänge = RF   Zeile 7 rechts
```
