# Sicherheitszonen für Netzwerk- und IT-Infrastruktur

## Dokumentstatus und Zielsetzung

Dieses Dokument beschreibt ein Zonenmodell zur sicherheits- und architekturgerechten Einteilung von Netzwerk-, Plattform- und IT-Infrastrukturen. Es ist für Architekturentscheidungen, Security Reviews, Plattformdesign, Segmentierungskonzepte, Berechtigungsmodelle, Recovery-Planung und Audit-Vorbereitung ausgelegt.

Das Modell ist bewusst so formuliert, dass es sowohl in klassischen Rechenzentren als auch in hybriden und cloudbasierten Umgebungen anwendbar ist. Es ist technologieoffen, aber hinreichend präzise, um als Bewertungsmaßstab in Architektur- und Security-Reviews zu dienen.

Zusätzlich verwendet dieses Dokument normative Steuerungsbegriffe:

* **MUSS**: verbindliche Anforderung
* **SOLL**: starke fachliche Empfehlung; Abweichungen sind zu begründen
* **KANN**: optionale Maßnahme, abhängig von Kontext, Risiko und Wirtschaftlichkeit

Dieses Dokument beantwortet insbesondere die Frage, wie Kernkomponenten für minimale Betriebs- und Automatisierungsfähigkeit – zum Beispiel Git, Auth, Vault, Artefakt-Storage und State-Storage – einzuordnen sind.

---

## Geltungsbereich

Dieses Zonenmodell gilt für:

* Netzwerksegmentierung
* Identitäts- und Berechtigungsarchitektur
* Plattform- und Automatisierungsdienste
* zentrale Infrastrukturdienste
* geschäftskritische Anwendungen
* Administrations- und Betriebszugänge
* Logging, Monitoring und Telemetrie
* Backup-, Restore- und Recovery-Architektur
* hybride und cloudbasierte Betriebsmodelle

Es gilt nicht automatisch unverändert für:

* OT-, ICS- oder SCADA-Umgebungen
* Hochsicherheitsumgebungen mit zusätzlichen Geheimhaltungsanforderungen
* stark regulierte Umgebungen mit abweichenden gesetzlichen Spezialanforderungen

Für solche Bereiche SOLLEN ergänzende Sonderzonen und domänenspezifische Kontrollen definiert werden.

---

## Architekturprinzipien

### 1. Zonierung nach Vertrauenswirkung, nicht nur nach Technik

Systeme DÜRFEN nicht allein nach Technologieklasse gruppiert werden. Die Einordnung MUSS anhand folgender Kriterien erfolgen:

* Funktion des Systems
* Grad der administrativen Einflussnahme auf andere Systeme
* Rolle in der Vertrauenskette
* Rolle im Wiederanlauf und in der Recovery
* Schadenswirkung bei Kompromittierung
* Schadenswirkung bei Ausfall

### 2. Administrative Macht ist höher zu bewerten als reine Datenverarbeitung

Systeme, die Änderungen an anderen Systemen ausrollen, Identitäten ausstellen, Secrets verteilen, Policies erzwingen oder Vertrauensanker verwalten, MÜSSEN grundsätzlich weiter innen eingeordnet werden als Systeme, die nur Geschäftslogik ausführen.

### 3. Vertrauen wird von innen nach außen aufgebaut

Innere Zonen MÜSSEN als Vertrauens- und Kontrollkern betrachtet werden. Äußere Zonen DÜRFEN keine unkontrollierten administrativen oder privilegierten Rückkanäle in innere Zonen besitzen.

### 4. Recovery ist Teil des Sicherheitsmodells

Die Zonierung MUSS nicht nur den Normalbetrieb, sondern auch Sicherheitsvorfälle, Domänenkompromittierung, Ransomware-Szenarien und Wiederanlauf berücksichtigen.

### 5. Eine Zone ist mehr als ein Netzsegment

Eine Zone DARF nicht als bloße VLAN- oder Subnetzdefinition verstanden werden. Die wirksame Umsetzung SOLL mindestens folgende Ebenen umfassen:

* Netzsegmentierung
* Identitäten und Rollen
* administrative Zugangswege
* Secrets- und Schlüsselverwaltung
* Policy Enforcement
* Logging und Nachvollziehbarkeit
* Build-, Deploy- und Change-Kontrollen

---

## Überblick über das Zielmodell

Das empfohlene Zielmodell besteht aus fünf Hauptzonen:

* **Zone 0 – Trust & Recovery Core**
* **Zone 1 – Automation & Platform Core**
* **Zone 2 – Shared Platform Services**
* **Zone 3 – Business Applications**
* **Zone 4 – User, Endpoint & External Edge**

Ergänzend KÖNNEN Sonderzonen definiert werden, zum Beispiel für Cyber Recovery, Security Operations, OT/ICS, Partneranbindung oder hochsensible Datenverarbeitung.

---

# Zone 0 – Trust & Recovery Core

## Zweck

Zone 0 enthält die Vertrauensanker und Recovery-Funktionen, ohne die die Gesamtumgebung nicht mehr sicher kontrolliert, attestiert oder wiederhergestellt werden kann.

Zone 0 ist die innerste Zone. Eine Kompromittierung dieser Zone ist als struktureller Kontrollverlust über die Gesamtumgebung zu bewerten.

## Einordnungskriterien

Ein System MUSS Zone 0 zugeordnet werden, wenn mindestens eines der folgenden Merkmale zutrifft:

