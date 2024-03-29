[10]: https://git-scm.com/downloads
[20]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows
[30]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/
[31]: working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows
[32]: https://man.openbsd.org/sshd_config
[33]: https://man.openbsd.org/sshd_config
[34]: https://www.thomas-krenn.com/de/wiki/OpenSSH_Public_Key_Authentifizierung_unter_Ubuntu

Inhaltsverzeichnis
====
* 01 - [Wichtige Linux-Kommandos](#-01---wichtige-linux-kommandos)
* 02 - [SSH-Keys und SSH-Agent](#-02---ssh-keys-und-ssh-agent)
* 03 - [Zentrale SSH Konfigurationsfiles](#-03---zentrale-ssh-konfigurationsfiles)
* 04 - [Analyse und Debugging](#-04---Analyse-und-debugging)
* 05 - [Quellenverzeichnis](#-05---quellenverzeichnis)


<br><br>
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

#### SSH-Befehle
Mit SSH kann man nicht nur sicher auf entfernten Systemen einloggen. Es gibt noch unzählige weitere Anwendungen, die damit realisierbar sind. Verschlüsselt und sehr schnell. Hier drei Beispiele, die in der Praxis sehr oft genutzt werden:

 * `ssh <benutzer>@<ip>` SSH-Verbindung eines Benutzers auf ein anderes System
 * `scp <benutzer>@<ip>:<Verzeichnis` Kopieren von Dateien auf ein anderes System
  * `ssh -X <benutzer>@<IP-Adresse>` X-Window öffnen. Danach kann man GUI-Applikationen des Remote-Systems lokal öffnen -> z.B. gedit



 
<br><br>

___

![](images/SSH_Logo_36x36.png "SSH") 02 - SSH-Keys und SSH-Agent 
====

> [⇧ **Nach oben**](#inhaltsverzeichnis)


### SSH Keypair erstellen
(z.B. auf Laptop oder PC)

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
**Vorteil:** Man muss sich nur einmal authentifizieren und kann anschliessend Projektänderungen **ohne** Passwort oder Passphrase-Eingabe "pushen" oder "pullen". Ausserdem kann ich anschliessend ohne Authentifizierung auf sämtliche Rechner zugreifen, welche meinen Public-Key hinterlegt haben. Damit habe ich sehr schnell, sicher und sehr unkompliziert Zugriff auf sämtliche Systeme, die ich administriere. 

**Windows und Linux:**

In diesem Tutorial benutze ich den **GitBash Client**. Dieser kann wie folgt installiert werden:
1. Installer unter [dieser Webseite](https://git-scm.com/downloads) herunterladen 
2. Die Installation erfolgt GUI-basiert, jedoch Standard (ohne speziellen Anpassungen). Daher wird an dieser Stelle auf eine Erklärung verzichtet.
3. Sobald der Vorgang abgeschlossen wurde, kann mit der Konfiguration fortgefahren werden.


Im folgenden Abschnitt werden zwei Varianten erklärt. Bei der ersten Variante muss der Ablauf bei jedem neuen Login durchgeführt werden (Non-Persistent). Bei der zweiten Variante wird ein Script in das .bash-profile eingefügt, welches beim Login des Benutzers automatisch ausgeführt wird (Persistent)

**Variante 1:** Non-Persistent (muss bei jedem Login neu ausgeführt werden)

1.  SSH-Agenten starten:
    ```Shell
      $ eval "$(ssh-agent -s)"
      Agent pid 635
    ```
    
    Beispiel:<br>
   ![Screenshot](images/SSH_Agent_starten_800.png)

2.  SSH-Agenten mit der aktuellen Shell "verlinken":
    ```Shell
      $ ssh-add 
    ```

    Beispiel:<br>
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
    Beispiel:<br>
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
    Beispiel:<br>
   ![Screenshot](images/bash_profile_inhalt_800.png)

**Check:**

Wenn alles korrekt konfiguriert worden ist, und der zugehörige Public-Key auf die zu verwaltenden Server verteilt wurde, kann ich nun von meinem Host aus per **SSH** auf alle diese Systeme zugreifen, **ohne** ein Passwort und **ohne** eine Passphrase einzugeben.

Wenn ich ja jetzt für den Verbindungsaufbau weder ein Passwort noch eine Passphrase benötige, stellt sich die Frage, ob dies ein Security-Issue darstellt. Die Antwort ist "**Nein**". Aber wie lautet die Begründung? 

Nun: Falls jemand in den Besitz unseres **Private-Keys** kommt, muss er oder sie zusätzlich noch die Passphrase kennen. Wir erinnern uns: Immer zu Beginn einer Session, also **vor** der Eingabe des **ersten** SSH-Befehls, **muss** die **Passphrase** eingegeben werden. In unserem Fall ist das System so konfiguriert, dass der **SSH-Agent** im **.bash_profile** automatisch gestartet wird. In der gleichen Codeabfolge wird ihm der Private-Key angehängt. Das funktioniert aber nur, wenn die entsprechende Eingabe mit der korrekten **Passphrase** übereinstimmt!

Dieses Verfahren ist also **sicher** und hilft uns anschliessend, **effizient** Code mit einem Repository abzugleichen (push/pull), oder **unkompliziert** auf andere Systeme zuzugreifen.



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


<br><br>
___

![](images/Configfile_Logo_36x36.png "SSH") 03 - Zentrale SSH Konfigurationsfiles 
====

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Es gibt verschiedene **Konfigurationsfiles**, die im Zusammenhang mit **SSH** eine Rolle spielen. 

Noch **bevor** der SSH-Daemon ins Spiel kommt, kann ich auf meinem System übergeordnete **Restriktionen** vornehmen. In folgenden Konfigurationsfiles kann ich definieren, wer Zugriff auf mein System erhält. Dabei kann ich von **grob** (ganze Topleveldomains oder Subnetze) bis **fein** (Einzelne Hosts oder einzelne Benutzer) Berechtigungen zuweisen oder entfernen.

### RSA rhost authentication 
  - /etc/hosts.allow
  - /etc/hosts.deny
  - /home/username/.rhosts
  - /home/username/.shosts

  ### Anwendungsfall

  Im folgenden Beispiel werden auf dem System **10.3.37.40** (Bild unten links, gelbes Feld) im **/etc/hosts.deny** sämtliche Zugriffe von "entfernten" Systemen unterbunden - mit einer Ausnahme: Das System mit der IP-Adresse **10.3.37.41** (Bild unten, Feld oben rechts), welches mit einem "EXCEPT"-Eintrag von dieser Regel ausgeschlossen wird. Beide Systeme auf der rechten Seite möchten auf das linke System zugreifen  (1a, 1b). Zugriff erhält aber nur dasjenige, welches im /etc/hosts.deny berechtigt wurde (3a).  
  
  Dieselbe Bedingung könnte auch erfüllt werden, wenn stattdessen ein entsprechender Eintrag im **/etc/hosts.allow** gemacht würde.

**Illustration:**

   ![Screenshot](images/12_SSH-Lab_v3_800.jpg)


**Host: 10.3.37.40** wie folgt konfigurieren:

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

Screenshot (Beispiel):<br>
   ![Screenshot](images/12_exception_800.jpg)


<br>

**Versuch 1**: in diesem Fall funktioniert es **nicht** (Reset/Abbruch) <br>
Zugriff von **10.3.37.42** (rechts unten, Blau) auf den Zielrechner **10.3.37.40** (links). Dieser Rechner hat **keine Exception** und darf somit **nicht zugreifen**:

Befehle auf System **10.3.37.42**:
```Shell
  $ hostname #Zeigt den Hostname an
  $ ssh mclaren@10.3.37.40 #Zugriff auf 10.3.37.40
```

Screenshot (Beispiel):<br>
   ![Screenshot](images/10_SSH_800.jpg)

Es erscheint die Meldung "Connection reset by peer". Der Verbindungsaufbau wurde somit abgebrochen; ich bleibe auf meinem Host.

<br>

**Versuch 2**: in diesem Fall funktioniert es (Success) <br>
Zugriff von **10.3.37.41** (rechts oben) auf den Zielrechner **10.3.37.42**. Dieser Rechner hat **eine Exception** und darf somit **zugreifen**:

Befehle auf System **10.3.37.41**:
```Shell
  $ hostname #Wir befinden uns auf 
  $ ssh ferrari@10.3.37.40 #Zugriff auf 10.3.37.40
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/11_SSH_800.png)

Da ich zum ersten Mal von diesem Rechner zugreife, erscheint noch die Nachfrage, ob der Fingerprint des Systems akzeptiert werden möchte. Nach der Bestätigung werde ich zugelassen.


### SSH User keys location 
  - **/home/username/.ssh** <br>
    *hier sind die Keys (private & public), sowie die config und known-hosts-Datei abgelegt*.

    Das erstellen der Keys wurde bereits weiter oben erklärt. Hier noch en paar Informationen zur **known-hosts**-Datei.
    Immer wenn ich mit meinem Host zum erstem Mal auf ein anderes System zugreife, erhalte ich von diesem System einen "Fingerprint", den ich mit **yes** bestätigen muss, um mich erfolgreich darauf anzumelden. Hier ist es wichtig, dass man sich nochmals vergewissert, ob man auf das richtige System zugreift - denn nach der Bestätigung wird dieser Fingerprint in der persönlichen **known-hosts**-Datei abgelegt. Bei späteren Zugriffen auf diesen Zielhost geht das System deshalb davon aus, dass es sich um einen vertrauenswürdigen Rechner handelt.  
    Diese Abfrage kann z.B. **Man-in-the-Middle**-Angriffe vermeiden (dieselbe IP-Adresse eines Dritten würde nicht zugelassen). Es kann aber auch sein, dass z.B. ein DHCP-Server eine bereits bestätigte IP-Adresse einem anderen System zuweist und der Zugriff dadurch nicht mehr funktioniert. In solchen Fällen muss einfach der entsprechende Fingerprint im **known-hosts**-File gelöscht werden.  


### SSH configuration files 
  - **/etc/ssh/ssh_config** (Control Client verhalten)<br>
    Wenn man vom aktuellen Host auf ein anderes System zugreifen will, benutzt man den SSH-Client. In diesem Fall greifen die Settings im **ssh_config** (z.B. Portnummer, Protokoll-Version, Encryption-Algorythmus etc..)

  - **/etc/ssh/sshd_config** (Control Server verhalten)<br>
  Dieses Konfigurationsfile wird vom lokalen SSH-Daemon genutzt (der Dienst, welcher standardmässig auf dem SSH-Port horcht und Verbindungsanfragen von entfernten Systemen entgegennimmt).
  Beispiel: Wenn jemand von einem entfernten System via SSH auf unseren Host zugreifen will. 
  Damit dieser Zugriff funktioniert, müssen die Client-Settings auf dem anderen System mit den **sshd-config**-Settings übereinstimmen (z.B. Port-Nummer, Version etc...).
  
### SSH-Hardening:
Wenn man von SSH-Hardening spricht, dann ist insbesondere die Anpassung des Files **/etc/ssh/sshd_config** gemeint. Die folgende imperative Anleitung zeigt, wie Systeme im Enterprise-Umfeld gehärtet werden.

Bei der Bearbeitung einer Konfigurationsdatei können standardmäßig einige Optionen mit einem einzelnen Hash-Zeichen (**#**) am Anfang der Zeile auskommentiert werden. Um diese Optionen zu bearbeiten oder die kommentierte Option anerkennen zu lassen, muss man sie unkommentieren, indem man vorne einfach den Hash (**#**) entfernt.

Konfigurationsdatei sichern:
```Shell
  $ sudo cd /etc/ssh #Ins Verzeichnis wechseln
  $ sudo cp sshd_config sshd_config.ORIG #Sicherheitskopie des Configfiles
```
Bevor man die Konfigurationsdatei bearbeitet, können die aktuell festgelegten Optionen überprüft werden. Mit folgendem Befehl wird auf dem OpenSSH-Server ein erweiterter Testmodus ausgeführt, welcher die vollständige Konfigurationsdatei validiert und die effektiven Konfigurationswerte ausgibt.
```Shell
  $ sudo sshd -T #Validierung/Ausgabe sämtlicher Parameter
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/12_sshd_config_800.png)

<br>

**1.** Als erstes muss für den Root-Account die Anmeldung über SSH **deaktiviert** werden:

```Shell
  $ vi /etc/ssh/sshd_config #Datei in einem Editor öffnen
```
Parameter **PermitRootLogin** auf **no** setzen
```Shell
  PermitRootLogin No
```

Das ist sehr wichtig, da es einen potenziellen Angreifer daran hindert, sich direkt als root anzumelden. Das unterstützt zudem gute betriebliche Sicherheitspraktiken wie das Operieren als nicht privilegierter Benutzer und die Verwendung von **sudo**, um Berechtigungen nur dann auszuweiten, wenn es unbedingt erforderlich ist.
Dieses Vorgehen ist inzwischen auch bei sämtlichen Banken in der Region Zürich "Best Practice". Da im **sudoers.conf** festgelegt wird, wer welche Rechte hat und sich alle via **sudo** authentifizieren müssen, wird zu jedem Zeitpunkt genau festgehalten, wer wann welches Kommando ausgeführt hat. Würden z.B. mehrere Administratoren Zugang zum Root-Passwort haben und wäre ein Root-Login erlaubt, wäre dies nicht möglich. 

Folgendes Bild verdeutlicht die Authentifizierung von den beiden Benutzern **Norris** (links oben) und **Ricciardo** (links unten). Der Ablauf wird weiter unten in vier Schritten erklärt:

**Illustration:**

   ![Screenshot](images/13_SSH_SUDO_v2_800.jpg)


- **Erster Schritt:** Beide Clients (Norris und Ricciardo) verbinden sich von unterschiedlichen Clients via **SSH** mit dem Server

- **Zweiter Schritt:** Die Authentifizierung findet mittels Private- und Public-Key statt. Da beide ihren Public-Key auf dem Server hinterlegt haben, können sie sich einloggen.

- **Dritter Schritt:** Beide durchlaufen das **/etc/sudoers.conf**. Hier wird jedem Benutzer die ihm zugewiesenen Berechtigungen zugeteilt. 

- **Vierter Schritt:** Beide sind nun auf dem Zielrechner eingeloggt, haben aber unterschiedliche Berechtigungen (Norris kann Root-Recht erlangen, Ricciardo nicht)

<br> 

**2.** Als Nächstes sollte die maximale Anzahl der Authentifizierungsversuche für eine bestimmte Anmeldesitzung wie folgt begrenzt werden:


Parameter **MaxAuthTries** auf **3** setzen
```Shell
  MaxAuthTries 3
```
Für die meisten Einstellungen ist ein Standardwert von 3 akzeptabel. Man kann hier jedoch je nach eigener Risikoschwelle einen höheren oder niedrigeren Wert festlegen.

<br>

**3.** Bei Bedarf eine reduzierte Anmeldefrist festgelegt werden. Das ist die Zeitspanne, in der ein Benutzer die Authentifizierung abschließen muss, nachdem er sich mit dem SSH-Server verbunden hat:

Parameter **LoginGraceTime** auf z.B. **20** setzen
```Shell
  LoginGraceTime 20
```
Die Konfigurationsdatei gibt diesen Wert in **Sekunden** an.

Eine Einstellung auf einen niedrigeren Wert kann helfen, bestimmte **Denial-of-Service-Angriffe** zu verhindern, bei denen mehrere Authentifizierungssitzungen für einen **längeren Zeitraum** offen gehalten werden.

<br>

**4.** Wenn SSH-Schlüssel für die Authentifizierung konfiguriert wurden anstatt Passwörter zu verwenden, kann die SSH-Passwortauthentifizierung deaktiviert werden. So wird verhindert, dass sich ein Angreifer mit **geleakten Benutzerpasswörtern** anmelden kann.
```Shell
  PasswordAuthentification no
```
Als eine weitere Härtungsmaßnahme in Bezug auf Passwörter kann bei Bedarf auch die Authentifizierung mit leeren Passwörtern deaktiviert werden. Dadurch wird die Anmeldung verhindert, wenn das Passwort eines Benutzers auf einen blanken oder leeren Wert gesetzt ist:
```Shell
  PermitEmptyPasswords no
```
In den meisten Anwendungsfällen wird SSH mit der Authentifizierung mit **öffentlichen Schlüsseln** als die **einzige** Authentifizierungsmethode konfiguriert. OpenSSH-Server unterstützt aber noch viele andere Authentifizierungsmethoden, von denen einige standardmäßig aktiviert sind. Wenn diese nicht erforderlich sind, können Sie sie deaktivieren, um die Angriffsoberfläche Ihres SSH-Servers weiter zu reduzieren:
```Shell
  ChallengeResponseAuthentication no
  KerberosAuthentication no
  GSSAPIAuthentication no
```
Weitere Ressourcen zu diesen Authentifizierungsvarianten:
- [Challenge–response authentication](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)
- [Kerberos und SSH](https://docstore.mik.ua/orelly/networking_2ndEd/ssh/ch11_04.htm)
- [GSSAPI-Authentifizierung](https://de.wikipedia.org/wiki/GSSAPI)

<br>

Die Syntax der neuen Konfiguration kann validiert werden, indem **sshd** nochmals im Testmodus ausgeführt wird:
```Shell
  $ sudo sshd -T #Validierung/Ausgabe sämtlicher Parameter
```

Wenn Sie mit der Konfigurationsdatei zufrieden sind, können Sie sshd neu laden, um die neuen Einstellungen anzuwenden:
```Shell
  $ sudo service sshd reload #SSH-Daemon neu starten / aktuelles Config-file einlesen
```

<br><br>
___

![](images/Lupe_36x36.png "Analyse") 04 - Analyse und Debugging 
====

> [⇧ **Nach oben**](#inhaltsverzeichnis)

Bei der Anwendung von SSH können immer wieder mal Probleme auftreten. Meistens gibt uns die entsprechende Rückmeldung gute Anhaltspunkte, um mit einem zielführenden **Debugging** loszulegen. Es ist deshalb unerlässlich, das Grundprinzip dieses Client/Server-Dienstes gut verstanden zu haben.

Die folgende Darstellung zeigt nochmals, wie der Verbindungsaufbau und die Authentisierung beim **SSH-Dienst** abläuft. Wie so oft bei Client/Server-Diensten, sind auch hier **drei Schritte** notwendig.

Der Ablauf wird weiter unten erklärt:

**Illustration:**

   ![Screenshot](images/14_SSH_Keyaustausch_800.jpg)

- **Erster Schritt:** Ein bestimmter Benutzer auf dem Client (z.B. ubuntu) sendet dem Server eine **Verbindungsanfrage**.

- **Zweiter Schritt:** Der Server schaut lokal bei sich nach, ob dieser bestimmte Benutzer einen **Public-Key** hinterlegt hat. Falls ja, schickt er dem Client eine Zufallszahl zurück, die er (**achtung wichtig!!**) vorher mit genau diesem **Public Key** des entsprechenden Benutzers verschlüsselt hat. Falls **kein Public-Key** hinterlegt ist, muss sich der Client mit seinem gewöhnlichen Passwort authentifizieren. Auch bei dieser Variante muss er natürlich auf dem Zielhost dafür berechtig sein.
In unserem Fall hat der Server einen **Public-Key** gefunden, damit die Zufallszahl **encrypted** und anschliessend dem Benutzer auf dem Client zurückgeschickt.

- **Dritter Schritt:** Damit der Benutzer des Clients auf dem Server zugelassen wird, muss er die erhaltene Zufallszahl mit seinem **Private Key decrypten** (entschlüsseln). Anschliessend sendet er die entschlüsselte Zufallszahl dem Server zurück, dieser vergleicht den erhaltenen Wert mit dem vorher versandten Wert. Falls beide übereinstimmen, wird die Verbindung zugelassen und aufgebaut (Grüner Doppelpfeil).

<br> 

### Debugging SSH:

**1.** Zuerst sollte, wie eigentlich immer, sichergestellt werden, dass die Netzwerkverbindung auf **TCP/IP-Layer 1 bis 3** funktioniert (z.B. mit einem Ping) - darauf achten dass das Layer-3 Protokoll **ICMP** von beiden Hosts zugelassen wird.

<br>

**2.** auf dem **SSH-Server** überprüfen, ob der Dienst korrekt läuft:
```Shell
  $ sudo systemctl status sshd #ssh-Dienst checken
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/15_SSH_Dienst_aktiv_800.png)

In diesem Screenshot sieht man weiter unten auch gleich noch einen Auszug aus dem **auth.log**-File. Benutzer, die auf dieses System zugegriffen haben hinterlassen Spuren (dazu gleich mehr bei Punkt 4)

<br>

**3.** Die **SSH-Verbindung** mit dem **Verbose-Flag** ergänzen. Es werden sehr viel mehr Informationen am Terminal ausgegeben: 
```Shell
  $ ssh -v <user@IP-Adresse> # v-Flag (Verbose)
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/16_SSH_verbose_800.png)

...auf dem Screenshot sind nur die ersten Zeilen des Outputs. Dieser ist sehr viel länger und zeigt Schritt für Schritt und nachvollziehbar auf, wie die Authentifizierung zwischen Client und Server abläuft.

<br>

**4. Logfile-Analyse**: Jeder Dienst muss nach der Installation konfiguriert werden (Details oben bereits behandelt). Dabei wird auch festgehalten, wie Anwendung journalisiert werden. In grösseren Firem werden dazu sogenannte **Loghosts** eingesetzt. Diese sammeln verschiedenste Informationen und bestimmen anschliessend, was damit geschieht. In unserem Fall wurde nichts angepasst und so sind die Logfiles auf Linux- und Unix-Umgebungen standardmässig im Verzeichnis **/var** abgelegt. Das Logfile für die Zugriffsauthentifizierung nennt sich **auth.log** und liegt unter **/var/log**.
```Shell
  $ sudo tail -30 /var/log/auth.log # Zeige die letzten 30 Zeilen des Logfiles
  $ sudo cat /var/log/auth.log | grep sshd # Zeige alle sshd-Einträge
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/16_SSH_logfile_800.png)

Im Beispiel oben sieht man einen Auszug dieses Logfiles. Dabei kann man dieselben Einträge wiedererkennen, die weiter oben bereits bei der Kontrolle des Dienstes ausgegeben wurden. Hier ist allerdings die gesamte Historie abgelegt. Mit dem **zweiten Befehl** werden sämtliche Zeilen ausgegeben, die vom **SSH-Dienst** verzeichnet wurden 

<br>

**5. Advanced-Analyse**:
Für Fortgeschrittene gibt es noch die Möglichkeit, auf der zugehörigen Netzwerkkarte den Port 22 zu sniffen. Dafür bietet sich z.B. Wireshark an. Man kann allerdings auch auf der Kommandozeile entsprechende Befehle eingeben und den Output z.B. in ein File umleiten, um dieses dann später in Ruhe zu **debuggen**. 

```Shell
  $ ifconfig -a # Netzwerkschnittstelle checken
  $ sudo tcpdump -n -i wg0 tcp port 22 and host 10.3.37.41 # Sniffe auf wg0 Port 22 mit IP 10.3.37.41 
```
Screenshot (Beispiel):<br>
   ![Screenshot](images/17_SSH_sniffing_800.jpg)

In diesem Beispiel oben wird das **tcpdump**-Kommando auf dem System mit der IP-Adresse 10.3.37.40 abgesetzt. Der **sniff** wird dabei so eingeschränkt, dass nur die Netzwerkkarte "wg0" (Wireguard-Schnittstelle) auf Port 22 überwacht wird. Hier wird noch zusätzlich eingeschränkt, dass **nur** Datenpackete des System mit der IP-Adresse **10.3.37.41** ausgegeben werden sollen.

Dies ist nur eine Möglichkeit, wie man mit Regeln und Parametern die Datenerhebung gezielt eingrenzen kann. Wie oben bereits erwähnt, eher für **Fortgeschrittene** :-)

<br>

___

![](images/Library_Logo_36x36.png "Quellenverzeichnis") 05 - Quellenverzeichnis 
====

> [⇧ **Nach oben**](#inhaltsverzeichnis)

- [SSH - Ubuntuusers][30]
- [Working with SSH Key Passphrases][31]
- [sshd_config - openbsd Manpages][32]
- [SSH härten][33]
- [OpenSSH Public Key Authentifizierung unter Ubuntu][34]

<br>




----

<br>

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/ch/"><img alt="Creative Commons Lizenzvertrag" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/ch/88x31.png" /></a><br />Dieses Werk ist lizenziert unter einer <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/ch/">Creative Commons Namensnennung - Nicht-kommerziell - Weitergabe unter gleichen Bedingungen 3.0 Schweiz Lizenz</a>

- - -

- Autor: Marcello Calisto
- Mail: marcello.calisto@tbz.ch
