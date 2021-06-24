Aktuelle Serverkonfiguration:

>>>
Allgemeine Anmerkungen:
Mindest-Ram: 8 GB
Prozessor Server-Prozessor mit mind. 4 Kernen

>>>


Installation:
* [x] Rollen und  Features
  * [x] Serverrollen
    * [x]  Webserver IIS ()
      * Features in Form von Verwaltungstools hinzufügen -> Ja
  * [x]  Features
    * [x] ASP.NET 4.5 (.NET Framework 4.5 -> ASP.NET 4.5)

  * [x]  Rolle Webserver (IIS) -> Rollendienste
    * [x]  Kompatibilität mit der IIS6-Verwaltung -> alle Punkte und deren Abhängigkeiten installieren
    *  **WICHTIG, steht nicht in der Anleitung von myfactory**
      * [x] Anwendungsentwicklung:
        * [x]  .NET Erweiterbarkeit 4.5 -> alle Abhänigkeiten
        * [x]  ASP.NET 4.5 -> und alle Abhängigkeiten

* [x]  SQL-Server Installieren
  * [x]  SQL Server herunterladen (Developer oder Express, je nach Lizenzanforderung)
  * [x]  Benutzerdefiniert, Verzeichnis kann normal belassen werden (z.B. C:\SQL2019 oder was auch immer vorgeschlagen wird)
  * [x]  Updates suchen -> Ja
  * [x]  Datenbankmoduldienste, der Rest ist optional
  * [x]  Benutzer der Dienste belassen
  * [x]  Gemischter Modus -> starkes Kennwort vergeben
    * [x]  ggf. aktuellen Benutzer als Admin hinzufügen

* [x]  SSMS installieren

* [x]  myfactory selbst
  * [x]  Setup von Disk ausführen
  * [x]  BWA als Admin starten
  * [x]  Installation vervollständigen
    * [x]  Lizenz eintragen
    * [x]  Vorlagen alle anwählen
    * [x]  Instanzname definieren (Verzeichnisse bitte an Instanznamen anpassen)
  * [x]  Email Adressen der Instanz anpassen
  * [x]  Domain eintragen
 
* [x]  IIS SSL
  * [x]  SSL Zertifikat erzeugen
``
New-SelfSignedCertificate `
    -DnsName "eme-srv-2020.local" `
    -CertStoreLocation "cert:\LocalMachine\My" `
    -FriendlyName "eme-srv-2020.local" `
    -TextExtension "2.5.29.37={text}1.3.6.1.5.5.7.3.1" `
    -KeyUsage DigitalSignature,KeyEncipherment,DataEncipherment `
    -Provider "Microsoft RSA SChannel Cryptographic Provider" `
    -HashAlgorithm "SHA256"
``
    * [x]  Self-Signed via Powershell möglich (zusätzlich eins für localhost erstellen)
        * auf Windows Server 2012 r2 sind die Optionen hier stark eingeschränkt, am einfachsten: auf einem 2016er erstellen und auf dem 2012er importieren
  * [x]  Unter Default Website die Bindungen bearbeiten
    * [ ]  http auf Port 8080 umstellen
    * [x]  Neuer Eintrag localhost mit https und IP (*) und Cert localhost
    * [x]  Neuer Eintrag https mit gewünschter Domain und dem dazugehörigen Cert
      * bitte den Haken bei SNI setzen
    * [ ]  ggf. bei der Default Website und/oder der Anwendung SSL erforderlich auf JA setzen
    * [x] alternativ urlrewrite2 installieren und web.config für http -> https einrichten
      - [x] donwload urlrewrite2 (https://www.iis.net/downloads/microsoft/url-rewrite)
      - [x] Installieren
      - [x] web.config in C:\inetpub\sitename erstellen (Standard ist wwwroot) \
            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <configuration>
              <system.webServer>
                <rewrite>
                  <rules>
                    <rule name="HTTP to HTTPS redirect" stopProcessing="true">
                      <match url="(.*)" />
                        <conditions>
                          <add input="{HTTPS}" pattern="off" />
                        </conditions>
                        <action type="Redirect" redirectType="Permanent" url="https://{HTTP_HOST}/{R:1}" />
                      </rule>
                    </rules>
                  </rewrite>
                </system.webServer>
            </configuration>
            ```

    * [ ] Zertifikat auf allen Endgeräten auch dem Server selbst installieren unter Vertrauenswürdige Stammzertifizierungsstellen
      * [x]  Das Cert ohne Private Key z.B. als DER exportieren, diese Datei via Rechtsklick auf den Geräten installieren, oder über Gruppenrichtlinie verteilen. Export z.B. via IIS auf dem Server -> Serverzertifikate Reiter Details, dann in Datei kopieren

  * [x]  weitere IIS Eintellungen:
    * [x]  bei Anwendungspool Default App Pool in Erweiterten Einstellungen:
      * [x]  Maximale Anzahl von Arbeitsprozessen anpassen (z.B. 2, max halbe Core Anzahl)
      * [x]  Schutz für schnelle Fehler aktiviert auf false setzen
      * [x]  Wiederverwendung -> bestimmte Zeiten einen Zeitpunkt für einen Neustart nachts eintragen (z.B. 02:55:00)

  * [x]  SQL-Server Einstellungen
    * [x]  Arbeitsspeicher begrenzen
      * [x]  via SSMS auf Server Rechtsklick->Eigenschaften
      * [x]  Reiter Arbeitsspeicher
      * [x]  bei maximaler Serverarbeitsspeicher max Hälfte des verfügbaren Arbeitsspeichers setzen

  * [x]  PDF werden nicht angezeigt:
    * [x] Registry Eintrag: \
