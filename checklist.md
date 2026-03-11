# Zonenmodell – Review-Checklist für Architektur, Security und Audit

## Zweck des Dokuments

Diese Checkliste dient zur strukturierten Bewertung von Sicherheitszonen, Netzsegmentierung, Plattformarchitektur, Betriebsmodellen, Identitäten, administrativen Zugängen, Recovery-Fähigkeit und Kontrollwirksamkeit.

Sie ist für folgende Einsatzfälle geeignet:

* Architektur-Review neuer Plattformen oder Anwendungen
* Security-Review bestehender Umgebungen
* Freigabe von Zielarchitekturen
* Audit-Vorbereitung und interne Kontrollnachweise
* Reifegradbewertung für Segmentierung und privilegierte Betriebsmodelle
* Review von Cloud-, Hybrid- und On-Prem-Architekturen

Die Checkliste ist auf das in `zones.md` definierte Zonenmodell abgestimmt:

* **Zone 0 – Trust & Recovery Core**
* **Zone 1 – Automation & Platform Core**
* **Zone 2 – Shared Platform Services**
* **Zone 3 – Business Applications**
* **Zone 4 – User, Endpoint & External Edge**

Zusätzlich werden Sonderzonen wie Cyber Recovery, Security Operations, OT/ICS, Partnerintegration und Hochsicherheitsdaten berücksichtigt.

---

## Normative Bewertungsskala

Für jeden Prüfpunkt SOLL eine der folgenden Bewertungen vergeben werden:

* **Erfüllt** – Anforderung ist vollständig und nachweisbar umgesetzt.
* **Teilweise erfüllt** – Anforderung ist nur teilweise umgesetzt, eingeschränkt wirksam oder noch nicht vollständig nachweisbar.
* **Nicht erfüllt** – Anforderung ist nicht umgesetzt oder fachlich nicht belastbar.
* **Nicht anwendbar** – Prüfpunkt ist für den betrachteten Scope fachlich nicht relevant; Begründung ist erforderlich.

Zusätzlich SOLLEN zu jedem Prüfpunkt dokumentiert werden:

* Nachweis oder Referenz
* identifizierte Risiken
* Abweichungen und Kompensationsmaßnahmen
* Verantwortliche Rolle oder Organisationseinheit
* Zieltermin bei offenen Maßnahmen

---

## Pflicht-Metadaten des Reviews

Vor Durchführung der fachlichen Bewertung MÜSSEN folgende Metadaten vollständig vorliegen.

### 1. Review-Identifikation

* Name des reviewed Systems oder Programms
* eindeutige Review-ID
* Datum des Reviews
* Version des Architekturstands
* Version dieser Checkliste
* verantwortliche Organisationseinheit

### 2. Scope

* fachlicher Scope
* technischer Scope
* ausgeschlossene Bereiche
* beteiligte Umgebungen, zum Beispiel Produktion, Pre-Production, Entwicklung, Recovery
* einbezogene Hosting-Modelle, zum Beispiel On-Prem, Cloud, Hybrid, SaaS-Anteile

### 3. Beteiligte Rollen

* Architekturverantwortung
* Security-Verantwortung
* Plattformverantwortung
* Betriebsverantwortung
* IAM-/PKI-/Secrets-Verantwortung
* Netzwerkverantwortung
* Backup-/Recovery-Verantwortung
* Fachverantwortung, sofern relevant

### 4. Pflichtunterlagen

Folgende Unterlagen MÜSSEN für ein belastbares Review vorliegen:

* Ziel- oder Ist-Architekturdiagramm
* Segmentierungs- oder Kommunikationsübersicht
* Systeminventar
* Identitäts- und Rollenmodell
* Administrationsmodell
* Beschreibung von Build-, Deploy- und Change-Prozessen
* Secrets- und Schlüsselkonzept
* Backup- und Recovery-Konzept
* Logging- und Monitoring-Konzept
* Liste externer Anbindungen und Partnerpfade

### Beispiele

Ein Review einer GitOps-Plattform ohne Nachweis des Rollenmodells ist nicht vollständig. Ein Review eines zentralen Vault-Clusters ohne Dokumentation des Unseal-Konzepts und der Root-Abhängigkeiten ist ebenfalls nicht belastbar.

---

# Abschnitt A – Systemverständnis und Kontext

## A.1 Geschäfts- und Betriebszweck

### Prüffragen

* Ist klar beschrieben, welchem fachlichen oder technischen Zweck das System dient?
* Ist beschrieben, ob es Vertrauensanker, Plattformdienst, Shared Service, Business Service oder Edge-Komponente ist?
* Ist dokumentiert, welche Systeme oder Geschäftsprozesse von ihm abhängen?
* Ist beschrieben, welche Schäden bei Ausfall entstehen?
* Ist beschrieben, welche Schäden bei Integritätsverlust oder Kompromittierung entstehen?

### Erwarteter Inhalt

Die Beschreibung MUSS klar zwischen Verfügbarkeit, Integrität, Vertraulichkeit und Kontrollwirkung unterscheiden.

### Beispiele

* Ein Vault-Cluster dient nicht nur der Secret-Speicherung, sondern der kontrollierten Secret-Ausgabe an Plattform und Anwendungen. Seine Kompromittierung kann zu breiter Credential-Übernahme führen.
* Ein gemeinsamer Kubernetes-Cluster stellt Runtime für mehrere Teams bereit. Sein Ausfall beeinträchtigt viele Anwendungen, seine Kompromittierung ermöglicht Seitwärtsbewegung zwischen Workloads.

## A.2 Kritikalität und Schutzbedarf

### Prüffragen

