# Gate 0 — Den vorhandenen Rushbot zuverlässig lauffähig machen

**Status:** BLOCKING  
**Priorität:** P0 / höchste technische Priorität  
**Gilt für:** `mleem97/RR_BOT_AI`  
**Muss abgeschlossen sein, bevor:** Dataset-Aufbau, Modelltraining, Behavior Cloning, Reinforcement Learning, Vision-Modelle oder autonome AI-Policies als produktive Botlogik integriert werden.

---

## 1. Verbindliche Grundregel

Das Projekt darf nicht mit einer neuen AI-Architektur beginnen, solange der vorhandene klassische Bot nicht reproduzierbar installiert, gestartet, verbunden, beobachtet, gesteuert und über vollständige Matches betrieben werden kann.

Die vorhandene Botlogik wird deshalb nicht nur als historische Referenz behandelt. Sie ist die erste funktionsfähige Baseline, die:

1. die technische Verbindung zu Emulator und Android-Gerät beweist;
2. Screenshots beziehungsweise Videoframes zuverlässig erfasst;
3. Klicks, Swipes und Android-Keyevents korrekt ausführt;
4. die wesentlichen Spielzustände erkennt;
5. einen vollständigen Matchablauf ohne manuellen Eingriff durchführen kann;
6. reproduzierbare Logs, Screenshots, Zustandsübergänge und Aktionen erzeugt;
7. später als `LegacyPolicy` und Sicherheitsfallback der AI-Version dient.

**Ohne bestandene Gate-0-Abnahme gilt das Repository nicht als entwicklungsbereit für AI-Arbeit.**

---

## 2. Warum Gate 0 zwingend vor dem Training liegt

Ein Modell kann nur sinnvoll trainiert und bewertet werden, wenn die technische Umgebung zuverlässig funktioniert. Andernfalls würden Fehler aus Capture, ADB, Koordinaten, Templates oder Zustandslogik fälschlich als Modellfehler erscheinen.

Gate 0 trennt deshalb vier Problemklassen voneinander:

| Schicht | Gate-0-Frage |
|---|---|
| Geräteverbindung | Ist das gewünschte Android-Ziel eindeutig erreichbar und steuerbar? |
| Capture | Kommen aktuelle, vollständige und zeitlich korrekt zuordenbare Frames an? |
| Wahrnehmung | Erkennt der klassische Bot Home, Match, Ergebnis, Dialoge und Boardzustände? |
| Entscheidung und Aktion | Führt die bestehende Regel-Policy nur zulässige Aktionen am richtigen Ziel aus? |

Erst wenn diese Schichten einzeln und gemeinsam funktionieren, darf eine lernende Policy mit der LegacyPolicy verglichen werden.

---

## 3. Aktuell erkennbare Blocker im Repository

Die folgenden Punkte müssen im Gate-0-Audit bestätigt und behoben werden. Die Liste ist ausdrücklich ein Startpunkt und keine Behauptung, dass dies bereits alle Laufzeitfehler sind.

### 3.1 Widersprüchlicher ADB-Unterbau

- `Src/bot_handler.py` erklärt, ADB sei durch `pure-python-adb` ohne manuelle Installation verfügbar.
- `Src/port_scan.py` ruft dagegen fest `.scrcpy\\adb` auf.
- `install.bat` installiert weder Android Platform Tools noch ein lokales `.scrcpy`-Paket.
- Damit ist der tatsächliche Besitzer des ADB-Prozesses derzeit nicht eindeutig definiert.

**Zielzustand:** Es gibt genau eine dokumentierte ADB-Strategie mit explizitem Binary-Finder, Serverstart, Device-Discovery, Verbindungsaufbau und Diagnoseausgabe.

### 3.2 Fehleranfälliger initialer Verbindungsaufbau

`Src/bot_core.py` versucht bei fehlendem `adb_device`, einen `adb connect`-Befehl über die allgemeine `shell()`-Methode auszuführen. Der Fallback von `shell()` ist jedoch für Befehle innerhalb eines bereits erreichbaren Android-Geräts ausgelegt. Der Host-Verbindungsaufbau und Android-Shellbefehle müssen getrennte APIs erhalten.

**Zielzustand:**