* Es ist kryptografischer Vertrauensursprung.
* Es verwaltet Root-Schlüssel oder gleichwertige Root-Geheimnisse.
* Es stellt Notfall- oder Break-glass-Kontrolle über die Gesamtumgebung her.
* Es ist erforderlich, um zentrale Vertrauensmechanismen nach einer Kompromittierung wiederherzustellen.
* Es kontrolliert die Root-Ebene der Management- oder Identitätsinfrastruktur.

## Typische Komponenten

### Vertrauensanker

* Root-CA
* Offline-CA oder stark geschützte Intermediate-CA mit Root-Charakter
* HSM
* Root-KMS
* Root-Secrets oder Unseal-Mechanismen

### Identitäts- und Kontrollkern

* Root-Administrationsidentitäten
* Break-glass-Identitäten
* hochprivilegierte Recovery-Identitäten
* Root-Funktionen eines zentralen Verzeichnis- oder IAM-Systems, sofern diese Vertrauensursprung sind

### Recovery-Funktionen

* Recovery-Tresore für Schlüsselmaterial
* besonders geschützte Wiederanlaufdokumentation
* isolierte Kontrollpfade zur Wiedererlangung administrativer Souveränität

### Basale Kernabhängigkeiten

Nur falls architekturell erforderlich und nicht anders trennbar:

* vertrauenskritische DNS-Kerninstanzen
* sicherheitskritische Zeitquellen
* Root-Funktionen für Secrets-Entsperrung

## Beispiele

* Root-CA im Offline- oder HSM-gestützten Betrieb
* HSM zur Absicherung der Root-Schlüssel
* KMS-Root-Schlüssel zur Entsperrung kritischer Plattformgeheimnisse
* Break-glass-Konten für vollständige Umgebungswiederherstellung
* isolierter Recovery-Vault mit Notfallmaterialien

## Architektur- und Sicherheitsanforderungen

### Zugang und Administration

* Zugriffe auf Zone 0 **MÜSSEN** über dedizierte, gehärtete Administrationspfade erfolgen.
* Direkte Zugriffe aus Benutzer- oder Standard-Admin-Umgebungen **DÜRFEN NICHT** zugelassen werden.
* Für Zone-0-Administration **MÜSSEN** getrennte Identitäten verwendet werden.
* Privilegierte Endgeräte für Zone 0 **SOLLEN** als dedizierte PAWs oder funktional gleichwertige gehärtete Zugangsgeräte ausgeprägt sein.

### Härtung und Reduktion

* Zone 0 **MUSS** minimal gehalten werden.
* Komponenten in Zone 0 **MÜSSEN** auf den kleinsten erforderlichen Funktionsumfang reduziert werden.
* Direkte Software- und Integrationsabhängigkeiten **SOLLEN** minimiert werden.

### Nachvollziehbarkeit

* Jede administrative Aktion in Zone 0 **MUSS** revisionsfähig protokolliert werden, soweit rechtlich und technisch zulässig.
* Änderungen an Root-Schlüsseln, Zertifikatsketten, Root-Policies und Break-glass-Mechanismen **MÜSSEN** besonders überwacht werden.

### Recovery

* Für Zone 0 **MÜSSEN** dokumentierte Wiederherstellungsverfahren vorliegen.
* Recovery der Zone 0 **MUSS** regelmäßig praktisch getestet werden.
* Root-Materialien **SOLLEN** teilweise offline, immutable oder physisch getrennt abgesichert werden, sofern technisch sinnvoll.

## Abgrenzung

Folgende Systeme gehören typischerweise **nicht** nach Zone 0, auch wenn sie sehr kritisch sind:

* normales Git für Entwicklungs- oder Plattformarbeit
* Standard-CI/CD-Plattformen
* gewöhnliche zentrale Monitoring-Plattformen
* normale Anwendungsruntimes
* Standard-SSO für den Tagesbetrieb

Diese Systeme sind in der Regel operativ kritisch, aber nicht der eigentliche Vertrauensursprung.

---

# Zone 1 – Automation & Platform Core

## Zweck

Zone 1 enthält die minimalen Betriebs- und Plattformdienste, die erforderlich sind, um Änderungen kontrolliert auszurollen, Konfigurationen zu verwalten, Secrets zu verteilen, Artefakte bereitzustellen und Automatisierung überhaupt zu ermöglichen.

Zone 1 ist der operative Kern einer modernen Infrastruktur. Fällt Zone 1 aus oder wird kompromittiert, sind Änderungsfähigkeit, Wiederanlaufgeschwindigkeit, Integrität von Deployments und kontrollierter Plattformbetrieb stark gefährdet.

## Einordnungskriterien

Ein System MUSS typischerweise Zone 1 zugeordnet werden, wenn es eine oder mehrere der folgenden Funktionen ausübt:

* zentrale Steuerung von Build-, Deploy- oder IaC-Prozessen
* Verteilung von Secrets an Plattform- oder Anwendungsdienste
* zentrale Haltung von Infrastruktur- oder Plattformcode
* Verwaltung von Deploy-Artefakten oder Plattformzuständen
* Bereitstellung von Betriebsidentitäten für zentrale Plattformfunktionen
* Kontrolle über großflächige Änderungen an Zone 2 oder Zone 3

## Typische Komponenten

### Quellcode und Konfiguration

* zentrale Git-Plattformen für Infrastruktur- und Plattformcode
* Repositories für Richtlinien, Policies und Konfigurationsstände
* Plattform- oder Infrastrukturdefinitionen als Code