* Wurde der Schutzbedarf für Verfügbarkeit, Integrität und Vertraulichkeit bewertet?
* Wurde die Kontrollwirkung des Systems auf andere Zonen oder Systeme bewertet?
* Wurde bewertet, ob das System für Recovery zwingend erforderlich ist?
* Wurde bewertet, ob das System als Hebel für Supply-Chain-, Privilege-Escalation- oder Lateral-Movement-Angriffe dienen kann?

### Erwarteter Inhalt

Die Bewertung SOLL nachvollziehbar und nicht nur pauschal sein. Ein System mit geringer fachlicher Sichtbarkeit kann trotzdem sehr hohe Kontrollwirkung besitzen.

### Beispiele

* Git für Plattformcode hat meist sehr hohe Integritäts- und Kontrollkritikalität, auch wenn Nutzer es nicht direkt sehen.
* Ein Edge-Webserver hat hohe Exposition, aber nicht automatisch Root-of-Trust-Charakter.

## A.3 Abhängigkeiten

### Prüffragen

* Sind eingehende und ausgehende Abhängigkeiten dokumentiert?
* Ist klar, welche Identitäts-, DNS-, Zertifikats-, Storage-, Messaging- oder Secret-Abhängigkeiten bestehen?
* Ist dokumentiert, welche Systeme das reviewed System ändern oder konfigurieren können?
* Ist dokumentiert, welche Systeme durch das reviewed System geändert werden können?

### Erwarteter Inhalt

Abhängigkeiten MÜSSEN funktional und sicherheitlich beschrieben sein, nicht nur technisch.

### Beispiele

* Ein CI/CD-System hängt von Git, IAM, Registry und Secret-Management ab und kann gleichzeitig Produktionsdeployments auslösen.
* Ein SSO-System hängt von Verzeichnisdienst, Zertifikaten und Zeitquelle ab und steuert gleichzeitig Zugriffe auf viele weitere Systeme.

---

# Abschnitt B – Zonenzuordnung

## B.1 Grundsätzliche Zuordnung

### Prüffragen

* Ist das System einer klaren Zone zugeordnet?
* Ist die Zuordnung begründet?
* Entspricht die Zuordnung der tatsächlichen Funktion statt nur dem technischen Hosting-Ort?
* Ist dokumentiert, ob das System Teil einer Hauptzone oder einer Sonderzone ist?

### Erwarteter Inhalt

Die Zonenzuordnung MUSS aus Funktion, Vertrauenswirkung, Änderungsmacht und Recovery-Relevanz abgeleitet werden.

### Beispiele

* Ein GitOps-Controller in einem Kubernetes-Cluster gehört funktional in Zone 1, auch wenn er technisch auf Shared Runtime in Zone 2 läuft.
* Ein HSM in einer separaten Appliance gehört in Zone 0, auch wenn es physisch im selben Rechenzentrum wie andere Dienste steht.

## B.2 Prüfung auf Zone 0

### Prüffragen

* Ist das System kryptografischer oder identitärer Vertrauensursprung?
* Verwaltet es Root-Schlüssel, Root-Zertifikate, Root-Secrets oder Break-glass-Kontrolle?
* Wird es benötigt, um zentrale Vertrauensketten nach einer Kompromittierung wiederherzustellen?
* Verwalten seine Administratoren Root- oder Recovery-Funktionen für die Gesamtumgebung?

### Erwarteter Inhalt

Wenn eine oder mehrere Fragen klar mit Ja beantwortet werden, MUSS die Einordnung in Zone 0 ernsthaft geprüft und begründet werden.

### Beispiele

* Root-CA, HSM, Root-KMS, Break-glass-Identitäten, Recovery-Tresor.

## B.3 Prüfung auf Zone 1

### Prüffragen

* Steuert das System Änderungen an anderen Systemen großflächig?
* Enthält es Infrastruktur- oder Plattformcode?
* Verteilt es Secrets oder Zertifikate an zentrale Dienste?
* Hält es zentrale Artefakte, Deploy-Artefakte oder Plattformzustände?
* Dient es als CI/CD-, GitOps-, IaC- oder zentrale Orchestrierungsinstanz?

### Erwarteter Inhalt

Wenn diese Merkmale vorliegen, SOLL das System in Zone 1 eingeordnet werden, auch wenn es fachlich nicht sichtbar ist.

### Beispiele

* Git-Plattform für Infrastrukturcode, Vault, Artifactory, Terraform-State-Backend, Argo CD.

## B.4 Prüfung auf Zone 2

### Prüffragen

* Stellt das System gemeinsame Laufzeit- oder Plattformdienste bereit?
* Wird es von mehreren Anwendungen oder Teams genutzt?
* Hat es breite Wirkung bei Ausfall, ohne selbst Root-of-Trust oder globaler Änderungshebel zu sein?

### Beispiele

* Shared Kubernetes Runtime, PostgreSQL-Plattform, Kafka, Monitoring-Plattform.

## B.5 Prüfung auf Zone 3

### Prüffragen

* Dient das System primär einem Fachprozess oder einer Business-Funktion?
* Verarbeitet es geschäftliche Nutzdaten, ohne selbst globaler Plattformkern zu sein?
* Nutzt es Plattformdienste aus Zone 1 und Zone 2?

### Beispiele

* Bestellservice, Kundenportal, HR-Anwendung, Reporting-Service.

## B.6 Prüfung auf Zone 4

### Prüffragen

* Ist das System benutzernah, extern exponiert oder Übergangspunkt?
* Ist die primäre Bedrohungslage Phishing, Browser-Angriff, Malware, Partnerzugriff oder Internet-Exposition?
* Dient das System primär als Zugriffspfad und nicht als innerer Kontrollkern?

### Beispiele

* Mitarbeiter-Laptop, VDI, VPN-Gateway, ZTNA-Zugang, WAF, Reverse Proxy.

## B.7 Misch- oder Mehrrollen-Systeme