- `HostAdb.connect(serial)` führt Host-ADB-Befehle aus;
- `AndroidDevice.shell(command)` führt Befehle auf einem verbundenen Gerät aus;
- ein nicht verbundenes Gerät kann niemals versehentlich als Shellziel verwendet werden.

### 3.3 Nicht reproduzierbare Installationsaussage

Das Repository bezeichnet sich als Python-3.13-kompatibel, ohne dass eine dokumentierte, automatisierte Installationsmatrix und ein Lockfile die Aussage absichern.

**Zielzustand:** Eine saubere Maschine kann über einen dokumentierten Befehl installiert werden. Alle verwendeten Pakete werden in CI und mindestens auf dem Referenzsystem tatsächlich importiert.

### 3.4 Auflösung, Orientierung und Koordinaten sind nicht kanonisch

Die Dokumentation nennt eine Emulatorauflösung, während die Botlogik feste Koordinaten verwendet, die auf ein anderes Koordinatensystem hindeuten können. Capture-Auflösung, Android-Displaygröße, Orientierung, Crop, Skalierung und Eingabekoordinaten müssen als ein gemeinsamer Vertrag behandelt werden.

**Zielzustand:** Jede Aktion wird aus normierten Koordinaten oder einem eindeutig versionierten Geräteprofil abgeleitet. Der Bot verweigert Aktionen, wenn das aktive Profil nicht zum erkannten Frame passt.

### 3.5 Fehlender Fail-fast-Startcheck

Der Startknopf startet aktuell direkt Initialisierung und Botthread. Es fehlt ein verbindlicher Preflight, der vor der ersten Aktion mindestens prüft:

- Python- und Paketversionen;
- ADB-Binary und ADB-Server;
- erreichbare Geräte und eindeutige Geräteauswahl;
- Paketname und Startbarkeit des Spiels;
- Capture;
- erwartete Auflösung und Orientierung;
- Vorhandensein und Lesbarkeit der Bildassets;
- Schreibrechte für Logs, Screenshots und Replays;
- Konfigurationsschema.

### 3.6 Keine belastbare Ende-zu-Ende-Abnahme

Ein GUI-Start allein beweist nicht, dass der Bot funktioniert. Es fehlt eine reproduzierbare Abnahme vom leeren Checkout bis zum abgeschlossenen Match.

---

## 4. Referenzsystem und Portabilitätsreihenfolge

Gate 0 wird in einer kontrollierten Reihenfolge abgeschlossen. Dadurch werden Fehler nicht gleichzeitig auf mehreren Plattformen gesucht.

### 4.1 Referenzprofil G0-R1

Das erste verbindliche Referenzprofil ist das System, auf dem die bestehende Legacylogik mit möglichst wenigen Variablen wiederhergestellt werden kann:

- Windows 11;
- unterstützte Python-Version aus der getesteten Kompatibilitätsmatrix;
- BlueStacks 5 als erstes Emulatorziel;
- ein einzelnes Rush-Royale-Fenster;
- festes, versioniertes Displayprofil;
- lokaler ADB-Server;
- LegacyPolicy ohne AI-Inferenz und ohne Training.

Python 3.13 bleibt Ziel, wird aber erst als freigegeben markiert, wenn alle Runtime-Abhängigkeiten installiert und der vollständige Smoke-Test bestanden wurden. Falls eine zwingende Abhängigkeit Python 3.13 noch nicht zuverlässig unterstützt, wird übergangsweise eine getestete Runtimeversion als Baseline verwendet und die 3.13-Migration als eigener, messbarer Schritt geführt. Eine Versionsbehauptung darf niemals wichtiger sein als ein funktionierender Bot.

### 4.2 Pflichtprofile nach erfolgreichem Referenzprofil

Nach G0-R1 werden noch innerhalb des Bootstrap-Programms folgende Gerätepfade vereinheitlicht:

1. Android-Gerät über USB-ADB;
2. Android-Gerät über Wireless Debugging beziehungsweise ADB über WLAN;
3. manuell konfiguriertes ADB-over-TCP-Ziel;
4. später die Linux-Emulatoradaption;
5. scrcpy-basierter Capturepfad als optimierte Alternative zum ADB-Screencap.