### Secrets und Zertifikatsbereitstellung

* Vault oder gleichwertige Secrets-Manager
* Zertifikats- und Token-Bereitstellung für Plattformdienste
* zentrale Secret-Broker für Automatisierung

### Artefakte und States

* Container-Registry
* Paket-Repository
* Build-Artefakt-Speicher
* Terraform-State-Backends
* zentrale Metadata- oder Config-Stores

### Automatisierungs- und Steuerungskomponenten

* CI/CD-Control-Planes
* GitOps-Controller
* Ansible-Automation-Controller
* IaC-Steuerinstanzen
* zentrale Deployment-Orchestrierung
* zentrale Image-Build-Steuerung

### Betriebsidentität

* SSO oder IAM-Komponenten für Plattform- und Betriebszugriff
* Föderationsdienste für zentrale Betriebssteuerung
* Service-Identitätsdienste mit Steuerwirkung auf Plattformänderungen

## Beispiele

* GitLab, GitHub Enterprise, Gitea oder Bitbucket für Plattform- und Infrastrukturcode
* Vault für Secret-Distribution
* Harbor, Nexus, Artifactory oder gleichwertige Artefakt-Repositories
* Terraform Enterprise oder selbst betriebene State-Backends
* Argo CD, Flux oder zentrale Deployment-Controller
* zentrale Build- oder Release-Steuerung

## Spezifische Einordnung der Ausgangsfrage

Die Infrastruktur, die minimale Dienste wie Git, Auth, Vault und Storage bereitstellt, um Automatisierung überhaupt zu ermöglichen, gehört **grundsätzlich in Zone 1**.

Dabei gilt die wichtige Unterteilung:

* Git selbst: **Zone 1**
* Vault selbst: **Zone 1**
* SSO/IAM für normalen Betriebszugriff: **Zone 1**
* Plattform-Storage für State, Artefakte und Konfigurationsstände: **Zone 1**

Die tieferen Vertrauensanker dieser Dienste gehören jedoch typischerweise in **Zone 0**, zum Beispiel:

* HSM
* Root-KMS
* Root-CA
* Break-glass-Mechanismen
* Unseal-Root oder äquivalente Root-Geheimnisse

## Warum Zone 1 nicht Zone 0 ist

Zone 1 ist betriebsentscheidend, aber in vielen Architekturen nicht der eigentliche Vertrauensursprung. Typische Abhängigkeiten sind:

* Git vertraut auf IAM, Signatur- und Berechtigungsmodelle.
* Vault vertraut auf Unseal-Mechanismen, HSM oder KMS.
* CI/CD vertraut auf signierte Artefakte, Identitäten und Policies.
* Artefakt-Storage vertraut auf Root-Admin-, Schlüssel- und Recovery-Kontrollen.

Damit ist Zone 1 der **operative Hebel**, während Zone 0 die **Vertrauens- und Recovery-Basis** darstellt.

## Architektur- und Sicherheitsanforderungen

### Integrität der Lieferkette

* Änderungen in Zone 1 **MÜSSEN** nachvollziehbar, prüfbar und autorisiert sein.
* Build-, Release- und Deployment-Pfade **MÜSSEN** manipulationsresistent ausgelegt werden.
* Signierte Artefakte **SOLLEN** verwendet werden.
* Signierte Commits, signierte Tags und attestierte Build-Prozesse **SOLLEN** verwendet werden, sofern technisch und organisatorisch umsetzbar.

### Trennung von Rollen

* Entwickler-, Betreiber- und Security-Rollen **MÜSSEN** getrennt werden.
* Direkter administrativer Vollzugriff auf Zone-1-Kernkomponenten **DARF NICHT** breit vergeben werden.
* Runner, Agenten und Controller **MÜSSEN** auf das Mindestmaß an Rechten begrenzt werden.

### Secrets-Sicherheit

* Secrets **DÜRFEN NICHT** dauerhaft unkontrolliert auf Build- oder Worker-Systemen verbleiben.
* Secret-Ausgabe **MUSS** an Identität, Kontext und Laufzeitbindung gekoppelt sein.
* Root- oder Unseal-Funktionen **MÜSSEN** von regulären Betriebsfunktionen getrennt sein.

### Administrationspfade

* Administration von Zone 1 **MUSS** über dedizierte und gehärtete Managementpfade erfolgen.
* Direkte Administration aus Standard-Benutzerumgebungen **SOLL** unterbunden werden.
* Besonders privilegierte Rollen **SOLLEN** Just-in-Time oder zeitlich begrenzt vergeben werden.

### Verfügbarkeit und Recovery

* Zone 1 **MUSS** hochverfügbar oder mit definiertem Wiederanlaufziel ausgelegt werden.
* Für Zone 1 **MÜSSEN** dokumentierte Restore- und Rebuild-Pfade vorliegen.
* Es **SOLL** möglich sein, Zone 1 aus vertrauenswürdigen Quellen neu aufzubauen, auch wenn äußere Zonen kompromittiert wurden.

---

# Zone 2 – Shared Platform Services

## Zweck

Zone 2 enthält gemeinsam genutzte Plattformdienste, die vielen Anwendungen, Teams oder Fachbereichen als Laufzeit-, Daten-, Messaging-, Observability- oder Integrationsbasis dienen, ohne selbst der operative Kern der gesamten Änderungs- und Vertrauenskette zu sein.

## Einordnungskriterien

