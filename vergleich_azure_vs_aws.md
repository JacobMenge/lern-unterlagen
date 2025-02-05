# Vergleich Azure vs. AWS für AZ-900  

## 1. Einführung  

Microsoft Azure und Amazon Web Services (AWS) sind die beiden führenden Cloud-Plattformen weltweit. Während AWS als Pionier des Cloud-Computings gilt, hat sich Azure als starker Konkurrent etabliert, insbesondere durch seine enge Integration mit Microsoft-Produkten.  

Die **AZ-900-Zertifizierung** prüft grundlegende Kenntnisse zu **Azure-Diensten und Cloud-Konzepten**. Ein Verständnis der **Unterschiede zwischen Azure und AWS** hilft, die jeweiligen Stärken und Schwächen der Plattformen besser einzuordnen.  

---

## 2. Cloud-Konzepte  

Grundlegendes Verständnis von Cloud-Computing, Bereitstellungsmodellen und Skalierung.  

| **Azure Begriff / Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|----------------------------|------------------|--------------------|
| **Öffentliche, private und hybride Cloud** | Bereitstellungsmodelle: Öffentliche Cloud (für alle zugänglich), private Cloud (internes Rechenzentrum), hybride Cloud (Mischform). | AWS ist primär eine **Public Cloud**, Hybrid-Cloud-Lösungen über **AWS Outposts**. |
| **IaaS, PaaS, SaaS** | Service-Modelle: Infrastruktur (IaaS), Plattform (PaaS) und Software (SaaS). | AWS bietet dieselben Modelle an. |
| **Skalierbarkeit & Elastizität** | Fähigkeit, Ressourcen dynamisch an Lasten anzupassen. | **AWS Auto Scaling** |
| **Hochverfügbarkeit & Fehlertoleranz** | Sicherstellung der kontinuierlichen Verfügbarkeit durch Redundanz. | **AWS Multi-AZ Deployments, Auto Scaling** |
| **Disaster Recovery & Geschäftskontinuität** | Strategien zur Wiederherstellung nach Ausfällen. | **AWS Elastic Disaster Recovery, AWS Backup** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure ist in Hybrid-Cloud-Szenarien überlegen** durch **Azure Arc** & **Azure Stack**.  
✔ **AWS ist Marktführer in Public Cloud** mit der größten globalen Infrastruktur.  

---

## 3. Compute-Dienste (Rechenleistung)  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Machines (VMs)** | Cloud-VMs für Windows & Linux. | **Amazon EC2** |
| **Azure App Service** | Verwalteter Hosting-Dienst für Web-Apps. | **AWS Elastic Beanstalk** |
| **Azure Functions** | Serverlose Code-Ausführung. | **AWS Lambda** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure hat eine tiefere Windows-Integration** – ideal für Unternehmen mit **Windows Server, Active Directory & .NET-Anwendungen**.  
✔ **AWS hat eine breitere Auswahl an Instanztypen & Optimierungen** für High-Performance-Anwendungen.  

---

## 4. Netzwerk- & Sicherheitsdienste  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Virtual Network (VNet)** | Privates virtuelles Netzwerk. | **Amazon VPC** |
| **Azure Load Balancer** | Lastverteilung von Netzwerkverkehr. | **Elastic Load Balancer (ELB)** |
| **Azure VPN Gateway** | Sichere Verbindung zwischen On-Premises & Azure. | **AWS VPN Gateway** |
| **Azure ExpressRoute** | Dedizierte Hochgeschwindigkeitsverbindung zu Azure. | **AWS Direct Connect** |
| **Azure Firewall** | Cloud-native Firewall für Azure-Umgebungen. | **AWS Network Firewall** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure ExpressRoute bietet stabilere On-Premises-Konnektivität** als **AWS Direct Connect**.  
✔ **AWS bietet flexiblere Routing-Optionen** innerhalb von VPCs.  

---

## 5. Speicher- & Datenbankdienste  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Blob Storage** | Skalierbarer Objektspeicher. | **Amazon S3** |
| **Azure File Storage** | Netzlaufwerk für Cloud- und On-Prem-Integration. | **Amazon EFS** |
| **Azure Cosmos DB** | Globale, verteilte NoSQL-Datenbank. | **Amazon DynamoDB** |
| **Azure SQL Database** | Verwalteter relationaler Datenbankdienst. | **Amazon RDS für SQL Server** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure Cosmos DB bietet Multi-API-Unterstützung**, AWS DynamoDB ist primär **Key-Value-basiert**.  
✔ **AWS S3 ist Marktführer im Objektspeicher**, Azure Blob Storage bietet aber **intensivere Azure-Integration**.  

---

## 6. Identitäts- & Zugriffsmanagement  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Active Directory (Azure AD)** | Verwaltung von Benutzeridentitäten & Zugriffskontrolle. | **AWS IAM** |
| **Azure Role-Based Access Control (RBAC)** | Zugriffskontrolle basierend auf Rollen. | **AWS IAM Policies & Roles** |
| **Azure Multi-Factor Authentication (MFA)** | Erhöhte Sicherheit durch 2FA. | **AWS MFA** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure AD bietet eine tiefere Integration in Unternehmensnetzwerke**.  
✔ **AWS IAM ist flexibler für API-gesteuerte Berechtigungsverwaltung**.  

---

## 7. Überwachung & Governance  

| **Azure Dienst** | **Beschreibung** | **AWS-Äquivalent** |
|------------------|------------------|--------------------|
| **Azure Monitor** | Überwachung von Infrastruktur & Logs. | **Amazon CloudWatch** |
| **Azure Security Center** | Zentrale Sicherheitsverwaltung. | **AWS Security Hub** |
| **Azure Policy** | Governance- & Compliance-Kontrolle. | **AWS Organizations, AWS Config** |
| **Azure Cost Management + Billing** | Verwaltung von Cloud-Kosten. | **AWS Cost Explorer, AWS Budgets** |

### 🏆 **Unterschiede & Vorteile**  
✔ **Azure Security Center bietet umfassendere Sicherheitsbewertungen** als AWS Security Hub.  
✔ **AWS Cost Explorer ist intuitiver für Multi-Konten-Analysen**.  
