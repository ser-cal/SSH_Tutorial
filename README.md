[10]: https://git-scm.com/downloads
[20]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows


SSH verstehen und anwenden
===================

Computer kommunizieren über Netzwerke miteinander. Daher wurde in der Vergangenheit eine Reihe von Regeln für die Kommunikation zwischen ICT-Systemen eingeführt und stetig weiterentwickelt.

Der erste dafür eingesetzte Dienst war **Telnet**. Dieser Dienst ist jedoch nicht sicher. Telnet übermittelt Daten im **Klartext**. Das heisst; die übertragenen Daten können, falls abgefangen, ohne weiteren Aufwand gelesen werden. Um genau dieses Problem zu lösen, wurde Mitte der 90er Jahre ein **sichereres** Protokoll namens **SSH** entwickelt und als zuverlässiger Nachfolger des Telnet-Protokolls eingeführt. Daten die mit **SSH** übermittelt werden, sind verschlüsselt (encrypted) und können nicht ohne entsprechenden **Schlüssel** gelesen (decrypted) werden. 

Dieses Tutorial soll helfen, **SSH** zu verstehen und praktisch anzuwenden. Es wird empfohlen, auch die Praxisbeispiele durchzuarbeiten, um das Wissen zu festigen. 
Diese **Skills** gehören, ähnlich wie z.B. **Netzwerkgrundlagen**, zum unverzichtbaren **Basiswissen** eines Informatikers.

### Lernziel

**SSH** richtig verstehen und praktisch anwenden

### Voraussetzungen

* Host (PC/Notebook mit min. 8 GB freiem RAM)
* Ausreichend Platz auf dem Host, um virtuelle Maschinen zu erstellen *(mind. 40GB freier HD- oder SSD-Speicher, falls kein Zugriff auf TBZ-Cloud, AWS-Cloud oder Azure-Cloud)*
* Einfache Linux-Kenntnisse (alles notwendige dazu weiter unten)
* Internetanschluss *(allenfalls VPN-Zugang zu Cloud-Umgebung)*


### Terminal (Kommandozeile) 

![](images/Gitbash_Logo_36x36.png "Gitbash") 

Die meisten Arbeiten erfolgen auf der Kommandozeile; hier als **Terminal** (*Bash*) bezeichnet. In diesem Tutorial wird die **Git-Bash** ([Download-URL][10]) auf einem Windows-System eingesetzt.

In der Kommandozeile bzw. im Terminal läuft die "Bash" Shell. Das ist nur die Shell von Linux und noch kein vollständiges Linux System. 

Diese Umgebung wird verwendet, weil benötige Programme wie `git`, `ssh-keygen` in der Powershell nicht zur Verfügung stehen. 


#### Inhaltsverzeichnis