Ein System SOLL Zone 2 zugeordnet werden, wenn es:

* von mehreren Anwendungen oder Teams gemeinsam genutzt wird,
* primär Laufzeit- oder Plattformfunktion bereitstellt,
* nicht selbst zentraler Änderungshebel für die Gesamtumgebung ist,
* aber erhebliche seitliche Auswirkungen bei Kompromittierung oder Ausfall haben kann.

## Typische Komponenten

* gemeinsame Kubernetes- oder Container-Laufzeitplattformen
* zentrale Datenbankplattformen
* Messaging-Systeme
* API-Gateways
* Service-Mesh-Kontrollkomponenten, abhängig vom Wirkungsgrad
* Monitoring- und Logging-Plattformen
* geteilte Datei-, Block- oder Objekt-Storage-Dienste
* zentrale Backup-Plattformen, sofern sie nicht Zone-0- oder Zone-1-Root-Funktionen tragen

## Beispiele

* PostgreSQL- oder MySQL-Cluster für mehrere Fachanwendungen
* Kafka oder RabbitMQ
* Prometheus, Grafana, Loki, Elasticsearch, Splunk
* Shared Kubernetes Cluster für Business Workloads
* zentrale Storage-Plattform für gemeinsame Workloads

## Abgrenzung zu Zone 1

Die Abgrenzung MUSS sauber erfolgen:

* Zone 1 steuert Änderungen, Policies und Deployment.
* Zone 2 stellt gemeinsam genutzte Betriebs- und Laufzeitdienste bereit.

Beispiele:

* GitOps-Controller: Zone 1
* gemeinsamer Kubernetes-Runtime-Cluster: Zone 2
* zentrale Artefakt-Registry mit Deploy-Steuerwirkung: eher Zone 1
* Shared Storage für Laufzeitdaten: meist Zone 2

## Architektur- und Sicherheitsanforderungen

* Mandantentrennung **MUSS** dort umgesetzt werden, wo mehrere Anwendungen, Teams oder Schutzbedarfe zusammenlaufen.
* Management-Schnittstellen **MÜSSEN** stärker geschützt sein als Dataplane-Zugriffe.
* Seitlicher Verkehr zwischen Workloads **SOLL** begrenzt und policybasiert kontrolliert werden.
* Zugriffe aus Zone 3 auf Zone 2 **MÜSSEN** fachlich oder technisch begründet sein.
* Administrative Rückkanäle von Zone 2 nach Zone 1 **DÜRFEN NICHT** pauschal offen sein.

---

# Zone 3 – Business Applications

## Zweck

Zone 3 enthält geschäftliche Fachanwendungen und Business Services. Hier liegen die Systeme, die Geschäftsprozesse, Fachlogik und produktive Geschäftsdatennutzung unmittelbar bereitstellen.

## Einordnungskriterien

Ein System SOLL Zone 3 zugeordnet werden, wenn es primär:

* Geschäftslogik umsetzt,
* einem Fachprozess dient,
* Daten für einen oder mehrere definierte Geschäftszwecke verarbeitet,
* auf Dienste aus Zone 1 und Zone 2 aufsetzt,
* aber nicht selbst Vertrauens- oder Plattformkern der Gesamtumgebung ist.

## Typische Komponenten

* ERP-, CRM-, HR- und Finance-Anwendungen
* Produktiv-Webanwendungen
* interne Portale
* fachliche APIs
* domänenspezifische Microservices
* Batch- und Reporting-Anwendungen
* Datenverarbeitung für Geschäftsprozesse

## Beispiele

* Kundenportal
* Bestellplattform
* internes HR-System
* SAP-nahe Geschäftsservices
* Abrechnungs- oder Reporting-Dienste

## Architektur- und Sicherheitsanforderungen

* Anwendungen in Zone 3 **MÜSSEN** mit minimalen Rechten gegenüber Zone 2 und Zone 1 betrieben werden.
* Statische, lang lebende Secrets **SOLLEN** vermieden werden.
* Workload-Identitäten **SOLLEN** bevorzugt eingesetzt werden.
* Direkte administrative Rückkanäle in Zone 1 oder Zone 0 **DÜRFEN NICHT** bestehen.
* Ost-West-Verkehr zwischen Fachanwendungen **SOLL** nur auf definierter Basis erlaubt werden.
* Exponierte Anwendungen **MÜSSEN** zusätzliche Schutzmaßnahmen wie Rate Limiting, WAF, Eingangsvalidierung oder vergleichbare Kontrollen erhalten, soweit das Bedrohungsmodell dies erfordert.

---

# Zone 4 – User, Endpoint & External Edge

## Zweck

Zone 4 ist die äußerste Zone. Sie umfasst Benutzerendgeräte, benutzernahe Zugänge, mobile Arbeitsumgebungen, Partner- und Fernzugriffe sowie internetnahe Übergangsbereiche.

Diese Zone ist typischerweise am stärksten exponiert und besitzt das höchste Risiko für Initialzugriffe durch Phishing, Malware, Credential Theft, Browser-Angriffe oder kompromittierte Partnerpfade.

## Einordnungskriterien

Ein System SOLL Zone 4 zugeordnet werden, wenn es:

* primär benutzernah ist,
* extern exponiert ist,
* als Zugangs- oder Übergangspunkt dient,
* oder ein erhöhtes Risiko durch unsichere Umgebungen und Fremdeinwirkung trägt.

## Typische Komponenten