Die fachliche Botlogik darf nicht wissen, ob das Ziel BlueStacks, ein echtes Gerät oder die spätere Linux-Adaption ist. Sie erhält ausschließlich den gemeinsamen `DeviceAdapter`.

---

## 5. Gate-0-Arbeitsphasen

## G0.0 — Bestand einfrieren und reproduzierbaren Fehler erfassen

### Aufgaben

- aktuellen `main`-Stand taggen oder mit unveränderlichem Commit dokumentieren;
- Original-Fork und Upstream-Commit eindeutig festhalten;
- vorhandenen Bot auf dem vorgesehenen Referenz-PC exakt nach aktueller README installieren;
- vollständiges Terminal-, GUI- und ADB-Log erfassen;
- jeden Fehler mit Zeitpunkt, Befehl, Traceback, Systemdaten und erwartetem Verhalten dokumentieren;
- keine Architekturänderung durchführen, bevor der erste Fehler reproduziert ist;
- vorhandene Assets, Templates und Konfigurationsdateien inventarisieren;
- bekannte funktionierende Altversionen, Screenshots oder Videos als Referenz sichern.

### Ergebnis

Ein `bootstrap-baseline-report` beschreibt, was heute tatsächlich startet, wo der Start abbricht und welche manuellen Schritte bisher nötig sind.

---

## G0.1 — Reproduzierbare Python- und Paketumgebung

### Aufgaben

- `requirements.txt` gegen einen modernen, gelockten Paketaufbau ablösen oder ergänzen;
- Kernruntime und optionale AI-/Trainingsabhängigkeiten trennen;
- Windows-, Linux-, NVIDIA- und Intel-Extras getrennt definieren;
- nur Pakete installieren, die für LegacyMode wirklich benötigt werden;
- Import-Smoke-Test für jedes Runtimepaket hinzufügen;
- veraltete oder nicht installierbare Pakete ersetzen;
- Python-Kompatibilitätsmatrix in CI ausführen;
- Installationsskripte idempotent machen;
- Installation darf kein bestehendes virtuelles Environment ungefragt löschen;
- klare Fehlermeldung mit Korrekturanweisung ausgeben.

### Abnahme

Auf einer sauberen Maschine funktionieren mindestens:

```text
bootstrap
runtime import smoke test
configuration validation
GUI dry start
```

ohne manuelle Bearbeitung von Python-Dateien.

---

## G0.2 — Einheitlicher ADB- und DeviceAdapter

### Aufgaben

- ADB-Binary über Konfiguration, Umgebungsvariable, PATH und bekannte Installationsorte finden;
- bei fehlendem ADB mit klarer Installationsanweisung abbrechen;
- Host-ADB-Befehle von Android-Shellbefehlen trennen;
- ADB-Server kontrolliert starten und Status protokollieren;
- `adb devices -l` robust parsen;
- Zustände `device`, `offline`, `unauthorized` und `no permissions` unterscheiden;
- explizite Geräteauswahl bei mehreren Geräten verlangen;
- BlueStacks-Ports nicht blind über zehntausende Ports scannen;
- bekannte Ziele zuerst prüfen und optional konfigurierbares Scanning verwenden;
- USB-, WLAN- und TCP-Ziele über dieselbe Schnittstelle abbilden;
- Reconnect mit Backoff und maximaler Versuchszahl implementieren;
- Device-Fingerprint und Displayinformationen loggen;
- keine Aktion an ein unerwartetes Gerät senden.

### Abnahme

Der Diagnosebefehl zeigt für jedes gefundene Gerät:

```text
serial
type
connection state
Android version
display size
orientation
package installed
package running
capture available
input available
```

---

## G0.3 — Preflight- und Doctor-Befehl

Ein eigener Doctor-Befehl wird Pflichtbestandteil der Anwendung. Er führt noch keine Spielaktion aus.

### Pflichtprüfungen

1. Repositorystruktur;
2. Konfigurationsschema;
3. Pythonversion;
4. Runtimeimports;
5. ADB-Binary;
6. ADB-Server;
7. Geräteauswahl;
8. Autorisierung;
9. Spielpaket;
10. Appstart im Diagnosemodus;
11. Screencapture;
12. Auflösung und Orientierung;
13. Assetinventar;
14. Template-Lesbarkeit;
15. Log- und Replaypfade;
16. CPU-, RAM- und optional VRAM-Baseline;
17. Systemzeit und monotone Zeitquelle;
18. Not-Aus-Konfiguration.