### Prüffragen

* Erfüllt das System mehrere Rollen, die eigentlich unterschiedlichen Zonen entsprechen?
* Ist die Trennung logisch, organisatorisch oder technisch ausreichend umgesetzt?
* Wurden Kontroll- und Betriebsfunktionen getrennt, wenn ein System sowohl Runtime als auch Steuerkomponente enthält?

### Erwarteter Inhalt

Mischsysteme SOLLEN vermieden werden. Wenn sie unvermeidbar sind, MUSS die Trennung von Rollen, Identitäten, Netzpfaden und Administrationsrechten besonders gründlich geprüft werden.

### Beispiele

* Ein Kubernetes-Cluster, der sowohl Shared Runtime als auch hochprivilegierte GitOps-Controller hostet, erfordert besonders strenge Trennung.
* Ein Verzeichnisdienst, der gleichzeitig Root-IAM und normalen SSO-Tagesbetrieb abbildet, muss funktional und administrativ sauber segmentiert werden.

---

# Abschnitt C – Kommunikationsmodell und Segmentierung

## C.1 Dokumentation der Kommunikationsbeziehungen

### Prüffragen

* Gibt es eine vollständige Übersicht aller eingehenden und ausgehenden Kommunikationsbeziehungen?
* Sind Richtung, Protokoll, Port, Authentifizierung, Zweck und Gegenstellen dokumentiert?
* Ist erkennbar, welche Beziehungen permanent, temporär oder nur im Notfall erforderlich sind?

### Erwarteter Inhalt

Jede Zonenüberschreitung MUSS dokumentiert und begründet sein.

### Beispiele

* Zone 1 GitOps-Controller → Zone 2 Kubernetes API über mTLS und Service-Identity.
* Zone 3 Anwendung → Zone 2 Datenbank über TLS, servicegebundene Identität und nur auf definiertem Datenbankkonto.

## C.2 Default-Deny und Freigabelogik

### Prüffragen

* Ist das Default-Modell deny by default?
* Werden nur explizit genehmigte Kommunikationsbeziehungen freigeschaltet?
* Ist dokumentiert, wie Freigaben beantragt, geprüft und gepflegt werden?
* Gibt es Rezertifizierung oder regelmäßige Prüfung alter Freigaben?

### Erwarteter Inhalt

Pauschale Offenheit zwischen Segmenten DARF NICHT akzeptiert werden.

## C.3 Rückkanäle und Seitwärtsbewegung

### Prüffragen

* Gibt es Rückkanäle von äußeren Zonen in innere Zonen?
* Sind administrative Schnittstellen aus nicht privilegierten Zonen erreichbar?
* Kann ein kompromittiertes System in Zone 3 oder Zone 4 direkt privilegierte Systeme in Zone 1 erreichen?
* Sind Ost-West-Verkehre innerhalb einer Zone begrenzt, wo dies fachlich sinnvoll ist?

### Beispiele

* Eine Anwendung in Zone 3 darf nicht beliebig auf Vault-Administrationsschnittstellen in Zone 1 zugreifen.
* Ein Entwickler-Laptop in Zone 4 darf nicht direkt Root-KMS-Administration auslösen.

## C.4 Protokoll- und Identitätssicherheit

### Prüffragen

* Werden sichere Protokolle verwendet?
* Ist die Gegenstelle authentisiert?
* Werden Zertifikate, mTLS, signierte Tokens oder gleichwertige Mechanismen eingesetzt?
* Ist Klartextverkehr in sensitiven Übergängen ausgeschlossen?

### Beispiele

* mTLS für Controller-zu-API-Kommunikation.
* Kurzlebige Tokens für Secret-Zugriffe statt lang lebender Shared Credentials.

## C.5 Management- und Dataplane-Trennung

### Prüffragen

* Sind Management- und Dataplane-Kommunikation getrennt?
* Sind Management-Schnittstellen restriktiver geschützt als normale Servicezugriffe?
* Gibt es separate Netze, Policies oder Identitäten für Managementpfade?

### Beispiele

* Storage-Datenpfad für Anwendungstraffic getrennt von Admin-Interface des Storage-Clusters.
* Kubernetes Workload-Traffic getrennt von API-Server-Administrationszugang.

---

# Abschnitt D – Administrationsmodell und privilegierte Zugänge

## D.1 Getrennte Administrationspfade

### Prüffragen

* Existieren separate Administrationspfade für Zone 0 und Zone 1?
* Sind diese Pfade gehärtet und überwacht?
* Werden privilegierte Tätigkeiten nicht von Standard-Endgeräten ausgeführt?

### Erwarteter Inhalt

Für innere Zonen MÜSSEN separate und belastbare Admin-Pfade vorhanden sein.

### Beispiele

* PAW für Vault-Administration statt Standard-Laptop.
* Separater Jump Host für Zone-2-Betrieb mit starker Authentisierung.

## D.2 Trennung von Identitäten

### Prüffragen

* Gibt es getrennte Konten für Standardarbeit und privilegierte Arbeit?
* Sind Zone-0-Rollen strikt von Zone-1- und Zone-2-Rollen getrennt?
* Werden Break-glass-Identitäten im Normalbetrieb nicht verwendet?

### Beispiele

* Ein Administrator nutzt ein normales Konto für Kommunikation und ein separates privilegiertes Konto für Plattformbetrieb.
* Root-IAM-Identität wird nur im dokumentierten Notfallprozess genutzt.

## D.3 Vergabe und Aktivierung privilegierter Rechte

### Prüffragen

* Werden privilegierte Rechte zeitlich begrenzt vergeben?
* Ist ein Genehmigungsprozess vorhanden?
* Werden Sitzungen oder Rollenwechsel protokolliert?
* Werden stehende Superuser-Rechte minimiert?

