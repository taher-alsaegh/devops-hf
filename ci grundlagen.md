## Definition und Ziele von Continuous Integration (CI)
- Continuous Integration (CI) ist eine Entwicklungspraxis, bei der Teammitglieder ihren Code regelmässig in ein gemeinsames Repository integrieren. Jede Integration wird durch einen automatisierten Build mit Tests verifiziert, um Fehler schnell zu erkennen.

## Kernprinzipen von CI
- **Häufige Integration**: Entwickler integrieren ihren Code mehrmals täglich
- **Automatisierte Builds**: Jede Integration löst einen automatisierten Build-Prozess aus
- **Automatisierte Tests**: Teil jedes Builds sind automatisierte Tests
- **Schnelles Feedback**: Entwickler erhalten sofort Rückmeldung über den Erfolg ihrer Integration
- **Fehlerfreie Hauptlinie**: Das Hauptziel ist, den Hauptentwicklungszweig stets in einem funktionsfähigen Zustand zu halten <br>
- CI sorgt dafür das Fehler frühzeitig erkannt werden und der Entwicklungsprozess nicht beeinträchtigt wird
- Bei jedem commit auf die Main-Branch läuft diese CI durch und prüft, ob der neue Code durch die CI "Stepts" erfolgreich durchgeht.
- Ziel von CI ist es, Fehler und Integrationsprobleme so früh wie möglich zu erkennen und zu beheben


## Continuous Integration vs. Continuous Delivery/Deployment

![[tag04_theorie_ci_vs_cd.png]]

- **Continuous Integration (CI)**: Häufiges Mergen von Code in ein zentrales Repository, gefolgt von automatischem Build und Test jedes Changes. Das Ziel ist eine laufend integrierte, getestete Codebasis. CI endet typischerweise damit, dass ein getestetes Artifact (z. B. eine ausführbare Datei, ein Container-Image) bereitsteht.

- **Continuous Delivery (CD)**: Baut auf CI auf. Der neu gebaute und getestete Code wird kontinuierlich in eine Staging-Umgebung oder ein Release-Repository ausgeliefert, sodass er jederzeit für ein Deployment bereitsteht. Wichtig: Der Schritt in die Produktionsumgebung erfolgt bei Continuous Delivery noch manuell – beispielsweise muss ein Operator oder das Team den endgültigen „Go-Live"-Knopf drücken. Die Pipeline automatisiert also alles bis kurz vor die Produktivsetzung.

- **Continuous Deployment (CD)**: Hier geht man noch einen Schritt weiter. Jeder erfolgreich getestete Build wird automatisch in Produktion ausgerollt, ohne dass ein Mensch eingreifen muss. Es gibt keine manuelle Freigabe mehr. Diese Methode verlangt sehr hohe Testsicherheit und Monitoring, da Fehler direkt Kunden betreffen könnten. Continuous Deployment maximiert die Liefergeschwindigkeit – jedes Commit, das die Pipeline passiert, kann in wenigen Minuten live gehen.

>[!info]
>Continuous Delivery wie auch Continuous Deployment setzt eine funktionstüchtige CI voraus.


## Vorteile von Continuous Integration

**Schnellere Fehlererkennung**: Da bei jedem Commit automatisiert getestet wird, fallen Integrationsfehler oder Bugs sofort auf, nicht erst am Ende eines Release-Zyklus.

**Weniger Merge-Konflikte**: Entwickler integrieren kleine Änderungen laufend statt riesiger Codeblöcke auf einmal. Dadurch treten Konflikte im Quellcode seltener auf, und wenn doch, sind sie einfacher zu lösen.

**Bessere Codequalität**: Probleme werden unmittelbar behoben, was zu einer saubereren, stabileren Codebasis führt. Kontinuierliche Tests stellen sicher, dass neue Änderungen die bestehenden Funktionen nicht brechen.

**Kontinuierliches Feedback & höhere Produktivität**: Entwickler erhalten schnell Rückmeldung, ob ihr Code funktioniert. Das fördert kurze Feedback-Schleifen und erlaubt es, früh gegenzusteuern.