### Ausgabeformat

Jeder Check liefert:

```text
check_id
status: PASS | WARN | FAIL | SKIP
duration_ms
observed
expected
remediation
```

Der Bot darf nur dann in einen aktiven Modus wechseln, wenn alle sicherheitskritischen Checks `PASS` liefern.

---

## G0.4 — Capturepfad zuerst isoliert beweisen

### Aufgaben

- ADB-Screencap als robuste Baseline implementieren;
- scrcpy als optionalen Low-Latency-Captureadapter implementieren;
- Frame-ID und monotone Capturezeit vergeben;
- stale Frames erkennen und verwerfen;
- BGR/RGB-Konvertierung zentralisieren;
- Auflösung, Orientierung und Crop im Frame-Metadatum führen;
- Screenshotdateien nicht als ungesicherten globalen Kommunikationskanal zwischen Threads verwenden;
- Ringbuffer beziehungsweise Latest-Frame-Puffer einführen;
- Capturefehler und Reconnect getrennt behandeln;
- Diagnoseansicht mit unverändertem Rawframe und Overlayframe bereitstellen.

### Abnahme

- 1000 aufeinanderfolgende Frames ohne beschädigte Bilddaten;
- keine rückwärts laufenden Zeitstempel;
- stale-frame-Schutz nachgewiesen;
- Capture kann unabhängig von GUI und Botpolicy gestartet und gestoppt werden;
- Displaywechsel führt zu kontrolliertem Profilwechsel oder sicherem Stopp.

---

## G0.5 — Eingaben und Koordinaten isoliert beweisen

### Aufgaben

- `tap`, `swipe`, `keyevent`, `app_start`, `app_stop` und `reconnect` als Adaptermethoden definieren;
- normierte Koordinaten auf das aktive Displayprofil transformieren;
- sichere Testfläche beziehungsweise Android-Test-App für Eingabetests verwenden;
- Klick- und Swipeergebnis per Folgescreenshot validieren;
- Aktions-ID, Zielkoordinate, Ursprung, Zeit und Ergebnis protokollieren;
- Rate Limits und Mindestabstände einführen;
- globale Not-Aus-Funktion implementieren;
- Aktionen bei unsicherem Zustand blockieren.

### Abnahme

Ein automatisierter Eingabetest bestätigt reproduzierbar alle definierten Testpunkte und Gesten, ohne das Spielkonto für den technischen Test zu benötigen.

---

## G0.6 — Legacy-Wahrnehmung reparieren und testen

### Kritische Zustände

Mindestens folgende Zustände müssen erkannt werden:

- App nicht gestartet;
- Ladebildschirm;
- Home;
- Modusauswahl;
- Deck- oder Kartenansicht;
- Matchmaking;
- Match aktiv;
- Match beendet;
- Sieg;
- Niederlage;
- Verbindungsdialog;
- Update- oder Wartungsdialog;
- Werbung beziehungsweise unerwarteter Overlaydialog;
- unbekannter Zustand.

### Aufgaben

- Templatebestand versionieren und jeder Spielversion zuordnen;
- ROI-Definitionen zentralisieren;
- kritische Erkennungen mit Konfidenz und Begründung ausgeben;
- unbekannten Zustand als normalen, sicheren Zustand behandeln;
- Testdatensatz aus eigenen Referenzscreenshots erstellen;
- Regressionstests für alle kritischen Zustände hinzufügen;
- Debugoverlay mit ROI, Treffer, Score und gewähltem Zustand erzeugen;
- Boardgrid, Karten, Mana und Mergeinformationen separat testen;
- falsche positive Treffer für gefährliche Aktionen priorisiert eliminieren.

### Abnahme

Der vollständige Referenzdatensatz wird offline ausgewertet. Kritische Zustände dürfen nicht ohne ausreichende Evidenz in einen aktiven Aktionszustand wechseln.

---

## G0.7 — LegacyPolicy als explizites Modul

### Aufgaben