* Notebooks und Desktops
* VDI-Arbeitsplätze
* mobile Endgeräte
* Remote-Access-Endpunkte
* Partnerzugänge
* VPN- oder ZTNA-Einstiege
* WAF-, Reverse-Proxy- oder DMZ-nahe Zugriffsschichten
* Browser- und E-Mail-nahe Arbeitsumgebungen

## Beispiele

* Mitarbeiter-Laptop
* Browserzugriff auf Unternehmensanwendungen
* VPN-Gateway
* ZTNA-Zugangslösung
* öffentliche Web-Eingangsschicht

## Architektur- und Sicherheitsanforderungen

* Zone 4 **DARF NICHT** als vertrauenswürdige Ausgangsbasis für privilegierte Zugriffe in innere Zonen behandelt werden.
* Direkte Administrationszugriffe aus Standard-Endgeräten in Zone 0 oder Zone 1 **DÜRFEN NICHT** allgemein zulässig sein.
* Gerätezustand, Identität und Sitzungskontext **SOLLEN** vor sensitiven Zugriffen geprüft werden.
* Endpoint Detection, Schutz vor Credential Theft und Schutz vor Browser-/Mail-basierten Angriffen **SOLLEN** implementiert werden.
* Externe oder Partnerzugriffe **MÜSSEN** explizit segmentiert, nachvollziehbar und minimiert sein.

---

# Sonderzonen und ergänzende Segmente

Abhängig von Risiko, Regulierung und Architektur KÖNNEN zusätzliche Zonen eingeführt werden.

## Backup- und Cyber-Recovery-Zone

Sinnvoll, wenn Wiederherstellungsfähigkeit explizit gegen Ransomware, Identitätskompromittierung oder großflächige Plattformmanipulation abgesichert werden soll.

Typische Inhalte:

* immutable Backups
* isolierte Backup-Administration
* Clean-Room- oder Recovery-Umgebung
* Wiederanlauf-Tooling

Diese Zone **SOLL** getrennt ausgeprägt werden, wenn das Schadensszenario „breite Kompromittierung von Zone 1 bis Zone 4“ relevant ist.

## Security-Operations-Zone

Für SIEM, Forensik, EDR-Backend, Threat-Detection und ähnliche Systeme mit weitreichender Sicht oder Interventionsfähigkeit.

Diese Zone **SOLL** erwogen werden, wenn Security-Werkzeuge weitreichende Zugriffsrechte über mehrere Zonen hinweg erhalten.

## OT-/ICS-Zone

Für Produktionsnetze, Fertigungssteuerung, SCADA oder andere betriebsnahe Umgebungen mit eigenen Sicherheits- und Verfügbarkeitsanforderungen.

OT-/ICS-Bereiche **MÜSSEN** separat betrachtet werden und DÜRFEN nicht pauschal in dieses IT-Zonenmodell eingegliedert werden.

## Partner- und Integrationszone

Für B2B-Kopplung, Datenaustausch, APIs mit Dritten oder technische Lieferantenanbindungen.

## Hochsicherheits-Datenzone

Für besonders schützenswerte Informationen, etwa aus Forschung, Verteidigung, Gesundheitswesen oder kritischer Infrastruktur.

---

# Entscheidungsmethodik zur Zonenzuordnung

Für Architektur- und Security-Reviews SOLL die Zonenzuordnung anhand einer standardisierten Prüfung erfolgen.

## Prüffrage 1: Ist das System Vertrauensursprung?

Wenn ja, ist Zone 0 wahrscheinlich.

Typische Indikatoren:

* Root-Schlüssel
* Root-Zertifikate
* Unseal-Geheimnisse
* Break-glass-Kontrolle
* Root-IAM-Verwaltung

## Prüffrage 2: Kann das System großflächige Änderungen auslösen?

Wenn ja, ist Zone 1 wahrscheinlich.

Typische Indikatoren:

* Deployment-Steuerung
* Policy-Verteilung
* Secret-Distribution
* IaC-Steuerung
* zentrale Build-/Release-Funktionen

## Prüffrage 3: Stellt das System gemeinsame Laufzeitdienste bereit?

Wenn ja, ist Zone 2 wahrscheinlich.

Typische Indikatoren:

* Shared Runtime
* Datenbank- oder Messaging-Plattform
* gemeinsame Observability
* Shared Storage

## Prüffrage 4: Liefert das System primär Fachlogik?

Wenn ja, ist Zone 3 wahrscheinlich.

## Prüffrage 5: Ist das System benutzernah oder extern exponiert?

Wenn ja, ist Zone 4 wahrscheinlich.

## Zusätzliche Review-Fragen

Für jedes System SOLLEN mindestens diese Fragen beantwortet werden:

* Welche anderen Systeme kann es direkt oder indirekt verändern?
* Welche Secrets oder Schlüssel verarbeitet es?
* Welche Zonen hängen funktional von ihm ab?
* Welche Folgen hätte Kompromittierung der Integrität?
* Welche Folgen hätte Ausfall der Verfügbarkeit?
* Kann es für Seitwärtsbewegung oder Privilege Escalation missbraucht werden?
* Ist es für Recovery zwingend erforderlich?

---

# Typische Zuordnung häufiger Komponenten

## Identität und Zugriff

* Root-Identitäten: **Zone 0**
* Root-Funktionen zentraler IAM-/Verzeichnisdienste: **Zone 0**
* SSO/IdP für normalen Betriebszugriff: **Zone 1**
* anwendungsnahe Access-Komponenten: **Zone 2** oder **Zone 3**
* Benutzer-Login- und Edge-Zugänge: **Zone 4**

