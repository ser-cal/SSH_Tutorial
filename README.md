SSH verstehen und anwenden
===================

Computer kommunizieren über Netzwerke miteinander. Daher wurde in der Vergangenheit eine Reihe von Regeln für die Kommunikation zwischen ICT-Systemen eingeführt und stetig weiterentwickelt.

Der erste dafür eingesetzte Dienst war **Telnet**. Dieser Dienst ist jedoch nicht sicher. Telnet übermittelt Daten im **Klartext**. Das heisst; die übertragenen Daten können, falls abgefangen, ohne weiteren Aufwand gelesen werden. Um genau dieses Problem zu lösen, wurde Mitte der 90er Jahre ein **sichereres** Protokoll namens **SSH** entwickelt und als zuverlässiger Nachfolger des Telnet-Protokolls eingeführt. Daten die mit **SSH** übermittelt werden, sind verschlüsselt (encryptede) und können nicht ohne entsprechenden **Schlüssel** gelesen (decrypted) werden. 

Dieses Tutorial soll helfen, **SSH** zu verstehen und praktisch anzuwenden. Es wird empfohlen, auch die Praxisbeispiele durchzuarbeiten, um das Wissen zu festigen. 
Diese **Skills** gehören, ähnlich wie z.B. **Netzwerkgrundlagen**, zum unverzichtbaren **Basiswissen** eines Informatikers.

#### Lernziele

Ein gutes Basiswissen in Bezug auf die Anwendung von SSH

#### Voraussetzungen

* PC/Notebook mit min. 8 GB freiem RAM, ca. 20 GB freiem HD-Speicher und einer Ethernet-Netzwerkkarte.
* **Einfache** Linux-Kenntnisse sind von Vorteil, aber nicht zwingend notwendig
* Ein schnelles Netzwerk- (Kabel!) und Internet-Anschluss

#### Allgemeine Hinweise

Die meisten Arbeiten erfolgen auf der Kommandozeile, hier als **Terminal** (*Bash*) bezeichnet.

In der Kommandozeile bzw. im Terminal läuft die "Bash" Shell. Das ist nur die Shell von Linux und noch kein vollständiges Linux System. 

Diese Umgebung wird verwendet, weil benötige Programme wie `git`, `ssh-keygen` in der Powershell nicht zur Verfügung stehen. 

Um sich im Filesystem zurechtzufinden, sind folgende Befehle nützlich:
* `cd /Verzeichnis` wechselt in Verzeichnis z.B. `cd /Users`, alternativ kann die Windows Schreibweise in " verwendet werden, z.B. `cd "C:\Users"`
* Alternativ kann im Windows Explorer jederzeit ein Terminal mittels rechter Maustaste und `Git Bash Here` geöffnet werden.
* `cd ~` Wechsel ins eigene Home-Verzeichnis. Dort werden SSH-Keys etc. abgelegt.
* `cd -` wird auf das zuletzt verwendete Verzeichnis gewechselt.
* Die Laufwerke von Windows stehen als `/c`, `/d/` zur Verfügung, Bsp. `cd /c/Users` und `cd "C:\Users"` sind indentisch
* `ls -l` zeigt die Dateien im aktuellen Verzeichnis an
* `pwd` zeigt den aktuellen Pfad an.
* Die Windows Befehle stehen auch im Terminal zur Verfügung, z.B. `notepad README.md` 

#### Inhaltsverzeichnis