### Beispiele

* JIT-Admin-Rolle für 30 Minuten mit Ticket-Bezug.
* Protokollierte Freigabe eines Produktionszugriffs für einen klaren Wartungsfall.

## D.4 Härtung administrativer Systeme

### Prüffragen

* Sind Admin-Arbeitsplätze oder Bastions speziell gehärtet?
* Ist riskante Nutzung wie allgemeines Browsing oder E-Mail auf diesen Systemen eingeschränkt?
* Werden lokale Administratorrechte minimiert?
* Sind Schutzmaßnahmen gegen Credential Theft aktiv?

### Beispiele

* PAW ohne E-Mail-Client, ohne Internet-Browsing, mit Device Control und starker MFA.
* Bastion mit restriktiven Zielsystemlisten und vollständiger Session-Protokollierung.

## D.5 Session-Sicherheit und Nachvollziehbarkeit

### Prüffragen

* Werden privilegierte Sitzungen nachvollziehbar protokolliert?
* Ist die Nutzung geteilter Administrationskonten ausgeschlossen oder minimiert?
* Sind Notfallzugriffe besonders überwacht?

---

# Abschnitt E – Identitäten, Rollen und Berechtigungen

## E.1 Rollenmodell

### Prüffragen

* Gibt es ein dokumentiertes Rollenmodell?
* Sind Rollen zonenspezifisch statt pauschal definiert?
* Ist zwischen Entwickler-, Betreiber-, Security-, Audit- und Recovery-Rollen unterschieden?

### Erwarteter Inhalt

Rollen MÜSSEN zweckgebunden und least-privileged sein.

## E.2 Menschliche und technische Identitäten

### Prüffragen

* Sind menschliche und technische Identitäten getrennt?
* Werden technische Identitäten nicht interaktiv von Menschen verwendet?
* Haben Service Accounts nur die minimal nötigen Rechte?

### Beispiele

* CI/CD-Controller besitzt nur Deploy-Rechte, aber keinen generischen Administrationszugang auf beliebige Systeme.
* Ein Anwendungspod nutzt eine eigene Workload-Identität statt geteiltem Datenbank-Passwort.

## E.3 Lebensdauer und Rotation von Credentials

### Prüffragen

* Sind Credentials kurzlebig, wo dies technisch möglich ist?
* Gibt es Rotation für Schlüssel, Tokens, Zertifikate und Passwörter?
* Ist dokumentiert, wie kompromittierte Credentials widerrufen oder ersetzt werden?

### Beispiele

* Kurzlebige Vault-Tokens für Jobs.
* Regelmäßige Rotation von Datenbank- oder API-Secrets.
* Zertifikatswiderruf für kompromittierte Maschinenidentitäten.

## E.4 Berechtigungsprüfung und Rezertifizierung

### Prüffragen

* Werden privilegierte Rechte regelmäßig überprüft?
* Gibt es Rezertifizierung für Rollen in Zone 0 und Zone 1?
* Sind verwaiste Rechte identifizierbar und entziehbar?

### Beispiele

* Vierteljährliche Prüfung aller Vault-Admin-Rollen.
* Rezertifizierung aller Git-Maintainer für Produktionsinfrastruktur-Repositories.

## E.5 Trennung kritischer Zuständigkeiten

### Prüffragen

* Ist Trennung kritischer Funktionen umgesetzt?
* Kann dieselbe Person nicht unkontrolliert Code ändern, Secrets verwalten und Deployment freigeben?
* Sind Recovery-Root-Rechte von Tagesbetriebsrechten getrennt?

### Beispiele

* Entwickler darf Pipeline-Definition anpassen, aber nicht allein Produktionsfreigabe erteilen.
* Vault-Root-Entsperrung erfordert separates Verfahren mit mehreren Verantwortlichen.

---

# Abschnitt F – Vertrauensanker, PKI, Secrets und Schlüssel

## F.1 Root-of-Trust-Klärung

### Prüffragen

* Ist klar definiert, was in dieser Architektur Root of Trust ist?
* Sind Root-Schlüssel, Root-Zertifikate, HSMs, Root-KMS und Break-glass-Mechanismen identifiziert?
* Sind diese Komponenten in Zone 0 verortet oder entsprechend geschützt?

### Beispiele

* Root-CA offline in Zone 0.
* KMS-Root für Plattform-Secrets in Zone 0.

## F.2 PKI- und Zertifikatsmanagement

### Prüffragen

* Gibt es klare Trennung zwischen Root-CA, Intermediate-CA und Zertifikatsnutzung?
* Sind Ausstellung, Verlängerung und Widerruf dokumentiert?
* Sind Schlüsselmaterialien angemessen geschützt?

### Beispiele

* Intermediate-CA für Plattformdienste in Zone 1, Root-CA in Zone 0.
* Widerrufsprozess für kompromittierte API-Gateway-Zertifikate.

## F.3 Secrets-Management

### Prüffragen

* Werden Secrets zentral und kontrolliert verwaltet?
* Ist festgelegt, welche Systeme Secrets lesen, erzeugen, rotieren oder administrieren dürfen?
* Gibt es Trennung zwischen Secret-Nutzung und Secret-Administration?
* Werden Secrets nicht dauerhaft ungeschützt auf Filesystemen, Images oder Build-Workern abgelegt?

### Beispiele

* Anwendung liest Datenbank-Credential zur Laufzeit aus Vault.
* CI-Runner erhält nur temporären Zugriff auf ein Build-Secret.

## F.4 Unseal-, Root- und Recovery-Mechanismen

### Prüffragen

* Sind Unseal- oder Recovery-Mechanismen dokumentiert?
* Sind sie getrennt von regulären Betriebsrechten?
* Werden sie nur kontrolliert und nachvollziehbar genutzt?

### Beispiele