Windows Registry Editor Version 5.00\
\
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_IVIEWOBJECTDRAW_DMLT9_WITH_GDI]
"w3wp.exe"=dword:00000001

  * [x]  PDF per SSL:
    * [x]  Reg Eintrag setzen: HKEY_LOCAL_MACHINE\SOFTWARE\myfactory\3.0\Options\Instanzname -> neuer REG_SZ mit Namen ForceDocumentsSSL und Wert true

  * [x]  TimerService installieren
    * [x]  .msi ausführen
    * [x]  Programme (x86)->myfactory->TimerService->app.config editieren
      * [x]  AppName = Instanzname
      * [x]  wfTimerService_wfTimerService_wfTimerService Pfad an Instanz anpassen (https ggf. nicht vergessen)
      * [x]  URL -> ggf auf https umstellen
  * [ ]  ServerPrint
    * [x]  Benutzerkonto anlegen
      * [x]  lokaler Admin
      * [x]  Kennwort läuft nie ab
      * [x]  recht als Dienst anmelden
        * [x]  Wählen Sie Start > Systemsteuerung > Verwaltung > Lokale Sicherheitsrichtlinie > Lokale Richtlinien > Zuweisen von Benutzerrechten.
        * [x]  Doppelklicken Sie auf Als Dienst anmelden und dann auf Benutzer oder Gruppe hinzufügen.
        * [x]  Geben Sie den Benutzernamen des Microsoft-Administrators ein und klicken Sie auf OK.
    * [x]  Druckdienst über Setup installieren
      * [x]  erstelltes Benutzerkonto verwenden
    * [ ]  Druckdienst konfigurieren
      * [ ]  myfactoryServerPrintConfig.xml im Installationsverzeichnis öffnen
      * [ ]  Drucker eintragen
      * [ ]  Verzeichnis eintragen
    * [ ]  Administration->Grundlagen->Druck: Verzeichnis konfigurieren
  * [x]  Kennwörter ändern (besonders wichtig bei öffentlicher Domain)
  * [x]  PDFChormeEngine aktivieren (muss pro Datenbank erfolgen)
    * [x]  Prüfen, ob bereits installiert (es müssten dann beide Einträge vorhanden sein): \
``Select * from tdGeneralProperties where PropertyName like 'PDFEngineChrome%'``
    
    * [x]  Wenn nicht installieren
      * [x]  Temp Verzeichnis erstellen (hier im Beispiel C:\work\temp)
      * [x]  tdGeneralProperies-Einträge setzen \
``
insert into tdGeneralProperties
values ('PDFEngineChrome',0,0,-1);
insert into tdGeneralProperties
values ('PDFEngineChromeTempPath',0,0,'C:\work\temp');
``