- bestehende regelbasierte Entscheidungen aus GUI, Capture und ADB-Code lösen;
- Eingabe als versionierten `GameState` definieren;
- Ausgabe als versionierten `ActionIntent` definieren;
- ActionValidator zwischen Policy und DeviceAdapter setzen;
- Zufallsanteile kontrollierbar und seedbar machen;
- jede Entscheidung mit Regel-ID und Eingangszustand protokollieren;
- Replaybetrieb ohne echtes Gerät ermöglichen;
- SafePolicy für unbekannte und fehlerhafte Zustände implementieren.

### Abnahme

Dieselbe aufgezeichnete Session erzeugt bei gleichem Seed dieselbe Folge fachlicher Entscheidungen. Gerätekoordinaten sind kein Bestandteil der LegacyPolicy.

---

## G0.8 — Vollständiger Matchablauf

### Referenzablauf

```text
Anwendung starten
→ Gerät auswählen
→ Doctor bestanden
→ Spiel starten oder laufende Instanz erkennen
→ Home erkennen
→ vorgesehenen Modus öffnen
→ Match starten
→ Matchmaking erkennen
→ Matchzustand verfolgen
→ nur validierte Legacyaktionen ausführen
→ Ergebnis erkennen
→ Belohnungs-/Ergebnisdialog behandeln
→ zu einem definierten sicheren Zustand zurückkehren
→ Sessionartefakte abschließen
```

### Pflichtartefakte pro Session

- Sessionmanifest;
- Hardware- und Softwareprofil;
- Deviceprofil;
- Ereignislog;
- Zustandsübergänge;
- ActionIntents;
- validierte und verworfene Aktionen;
- ausgewählte Diagnoseframes;
- Fehler und Recoverys;
- Endstatus;
- Ressourcenstatistik.

---

## G0.9 — Recovery, Watchdog und sicherer Betrieb

### Aufgaben

- ADB-Verbindungsverlust erkennen;
- Appcrash erkennen;
- eingefrorenen oder unveränderten Frame erkennen;
- unerwartete Orientierung erkennen;
- Endlosschleifen anhand identischer Zustände erkennen;
- abgestufte Recovery definieren: warten, erneut capturen, zurück, App neu starten, ADB reconnecten, sicher stoppen;
- maximale Recoveryanzahl pro Session definieren;
- Not-Aus aus GUI und Tastatur anbieten;
- bei jeder Unsicherheit keine Eingabe senden;
- Recoveryereignisse im Replay kenntlich machen.

### Abnahme

Kontrolliert provozierte Fehler führen entweder zur dokumentierten Wiederherstellung oder zu einem sicheren Stopp. Ein Fehler darf nicht zu unkontrollierten Klickserien führen.

---

## G0.10 — Installierbares Legacy-MVP

### Lieferumfang

- dokumentierter Einzeiler oder klarer Bootstrap-Befehl je unterstütztem Betriebssystem;
- Doctor-Befehl;
- Observe-only-Modus;
- LegacyMode;
- Konfigurationsassistent;
- Geräteauswahl;
- Diagnoseseite;
- Logexport;
- Session-/Replayexport;
- Deinstallation beziehungsweise saubere Entfernung des virtuellen Environments;
- Troubleshooting-Dokumentation.

---

## 6. Verbindliche Gate-0-Abnahmekriterien

Gate 0 ist erst bestanden, wenn **alle** folgenden Punkte erfüllt sind:

### Installation

- frischer Checkout auf einer sauberen Referenzmaschine;
- kein manuelles Kopieren versteckter `.scrcpy`-, ADB- oder DLL-Dateien;
- reproduzierbare Abhängigkeitsauflösung;
- keine manuelle Änderung im Quellcode;
- verständliche Fehlermeldung bei fehlender Systemvoraussetzung.

### Start und Diagnose

- Doctor läuft ohne GUI;
- GUI kann separat gestartet werden;
- fehlendes Gerät verursacht keinen Traceback ohne Handlungsanweisung;
- mehrere Geräte werden nicht stillschweigend verwechselt;
- alle Pfade funktionieren unabhängig vom aktuellen Arbeitsverzeichnis.

### Capture und Steuerung

- aktuelle Frames werden zuverlässig erfasst;
- Framealter ist messbar;
- Tap und Swipe treffen im aktiven Profil die vorgesehenen Ziele;
- Eingaben werden bei Profilabweichung blockiert;
- Not-Aus stoppt neue Aktionen unmittelbar.