* Vault-Unseal über HSM/KMS mit zusätzlichem Break-glass-Prozess.
* Recovery-Schlüssel in getrennter, offline geschützter Verwahrung.

## F.5 Geheimnisverbreitung und Blast Radius

### Prüffragen

* Ist der Blast Radius eines kompromittierten Secrets bewertet?
* Werden Secrets zonenspezifisch begrenzt?
* Sind Shared Credentials minimiert?

### Beispiele

* Ein Secret für ein einzelnes Deployment darf nicht gleichzeitig globale Schreibrechte auf Registry, Git und Cluster besitzen.

---

# Abschnitt G – Software-Lieferkette, Git, CI/CD und Automatisierung

## G.1 Integrität von Quellcode und Konfiguration

### Prüffragen

* Sind kritische Repositories identifiziert?
* Gibt es Branch-Schutz, Review-Pflicht und Änderungsnachvollziehbarkeit?
* Werden signierte Commits oder gleichwertige Maßnahmen genutzt?
* Sind besonders kritische Repositories für Infrastruktur und Policies restriktiver geschützt als normaler Business-Code?

### Beispiele

* Infrastruktur-Repository erfordert Pull Request, Zwei-Personen-Freigabe und geschützten Main-Branch.
* Policy-Repository darf nur von kleiner Plattform-/Security-Gruppe geändert werden.

## G.2 Build- und Pipeline-Sicherheit

### Prüffragen

* Sind Build-Pipelines gegen Manipulation geschützt?
* Ist klar, wer Pipelines ändern darf?
* Werden Build-Umgebungen isoliert?
* Können Runner nicht beliebig auf alle Secrets oder Zielsysteme zugreifen?

### Beispiele

* Getrennte Runner für untrusted und trusted Workloads.
* Produktionsdeployments nur aus attestierten Build-Artefakten.

## G.3 Deployment-Sicherheit

### Prüffragen

* Ist dokumentiert, welche Instanz Deployments auslösen darf?
* Gibt es Trennung zwischen Build und Deploy?
* Sind Produktionsdeployments stärker kontrolliert als Entwicklungsdeployments?
* Sind manuelle Notfallpfade dokumentiert und abgesichert?

### Beispiele

* Argo CD deployt nur aus signierten, freigegebenen Repositories.
* Notfall-Deployment erfordert separaten Freigabeprozess und wird besonders überwacht.

## G.4 Artefakte und Registry

### Prüffragen

* Sind Artefaktquellen vertrauenswürdig und nachvollziehbar?
* Wird Signatur oder Attestierung genutzt?
* Ist dokumentiert, wer Artefakte schreiben, löschen oder promoten darf?
* Sind Produktionsartefakte vor Überschreibung und unkontrollierter Löschung geschützt?

### Beispiele

* Container-Registry mit separaten Rechten für Push, Pull und Promotion.
* Paket-Repository mit Freigabestufen für Dev, Test und Prod.

## G.5 IaC und Policy as Code

### Prüffragen

* Ist dokumentiert, welche Infrastruktur per Code gesteuert wird?
* Sind State-Backends geschützt?
* Werden Plan-, Apply- und Policy-Schritte nachvollziehbar getrennt?
* Sind Drift, Out-of-Band-Änderungen und privilegierte Direktzugriffe kontrolliert?

### Beispiele

* Terraform-State in geschütztem Backend mit separaten Schreibrechten.
* Direkte Produktionsänderung an Cloud-Ressourcen nur in Notfällen mit Audit-Nachweis.

---

# Abschnitt H – Plattformdienste und Shared Services

## H.1 Gemeinsame Runtimes

### Prüffragen

* Ist bei Shared Kubernetes, VM-Plattform oder Container-Runtime die Mandantentrennung ausreichend?
* Sind Namespaces, Clusterrollen, Netzpolicies und Secret-Zugriffe restriktiv ausgeprägt?
* Kann ein kompromittierter Workload auf andere Mandanten oder Plattformkomponenten übergreifen?

### Beispiele

* Trennung von Plattform-Namespaces und Business-Namespaces.
* Keine Cluster-Admin-Rechte für normale Anwendungsteams.

## H.2 Datenbank- und Messaging-Plattformen

### Prüffragen

* Sind Datenbank- oder Messaging-Rechte pro Anwendung getrennt?
* Gibt es kein generisches Shared Admin-Credential für alle Anwendungen?
* Sind Admin-Schnittstellen stärker geschützt als Nutzpfade?

### Beispiele

* Pro Anwendung separates Datenbankkonto statt gemeinsamer DB-Superuser.
* Kafka-ACLs pro Topic oder Produzent/Konsument.

## H.3 Monitoring, Logging, Observability

### Prüffragen

* Sind Observability-Plattformen gegen Manipulation geschützt?
* Ist geregelt, wer Logs sehen, ändern oder löschen darf?
* Haben Telemetrie-Agenten nur minimale Rechte?
* Ist die besondere Sensitivität von Logs aus Zone 0 und Zone 1 berücksichtigt?

### Beispiele

* Log-Backend erlaubt Löschen nur für eng begrenzte Rollen.
* Audit-Logs werden separat und manipulationsarm gespeichert.

## H.4 Backup-Plattformen

### Prüffragen

* Ist die Backup-Plattform selbst segmentiert und gehärtet?
* Sind Backup-Administrationsrechte getrennt von normalen Plattformrechten?
* Gibt es Schutz vor unautorisierter Löschung oder Verschlüsselung von Backups?

### Beispiele

* Immutable Backup-Storage.
* Separater Backup-Admin ohne generischen Domain-Admin-Zugang.

---

# Abschnitt I – Business Applications

## I.1 Minimale Rechte gegenüber inneren Zonen

### Prüffragen