* 01 - [Wichtige Linux-Kommandos](#-01---Wichtige-Linux-Kommandos)
* 02 - [SSH Keypair erstellen](#-02---SSH-Keypair-erstellen)
* 03 - [Zentrale SSH Konfigurationsfiles](#-03---Zentrale-SSH-Konfigurationsfiles)
* 04 - [Vagrant](#--04---vagrant)
* 03 - [Zentrale SSH Konfigurationsfiles](#-03---Zentrale-SSH-Konfigurationsfiles)
* 06 - [Quellenverzeichnis](#-06---quellenverzeichnis)


___

![](images/Dollarprompt_Logo_36x36.png "SSH") 01 - Wichtige Linux-Kommandos 
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

#### Unix-Befehle

Um sich im Filesystem zurechtzufinden, sind folgende Unix-Commands nützlich:
* `pwd` zeigt den aktuellen Pfad an.
* `cd /Verzeichnis` wechselt in Verzeichnis z.B. `cd /Users`, alternativ kann die Windows Schreibweise in " verwendet werden, z.B. `cd "C:\Users"`
* Alternativ kann im Windows Explorer jederzeit ein Terminal mittels rechter Maustaste und `Git Bash Here` geöffnet werden.
* `cd ~` Wechsel ins eigene Home-Verzeichnis. 
* `cd -` wird auf das zuletzt verwendete Verzeichnis gewechselt.
* Die Laufwerke von Windows stehen als `/c`, `/d/` zur Verfügung, Bsp. `cd /c/Users` und `cd "C:\Users"` sind indentisch
* `ls -l` zeigt die Dateien im aktuellen Verzeichnis an
* `ls -al` zeigt sämtliche Dateien im aktuellen Verzeichnis an (auch die versteckten, wie z.B. **.ssh**, wo sämtliche Keys abgelegt werden)
* Die Windows Befehle stehen auch im Terminal zur Verfügung, z.B. `notepad README.md` 
 


___

![](images/SSH_Logo_36x36.png "SSH") 02 - SSH-Keys und SSH-Agent 
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)


### SSH Keypair erstellen
(z.B. auf persönlichem Laptop)

Als erstes muss ein SSH-Keypair (Private/Public-Key) erstellt werden. 
Der **Private-Key** wird persönlich verwaltet und sollte **niemals** weitergegeben werden.
Der **Public-Key** (Endung .pub) kann  weitergegeben werden und ist somit auch für alle einsehbar. 

1.  Terminal (*Bash*) öffnen

2.  Folgenden Befehl mit der Account-E-Mail von GitHub einfügen:
    ```Shell
      $  ssh-keygen -t rsa -b 4096 -C "livio.brugger@edu.tbz.ch"
    ```
3. Neuer SSH-Key wird erstellt:
    ```Shell
      Generating public/private rsa key pair.
    ```
4. Bei der Abfrage, unter welchem Namen der Schlüssel gespeichert werden soll, die Enter-Taste drücken (für Standard):
    ```Shell
      Enter a file in which to save the key (~/.ssh/id_rsa): [Press enter]
    ```
5. Nun kann ein Passwort für den Key festgelegt werden. Es wird empfohlen dieses zu setzen und anschliessend dem SSH-Agent zu hinterlegen, sodass keine erneute Eingabe (z.B. beim Pushen) notwendig ist:
    ```Shell
      Enter passphrase (empty for no passphrase): [Passwort]
      Enter same passphrase again: [Passwort wiederholen]
    ```

--- 
### SSH-Key dem SSH-Agent hinzufügen 

Um das Abgleichen der Inhalte zwischen dem Origin und dem Main/Master-Repository zu beschleunigen, empfiehlt es sich, den SSH-Key dem SSH-Agent hinzuzufügen.
**Vorteil:** Man muss sich nur einmal authentifizieren und kann anschliessend Projektänderungen **ohne** Passwort oder Passphrase-Eingabe "pushen" oder "pullen".  

**Windows und Linux:**
Im folgenden Abschnitt werden zwei Varianten erklärt. Bei der ersten Variante muss der Ablauf bei jedem neuen Login durchgeführt werden (Non-Persistent). Bei der zweiten Variante wird ein Script in das .bash-profile eingefügt, welches beim Login des Benutzers automatisch ausgeführt wird (Persistent)

**Variante 1:** Non-Persistent (muss bei jedem Login neu ausgeführt werden)

1.  SSH-Agenten starten:
    ```Shell
      $ eval "$(ssh-agent -s)"
      Agent pid 635
    ```
    Beispiel:
   ![Screenshot](images/SSH_Agent_starten_800.png)

2.  SSH-Agenten mit der aktuellen Shell "verlinken":
    ```Shell
      $ ssh-add 
    ```

    Beispiel:
   ![Screenshot](images/SSH_Agent-2-Session_800.png)

    ...danach muss die korrekte "Passphrase" noch eingegeben werden (falls gesetzt)

    Achtung: dieses Vorgehen ist nicht persistent. Das heisst, dass diese Prozedur beim nächsten Login erneut durchgeführt werden muss:

<br>

**Variante 2:** Persistent - SSH-Agent startet nach dem Login automatisch (Code liegt im .bash_profile, welches bei jedem Login gleich zu Beginn ausgeführt wird). Passphrase muss nur einmal (am Anfang) eingegeben werden -  [Quelle][20]

1.  Bash_Profile sichern, bevor es geändert wird und anschliessend mit Editor öffnen:
    ```Shell
      $ cd ~
      $ cp .bash_profile .bash_profile.ORIG
      $ vi .bash_profile
    ```
    Beispiel:
   ![Screenshot](images/bash_profile_800.png)

2.  Folgenden Code ins .bash_profile einfügen:
    ```Shell
      env=~/.ssh/agent.env
      agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }
      
      agent_start () {
        (umask 077; ssh-agent >| "$env")
        . "$env" >| /dev/null ; }

      agent_load_env

      # agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
      agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

      if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
        agent_start
        ssh-add
      elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
        ssh-add
      fi

      unset env

    ```
    Beispiel:
   ![Screenshot](images/bash_profile_inhalt_800.png)

**Check:**

Wenn alles korrekt konfiguriert worden ist, und der zugehörige Public-Key auf die zu verwaltenden Server verteilt wurde, kann ich nun von meinem Host aus per **SSH** auf alle diese Systeme zugreifen, **ohne** ein Passwort und **ohne** eine Passphrase einzugeben.

Wenn ich ja jetzt für den Verbindungsaufbau weder ein Passwort noch eine Passphrase benötige, stellt sich die Frage, ob dies ein Security-Issue darstellt. Die Antwort ist "**Nein**". Aber wie lautet die Begründung? 

Nun: Falls jemand in den Besitz unseres **Private-Keys** kommt, muss er oder sie zusätzlich noch die Passphrase kennen. Wir erinnern uns: Immer zu Beginn einer Session, also bei der Eingabe des ersten SSH-Befehls, muss die Passphrase eingegeben werden. In unserem Fall ist das System so konfiguriert, dass der **SSH-Agent** im **.bash_profile** automatisch gestartet wird. In der gleichen Codeabfolge wird ihm der Private-Key angehängt. 

Dieses Verfahren ist also **sicher** und hilft uns, **effizient** Code mit einem Repository abzugleichen (push/pull), oder **unkompliziert** auf andere Systeme zuzugreifen.



---

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



___

![](images/Configfile_Logo_36x36.png "SSH") 03 - Zentrale SSH Konfigurationsfiles 
======

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Es gibt verschiedene Konfigurationsfiles, die im Zusammenhang mit **SSH** eine Rolle spielen. 

### Session Types 

1. RSA rhost authentication
    - /etc/hosts.allow
    - /etc/hosts.deny
    - /home/username/.rhosts
    - /home/username/.shosts

  Hier können entweder **pro System** oder **pro Benutzer** Einschränkungen konfiguriert werden. 

  Im folgenden Beispiel werden auf dem System **10.3.37.42** (Bild unten, gelbes Feld links) im **/etc/hosts.deny** sämtliche Zugriffe von "entfernten" Systemen unterbunden - mit einer Ausnahme: Das System mit der IP-Adresse **10.3.37.41** (Bild unten, oranges Feld oben rechts), welches mit einem "EXCEPT"-Eintrag von dieser Regel ausgeschlossen wird. Dieselbe Bedingung könnte auch erfüllt werden, wenn stattdessen ein entsprechender Eintrag im **/etc/hosts.allow** gemacht würde.

**Illustration:**

   ![Screenshot](images/12_SSH-Lab_v2_800.jpg)


**Host: 10.3.37.42**

Konfigurationsfile mit Editor öffnen
```Shell
  $ sudo vi /etc/hosts.deny
```

Eintrag in der letzten Zeile
```Shell
  # /etc/hosts.deny: list of hosts that are _not_ allowed to access the system.
  #                  See the manual pages hosts_access(5) and hosts_options(5).
  #
  # Example:    ALL: some.host.name, .some.domain
  #             ALL EXCEPT in.fingerd: other.host.name, .other.domain
  #
  # If you're going to protect the portmapper use the name "rpcbind" for the
  # daemon name. See rpcbind(8) and rpc.mountd(8) for further information.
  #
  # The PARANOID wildcard matches any host whose name does not match its
  # address.
  #
  # You may wish to enable this to ensure any programs that don't
  # validate looked up hostnames still leave understandable logs. In past
  # versions of Debian this has been the default.
  # ALL: PARANOID
  ALL: ALL EXCEPT 10.3.37.41
```

**1. Beweis:** (Reset/Abbruch) <br>
Zugriff von **10.3.37.40** auf den Zielrechner **10.3.37.42**. Dieser Rechner hat **keine Exception** und darf somit **nicht zugreifen**:

   ![Screenshot](images/10_SSH_800.png)

Es erscheint die Meldung "Connection reset by peer". Der Verbindungsaufbau wurde somit abgebrochen; ich bleibe auf meinem Host.

**2. Beweis:** (Success) <br>
Zugriff von **10.3.37.41** auf den Zielrechner **10.3.37.42**. Dieser Rechner hat **eine Exception** und darf somit **zugreifen**:

   ![Screenshot](images/11_SSH_800.png)

Da ich zum ersten Mal von diesem Rechner zugreife, erscheint noch die Nachfrage, ob der Fingerprint des Systems akzeptiert werden möchte. Nach der Bestätigung werde ich zugelassen.



2. Private/Public Keypair authentication <br>

3. Password authentication



### Client installieren
***
1. Für die Client-Installation muss der Installer unter [dieser Webseite](https://git-scm.com/downloads) heruntergeladen werden 
2. Die Installation erfolgt GUI-basiert, jedoch Standard (ohne speziellen Anpassungen). Daher wird an dieser Stelle auf eine Erklärung verzichtet.
3. Sobald der Vorgang abgeschlossen wurde, kann mit der Konfiguration fortgefahren werden.
