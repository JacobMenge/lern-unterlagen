# Vergleich Azure vs. AWS für AZ-900  

## 1. Einführung  

Microsoft Azure und Amazon Web Services (AWS) sind die beiden führenden Cloud-Plattformen weltweit. Während AWS als Pionier des Cloud-Computings gilt, hat sich Azure als starker Konkurrent etabliert, insbesondere durch seine enge Integration mit Microsoft-Produkten.  

# Vergleich Azure vs. AWS für AZ-900

## 1. Cloud-Konzepte

### Grundlegendes Verständnis von Cloud-Computing, Bereitstellungsmodellen und Skalierung.

Azure und AWS bieten ähnliche Cloud-Bereitstellungsmodelle (Public, Private, Hybrid) sowie Service-Modelle wie IaaS, PaaS und SaaS. Die Fähigkeit zur automatischen Skalierung und Fehlertoleranz ist entscheidend für die Verfügbarkeit von Anwendungen.

| **Azure Begriff / Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|----------------------------|------------------|--------------------|
| **Öffentliche, private und hybride Cloud** | Verschiedene Bereitstellungsmodelle: öffentliche Cloud (für alle zugänglich), private Cloud (internes Rechenzentrum), hybride Cloud (Kombination aus beiden). | AWS ist primär eine **Public Cloud**, Hybrid-Cloud-Lösungen über **AWS Outposts**. |
| **IaaS, PaaS, SaaS** | Service-Modelle: Infrastruktur (IaaS), Plattform (PaaS) und Software (SaaS). | AWS bietet dieselben Modelle an. |
| **Skalierbarkeit & Elastizität** | Fähigkeit, Ressourcen dynamisch an Lasten anzupassen. | **AWS Auto Scaling** |
| **Hochverfügbarkeit & Fehlertoleranz** | Sicherstellung der kontinuierlichen Verfügbarkeit durch Redundanz. | **AWS Multi-AZ Deployments, AWS Auto Scaling** |
| **Disaster Recovery & Geschäftskontinuität** | Strategien zur Wiederherstellung nach Ausfällen. | **AWS Elastic Disaster Recovery, AWS Backup** |
| **Verbrauchsbasierte Abrechnung (Pay-as-you-go)** | Abrechnung nur für tatsächlich genutzte Ressourcen. | **AWS Pay-as-you-go** |
| **Serverless Computing** | Ausführung von Code ohne Verwaltung der zugrunde liegenden Serverinfrastruktur. | **AWS Lambda** |

---

## 2. Azure-Architektur und -Dienste

Kernkomponenten der Cloud-Infrastruktur.

### Compute-Dienste (Rechenleistung)
Compute-Dienste ermöglichen die Bereitstellung und Verwaltung von virtuellen Maschinen, Serverless-Computing und Containern. Virtuelle Maschinen (VMs) sind die Basis vieler Cloud-Anwendungen, während Serverless-Computing (z. B. Azure Functions, AWS Lambda) Entwicklern ermöglicht, Code auszuführen, ohne sich um die Infrastruktur zu kümmern.
Container-Orchestrierung ist ein weiterer wichtiger Bestandteil, da Anwendungen in isolierten Containern betrieben werden können, die über Kubernetes verwaltet werden.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Machines (VMs)** | Cloud-VMs für Windows & Linux. | **Amazon EC2** |
| **Azure App Service** | Verwalteter Hosting-Dienst für Web-Apps. | **AWS Elastic Beanstalk** |
| **Azure Functions** | Serverlose Code-Ausführung. | **AWS Lambda** |
| **Azure Kubernetes Service (AKS)** | Verwalteter Kubernetes-Dienst für Container. | **Amazon EKS** |
| **Azure Container Instances (ACI)** | Container-Ausführung ohne Orchestrierungsdienst. | **AWS Fargate** |
| **Azure Virtual Desktop** | Virtuelle Desktop-Infrastruktur. | **Amazon WorkSpaces** |

---

### Netzwerk- & Sicherheitsdienste  

Cloud-Netzwerke müssen sicher, skalierbar und performant sein. Virtuelle Netzwerke ermöglichen die Trennung und Verwaltung von Ressourcen, während Load Balancer den Traffic effizient auf mehrere Instanzen verteilen.
Sicherheitslösungen wie VPN-Gateways und Firewalls schützen Cloud-Ressourcen vor Bedrohungen. In Hybrid-Szenarien spielen dedizierte Verbindungen wie Azure ExpressRoute oder AWS Direct Connect eine zentrale Rolle, um sichere, schnelle Verbindungen zwischen On-Premises und der Cloud herzustellen.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Network (VNet)** | Private Netzwerke in Azure. | **Amazon VPC** |
| **Azure Load Balancer** | Verteilung von Datenverkehr auf mehrere Ressourcen. | **Elastic Load Balancer (ELB)** |
| **Azure VPN Gateway** | Sichere Verbindung zwischen On-Premises & Azure. | **AWS VPN Gateway** |
| **Azure ExpressRoute** | Dedizierte Hochgeschwindigkeitsverbindung. | **AWS Direct Connect** |
| **Azure Firewall** | Cloud-native Firewall für Azure-Umgebungen. | **AWS Network Firewall** |

