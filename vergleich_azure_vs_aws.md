# Vergleich Azure vs. AWS für AZ-900

## 1. Cloud-Konzepte  
Grundlegendes Verständnis von Cloud-Computing, Bereitstellungsmodellen und Skalierung.

| **Azure Begriff / Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|----------------------------|------------------|--------------------|
| **Öffentliche, private und hybride Cloud** | Verschiedene Bereitstellungsmodelle: öffentliche Cloud (für alle zugänglich), private Cloud (internes Rechenzentrum), hybride Cloud (Kombination aus beiden). | AWS bietet hauptsächlich öffentliche Cloud-Dienste an, mit hybriden Lösungen über **AWS Outposts**. |
| **IaaS, PaaS, SaaS** | Verschiedene Service-Modelle: Infrastruktur (IaaS), Plattform (PaaS) und Software (SaaS) als Dienstleistung. | Entsprechende Modelle in **AWS** verfügbar. |
| **Skalierbarkeit und Elastizität** | Fähigkeit, Ressourcen dynamisch anzupassen, um auf wechselnde Anforderungen zu reagieren. | **AWS Auto Scaling** |
| **Hochverfügbarkeit und Fehlertoleranz** | Sicherstellung der kontinuierlichen Verfügbarkeit und Widerstandsfähigkeit gegenüber Ausfällen. | **AWS Multi-AZ Deployments, AWS Auto Scaling** |
| **Disaster Recovery und Geschäftskontinuität** | Strategien zur Wiederherstellung nach Ausfällen und zur Aufrechterhaltung des Geschäftsbetriebs. | **AWS Elastic Disaster Recovery, AWS Backup** |

---

## 2. Azure-Architektur und -Dienste  
Kernkomponenten der Cloud-Infrastruktur.

### Compute-Dienste (Rechenleistung)

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Machines (VMs)** | Bereitstellung von virtuellen Maschinen in der Cloud. | **Amazon EC2** |
| **Azure App Service** | Verwalteter Dienst für das Hosten von Webanwendungen und APIs. | **AWS Elastic Beanstalk** |
| **Azure Functions** | Serverloser Dienst zur Ausführung von Code als Reaktion auf Ereignisse. | **AWS Lambda** |
| **Azure Kubernetes Service (AKS)** | Verwalteter Kubernetes-Dienst für die Orchestrierung von Containern. | **Amazon EKS** |
| **Azure Container Instances (ACI)** | Ausführung von Containern ohne Orchestrierungsdienst. | **AWS Fargate** |
| **Azure Virtual Desktop** | Virtuelle Desktop-Infrastruktur in der Azure-Cloud. | **Amazon WorkSpaces** |

### Netzwerkdienste

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Network (VNet)** | Ermöglicht die Einrichtung privater Netzwerke in Azure. | **Amazon VPC** |
| **Azure Load Balancer** | Verteilt eingehenden Datenverkehr auf mehrere Ressourcen. | **Elastic Load Balancer (ELB)** |
| **Azure Application Gateway** | Lastenausgleich mit Web Application Firewall (WAF) für HTTP/HTTPS-Datenverkehr. | **AWS Application Load Balancer** |
| **Azure Traffic Manager** | DNS-basierter Lastenausgleich für die Verteilung des Datenverkehrs über mehrere Regionen. | **Amazon Route 53** |
| **Azure VPN Gateway** | Sichere Verbindung zwischen lokalen Netzwerken und Azure über VPN. | **AWS VPN Gateway** |
| **Azure ExpressRoute** | Dedizierte, private Verbindung zwischen lokalen Umgebungen und Azure. | **AWS Direct Connect** |

### Speicher- und Datenbankdienste

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Blob Storage** | Objektspeicher für unstrukturierte Daten. | **Amazon S3** |
| **Azure File Storage** | Vollständig verwalteter Dateifreigabedienst. | **Amazon EFS** |
| **Azure Table Storage** | NoSQL-Speicher für strukturierte Daten. | **Amazon DynamoDB** |
| **Azure Queue Storage** | Dienst für die Speicherung von Nachrichten in Warteschlangen. | **Amazon SQS** |
| **Azure Disk Storage** | Hochleistungsfähiger, langlebiger Blockspeicher für VMs. | **Amazon EBS** |
| **Azure Cosmos DB** | Globale, verteilte NoSQL-Datenbank mit mehreren APIs. | **Amazon DynamoDB** |
| **Azure SQL Database** | Verwalteter relationaler Datenbankdienst basierend auf SQL Server. | **Amazon RDS für SQL Server** |
| **Azure Database for MySQL/PostgreSQL/MariaDB** | Verwaltete Datenbankdienste für MySQL, PostgreSQL und MariaDB. | **Amazon RDS für MySQL/PostgreSQL/MariaDB** |

---

## 3. Identitäts- und Zugriffsmanagement  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Active Directory (Azure AD)** | Cloud-basierter Identitäts- und Zugriffsverwaltungsdienst. | **AWS IAM** |
| **Azure AD Domain Services** | Domänendienste wie Domänenbeitritt, Gruppenrichtlinien und LDAP. | **AWS Directory Service** |
| **Azure Multi-Factor Authentication (MFA)** | Zusätzliche Sicherheitsebene durch MFA. | **AWS MFA** |

---

## 4. Sicherheitsdienste  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Security Center** | Einheitliche Sicherheitsverwaltung und Bedrohungsschutz. | **AWS Security Hub** |
| **Azure Sentinel** | Cloud-nativer SIEM-Dienst für intelligente Sicherheitsanalysen. | **Amazon GuardDuty** |
| **Azure Key Vault** | Sicherer Speicher für Schlüssel, Geheimnisse und Zertifikate. | **AWS KMS, AWS Secrets Manager** |
| **Azure DDoS Protection** | Schutz vor DDoS-Angriffen auf Azure-Ressourcen. | **AWS Shield** |

---

## 5. Überwachungs- und Verwaltungsdienste  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Monitor** | Vollständige Überwachungslösung für die Erfassung, Analyse und Reaktion auf Metriken und Logs. | **Amazon CloudWatch** |
| **Azure Log Analytics** | Tool zur Abfrage und Analyse von Log-Daten. | **Amazon CloudWatch Logs Insights** |
