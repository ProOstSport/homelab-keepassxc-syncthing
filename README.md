# Passwortverwaltung mit KeePassXC und Syncthing

> Selbst gehostete Passwortdatenbank: KeePassXC läuft als Docker-Container auf dem Homeserver, Syncthing hält die Datenbank auf dem Laptop aktuell, und lokal ist sie auch offline nutzbar.

**Status:** Testbetrieb mit Testdaten. Echte Passwörter sind noch nicht migriert.

## Architektur

```
Laptop (KeePassXC lokal, offline nutzbar)
   ⇅  Syncthing (beide Richtungen)
Homeserver (KeePassXC im Docker-Container)
   ↑  Browserzugriff über Tailscale
```

## Umsetzung

- KeePassXC im Container, Port nur lokal gebunden und über Tailscale Serve erreichbar
- Datenbank liegt in einem Syncthing-Ordner, der zwischen Server und Laptop abgeglichen wird
- Backup der Datenbank als Teil des regulären Backup-Laufs (siehe [NUC-Projekt](https://github.com/ProOstSport/homelab-nuc-server))
- Oberflächenlogin und Datenbank-Masterpasswort sind getrennt

## Tests

| Test | Ergebnis |
|---|---|
| Browserlogin, speichern, sperren, entsperren | erfolgreich |
| Testeintrag nach Containerneustart | vorhanden |
| Zugriff über Tailscale aus fremdem Netz (Mobilfunk) | erfolgreich |
| Synchronisation in beide Richtungen | erfolgreich |
| Offline-Öffnung auf dem Laptop | erfolgreich |
| Backup mit identischen Prüfsummen | erfolgreich |
| Restore-Kopie geöffnet und Testeintrag geprüft | erfolgreich |
| Wiederherstellung aus Syncthing-Versionierung | offen |
| Restore des neuesten Datenbankstands | offen |

## Sicherheit

- **Arbeitsregel:** speichern, Datenbank schließen, Sync abwarten, erst dann auf dem anderen Gerät öffnen. Keine gleichzeitige Bearbeitung auf beiden Geräten.
- **Inaktivitätssperre:** 60 Sekunden getestet und ausgelöst. Der endgültige Wert von 240 Sekunden ist noch nicht bestätigt.
- **Zwischenablage:** Leerung nach 10 Sekunden eingestellt. Die Wirkung auf die lokale Zwischenablage ist nicht getestet.

## Bekannte Schwachstellen

- Die Sync-Ports von Syncthing sind auf allen Host-Adressen veröffentlicht, nicht nur im Tailnet.
- Die Backup-Platte enthält die `.env` mit dem Oberflächenpasswort im Klartext.

## Lessons Learned

- **Pfade verwechselt:** Host- und Containerpfade waren zunächst durcheinander. Lösung: die Mounts gezielt prüfen und die richtigen Pfade verwenden.
- **Falsches Proxy-Ziel:** Das Syncthing-Serve-Ziel zeigte auf den falschen Port, und eine Freigabe war doppelt. Beides wurde korrigiert.
- **Syntaxprüfung reicht nicht:** Ein Fehler (Exit 127) im Backup-Skript fiel bei `bash -n` nicht auf. Der Fehler zeigte sich erst im echten Lauf.
- **Zwei mögliche Hauptdateien:** Die ursprüngliche Datenbank wurde als Rückfallkopie umbenannt, damit nicht versehentlich an der falschen Datei gearbeitet wird.

## Offene Punkte

- Syncthing-Versionierung praktisch wiederherstellen
- Finale Sperr- und Zwischenablagewerte prüfen
- Feste Image-Versionen oder Digests dokumentieren
- Sync-Ports absichern
- Zweite, unabhängige Sicherung der Datenbank einrichten