---

## 3. Speicher- & Datenbankdienste  

Daten sind das Herzstück moderner Anwendungen. Cloud-Anbieter bieten unterschiedliche Speicherlösungen: Blob Storage für unstrukturierte Daten, Dateispeicher für gemeinsam genutzte Dateien und Datenbanken für relationale und NoSQL-Daten.
Azure Cosmos DB und Amazon DynamoDB sind skalierbare NoSQL-Datenbanken mit globaler Verfügbarkeit, während verwaltete SQL-Datenbanken in Azure und AWS eine hohe Verfügbarkeit für traditionelle Anwendungen bieten.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Blob Storage** | Skalierbarer Objektspeicher. | **Amazon S3** |
| **Azure File Storage** | Netzlaufwerk für Cloud- und On-Prem-Integration. | **Amazon EFS** |
| **Azure Cosmos DB** | Globale, verteilte NoSQL-Datenbank. | **Amazon DynamoDB** |
| **Azure SQL Database** | Verwalteter relationaler Datenbankdienst. | **Amazon RDS für SQL Server** |

---

## 4. Identitäts- & Zugriffsmanagement  

Sichere Cloud-Umgebungen benötigen effektives Identitäts- und Zugriffsmanagement (IAM). Azure AD und AWS IAM verwalten Benutzeridentitäten, Rollen und Berechtigungen. Multi-Faktor-Authentifizierung (MFA) erhöht die Sicherheit durch zusätzliche Verifizierungsschritte.
Azure bietet mit RBAC (Role-Based Access Control) eine detaillierte Zugriffskontrolle für Ressourcen, während AWS über IAM Policies & Roles ein flexibles Berechtigungssystem bereitstellt.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Active Directory (Azure AD)** | Verwaltung von Benutzeridentitäten & Zugriffskontrolle. | **AWS IAM** |
| **Azure Role-Based Access Control (RBAC)** | Zugriffskontrolle basierend auf Rollen. | **AWS IAM Policies & Roles** |
| **Azure Multi-Factor Authentication (MFA)** | Erhöhte Sicherheit durch 2FA. | **AWS MFA** |

---

## 5. Überwachung, Governance & Sicherheit  

Die Überwachung und Verwaltung von Cloud-Ressourcen ist entscheidend, um Leistung, Sicherheit und Compliance zu gewährleisten. Azure Monitor und AWS CloudWatch ermöglichen die Erfassung und Analyse von Logs und Metriken.
Azure Security Center bietet Bedrohungserkennung und Compliance-Überwachung, während AWS Security Hub ähnliche Funktionen bereitstellt. Governance-Tools wie Azure Policy und AWS Organizations helfen Unternehmen, Richtlinien und Sicherheitsstandards durchzusetzen.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Monitor** | Überwachung von Infrastruktur & Logs zur Leistungsanalyse. | **Amazon CloudWatch** |
| **Azure Service Health** | Echtzeit-Statusinformationen zu Azure-Diensten mit Warnmeldungen bei Ausfällen oder Wartungen. | **AWS Health Dashboard** |
| **Azure Resource Health** | Gibt den spezifischen Status einzelner Ressourcen an, um Probleme zu identifizieren. | **AWS Personal Health Dashboard** |
| **Microsoft Defender for Cloud** | Grundlegende Sicherheitsverwaltung und Empfehlungen zur Verbesserung der Sicherheit. | **AWS Security Hub** |
| **Azure Policy** | Governance- & Compliance-Kontrolle zur Durchsetzung von Unternehmensrichtlinien. | **AWS Organizations, AWS Config** |
| **Azure Cost Management + Billing** | Verwaltung und Optimierung von Cloud-Kosten. | **AWS Cost Explorer, AWS Budgets** |
| **Azure Sentinel** *(am Rande erwähnt)* | Cloud-nativer SIEM-Dienst für Sicherheitsanalysen und Bedrohungserkennung. | **Amazon GuardDuty** |
| **Azure Key Vault** *(am Rande erwähnt)* | Sicherer Speicher für Zertifikate, Passwörter & Verschlüsselungsschlüssel. | **AWS KMS, AWS Secrets Manager** |
| **Azure Advisor** *(am Rande erwähnt)* | Empfehlungen zur Verbesserung von Leistung, Sicherheit, Verfügbarkeit und Kosten. | **AWS Trusted Advisor** |

---

## 6. Hybrid- und Multi-Cloud-Lösungen  

Viele Unternehmen kombinieren Cloud- und On-Premises-Infrastrukturen. Azure Arc ermöglicht die Verwaltung von Multi-Cloud- und On-Premises-Ressourcen in Azure, während AWS Systems Manager eine ähnliche Funktionalität bietet.
Azure Stack und AWS Outposts bringen Cloud-Dienste direkt in das lokale Rechenzentrum, um eine einheitliche Hybrid-Cloud-Erfahrung zu ermöglichen.

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Arc** | Verwaltung von On-Premises- und Multi-Cloud-Ressourcen. | **AWS Systems Manager** |
| **Azure Stack** | Erweiterung von Azure-Diensten auf lokale Rechenzentren. | **AWS Outposts** |