### Wahrnehmung

- alle kritischen Zustände besitzen Testfälle;
- unbekannte Zustände führen zur SafePolicy;
- Templates und ROIs sind versioniert;
- Diagnoseoverlay erklärt die Zustandsentscheidung.

### Ende-zu-Ende

- mindestens zehn vollständige Referenzmatches hintereinander werden ohne Quellcodeänderung gestartet, begleitet und abgeschlossen;
- Netzwerk- oder Dialogfehler werden kontrolliert behandelt;
- kein Lauf endet in einer ungebremsten Klick- oder Restartschleife;
- ein mindestens zweistündiger Soak-Test bleibt ohne nicht behandelten Ausnahmefehler;
- jede Session ist anhand der Artefakte rekonstruierbar.

### Ressourcen

Der LegacyMode muss neben dem Referenzemulator auf einem System mit 16 GB RAM stabil laufen. Gate 0 benötigt noch keine GPU-Inferenz. CPU-, RAM- und optional GPU-Werte werden gemessen und als Ausgangsbasis für spätere AI-Budgets gespeichert.

### Dokumentation

- Installation;
- erster Start;
- Emulator- und Gerätekonfiguration;
- USB-ADB;
- Wireless Debugging;
- ADB-over-TCP;
- Displayprofile;
- Doctor-Auswertung;
- häufige Fehler;
- Log- und Replayexport;
- sichere Beendigung.

---

## 7. Gate-0-Arbeitspakete

Die folgenden IDs sind stabil und können direkt als GitHub-Issues angelegt werden.

### Audit und Reproduktion

- `RB-AI-G0-001` Upstream, Forkbasis und aktuellen Legacycommit dokumentieren.
- `RB-AI-G0-002` Referenzhardware und Referenzemulator erfassen.
- `RB-AI-G0-003` Ist-Installation auf sauberem Windows-System durchführen.
- `RB-AI-G0-004` ersten reproduzierbaren Startfehler protokollieren.
- `RB-AI-G0-005` Asset-, Template- und Konfigurationsinventar erstellen.
- `RB-AI-G0-006` bekannte Altaufnahmen und funktionierende Referenzen sichern.

### Runtime und Installation

- `RB-AI-G0-007` Runtime- und AI-Abhängigkeiten trennen.
- `RB-AI-G0-008` getestete Pythonmatrix definieren.
- `RB-AI-G0-009` lockbare Abhängigkeitsverwaltung einführen.
- `RB-AI-G0-010` idempotenten Windows-Bootstrap erstellen.
- `RB-AI-G0-011` Linux-Bootstrap vorbereiten.
- `RB-AI-G0-012` Import-Smoke-Tests hinzufügen.
- `RB-AI-G0-013` Pfadauflösung unabhängig vom Arbeitsverzeichnis machen.

### ADB und Geräte

- `RB-AI-G0-014` ADB-Binary-Finder implementieren.
- `RB-AI-G0-015` Host-ADB-API implementieren.
- `RB-AI-G0-016` Android-Shell-API von Hostbefehlen trennen.
- `RB-AI-G0-017` robustes Parsing von `adb devices -l` implementieren.
- `RB-AI-G0-018` explizite Geräteauswahl implementieren.
- `RB-AI-G0-019` BlueStacks-Verbindungsprofil implementieren.
- `RB-AI-G0-020` USB-ADB-Profil implementieren.
- `RB-AI-G0-021` Wireless-Debugging-Profil implementieren.
- `RB-AI-G0-022` konfigurierbares TCP-Profil implementieren.
- `RB-AI-G0-023` Reconnect mit Backoff implementieren.

### Doctor

- `RB-AI-G0-024` Doctor-CLI-Grundgerüst erstellen.
- `RB-AI-G0-025` Runtime- und Importchecks implementieren.
- `RB-AI-G0-026` ADB- und Gerätechecks implementieren.
- `RB-AI-G0-027` Spielpaket- und Appstartcheck implementieren.
- `RB-AI-G0-028` Capture- und Displaycheck implementieren.
- `RB-AI-G0-029` Asset- und Templatecheck implementieren.
- `RB-AI-G0-030` maschinenlesbaren Doctor-Report exportieren.