## PKI

* Root-CA: **Zone 0**
* Intermediate-CA mit breiter Betriebsfunktion: **Zone 1**, sofern nicht Root-charakteristisch
* Zertifikatsnutzer: entsprechend ihrer Betriebszone

## Secrets und Schlüssel

* Root-Schlüssel, Unseal-Mechanismen, HSM: **Zone 0**
* Vault-Plattform: **Zone 1**
* Anwendungsspezifische Secret-Nutzung: **Zone 3**

## Git und Software-Lieferkette

* Git für Infrastruktur- und Plattformcode: **Zone 1**
* Build- und Release-Steuerung: **Zone 1**
* hochprivilegierte Runner oder Controller: **Zone 1**
* Business-Code-Repositories: je nach Steuerwirkung **Zone 1** oder randnah **Zone 2**

## Container und Kubernetes

* GitOps- oder Cluster-Steuerung: **Zone 1**
* gemeinsamer Runtime-Cluster: **Zone 2**
* Business-Workloads im Cluster: **Zone 3**
* öffentlicher Ingress: **Zone 4-Übergang**

## Storage

* Root-Admin, Schlüssel-Root oder Entsperr-Root: **Zone 0**
* Artefakt-, State- und Plattform-Storage: **Zone 1**
* Shared Datenplattform: **Zone 2**
* Anwendungsdaten: **Zone 3**
* benutzernahe Dateibestände und Endgeräte-Caches: **Zone 4**

## Monitoring und Logging

* hochprivilegierte Security-/Forensik-Sicht: Sonderzone oder gehärtete Zone 2
* zentrales Monitoring: meist **Zone 2**
* anwendungsnahe Logs: **Zone 3**

## Backup

* Root-Recovery-Funktion: **Zone 0** oder Sonderzone
* zentrale Backup-Plattform: **Zone 1** oder **Zone 2**, abhängig von Kontrollwirkung
* anwendungsnahe Restore-Funktionen: **Zone 3**

---

# Kommunikationsmodell und Zonenübergänge

## Grundsatz

Kommunikation zwischen Zonen **MUSS** explizit erlaubt, begründet und dokumentiert sein. Das Default-Modell SOLL „deny by default“ sein.

## Zone 4 nach innen

* Zone 4 → Zone 3: zulässig über veröffentlichte, abgesicherte Dienste
* Zone 4 → Zone 2: nur in begründeten Sonderfällen
* Zone 4 → Zone 1: nur über definierte Management- oder Entwicklungszugänge
* Zone 4 → Zone 0: grundsätzlich unzulässig, außer dedizierte und stark kontrollierte Sonderpfade

## Zone 3 nach innen

* Zone 3 → Zone 2: zulässig bei fachlich erforderlicher Plattformnutzung
* Zone 3 → Zone 1: nur minimale technische Abhängigkeiten, keine allgemeinen Admin-Rückkanäle
* Zone 3 → Zone 0: grundsätzlich unzulässig

## Zone 2 nach innen

* Zone 2 → Zone 1: nur explizit freigegebene technische oder administrative Pfade
* Zone 2 → Zone 0: nur in seltenen, klar begründeten Fällen

## Zone 1 nach außen

* Zone 1 → Zone 2 und Zone 3: zulässig für Deployment, Konfiguration, Secret-Ausgabe und kontrollierte Betriebssteuerung
* Zone 1 → Zone 4: nur soweit für Benutzer- oder Entwicklerinteraktion fachlich erforderlich

## Zusätzliche Anforderungen

* Pauschal offene Managementnetze **DÜRFEN NICHT** verwendet werden.
* Kommunikationsbeziehungen **MÜSSEN** pro Protokoll, Richtung, Authentifizierung und Zweck dokumentiert werden.
* Wo möglich, **SOLLEN** Pull-Modelle oder kurzlebige, identitätsgebundene Zugriffe bevorzugt werden.

---

# Administrationsmodell

Die Wirksamkeit des Zonenmodells hängt maßgeblich von der Trennung administrativer Pfade ab.

## Anforderungen

* Für innere Zonen **MÜSSEN** getrennte Administrationspfade existieren.
* Normale Benutzerendgeräte **DÜRFEN NICHT** als primäre Administrationsbasis für Zone 0 oder Zone 1 dienen.
* Für privilegierte Tätigkeiten **MÜSSEN** getrennte Identitäten verwendet werden.
* Hochprivilegierte Rollen **SOLLEN** just-in-time, zeitlich begrenzt und protokolliert vergeben werden.
* Administrative Sprungpunkte, Bastions oder Remote-Management-Lösungen **MÜSSEN** gehärtet, überwacht und minimal berechtigt sein.
* Auf privilegierten Admin-Systemen **SOLLEN** E-Mail, allgemeines Browsing und sonstige Hochrisiko-Nutzung unterbunden werden.

## Zielbild

* Standardarbeit: Zone 4
* privilegierte Administration Zone 2: separierte Operator-Pfade
* privilegierte Administration Zone 1: gehärtete Kern-Admin-Pfade
* privilegierte Administration Zone 0: besonders restriktive Root-/Recovery-Pfade

---

# Identitäten, Rollen und Berechtigungen

Ein Zonenmodell ohne saubere Berechtigungsarchitektur ist formal vorhanden, aber praktisch schwach.

