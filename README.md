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

Zum Spielen mit dem Tempo siehe den Kopfkommentar in [mole.mic](mole.mic) (Tick, Lebensdauer, Chance, Stufenobergrenze).

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