* Greift die Anwendung nur auf die unbedingt nötigen Plattformdienste zu?
* Sind Zugriffe auf Zone 1 und Zone 2 minimal und zweckgebunden?
* Gibt es keine allgemeinen Admin-APIs oder breiten Secrets-Zugriffe aus der Anwendung heraus?

### Beispiele

* Anwendung liest nur ihr eigenes Secret, nicht beliebige Vault-Pfade.
* Anwendung nutzt nur das eigene Datenbankschema.

## I.2 Anwendungssegmentierung

### Prüffragen

* Ist Ost-West-Kommunikation zwischen Anwendungen kontrolliert?
* Sind unterschiedliche Schutzbedarfe oder Mandanten getrennt?
* Sind exponierte Komponenten von internen Backends getrennt?

### Beispiele

* Frontend und Backend in getrennten Segmenten oder Namespaces.
* Kein ungefilterter Direktzugriff von einer Reporting-Anwendung auf alle Fachservices.

## I.3 Laufzeit- und Schnittstellensicherheit

### Prüffragen

* Sind Eingaben, APIs und externe Integrationen abgesichert?
* Sind Ratenbegrenzung, Authentisierung und Transportverschlüsselung umgesetzt?
* Werden Abhängigkeiten zu Drittkomponenten kontrolliert?

### Beispiele

* Kundenportal hinter WAF, TLS, API-Auth und Rate Limiting.
* Interne Service-zu-Service-Kommunikation mit Service-Identity statt Shared Token.

---

# Abschnitt J – User, Endpoints und Edge

## J.1 Endgerätesicherheit

### Prüffragen

* Sind Endgeräte verwaltet, gehärtet und überwacht?
* Sind EDR, Verschlüsselung, Patch-Management und Gerätezustandsprüfung aktiv?
* Ist klar, welche Endgeräte überhaupt sensitive Zugriffe durchführen dürfen?

### Beispiele

* Nur compliant verwaltete Geräte dürfen auf Admin-Portale zugreifen.
* Unmanaged BYOD-Geräte erhalten keinen direkten Zugriff auf sensitive interne Dienste.

## J.2 Benutzerzugänge

### Prüffragen

* Sind Authentisierung und MFA für sensitive Zugriffe angemessen?
* Wird Zugriffsrisiko anhand von Kontext, Standort, Gerät oder Anomalien bewertet?
* Sind Partner- oder Drittzugänge gesondert behandelt?

### Beispiele

* Partnerzugang nur auf definierte API-Endpunkte, nicht ins interne Managementnetz.
* Höhere Authentisierungsstufe für Zugriffe auf Plattformadministration.

## J.3 Edge- und Zugangsschichten

### Prüffragen

* Sind WAF, Reverse Proxies, ZTNA, VPN oder Zugangsgateways segmentiert?
* Ist klar, welche inneren Dienste veröffentlicht werden?
* Bestehen keine unnötigen Direktverbindungen von Edge zu inneren Verwaltungsfunktionen?

### Beispiele

* Reverse Proxy veröffentlicht nur definierte Frontends, nicht Vault-Admin-UI oder Kubernetes-API.
* VPN gewährt keine pauschale Erreichbarkeit aller internen Netze.

---

# Abschnitt K – Logging, Auditierbarkeit und Security Monitoring

## K.1 Protokollierung sicherheitsrelevanter Ereignisse

### Prüffragen

* Werden privilegierte Anmeldungen, Rollenwechsel, Secret-Zugriffe, Policies, Deployments und Konfigurationsänderungen protokolliert?
* Sind Logquellen vollständig identifiziert?
* Sind kritische Ereignisse aus Zone 0 und Zone 1 besonders hervorgehoben?

### Beispiele

* Jeder Zugriff auf Root-KMS wird protokolliert.
* Änderungen an Git-Branch-Schutzregeln werden erfasst.

## K.2 Integrität und Verfügbarkeit von Logs

### Prüffragen

* Sind Logs manipulationsarm gespeichert?
* Können Angreifer nach einer Kompromittierung ihre Spuren nicht einfach löschen?
* Bestehen Aufbewahrungs- und Schutzmechanismen für Forensik?

### Beispiele

* Write-once- oder immutable Log-Storage für Audit-Daten.
* Separates Security-Logging außerhalb des primären Anwendungskontos.

## K.3 Erkennung und Reaktion

### Prüffragen

* Gibt es Erkennung für ungewöhnliche Admin-Aktivität, Secret-Missbrauch, Policy-Änderungen oder verdächtige Deployments?
* Sind Eskalations- und Incident-Prozesse definiert?
* Ist klar, welche Telemetrie für welche Zone zwingend ist?

### Beispiele

* Alarm bei Nutzung von Break-glass-Identität.
* Alarm bei Deaktivierung von Audit-Logging in einer CI/CD-Plattform.

---

# Abschnitt L – Backup, Restore, Cyber Recovery und Wiederanlauf

## L.1 Schutz der Backups

### Prüffragen

* Sind Backups gegen Löschung, Überschreibung und Verschlüsselung geschützt?
* Gibt es getrennte Rollen für Backup-Verwaltung?
* Sind besonders kritische Backups unveränderlich oder logisch isoliert?

### Beispiele

* Immutable Snapshots für Plattform- und Identitätsdaten.
* Air-gapped oder logisch getrennte Recovery-Kopien.

## L.2 Wiederherstellungsfähigkeit pro Zone

### Prüffragen

* Ist dokumentiert, wie Zone 0 wiederhergestellt wird?
* Ist dokumentiert, wie Zone 1 nach einem Sicherheitsvorfall wiederhergestellt wird?
* Können Zone 2 und Zone 3 kontrolliert aus einer vertrauenswürdigen Basis neu aufgebaut werden?
* Ist klar, in welcher Reihenfolge die Wiederherstellung erfolgt?

