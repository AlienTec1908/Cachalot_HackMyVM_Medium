# Cachalot - HackMyVM - Medium

![Cachalot HackMyVM Icon](Cachalot.png)

## Maschineninformation

*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Cachalot](https://hackmyvm.eu/machines/machine.php?vm=Cachalot)
*   **Schwierigkeitsgrad:** Medium

## Autor

*   **Pentest Report Author:** DarkSpirit

## Bericht Online

*   **Link zum detaillierten HTML Bericht:** [Cachalot - HackMyVM - Medium Report](https://alientec1908.github.io/Cachalot_HackMyVM_Medium/)

## Zusammenfassung

Dieser Bericht dokumentiert den Penetrationstest der HackMyVM-Maschine "Cachalot", die als "Medium" eingestuft ist. Die Maschine präsentierte eine komplexe Umgebung mit mehreren exponierten Diensten, die auf Docker-Containern liefen. Der Test umfasste die initiale Aufklärung, die detaillierte Analyse verschiedener Webanwendungen, das Erringen des ersten Systemzugriffs (Initial Access) durch die Ausnutzung einer Authentifizierten Remote Code Execution (RCE) in GitLab, sowie die Ausweitung der Berechtigungen bis hin zum Administrator (Privilege Escalation) durch eine unsichere Docker-Konfiguration, um schließlich die User- und Root-Flags zu erlangen.

## Gefundene Schwachstellen & Vorgehen

Der Weg zur vollständigen Kompromittierung der Maschine umfasste folgende Schritte und Schwachstellen:

1.  **Reconnaissance:** Identifizierung der Ziel-IP und eines breiten Spektrums offener Ports (22, 80, 3000, 5022, 5080, 8080, 9000) mit verschiedenen Diensten (Apache, Grafana, GitLab, Docker Web UI, Python Debug Server, SSH).
2.  **Web Enumeration:** Detaillierte Untersuchung der Webdienste. Entdeckung einer unauthentifizierten Docker Web UI auf Port 9000, die Informationen über laufende Container und Images preisgab. Identifizierung von Grafana auf Port 3000 und GitLab (Version 11.4.7) auf Port 5080 mit jeweiligen Login-Seiten. Ein Debug-Server auf Port 8080 gab Systeminformationen preis.
3.  **Local File Inclusion (LFI):** Entdeckung und Ausnutzung einer LFI-Schwachstelle im Grafana-Dienst (Port 3000) über den Pfad `/public/plugins/alertlist/`, die das Lesen beliebiger Systemdateien ermöglichte (z.B. `/etc/passwd`, `/etc/grafana/grafana.ini`).
4.  **Informationsleck (Initial Root Password):** Kombination der Docker Web UI Informationen (gemountetes Volume `/srv/initial_root_password`) mit der Grafana LFI, um den Inhalt der Datei `/srv/initial_root_password` zu lesen, die das initiale Root-Passwort für GitLab enthielt.
5.  **Authentifizierte Remote Code Execution (RCE) in GitLab:** Ausnutzung einer bekannten RCE-Schwachstelle (CVE-2018-19585/CVE-2018-19571) in GitLab Version 11.4.7. Unter Verwendung des kompromittierten Root-Passworts und eines angepassten Exploits (SSRF via `git://` Import URL zur Redis Command Injection) konnte Code als Benutzer `git` im GitLab-Container ausgeführt werden. Debugging und manuelle Anpassung des Payloads über Burp Suite waren notwendig, um eine Hostname-Validierung zu umgehen und die korrekte Ruby/Bash-Syntax für die Remote Code Execution zu gewährleisten.
6.  **Initial Access:** Erlangung einer interaktiven Reverse Shell als Benutzer `git` auf der Angreifer-Maschine durch die erfolgreiche Ausnutzung der GitLab RCE.
7.  **Privilege Escalation (Root):** System-Enumeration als Benutzer `git` enthüllte eine kritische Fehlkonfiguration in der `sudoers`-Datei: Der Benutzer `git` durfte den Befehl `/usr/bin/docker` als Root *ohne Passworteingabe* ausführen (`(root) NOPASSWD: /usr/bin/docker`).
8.  **Docker Escape:** Nutzung der `sudo docker` Berechtigung, um einen neuen Container mit gemountetem Host-Root-Dateisystem (`-v /:/mnt`) zu starten und mittels `chroot` Root-Zugriff auf das Host-System zu erlangen.
9.  **Flags:** Sichern der User-Flag (`local.txt`) im Home-Verzeichnis des Benutzers `cachalot` und der Root-Flag (`proof.txt`) im Wurzelverzeichnis des Host-Dateisystems.

## Verwendete Tools

*   arp-scan
*   echo
*   nmap
*   nikto
*   gobuster
*   curl
*   jq
*   git
*   searchsploit
*   python3
*   vi
*   Metasploit
*   wget
*   nc
*   Wappalyzer
*   CyberChef
*   Burpsuite
*   docker

## Berichtsdatum

06 Jun 2025