## Anforderungen

* Rollen **MÜSSEN** zonenspezifisch und zweckgebunden definiert werden.
* Hochprivilegierte Identitäten **DÜRFEN NICHT** in äußeren Zonen wiederverwendet werden.
* Menschliche und technische Identitäten **MÜSSEN** getrennt werden.
* Technische Credentials **SOLLEN** kurzlebig, rotierbar und kontextgebunden sein.
* Berechtigungen **MÜSSEN** nach Least Privilege und soweit möglich nach Just Enough Administration vergeben werden.
* Jede privilegierte Berechtigung **MUSS** einem Verantwortlichen und einem fachlichen Zweck zugeordnet sein.

## Beispielhafte Trennung

* Entwickler: Commit- und Review-Rechte, aber keine Root-Administration für Vault oder KMS
* Plattformoperator: Deploy- und Plattformrechte, aber keine Root-Schlüsselverwaltung
* Recovery-Administrator: Zugriff auf Notfallmechanismen, aber keine Nutzung im Regelbetrieb

---

# Logging, Monitoring und Nachvollziehbarkeit

## Anforderungen

* Sicherheitsrelevante Ereignisse aus Zone 0 und Zone 1 **MÜSSEN** zentral, manipulationsarm und korrelierbar erfasst werden.
* Administrative Anmeldungen, Rollenwechsel, Secret-Zugriffe, Policy-Änderungen, Deployment-Ausführungen und Konfigurationsänderungen **MÜSSEN** protokolliert werden.
* Zeitsynchronisation **MUSS** für revisionsfähige Auswertung sichergestellt sein.
* Zugriff auf privilegierte Logs **MUSS** selbst wieder restriktiv kontrolliert sein.
* Für forensisch relevante Daten **SOLLEN** Aufbewahrung, Integritätsschutz und Zugriffstrennung definiert werden.

## Besonders review-relevante Ereignisse

* Änderung an Zertifikatsketten
* Änderung an Root- oder Admin-Rollen
* Ausgabe hochkritischer Secrets
* Deaktivierung von Sicherheitskontrollen
* Pipeline-Ausführungen mit Produktionswirkung
* Manipulation oder Löschung von Audit-Trails

---

# Backup, Restore und Recovery

## Grundsatz

Das Zonenmodell **MUSS** in ein Wiederanlaufmodell übersetzt werden. Ohne dies bleibt die Segmentierung im Incident unvollständig.

## Anforderungen

* Es **MUSS** dokumentiert sein, in welcher Reihenfolge Zonen wiederhergestellt werden.
* Zone 0 **MUSS** vor oder gemeinsam mit Zone 1 wiederherstellbar sein.
* Zone 1 **MUSS** die kontrollierte Wiederherstellung von Zone 2 und Zone 3 ermöglichen.
* Backups **MÜSSEN** gegen Manipulation, Verschlüsselung durch Angreifer und unautorisierte Löschung geschützt werden.
* Wiederherstellung **MUSS** regelmäßig praktisch getestet werden.
* Für kritische Kernkomponenten **SOLLEN** Clean-Room-, Immutable- oder isolierte Recovery-Verfahren vorhanden sein.

## Typische Wiederanlaufreihenfolge

1. Trust & Recovery Core
2. Automation & Platform Core
3. Shared Platform Services
4. Business Applications
5. User-, Endpoint- und Edge-Zugänge

Abweichungen KÖNNEN zulässig sein, MÜSSEN aber architektonisch begründet werden.

---

# Review-Kriterien für Architektur- und Security-Freigaben

Ein Architektur- oder Security-Review SOLL mindestens diese Prüfpunkte enthalten:

## Zonenzuordnung

* Ist die Zuordnung anhand Funktion, Vertrauenswirkung und Änderungsmacht nachvollziehbar?
* Ist Zone 0 klein und klar abgegrenzt?
* Sind operative Kernsysteme korrekt in Zone 1 und nicht fälschlich zu weit außen eingeordnet?

## Kommunikationsmodell

* Gibt es dokumentierte, minimierte Kommunikationspfade?
* Ist Default-Deny technisch umgesetzt?
* Gibt es unkontrollierte Rückkanäle von außen nach innen?

## Administration

* Sind privilegierte Pfade getrennt?
* Gibt es PAWs oder äquivalente Härtung für innere Zonen?
* Sind Rollen und Identitäten sauber getrennt?

## Lieferkette und Automatisierung

* Sind Build-, Release- und Deployment-Pfade manipulationsresistent?
* Sind Secrets kontrolliert verteilt?
* Haben Runner, Agenten oder Controller zu breite Rechte?

## Recovery

* Ist der Wiederanlauf von Zone 0 und Zone 1 praktisch möglich?
* Gibt es getestete Wiederherstellungspfade?
* Sind Backups und Recovery-Assets gegen Plattformkompromittierung geschützt?

## Nachvollziehbarkeit

* Werden privilegierte und sicherheitsrelevante Aktionen protokolliert?
* Ist die Log-Integrität ausreichend geschützt?
* Können Veränderungen an Vertrauensankern und Policies sicher nachvollzogen werden?

---

# Häufige Fehlmuster

## 1. Zone 0 wird zu groß

Wenn alles Kritische in Zone 0 landet, verliert die Zone ihre Funktion als kleiner Vertrauens- und Recovery-Kern. Zone 0 **MUSS** klein bleiben.

## 2. Plattformsteuerung wird unterschätzt