### Beispiele

* Zuerst Root-KMS und Root-CA, danach Vault und GitOps, danach Shared Runtime und Anwendungen.

## L.3 Praktische Tests

### Prüffragen

* Wurden Wiederherstellungsverfahren praktisch getestet?
* Wurden Szenarien wie Ransomware, Credential-Komprimittierung, Registry-Manipulation oder Git-Kompromittierung geübt?
* Sind Testergebnisse und Lessons Learned dokumentiert?

### Beispiele

* Test eines Neuaufbaus der GitOps-Steuerung aus signierten Repositories.
* Test des Restore einer PKI-Intermediate aus dokumentiertem Verfahren.

## L.4 Clean-Room- und Recovery-Design

### Prüffragen

* Gibt es eine isolierte Recovery- oder Clean-Room-Option für Kernsysteme?
* Können kritische Systeme neu aufgesetzt werden, ohne kompromittierte Hilfssysteme zu vertrauen?
* Sind Recovery-Assets außerhalb der normalen Managementdomäne geschützt?

### Beispiele

* Separater Recovery-Tenant oder Recovery-VLAN.
* Offline verwahrte Root-Materialien für Notfall-Neuinitialisierung.

---

# Abschnitt M – Sonderzonen

## M.1 Cyber-Recovery-Zone

### Prüffragen

* Existiert eine separate Recovery-Zone?
* Ist sie von Produktiv-Identitäten und Standard-Admin-Pfaden getrennt?
* Sind dort nur klar definierte Recovery-Komponenten zugelassen?

## M.2 Security-Operations-Zone

### Prüffragen

* Haben EDR-, SIEM- oder Forensiksysteme weitreichende Sicht oder Eingriffsrechte?
* Ist diese besondere Macht segmentiert und kontrolliert?
* Sind Security-Administratoren von Plattform-Administratoren getrennt?

## M.3 OT-/ICS-Zone

### Prüffragen

* Gibt es OT-/ICS-Komponenten im Scope?
* Wurden deren besondere Verfügbarkeits- und Sicherheitsanforderungen separat betrachtet?
* Sind Übergänge zwischen IT und OT streng definiert?

## M.4 Partner- und Integrationszonen

### Prüffragen

* Sind Partneranbindungen gesondert segmentiert?
* Haben Partner keinen unnötigen Zugang zu inneren Zonen?
* Sind Verträge, technische Kontrollen und Monitoring abgestimmt?

## M.5 Hochsicherheits-Datenzonen

### Prüffragen

* Gibt es Daten mit besonders hohem Geheimhaltungsbedarf?
* Sind diese Daten zusätzlich logisch oder physisch getrennt?
* Ist geregelt, welche Personen, Systeme und Transferpfade zulässig sind?

---

# Abschnitt N – Cloud-, SaaS- und Hybridbesonderheiten

## N.1 Funktionsbasierte Zonierung trotz Cloud-Abstraktion

### Prüffragen

* Wurde die Zonierung funktional und nicht nur nach Cloud-Konto oder VPC vorgenommen?
* Sind Management-Plane, Workload-Plane und Recovery-Pfade getrennt bewertet?

### Beispiele

* Cloud-KMS-Root-Berechtigungen entsprechen funktional Zone 0.
* GitOps und zentrale CI/CD in Cloud entsprechen funktional Zone 1.

## N.2 Mandanten, Accounts, Subscriptions und Projekte

### Prüffragen

* Sind Cloud-Accounts oder Subscriptions entlang von Kontroll- und Schutzgrenzen getrennt?
* Können Workloads nicht unkontrolliert in Management- oder Shared-Service-Accounts eingreifen?
* Sind Organisations-Policies und Guardrails vorhanden?

### Beispiele

* Separater Management-Account für zentrale Plattformsteuerung.
* Getrennter Recovery-Account mit eingeschränkter Vertrauensstellung.

## N.3 SaaS-Abhängigkeiten

### Prüffragen

* Ist klar, welche SaaS-Dienste zonenkritische Funktionen übernehmen?
* Sind Admin-, API- und Break-glass-Zugänge für SaaS kontrolliert?
* Ist dokumentiert, welche Daten, Identitäten und Schlüssel in SaaS liegen?

### Beispiele

* SaaS-Git-Plattform für Plattformcode ist funktional weiterhin Zone 1.
* Externes IdP mit Root-ähnlicher Wirkung benötigt Zone-0- oder Zone-1-gerechte Kontrollen.

---

# Abschnitt O – Governance, Ausnahmen und Nachweisführung

## O.1 Ausnahmebehandlung

### Prüffragen

* Gibt es dokumentierte Ausnahmen von MUSS- oder SOLL-Vorgaben?
* Sind diese risikobewertet und genehmigt?
* Sind Kompensationsmaßnahmen dokumentiert?
* Haben Ausnahmen ein Verfallsdatum oder Review-Intervall?

### Beispiele

* Temporär nicht trennbare Altanwendung in Zone 2 mit zusätzlicher Netzwerkisolation und eng überwachten Admin-Pfaden.

## O.2 Verantwortlichkeiten

### Prüffragen

* Ist pro Zone und pro Kernsystem ein Owner benannt?
* Ist klar, wer Zugriffe genehmigt, wer Änderungen bewertet und wer Recovery verantwortet?
* Sind Security- und Architekturentscheidungen nachvollziehbar dokumentiert?

## O.3 Nachweisführung

### Prüffragen

* Sind für Erfüllungsaussagen belastbare Nachweise vorhanden?
* Wurden Diagramme, Policies, Konfigurationen, Rollenlisten, Logauszüge, Testprotokolle oder Runbooks referenziert?
* Ist die Nachweisführung auditierbar und reproduzierbar?