### Capture und Input

- `RB-AI-G0-031` ADB-Screencap-Adapter stabilisieren.
- `RB-AI-G0-032` scrcpy-Captureadapter entkoppeln.
- `RB-AI-G0-033` Frame-Metadaten und monotone Zeit einführen.
- `RB-AI-G0-034` Latest-Frame-Puffer implementieren.
- `RB-AI-G0-035` stale-frame-Schutz implementieren.
- `RB-AI-G0-036` normiertes Koordinatensystem definieren.
- `RB-AI-G0-037` Displayprofile implementieren.
- `RB-AI-G0-038` Tap-, Swipe- und Keyevent-Tests implementieren.
- `RB-AI-G0-039` Not-Aus implementieren.

### Wahrnehmung und LegacyPolicy

- `RB-AI-G0-040` kritische UI-Zustände inventarisieren.
- `RB-AI-G0-041` eigenen Referenzscreenshot-Datensatz erstellen.
- `RB-AI-G0-042` Templates und ROIs versionieren.
- `RB-AI-G0-043` Offline-State-Regressionstests implementieren.
- `RB-AI-G0-044` Diagnoseoverlay implementieren.
- `RB-AI-G0-045` `GameState`-Vertrag einführen.
- `RB-AI-G0-046` bestehende Regeln in `LegacyPolicy` kapseln.
- `RB-AI-G0-047` `ActionIntent`-Vertrag einführen.
- `RB-AI-G0-048` ActionValidator implementieren.
- `RB-AI-G0-049` SafePolicy implementieren.

### Ende-zu-Ende und Stabilität

- `RB-AI-G0-050` Home-bis-Match-Start-Flow reparieren.
- `RB-AI-G0-051` aktiven Matchloop reparieren.
- `RB-AI-G0-052` Ergebnis- und Rückkehrflow reparieren.
- `RB-AI-G0-053` Sessionmanifest und Ereignislog implementieren.
- `RB-AI-G0-054` Replayexport implementieren.
- `RB-AI-G0-055` Watchdog und Recoveryleiter implementieren.
- `RB-AI-G0-056` kontrollierte Fehlerszenarien testen.
- `RB-AI-G0-057` Zehn-Match-Abnahme durchführen.
- `RB-AI-G0-058` Zwei-Stunden-Soak-Test durchführen.
- `RB-AI-G0-059` Ressourcenbaseline erfassen.
- `RB-AI-G0-060` Gate-0-Abnahmebericht freigeben.

---

## 8. Konsequenz für die weitere Roadmap

Die bisherige Roadmap wird logisch wie folgt verschoben:

```text
Gate 0: funktionierender klassischer Bot
→ Gate 1: moderne, getestete Projektbasis
→ Gate 2: Verträge und Replaykern
→ Gate 3: Dataset und Labeling
→ Gate 4: Perception-Modelle
→ Gate 5: lernende Policy
→ Gate 6: Shadow- und Assistbetrieb
→ Gate 7: optimierte lokale AI-Runtime
→ Gate 8: produktionsreife AI-Version
```

Ein Arbeitspaket aus späteren Gates darf zwar recherchiert oder konzeptionell vorbereitet werden. Es darf jedoch nicht als produktiv integriert oder abgeschlossen markiert werden, solange `RB-AI-G0-060` nicht freigegeben ist.

---

## 9. Definition of Ready für die erste AI-Funktion

Die erste AI-Komponente darf erst in den Runtimepfad aufgenommen werden, wenn:

- Gate 0 vollständig bestanden ist;
- dieselben Sessions offline replaybar sind;
- LegacyPolicy und SafePolicy verfügbar sind;
- Capture-, State- und Actionverträge versioniert sind;
- ein AI-Fehler nicht die Gerätesteuerung umgehen kann;
- die AI im Shadowmodus gegen die LegacyPolicy verglichen werden kann;
- ein Rollback auf den funktionierenden LegacyMode jederzeit möglich ist.

Damit ist „der Bot funktioniert“ keine lose Vorphase, sondern ein messbarer, geschützter und dauerhaft regressionsgetesteter Bestandteil des finalen Produkts.