Git, Vault, CI/CD, Registry und State-Backends werden oft als „normale Infrastruktur“ behandelt. Das ist architektonisch falsch. Diese Systeme **MÜSSEN** als Kern der Änderungs- und Steuerkette betrachtet werden.

## 3. Segmentierung wird nur netztechnisch gedacht

Ohne Identitätsmodell, Secrets-Modell, Admin-Trennung und Logging bleibt die Zonierung unzureichend.

## 4. Standard-Endgeräte administrieren innere Zonen

Das unterläuft das gesamte Schutzmodell und **DARF NICHT** akzeptiert werden.

## 5. Recovery wird nicht mitmodelliert

Ein Modell, das im Incident nicht wiederanlaufbar ist, ist unvollständig.

## 6. Zu grobe oder zu feine Segmentierung

Zu grob verhindert wirksame Schutzdifferenzierung; zu fein erzeugt unbeherrschbare Komplexität. Das hier beschriebene Fünf-Zonen-Modell ist für viele Unternehmen ein belastbarer Ausgangspunkt.

---

# Beispielhafte Referenzzuordnung für ein typisches Unternehmen

## Zone 0

* Root-CA
* HSM
* Root-KMS
* Break-glass-Identitäten
* isolierter Recovery-Tresor

## Zone 1

* Git-Plattform für Infrastruktur- und Plattformcode
* Vault
* Artefakt-Registry
* Terraform-State-Backend
* GitOps- oder Deployment-Controller
* SSO für Plattform- und Betriebszugriff

## Zone 2

* Shared Kubernetes
* Datenbankplattform
* Messaging-Plattform
* Monitoring- und Logging-Plattform
* Shared Storage

## Zone 3

* Kundenportal
* Bestell- und Abrechnungsservices
* ERP-nahe Anwendungen
* interne Fachservices

## Zone 4

* Mitarbeiter-Laptops
* VDI
* VPN-/ZTNA-Zugänge
* Reverse Proxies und WAF
* externe Zugangsflächen

---

# Konkrete Schlussfolgerung zur Ausgangsfrage

Die Infrastruktur, die benötigt wird, um minimale Dienste wie Git, Auth, Vault und Storage bereitzustellen und damit Automatisierung überhaupt zu ermöglichen, ist **architektonisch und sicherheitlich als Zone 1 – Automation & Platform Core – einzuordnen**.

Dabei ist zwingend zu unterscheiden zwischen:

## Operativer Kern – Zone 1

Dazu gehören typischerweise:

* Git
* Vault
* SSO/IAM für den regulären Betriebszugriff
* Artefakt-Registry
* State-Storage
* Plattform-Konfigurationsspeicher
* Deployment- und IaC-Steuerung

## Vertrauens- und Recovery-Ursprung – Zone 0

Dazu gehören typischerweise:

* HSM
* Root-KMS
* Root-CA
* Break-glass-Mechanismen
* Unseal-Root oder gleichwertige Root-Secrets
* Root-Funktionen der Identitäts- und Wiederherstellungskette

Die belastbare Antwort lautet daher:

* **Die minimale Automatisierungsbasis gehört in Zone 1.**
* **Deren Vertrauensursprung und Recovery-Kern gehören in Zone 0.**

Genau diese Trennung macht das Modell review-tauglich: Sie unterscheidet sauber zwischen operativer Steuerungsfähigkeit und eigentlichem Root of Trust.

---

# Umsetzungsempfehlung

## Schritt 1 – Inventarisierung

Alle zentralen Systeme, Identitäten, Vertrauensanker, Steuerinstanzen und Recovery-Abhängigkeiten **MÜSSEN** vollständig inventarisiert werden.

## Schritt 2 – Abhängigkeitsanalyse

Für jedes System **MUSS** bewertet werden:

* wer es administriert,
* welche Systeme es ändern kann,
* welche Secrets oder Schlüssel es benötigt,
* welche Ausfall- und Kompromittierungsfolgen bestehen,
* welche Zonen davon abhängen.

## Schritt 3 – Zonenzuordnung

Die Zuordnung **MUSS** funktions- und risikobasiert erfolgen, nicht nach Teamstruktur oder Hosting-Ort.

## Schritt 4 – Kommunikations- und Admin-Pfade

Kommunikations- und Administrationspfade **MÜSSEN** explizit dokumentiert, minimiert und technisch erzwungen werden.

## Schritt 5 – Nachweis der Wirksamkeit

Die Wirksamkeit des Modells **SOLL** durch Architektur-Review, technische Tests, Rechte-Reviews und Recovery-Übungen nachgewiesen werden.

## Schritt 6 – Governance

Abweichungen von MUSS- und SOLL-Vorgaben **MÜSSEN** dokumentiert, risikobewertet und genehmigt werden.

---

# Kurzfassung

* Zone 0 = Vertrauens- und Recovery-Kern
* Zone 1 = Automatisierungs- und Plattform-Kern
* Zone 2 = gemeinsam genutzte Plattformdienste
* Zone 3 = Fachanwendungen
* Zone 4 = Benutzer, Endgeräte, Edge und externe Zugänge

Für Git, Auth, Vault und Storage als minimale Betriebsbasis gilt:

* **die eigentlichen Betriebsdienste gehören in Zone 1**
* **deren Root-of-Trust-Komponenten gehören in Zone 0**

Damit ist das Modell sowohl architekturseitig konsistent als auch für Security Reviews, Governance und Auditierung belastbar.