**Automatisierung spart Zeit**: Routineaufgaben wie Builds, Tests und Code-Analysen laufen automatisch ab. Deployments werden vorbereitet, ohne dass jemand händisch Skripte ausführen muss – Continuous Integration ist ein Enabler für schnelleres Delivery.

## Herausforderungen bei der Einführung von CI

- **Kultureller Wandel & Zusammenarbeit**: Continuous Integration erfordert ein Umdenken im Team. Entwickler, Tester und Ops müssen eng zusammenarbeiten, was in vielen Organisationen anfangs schwierig ist.

- **Initialer Einrichtungsaufwand**: Die Setup-Phase für CI kann komplex sein. Es müssen Build-Server oder CI-Dienste eingerichtet, Build-Skripte und Testautomatisierung entwickelt werden. Anfangs kostet das Zeit und Mühe, bevor die Automatisierung Früchte trägt.

- **Testabdeckung und -qualität**: CI steht und fällt mit automatisierten Tests. Um wirklich Nutzen zu bringen, muss eine hohe Testabdeckung erreicht werden (Unit-Tests, Integrations-Tests, ggf. End-to-End-Tests). Das Schreiben und Pflegen dieser Tests erfordert Disziplin. Flaky (instabile) Tests können zudem die CI-Pipeline stören und Vertrauen unterminieren – das Team muss in die Zuverlässigkeit der Tests investieren.

- **Umgang mit Fehlschlägen**: In einer CI-Umgebung ist es normal, dass Builds auch mal fehlschlagen. Wichtig ist eine Fehlerkultur, in der ein roter Build sofort Aufmerksamkeit bekommt und schnell korrigiert wird. (If you break it, fix it)

- **Werkzeug- und Infrastrukturvielfalt**: Die Wahl und Integration der richtigen Tools kann zur Hürde werden. Es gibt viele CI-Server und -Services (Jenkins, GitLab CI, GitHub Actions, etc.). Jedes Tool hat seine Konfiguration. (overwhelming)

## Typischer CI-Workflow

![[image.png]]

1. **Lokale Entwicklung**: Entwickler ziehen den aktuellen Code aus dem Versionsverwaltungssystem (Checkout des Repos) und entwickeln in einer privaten Arbeitskopie ein neues Feature oder einen Fix. Idealerweise führen sie lokal schon Tests aus, bevor sie den Code integrieren.

2. **Code Commit/Pull-Request**: Sobald eine Codeänderung fertiggestellt ist, wird sie in das gemeinsame Repository eingespielt (durch Commit + Push in den gemeinsamen Branch, z. B. direkt in main oder via Pull-Request in ein Integrations-Branch).

3. **Trigger durch Repository-Hook**: Der CI-Server oder -Dienst (z. B. GitHub Actions, Jenkins, etc.) überwacht das Repository. Sobald ein neuer Commit erkannt wird, startet automatisch ein Build. (Bei GitHub Actions geschieht dies über Ereignisse wie push oder pull_request.)

4. **Automatisierter Build-Prozess**: Der CI-Dienst führt nun ein vordefiniertes Build-Skript aus. Typische Schritte sind:

    - **Code auschecken**: Der aktuelle Stand des Repositories wird auf dem CI-Server ausgecheckt (geladen).

    - **Kompilieren/Bauen**: Falls nötig, wird der Code kompiliert oder aufgebaut (z. B. Ausführen von npm build für ein Node-Projekt, Erstellen einer ausführbaren Datei bei C/Java, etc.).

    - **Abhängigkeiten installieren**: Externe Bibliotheken oder Pakete werden installiert (z. B. via pip install oder npm install), damit die Anwendung lauffähig ist.

    - **Automatisierte Tests ausführen**: Eine definierte Testsuite läuft durch – typischerweise zuerst Unit-Tests, evtl. Integrationstests. So wird überprüft, ob der neue Code mit dem bestehenden zusammenarbeitet und keine Funktionen bricht.

    - **Statischer Code-Check (optional)**: Manchmal werden zusätzliche Prüfungen durchgeführt, z. B. Linter (Code-Stilprüfung) oder statische Codeanalyse-Tools, um Codequalität und Sicherheitsprobleme zu erkennen.

    - **Artifact erzeugen**: Ist der Build erfolgreich, wird ein Artefakt (Artifact) erstellt – z. B. ein kompilierter Programm-Build, ein Docker-Image oder ein Paket (ZIP/Deployment-Bundle). Dieses Artifact stellt die erfolgreich gebaute Version der Anwendung dar und kann für Deployments verwendet werden.