* 01 - [GitHub Account](#-01---github-account)
* 02 - [Git Client](#--02---git-client)
* 03 - [VirtualBox](#--03---virtualbox)
* 04 - [Vagrant](#--04---vagrant)
* 05 - [Visual Studio Code](#-05---visual-studio-code) / [Alternative Markdown Editoren](#alternative-editoren)
* 06 - [Quellenverzeichnis](#-06---quellenverzeichnis)

___

![](../images/GitHub_36x36.png "GitHub") 01 - GitHub Account 
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Als erster Schritt muss ein GitHub-Account eingerichtet werden. Dieser dient uns später als "Cloud-Speicher" unserer Dokumentation und weiteren Dateien.

Folgende Arbeiten müssen gemacht werden:

### Account erstellen
***
1. Auf www.github.com ein Benutzerkonto erstellen (Angabe von Username, E-Mail und Passwort)
2. E-Mail zur Verifizierung des Kontos bestätigen und anschliessend auf GitHub anmelden


### Repository erstellen
***
1. Anmelden unter www.github.com 
2. Innerhalb der Willkommens-Seite auf <strong>Start a project</strong> klicken
3. Unter <strong>Repository name</strong> einen Name definieren (z.B. M300-Services)
4. Optional: kurze Beschreibung eingeben
5. Radio-Button bei <strong>Public</strong> belassen
6. Haken bei <strong>Initialize this repository with a README</strong> setzen
7. Auf <strong>Create repository</strong> klicken
   
### SSH-Key erstellen (lokal)
***

**ACHTUNG**: Auf Windows muss zuerst [Git/Bash](#--02---git-client) installiert werden. Anschliessend können die Befehle in der Git/Bash ausgeführt werden. Dabei handelt es sich nur um die Shell von Linux, die auf Windows ausgeführt wird. Alternativ können Sie für die meisten Befehle auch die *PowerShell* verwenden.

1.  Terminal (*Bash*) öffnen
2.  Folgenden Befehl mit der Account-E-Mail von GitHub einfügen:
    ```Shell
      $  ssh-keygen -t rsa -b 4096 -C "beispiel@beispiel.com"
    ```
3. Neuer SSH-Key wird erstellt:
    ```Shell
      Generating public/private rsa key pair.
    ```
4. Bei der Abfrage, unter welchem Namen der Schlüssel gespeichert werden soll, die Enter-Taste drücken (für Standard):
    ```Shell
      Enter a file in which to save the key (~/.ssh/id_rsa): [Press enter]
    ```
5. Nun kann ein Passwort für den Key festgelegt werden. Ich empfehle dieses zu setzen und anschliessend dem SSH-Agent zu hinterlegen, sodass keine erneute Eingabe (z.B. beim Pushen) notwendig ist:
    ```Shell
      Enter passphrase (empty for no passphrase): [Passwort]
      Enter same passphrase again: [Passwort wiederholen]
    ```

### SSH-Key dem SSH-Agent hinzufügen 
***

**Windows und Linux**

Datei %HOME%/.ssh/id_rsa.pub oder $HOME/.ssh/id_rsa.pub in Zwischenablage kopieren.

**macOS**

1.  Terminal (*Bash*) öffnen
2.  SSH-Agent starten:
    ```Shell
      $ eval "$(ssh-agent -s)"
      Agent pid 931
    ```
3.  Ab Version macOS High Sierra 10.12.2 muss das `~/.ssh/config` File angepasst werden, damit SSH-Keys automatisch dem SSH-Agent hinzugefügt werden:
    ```Shell
      $ sudo nano ~/.ssh/config
      
      Host *
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
    ```
4.  Nun muss der Schlüssel dem Agent nur noch hinzugefügt werden:
    ```Shell
      $ ssh-add -k ~/.ssh/id_rsa
    ```
5.  Der SSH-Key muss nun nur noch kopiert und anschliessend dem GitHub-Account hinzugefügt werden (siehe "SSH-Key hinzufügen"):
    ```Shell
      $ cat ~/.ssh/id_rsa.pub
      # Kopiert den Angezeiten Inhalt der id_rsa.pub Datei in die Zwischenablage
    ``` 

### SSH-Key hinzufügen
***
1.  Anmelden unter www.github.com
2.  Auf Benutzerkonto klicken (oben rechts) und den Punkt <strong>Settings</strong> aufrufen
3.  Unter den Menübereichen auf der linken Seite zum Abschnitt <strong>SSH und GPG keys</strong> wechseln
4.  Auf <strong>New SSH key</strong> klicken
5.  Im Formular unter <strong>Title</strong> eine Bezeichnung vergeben (z.B. MB SSH-Key)
6.  Den zuvor kopierten Key mit <i>CTRL + V</i> einfügen und auf <strong>Add SSH key</strong> klicken
7.  Der Schlüssel (SSH-Key) sollte nun in der übergeordneten Liste auftauchen


> Weiter Infos zu SSH-Keys in Zusammenhang mit GitHub und dem SSH-Agent findet man unter:

> **GitHub-Help:**  https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

> **Wikipedia:**    https://en.wikipedia.org/wiki/Ssh-agent


![](../images/Git_36x36.png "Git Client")  02 - Git Client
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Damit die Arbeiten lokal auf dem eigenen PC erfolgen können, muss der sogenannte "Git Client", auf Windows "Git/Bash" installiert werden. Dieser ermöglicht uns,
Cloud-Repositories zu klonen, zu pullen (herunterladen) oder ein lokales Repository zu pushen (hochladen).

Hierzu müssen folgende Schritte durchgeführt werden: 

### Client installieren
***
1. Für die Client-Installation muss der Installer unter [dieser Webseite](https://git-scm.com/downloads) heruntergeladen werden 
2. Die Installation erfolgt GUI-basiert, jedoch Standard (ohne speziellen Anpassungen). Daher wird an dieser Stelle auf eine Erklärung verzichtet.
3. Sobald der Vorgang abgeschlossen wurde, kann mit der Konfiguration fortgefahren werden.
