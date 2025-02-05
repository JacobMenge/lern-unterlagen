# Vergleich Azure vs. AWS für AZ-900  

## 1. Einführung  

Microsoft Azure und Amazon Web Services (AWS) sind die beiden führenden Cloud-Plattformen weltweit. Während AWS als Pionier des Cloud-Computings gilt, hat sich Azure als starker Konkurrent etabliert, insbesondere durch seine enge Integration mit Microsoft-Produkten.  

Die **AZ-900-Zertifizierung** prüft grundlegende Kenntnisse zu **Azure-Diensten und Cloud-Konzepten**. Ein Verständnis der **Unterschiede zwischen Azure und AWS** hilft, die jeweiligen Stärken und Schwächen der Plattformen besser einzuordnen.  

# Vergleich Azure vs. AWS für AZ-900

## 1. Cloud-Konzepte

Grundlegendes Verständnis von Cloud-Computing, Bereitstellungsmodellen und Skalierung.

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

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Network (VNet)** | Private Netzwerke in Azure. | **Amazon VPC** |
| **Azure Load Balancer** | Verteilung von Datenverkehr auf mehrere Ressourcen. | **Elastic Load Balancer (ELB)** |
| **Azure VPN Gateway** | Sichere Verbindung zwischen On-Premises & Azure. | **AWS VPN Gateway** |
| **Azure ExpressRoute** | Dedizierte Hochgeschwindigkeitsverbindung. | **AWS Direct Connect** |
| **Azure Firewall** | Cloud-native Firewall für Azure-Umgebungen. | **AWS Network Firewall** |

---

## 3. Speicher- & Datenbankdienste  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Blob Storage** | Skalierbarer Objektspeicher. | **Amazon S3** |
| **Azure File Storage** | Netzlaufwerk für Cloud- und On-Prem-Integration. | **Amazon EFS** |
| **Azure Cosmos DB** | Globale, verteilte NoSQL-Datenbank. | **Amazon DynamoDB** |
| **Azure SQL Database** | Verwalteter relationaler Datenbankdienst. | **Amazon RDS für SQL Server** |

---

## 4. Identitäts- & Zugriffsmanagement  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Active Directory (Azure AD)** | Verwaltung von Benutzeridentitäten & Zugriffskontrolle. | **AWS IAM** |
| **Azure Role-Based Access Control (RBAC)** | Zugriffskontrolle basierend auf Rollen. | **AWS IAM Policies & Roles** |
| **Azure Multi-Factor Authentication (MFA)** | Erhöhte Sicherheit durch 2FA. | **AWS MFA** |

---

## 5. Überwachung, Governance & Sicherheit  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Monitor** | Überwachung von Infrastruktur & Logs. | **Amazon CloudWatch** |
| **Azure Security Center** | Zentrale Sicherheitsverwaltung. | **AWS Security Hub** |
| **Azure Policy** | Governance- & Compliance-Kontrolle. | **AWS Organizations, AWS Config** |
| **Azure Cost Management + Billing** | Verwaltung von Cloud-Kosten. | **AWS Cost Explorer, AWS Budgets** |
| **Azure Sentinel** | Cloud-nativer SIEM-Dienst für Sicherheitsanalysen. | **Amazon GuardDuty** |

---

## 6. Hybrid- und Multi-Cloud-Lösungen  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Arc** | Verwaltung von On-Premises- und Multi-Cloud-Ressourcen. | **AWS Systems Manager** |
| **Azure Stack** | Erweiterung von Azure-Diensten auf lokale Rechenzentren. | **AWS Outposts** |