---

# Abschnitt P – Bewertungszusammenfassung

Für den Abschluss des Reviews SOLL folgende Zusammenfassung erstellt werden.

## P.1 Zoneneinstufung des reviewed Systems

* primäre Zone
* ggf. Sonderzone
* Begründung der Einordnung
* relevante Abhängigkeiten zu anderen Zonen

## P.2 Hauptrisiken

* Top-Risiken nach Kritikalität
* betroffene Zonen
* mögliche Schadenswirkungen
* Eintrittspfad oder Missbrauchsszenario

## P.3 Positive Feststellungen

* wirksame Kontrollen
* sauber getrennte Pfade
* vorbildliche Nachweise
* gute Recovery-Fähigkeit

## P.4 Abweichungen und Maßnahmen

* offene Befunde
* Klassifikation nach Kritikalität
* empfohlene Maßnahmen
* Owner
* Zieltermine

## P.5 Freigabeempfehlung

Eine Freigabeempfehlung SOLL mindestens eine der folgenden Aussagen enthalten:

* freigabefähig ohne Auflagen
* freigabefähig mit Auflagen
* freigabefähig nur für begrenzten Scope oder befristet
* nicht freigabefähig

Die Begründung MUSS die wichtigsten Abweichungen und Restrisiken benennen.

---

# Kompakte Schnellprüfung für häufige Kernsysteme

## 1. Git für Plattform- oder Infrastrukturcode

MUSS geprüft werden:

* Zone-1-Einstufung
* geschützte Branches
* Review-Zwang
* restriktive Maintainer-Rollen
* Signatur oder gleichwertige Nachvollziehbarkeit
* Protokollierung kritischer Änderungen
* Trennung von Business-Code und Plattform-Code, sofern sinnvoll

Beispiel:
Ein Git-Repository, das Clusterkonfigurationen und Netzwerkpolicies enthält, ist nicht nur Entwicklerwerkzeug, sondern Teil des operativen Kerns.

## 2. Vault oder Secrets-Plattform

MUSS geprüft werden:

* Zone-1-Einstufung für den Dienst
* Zone-0-Einstufung für Unseal-/Root-Komponenten
* Trennung von Secret-Nutzung und Administration
* Protokollierung aller sensitiven Zugriffe
* Rotation und kurze Lebensdauer von Tokens
* Recovery- und Break-glass-Verfahren

Beispiel:
Wenn ein kompromittierter CI-Runner globale Vault-Leserechte besitzt, ist die Architektur nicht ausreichend segmentiert.

## 3. SSO/IAM

MUSS geprüft werden:

* Trennung von Root-Funktionen und Tagesbetrieb
* Schutz besonders privilegierter Rollen
* MFA und starke Authentisierung
* Protokollierung von Rollenänderungen
* Break-glass-Prozess
* Abhängigkeit zu Verzeichnisdienst, PKI und Recovery

Beispiel:
Ein IdP, der Zugang zu Git, Vault, Cloud-Management und Produktionsplattformen vermittelt, besitzt massive Kontrollwirkung und braucht innere Zonenkontrollen.

## 4. Registry / Artefakt-Repository

MUSS geprüft werden:

* Zone-1-Einstufung bei Steuerwirkung auf Deployments
* getrennte Rechte für Push, Pull, Promotion, Delete
* Schutz vor Überschreibung produktiver Artefakte
* Signatur, Provenance oder Attestierung
* Backup und Restore kritischer Metadaten

## 5. Terraform / IaC-Steuerung

MUSS geprüft werden:

* Zone-1-Einstufung
* Schutz des State-Backends
* Trennung von Plan und Apply
* Rechte auf Cloud- oder Infrastrukturkonten
* Umgang mit Drift und Out-of-Band-Änderungen
* Auditierbarkeit aller Infrastrukturänderungen

## 6. Shared Kubernetes Plattform

MUSS geprüft werden:

* Zone-2-Einstufung für Runtime
* Trennung von Control-Funktionen und Workloads
* keine unnötigen Cluster-Admin-Rechte
* Netzpolicies und Namespace-Trennung
* Secret-Zugriffe nur minimal
* Admin-Zugriff auf API-Server restriktiv

## 7. Edge / VPN / ZTNA

MUSS geprüft werden:

* Zone-4-Einstufung
* keine pauschale Netzfreischaltung nach innen
* segmentierte Freigaben je Zielsystem oder Zielgruppe
* starke Authentisierung und Gerätezustand
* Nachvollziehbarkeit privilegierter Zugriffe

---

# Minimaler Pflichtsatz für jede Review-Freigabe

Eine Architektur SOLL nicht freigegeben werden, wenn einer der folgenden Punkte nicht belastbar geklärt ist:

* Zonenzuordnung des Systems
* Kommunikationspfade über Zonengrenzen
* Administrationspfade und privilegierte Identitäten
* Root-of-Trust und Recovery-Abhängigkeiten
* Rollen- und Berechtigungsmodell
* Logging sicherheitsrelevanter Aktionen
* Restore- und Wiederanlaufverfahren für kritische Komponenten

---

# Abschluss

Diese Checkliste ist absichtlich vollständig und streng. Sie soll verhindern, dass Sicherheitszonen nur formal auf Diagrammen existieren, während Identitäten, Admin-Zugänge, Supply Chain, Secrets und Recovery in Wirklichkeit ungetrennt bleiben.

Der wichtigste Prüfgedanke lautet:

**Nicht nur fragen, wo ein System läuft, sondern was es kontrollieren, kompromittieren, wiederherstellen oder großflächig beeinflussen kann.**

Erst daraus ergibt sich die richtige Zoneneinordnung – und daraus wiederum die richtige Tiefe an Segmentierung, Härtung, Governance und Recovery-Vorsorge.
