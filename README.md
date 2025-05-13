# Momentum1 - HackMyVM (Medium)

![Momentum.png](Momentum.png)

## Übersicht

*   **VM:** Momentum1
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Momentum)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 23. November 2020
*   **Original-Writeup:** https://alientec1908.github.io/Momentum_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Momentum1" zu erlangen. Der initiale Zugriff erfolgte durch Ausführung eines Node.js-Skripts (`main.js`), dessen Ursprung im Bericht nicht klar ist, das aber SSH-Zugangsdaten (`auxerre:auxerre-alienum`) preisgab. Nach dem SSH-Login als `auxerre` wurde ein lokal laufender Redis-Server ohne Authentifizierung entdeckt. In diesem Redis-Server war das Root-Passwort (`m0mentum-al1enum`) im Klartext unter dem Schlüssel `rootpass` gespeichert. Dies ermöglichte den direkten Wechsel zum Root-Benutzer mittels `su`.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `nodejs`
*   `vi`
*   `hydra`
*   `ssh`
*   `redis-cli`
*   `su`
*   Standard Linux-Befehle (`ls`, `cat`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Momentum1" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Adresse des Ziels (192.168.2.112) mit `arp-scan` identifiziert.
    *   `nmap`-Scan offenbarte Port 22 (SSH, OpenSSH 7.9p1) und Port 80 (HTTP, Apache 2.4.38).
    *   `gobuster` auf Port 80 fand Standardverzeichnisse und `/manual`.
    *   Ein Node.js-Skript (`main.js`, Herkunft unklar) wurde ausgeführt und gab die Zeichenkette `auxerre-alienum` aus, die als potenzielle SSH-Credentials interpretiert wurde.

2.  **Initial Access (SSH als `auxerre`):**
    *   Eine Passwortliste mit Variationen von "auxerre" und "alienum" wurde erstellt.
    *   Mittels `hydra` wurde ein Brute-Force-Angriff auf den SSH-Dienst für den Benutzer `auxerre` mit der erstellten Passwortliste durchgeführt. Die Credentials `auxerre:auxerre-alienum` wurden als gültig bestätigt.
    *   Erfolgreicher SSH-Login als `auxerre`.

3.  **Privilege Escalation (von `auxerre` zu `root` via Redis):**
    *   Als `auxerre` wurde mit `redis-cli` auf einen lokal laufenden Redis-Server (Port 6379) ohne Authentifizierung zugegriffen.
    *   Der Befehl `KEYS *` in `redis-cli` zeigte den Schlüssel `rootpass`.
    *   Der Befehl `GET rootpass` gab den Wert `m0mentum-al1enum` zurück, bei dem es sich um das Root-Passwort handelte.
    *   Mittels `su root` und Eingabe des Passworts `m0mentum-al1enum` wurde Root-Zugriff erlangt.
    *   Die User-Flag (`84157165c30ad34d18945b647ec7f647`) in `/home/auxerre/user.txt` und die Root-Flag (`658ff660fdac0b079ea78238e5996e40`) in `/root/root.txt` wurden gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Information Disclosure (Credentials Leak):** Ein Node.js-Skript gab direkt oder indirekt SSH-Zugangsdaten preis.
*   **Unsicher konfigurierter Redis-Server:** Ein Redis-Server lief ohne Authentifizierung und speicherte das Root-Passwort im Klartext.
*   **Schwache Passwörter (impliziert):** Das Root-Passwort war im Klartext zugänglich.

## Flags

*   **User Flag (`/home/auxerre/user.txt`):** `84157165c30ad34d18945b647ec7f647`
*   **Root Flag (`/root/root.txt`):** `658ff660fdac0b079ea78238e5996e40`

## Tags

`HackMyVM`, `Momentum1`, `Medium`, `Information Disclosure`, `Node.js`, `Redis Exploit`, `Password Leak`, `SSH`, `Linux`, `Privilege Escalation`, `Apache`
