[01]: https://github.com/ser-cal/SSH_Tutorial/tree/main/Theorie#-01---wichtige-linux-kommandos
[02]: https://github.com/ser-cal/SSH_Tutorial/tree/main/Theorie#-02---ssh-keys-und-ssh-agent
[03]: https://github.com/ser-cal/SSH_Tutorial/tree/main/Theorie#-03---zentrale-ssh-konfigurationsfiles


# SSH verstehen und anwenden

Computer kommunizieren über Netzwerke miteinander. Daher wurde in der Vergangenheit eine Reihe von Regeln für die Kommunikation zwischen ICT-Systemen eingeführt und stetig weiterentwickelt.

Der erste dafür eingesetzte Dienst war **Telnet**. Dieser Dienst ist jedoch nicht sicher. Telnet übermittelt Daten im **Klartext**. Das heisst; die übertragenen Daten können, falls abgefangen, ohne weiteren Aufwand gelesen werden. Um genau dieses Problem zu lösen, wurde Mitte der 90er Jahre ein **sichereres** Protokoll namens **SSH** entwickelt und als zuverlässiger Nachfolger des Telnet-Protokolls eingeführt. Daten die mit **SSH** übermittelt werden, sind verschlüsselt (encrypted) und können nicht ohne entsprechenden **Schlüssel** gelesen (decrypted) werden. 

Dieses Tutorial soll helfen, **SSH** zu verstehen und praktisch anzuwenden. Es wird empfohlen, auch die Praxisbeispiele durchzuarbeiten, um das Wissen zu festigen. 
Diese **Skills** gehören, ähnlich wie z.B. **Netzwerkgrundlagen**, zum unverzichtbaren **Basiswissen** eines Informatikers.

## Lernziel

**SSH** verstehen und praktisch anwenden

## Voraussetzungen

* Host (PC/Notebook mit min. 8 GB freiem RAM)
* Ausreichend Platz auf dem Host, um virtuelle Maschinen zu erstellen *(mind. 40GB freier HD- oder SSD-Speicher, falls kein Zugriff auf TBZ-Cloud, AWS-Cloud oder Azure-Cloud)*
* Einfache Linux-Kenntnisse (alles notwendige dazu weiter unten)
* Internetanschluss *(allenfalls VPN-Zugang zu Cloud-Umgebung)*

---

## Hier geht es zu den Inhalten

### [**Theorie:** Hauptseite](Theorie)
* [01 - Wichtige Linux-Kommandos][01]
* [02 - SSH-Keys und SSH-Agent][02]
* [03 - Zentrale SSH Konfigurationsfiles][03]


---

### [**Praxis:**](Praxis)
- [**Lab 1**](Praxis/Lab1)
- [**Lab 2**](Praxis/Lab2)
<br>