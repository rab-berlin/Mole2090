# Mole2090

Whac-a-Mole für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4x4-Tastenfeld.

- 243 Programmschritte, 13 frei
- Datei: [mole.mic](mole.mic) (Version 18)

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

- Die Matrix im Speicher M0-MF ist die Wahrheit. Maulwürfe werden per Addition gesetzt (+3 / +C) und gelöscht (+D / +4). Setzen und Löschen nutzen denselben Code (PLACE).
- Fenster statt MAS: untere Bank mit `EXRL`, obere Bank mit `EXRA, EXRM, EXRA`.
- Treffer: 3 Hex-Stellen in R5-R7, am Ende über `EXRA` und `HXDZ` dezimal.
- Tastenabfrage (SCAN): vier Strobes, die höchste Spalte zählt. Ein Tick alle 3 Abfragen.

| Register | Bedeutung |
|---|---|
| R0 / R1, R2 / R3, R4 / R9 | Feld / Lebensdauer der 3 Maulwurf-Plätze |
| R5 R6 R7 | Treffer hexadezimal |
| RA | Tick-Zähler |
| RB | Misses |
| R8, RC-RF | Scratch |

## Tempo und Schwierigkeit einstellen

Alle Werte stehen als `MOVI`/`CMPI`-Befehle im Programm. Die Adressen gelten für Version 18, nach Änderungen am Programm verschieben sie sich.

| Was | Adresse | Befehl | Wirkung |
|---|---|---|---|
| Tick | 06 | `CMPI 2,A` | Ein Tick alle n + 1 Tastenabfragen (n = 2: alle 3) |
| Startwert der Lebensdauer | 4D | `MOVI F,E` | Lebensdauer ohne Stufe |
| Abzug je Stufe | 4C | `ADD C,C` | Faktor 2 je Stufe, ohne den Befehl 1 je Stufe |
| Treffer je Stufe | 75 | `MOV 6,C` | Die Stufe ist R6 = Treffer / 16 |
| Höchste Stufe | 78 und 7B | `CMPI 5,C`, `MOVI 5,C` | höchstens 7, sonst wird die Lebensdauer 0 |
| Chance auf weiteren Maulwurf | 3E und 3F | `ADD C,C`, `ADDI 2,C` | (2·S + 2) von 16, höchstens F |

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Eingang 1 und die Ausgänge sind frei für das Tastenfeld.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).

Bekannt: Wird beim Wecken eine Taste der Spalte 1 (Felder 0, 4, 8, C) gehalten, kann das Bild zu früh gesendet werden und verloren gehen. Das nächste Bild stimmt wieder.

## Versionen

| Version | Änderung |
|---|---|
| 17 | Unabhängige Maulwürfe mit Lebensdauer, altes Matrixprotokoll (Umschalten mit F) |
| 18 | Matrixprotokoll des ESP2090 Studios: Schlafen und Wecken, kein Umschalten mehr |

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung und nur eine Rücksprungadresse. Ein paar Kniffe, mit denen Mole in 243 Schritte passt:

### Fenster: Logik und Bild tauschen die Plätze

Das Bild liegt in M0-MF, die Logik in R0-RF. `EXRL` tauscht R0-R7 mit M0-M7, `EXRM` tauscht R8-RF mit M8-MF und `EXRA` tauscht R0-R7 mit R8-RF. Wer die untere Bildhälfte bearbeiten will, öffnet sie mit `EXRL`, rechnet in R0-R7 und schließt sie mit demselben `EXRL` wieder. Die Folge `EXRA, EXRM, EXRA` öffnet die obere Hälfte genauso. Jede dieser Folgen ist ihre eigene Umkehrung, und die Logik-Register kehren nach dem zweiten Tausch unverändert zurück. Deshalb kommt das Programm ganz ohne `MAS` aus, und die Maulwurf-Register überleben jeden Zugriff auf das Bild.

### Setzen und Löschen: dieselbe Addition

Ein Maulwurf ist ein Feld aus 2x2 LEDs. Ein Matrixregister hat 4 LEDs, also zwei Felder: Das linke Feld ist der Wert 3, das rechte C. Das Programm addiert diesen Wert in den zwei Registern, in denen die beiden LED-Zeilen des Feldes liegen. Löschen ist dieselbe Addition mit dem Negativ (D = −3, 4 = −C modulo 16). Die Bits zweier Maulwürfe überlappen sich nie, deshalb gibt es keinen Übertrag. Voraussetzung ist, dass Setzen nur auf ein freies und Löschen nur auf ein aktives Feld geht. Das Programm prüft das vorher.

Die Masken stehen in RE (P) und RF (Q). Statt sie jedes Mal zu laden, hat `PLACE` zwei Einsprünge: `PSET` lädt P = 3 und Q = C, `PCLR` lädt P = D und Q = 4, und beide laufen in dieselbe Routine hinein. Der Prolog ist der einzige Unterschied, der Rest ist gemeinsamer Code.