5. **Benachrichtigung & Ergebnis**: Nachdem Build und Tests durchlaufen sind, informiert das CI-System über das Ergebnis: Bei Erfolg (grüner Build) kann der Code als integriert betrachtet werden. Bei Fehlschlag (roter Build) werden die relevanten Entwickler automatisch benachrichtigt (z. B. via E-Mail, Chat oder im Pull-Request). Die Entwickler beheben den Fehler zeitnah und committen erneut, womit der Zyklus von vorn beginnt. Wichtig ist, dass ein roter Build nicht ignoriert wird – er hält idealerweise weitere Integrationen an, bis das Problem gelöst ist (Stichwort „Stop-the-Line"-Prinzip).
## Continuous Integration mit GitHub Actions

- **Ereignisgesteuert:** Workflows starten auf fast jedem GitHub-Event (push, pull_request, schedule, release, Issue/PR-Events etc.). Passt sich sauber in Dev- und PM-Abläufe ein.

- **Skalierbare Runner:** Cloud-Runner von GitHub, sofort nutzbar, skalieren automatisch. Open-Source gratis, private Repos mit Freikontingent. Viele Tools vorinstalliert → kein eigener Build-Server nötig.

- **Wiederverwendbare Actions:** Riesiger Marketplace mit fertigen Bausteinen (Sprach-Setups, Docker, Slack, etc.). Weniger eigenes Scripting, schneller Aufbau.

- **CI/CD aus einer Hand:** Nicht nur CI – auch Deployments möglich. Nach dem Build direkt automatisiert auf Server/Cloud deployen, wenn gewünscht.


### Beispiel: CI-Prozess für ein kleines Cloud-Projekt

1. **Code-Repository auf GitHub**: Der Projektcode liegt auf GitHub. Entwickler erstellen für Änderungen Feature-Branches und stellen Pull-Requests, oder committen kleine Änderungen direkt in main (wenn Teamprozesse das erlauben).

2. **Trigger der CI-Pipeline**: Bei jedem Push in den main-Branch (oder bei jedem neuen Commit in einem PR) startet automatisch die CI-Pipeline via GitHub Actions. Das Workflow-Skript dafür ist Teil des Repos (z. B. `.github/workflows/ci.yml`).

3. **Build und Unit-Tests**: In der Pipeline wird zunächst die Anwendung gebaut und getestet. Beispielsweise installiert der Workflow alle Dependencies (Bibliotheken) und führt Unit-Tests aus.

4. **Integrationstests (optional)**: Für ein Cloud-Projekt sind oft weitere Tests sinnvoll – z. B. Integrationstests oder Tests gegen Cloud-Services. In einem einfachen Projekt könnten wir einen Integrationstest haben, der z. B. prüft, ob unsere Web-API-Endpunkte antworten. Diese Tests können auch manuell geprüft werden.

5. **Container-Build**: Dockerfile im Repository wird genutzt, um ein frisches Image unserer Anwendung zu erstellen (z. B. `docker build -t myapp:commit123 .`). Dieses Image repräsentiert das getestete Application-Paket.

6. **Artifact/Registry (optional)**: Das gebaute Docker-Image wird als Artifact gespeichert. Oftmals würde man das Image nun in eine Container-Registry pushen (z. B. die GitHub Container Registry oder Docker Hub)

7. **Benachrichtigung und Review**: Sobald die Pipeline durchgelaufen ist, werden die Entwickler informiert. Via Email oder GUI red/green action



## Aufgabe 1: Erste Schritte mit GitHub Actions (Hello CI)

1. Neues Repo anlegen und initialisieren (lokal) oder repo einchecken (from remote repo).
2. Workflow datei ersetellen (falls noch nicht existent):
```
cd DEIN_REPO
mkdir -pv .github/workflows
```
3. `hello-ci.yml` anlegen:
```
touch .github/workflows/hello-ci.yml
```
4. folgendes einfuegen `vim .github/workflows/hello-ci.yml`:
```yaml
name: Hello CI Workflow

on: [push]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Print Hello
        run: echo "🎉 Hello Continuous Integration!"
```
name: = Name der CI Pipeline
on: = Wann soll es ausgefuehrt werden --> in dem Fall bei jedem push
jobs: = Ein CI Job wird definiert. evtl. mehrere jobs moeglich in einem workflow?
hello: = Name des jobs
steps: = Welche Schritte sollen ausgefuerhrt werden
-uses: = 1. Schritt ist das auchecken des repos damit der runner die dateien hat
-name: = Name fuer den kommenden run schritt definieren. Gleich wie # um zu kommentieren
-run: = bash cmd ausfuehren

5. commit & push. Anschliessend im repo unter action den Workflow tracken


## Aufgabe 2: Automatisierter Build und Test eines Projekts

1. Erstelle die benoetigten files:
   ```
   cd DEIN_REPO
   touch app.py && touch test_app.py
   ```
2. Fuege den Code in `app.py` ein:
   ```python
   def add(a, b):
     """Gibt die Summe von a und b zurück."""
     return a + b
   ```
3. Fuege den Code in `app_test.py` ein:
```python
from app import add
def test_add():
     assert add(2, 3) == 5
     assert add(0, 0) == 0
```
4. Lege ein `requirements.txt`  File an:
```
touch requirements.txt && echo "pytest" >> requirements.txt  ``` 5. Erstelle folgendes File unter `.github/workflows/ci-build-test.yml` und fuege diesen inhalt ein:

```yaml
name: Build and Test
on: [push]
jobs:
  build-test:
     runs-on: ubuntu-latest
     steps:
        - uses: actions/checkout@v3
        - uses: actions/setup-python@v4
          with:
             python-version: "3.10"
        - name: Install Dependencies
          run: pip install -r requirements.txt
        - name: Run Tests
          run: pytest -q
```

`pytest -q`: fuehrt den test quiet durch

6.  commit & push. Anschliessend im repo unter action den Workflow tracken


## Aufgabe 3: Erweiterte CI-Pipeline (Linter und Docker-Build)

1. flake8 dependencies hinzufuegen:

```
cd DEIN_REPO
echo "flake8" >> requirements.txt
```
2. Fuege den Teil in `.github/workflows/ci-build-test.yml` ein:

```yaml
- name: Run Linter
    run: flake8 .
```

2. Dockerfile erstellen mit `vim Dockerfile`:

```Dockerfile
# Basis-Image: schlankes Python-Laufzeitsystem FROM python:3.10-slim # Arbeitsverzeichnis im Container WORKDIR /app # Abhängigkeiten kopieren und installieren COPY requirements.txt .
RUN pip install -r requirements.txt
# Application Code kopieren
COPY . .
# Standard Kommando (hier ein einfaches Beispiel, z.B. das Script ausführen) CMD ["python", "app.py"] ``` :wq to save & exit vim

3. Fuege den Teil in `.github/workflows/ci-build-test.yml` ein:

```yaml
- name: Build Docker Image
    run: docker build -t myapp:latest .
```


# Projekt Aufgaben

## Aufgabe 1: Repository-Setup und erste CI-Pipeline (45 Minuten)

**Deine Aufgabe**:

- Nutze das bestehende GitHub-Repository mit dem E-Commerce-Code
- Entwickle eine Strategie für eine grundlegende CI-Pipeline, die bei jeder Code-Änderung automatisch ausgeführt wird
- Die Pipeline soll zunächst einfache Qualitätsprüfungen durchführen und das Team über den Status informieren
- Berücksichtige dabei die Besonderheiten einer Flask/Python-Anwendung und die bereits etablierte Git-Workflow-Struktur

1. SSH into webserver with ssh-key
2. workflows dir erstellen
```
mkdir -pv .github/workflows
```
3.