### Das Register aus k mod 8

Der Maulwurf k liegt in zwei Registern der unteren (k < 8) oder oberen Bildhälfte. Das Muster ist in beiden Hälften gleich, mit j = k mod 8: j = 0 und 1 betreffen R0 und R2, j = 2 und 3 betreffen R1 und R3, j = 4 und 5 betreffen R4 und R6, j = 6 und 7 betreffen R5 und R7. Bei geradem j wird P addiert, bei ungeradem Q. Eine Kette `CMPI j,R8` / `BRZ` wählt den Fall, j = 7 läuft durch. Weil das Programm nur einen CALL hat, steht alles in einem Stück: Fenster öffnen, Verzweigung, addieren, `GOTO TAIL`, Fenster schließen, `RET`.

### Treffer: hexadezimal zählen, dezimal anzeigen

Die Treffer zählen in R5, R6 und R7 hexadezimal: `ADDI 1,5`, dann `ADC 6` und `ADC 7` tragen den Übertrag weiter. Am Ende tauscht `EXRA` die drei Register nach D, E und F. Genau dort erwartet `HXDZ` seine Zahl (F ist die höchste Stelle) und liefert sie dezimal zurück, `DISP 3,D` zeigt sie an.

Die mittlere Stelle R6 ist außerdem die Stufe: Sie zählt Treffer geteilt durch 16, ganz ohne Rechnung. Die Stufe S ist R6, höchstens 5, und ab 256 Treffern (R7 > 0) immer 5.

### Zeit: Tick, Lebensdauer und Wahrscheinlichkeit

Das Programm hat keine Uhr. Jede Tastenabfrage (SCAN) kostet etwa gleich viel Zeit, also zählt es Abfragen (die Bilder verlängern den Tick etwas): Alle 3 Abfragen ist ein Tick. Bei jedem Tick sinkt die Lebensdauer jedes aktiven Maulwurfs um 1. `SUBI 1,L` setzt den Zero-Flag, wenn der Wert 0 erreicht, dann läuft er ab. Eine Lebensdauer von 0 bedeutet zugleich „Platz frei“, ein alter Feldwert im Platz bleibt deshalb stehen und wird nie gelöscht. Ein Treffer zählt nur, wenn die Lebensdauer größer als 0 ist.

Der Zufall kommt aus einem einzigen `RND`: D ist der Würfel für die Chance, E das neue Feld. Ob ein weiterer Maulwurf kommt, ist ein Vergleich `CMP D,C` mit Schwelle C = 2·S + 2 (Carry = Würfel < Schwelle): Die Wahrscheinlichkeit ist (2·S + 2)/16, also 12,5 % bis 75 %. Sind alle Plätze frei, kommt sofort einer. Die Start-Lebensdauer ist F − 2·S.

### Die Tastenabfrage in einer Schleife

`SCAN` aktiviert eine Keypad-Zeile nach der anderen mit `DOT 1, 2, 4, 8` und liest die Spalten mit `DIN`. Dass der Strobe von 1 über 2 und 4 bis 8 geht, bildet die Schleife: Jedes Mal wird er mit `ADD C,C` verdoppelt, und der Überlauf nach 8 (Carry) beendet die Abfrage. `DIN` setzt den Zero-Flag genau dann, wenn 0 gelesen wurde, also fällt der Test „keine Taste in dieser Zeile“ direkt in ein `BRZ`. Die Zeilen werden mit einem Sprung rückwärts durchlaufen, das spart ein `GOTO` je Zeile. Bei mehreren Tasten in einer Zeile zählt die höchste Spalte (Vergleiche mit 7, 3 und 1 auf die gelesenen Spaltenbits).

### Eine gehaltene Taste blockiert nichts

Das Programm merkt sich nicht, ob eine Taste schon bearbeitet wurde. Es braucht das auch nicht: Eine Taste auf ein leeres Feld hat keine Folgen, und ein Treffer entfernt den Maulwurf, sodass dieselbe Taste beim nächsten Durchlauf ins Leere geht. Nur ein Maulwurf, der genau auf diesem Feld neu erscheint, wird dann getroffen. Die Hauptschleife kehrt nach jeder Abfrage zurück, gehalten oder nicht.

### Spielende ohne Rücksprung

Nach dem dritten Miss springt `MISS` direkt nach `END`, ohne `RET`. Das macht nichts: Das Programm bleibt danach in einer Endlosschleife stehen, und es gibt nur eine Rücksprungadresse, die ohnehin überschrieben würde.

### Ein Frame, der nie anders aussieht

Die Matrix verlangt die 16 Register im Takt von 9,2 ms. Der Frame `SHOWS` ist deshalb immer derselbe Block ohne eingeschobene Befehle: `DOT 0` bis `DOT F`. Das Programm ruft ihn nach jeder Änderung des Bildes auf (Treffer, Ablauf, neuer Maulwurf) und am Start einmal mit leerer Matrix.
