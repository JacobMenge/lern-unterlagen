# Terraform mit AWS 

Dieses Tutorial führt dich in die Grundlagen von Terraform mit AWS ein. Du lernst, wie du deine Cloud-Infrastruktur als Code definieren, verwalten und automatisieren kannst - ein unerlässlicher Skill für moderne DevOps-Praktiken.

Schaut auch gerne in die Doku: https://developer.hashicorp.com/terraform/docs

## Inhaltsverzeichnis
- [1. Einführung in Terraform](#1-einführung-in-terraform)
  - [1.1 Was ist Infrastructure as Code (IaC)?](#11-was-ist-infrastructure-as-code-iac)
  - [1.2 Was ist Terraform und warum solltest du es nutzen?](#12-was-ist-terraform-und-warum-solltest-du-es-nutzen)
  - [1.3 Terraform vs. andere IaC-Tools](#13-terraform-vs-andere-iac-tools)
  - [1.4 Kernkonzepte: Provider, Ressourcen, Module, State](#14-kernkonzepte-provider-ressourcen-module-state)
- [2. Terraform-Architektur](#2-terraform-architektur)
  - [2.1 HashiCorp Configuration Language (HCL)](#21-hashicorp-configuration-language-hcl)
  - [2.2 Terraform-Workflow: Init, Plan, Apply, Destroy](#22-terraform-workflow-init-plan-apply-destroy)
  - [2.3 State-Management](#23-state-management)
  - [2.4 Terraform-Module](#24-terraform-module)
- [3. Terraform und AWS](#3-terraform-und-aws)
  - [3.1 Warum AWS mit Terraform?](#31-warum-aws-mit-terraform)
  - [3.2 AWS-Provider für Terraform](#32-aws-provider-für-terraform)
  - [3.3 Berechtigungsmanagement](#33-berechtigungsmanagement)
  - [3.4 Best Practices](#34-best-practices)

## 1. Einführung in Terraform

### 1.1 Was ist Infrastructure as Code (IaC)?

Stell dir vor, du müsstest jeden Server, jede Datenbank und jedes Netzwerk manuell über Web-Konsolen oder Kommandozeilen einrichten. Klingt mühsam, oder? Genau hier kommt Infrastructure as Code (IaC) ins Spiel.

**Infrastructure as Code** bedeutet, dass du deine gesamte IT-Infrastruktur in Textdateien definierst – genau wie beim Programmieren. Statt Buttons zu klicken oder Shell-Befehle einzugeben, schreibst du Code, der beschreibt, wie deine Infrastruktur aussehen soll.

Der große Vorteil: Was einmal als Code vorliegt, kannst du:
- **Versionieren** (mit Git oder ähnlichen Tools)
- **Wiederverwenden** in verschiedenen Projekten oder Umgebungen
- **Automatisiert testen** mit CI/CD-Pipelines
- **Einfach replizieren** (z.B. für Test-, Staging- und Produktionsumgebungen)
- **Nachvollziehen und prüfen**, wer wann welche Änderungen vorgenommen hat

IaC ist wie ein Bauplan für deine IT-Landschaft – präzise dokumentiert und jederzeit reproduzierbar. Du vermeidest das "Snowflake-Problem" (jeder Server ist ein einzigartiges Schneeflocken-Unikat), das oft zu unvorhersehbaren Fehlern und schwer zu behebenden Problemen führt.

### 1.2 Was ist Terraform und warum solltest du es nutzen?

**Terraform** ist ein Open-Source-Tool von HashiCorp, das Infrastructure as Code zum Leben erweckt. Es ermöglicht dir, Cloud-Ressourcen wie Server, Datenbanken, Netzwerke und vieles mehr in deklarativen Konfigurationsdateien zu beschreiben.

**Warum Terraform?**

1. **Anbieterunabhängig**: Terraform unterstützt nicht nur AWS, sondern auch Azure, Google Cloud, DigitalOcean und über 100 weitere Anbieter. Du lernst einmal die Syntax und kannst sie überall anwenden.

2. **Deklarativ statt imperativ**: Du beschreibst nur den Zielzustand ("Ich möchte 3 Server mit diesen Eigenschaften"), nicht die Schritte dorthin. Terraform kümmert sich um den Rest.

3. **State-Management**: Terraform behält den Überblick über alle erstellten Ressourcen und deren aktuelle Konfiguration, so dass Änderungen präzise und vorhersehbar sind.

4. **Planungsphase**: Mit `terraform plan` kannst du Änderungen simulieren, bevor sie tatsächlich umgesetzt werden – besonders wertvoll in produktiven Umgebungen.

5. **Modularität**: Du kannst Konfigurationen in wiederverwendbare Module verpacken und so komplexe Infrastrukturen einfach managen.

Hier ein einfaches Beispiel, wie Terraform-Code aussieht:

```hcl
# Eine EC2-Instance in AWS erstellen
resource "aws_instance" "webserver" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

Dieser Code erstellt einen einfachen Webserver in AWS – und zwar immer genau so, wie hier beschrieben. Terraform erstellt nicht nur die Ressource, sondern überwacht auch ihren Zustand und kann Änderungen erkennen und anwenden.

### 1.3 Terraform vs. andere IaC-Tools

Auf dem Markt gibt es mehrere IaC-Tools. Wie unterscheidet sich Terraform von diesen?

**AWS CloudFormation**:
- Nur für AWS-Ressourcen
- Verwendet JSON oder YAML
- Tief in AWS integriert
- Kein expliziter State (AWS behält den Überblick)

**Ansible**:
- Primär ein Konfigurationsmanagement-Tool
- Procedural/imperativ statt deklarativ
- Gut für Software-Installation und -Konfiguration
- Weniger fokussiert auf Infrastrukturerstellung

**Pulumi**:
- Nutzt echte Programmiersprachen (Python, TypeScript, etc.)
- Ähnliches Konzept wie Terraform, aber mit mehr Programmierflexibilität
- Neueres Projekt mit kleinerer Community

**Chef/Puppet**:
- Fokus auf Konfigurationsmanagement, nicht Infrastruktur
- Komplexere Lernkurve
- Etablierte Enterprise-Lösungen

**Terraform-Vorteile**:
- Anbieterunabhängigkeit
- Große Community und Ökosystem
- Einfache, deklarative Syntax
- Explizites State-Management
- Gute Dokumentation und Support

In der Praxis werden oft mehrere Tools kombiniert: Terraform für die grundlegende Infrastruktur und Ansible für die Konfiguration der erstellten Server.

### 1.4 Kernkonzepte: Provider, Ressourcen, Module, State

Um Terraform effektiv zu nutzen, solltest du diese vier Kernkonzepte verstehen:

**1. Provider**

Provider sind Plugins, die Terraform die Kommunikation mit verschiedenen Cloud-Plattformen oder Diensten ermöglichen. Der AWS-Provider beispielsweise enthält alle nötigen Funktionen, um mit Amazon Web Services zu interagieren.

Beispiel für die Provider-Konfiguration:

```hcl
provider "aws" {
  region = "eu-central-1"
}
```

In diesem einfachen Beispiel teilst du Terraform mit, dass du mit AWS-Ressourcen in der Region Frankfurt (eu-central-1) arbeiten möchtest. Der Provider fungiert als Übersetzer zwischen deinen Terraform-Befehlen und der AWS-API.

**2. Ressourcen**

Ressourcen sind die eigentlichen Infrastrukturkomponenten, die du erstellen möchtest – von EC2-Instances über S3-Buckets bis hin zu Datenbanken oder Netzwerken. Jede Ressource hat ihre eigenen Konfigurationsoptionen.

```hcl
resource "aws_s3_bucket" "beispiel" {
  bucket = "mein-beispiel-bucket"
  acl    = "private"
  
  tags = {
    Name        = "Mein Bucket"
    Environment = "Dev"
  }
}
```

In diesem Beispiel:
- `aws_s3_bucket` ist der Ressourcentyp (ein S3-Bucket in AWS)
- `beispiel` ist der lokale Name, mit dem du diese Ressource im Terraform-Code referenzieren kannst
- Innerhalb des Blocks konfigurierst du die spezifischen Eigenschaften des Buckets

**3. Module**

Module sind wiederverwendbare Terraform-Konfigurationen. Sie helfen dir, komplexe Infrastrukturen in kleinere, besser verwaltbare Teile zu zerlegen. Ein Modul kann beispielsweise eine komplette Netzwerkkonfiguration oder einen Cluster von Servern definieren.

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "meine-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["eu-central-1a", "eu-central-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}
```

**4. State**

Der Terraform-State ist das Herzstück von Terraform. Er speichert Informationen über alle erstellten Ressourcen und deren Konfigurationen. Standardmäßig wird er in einer lokalen Datei (`terraform.tfstate`) gespeichert, aber für Teamarbeit solltest du einen Remote-State verwenden.

Der State ermöglicht es Terraform:
- Den aktuellen Zustand mit dem gewünschten zu vergleichen
- Ressourcen korrekt zu aktualisieren oder zu löschen
- Abhängigkeiten zwischen Ressourcen zu verwalten

```hcl
# Beispiel für Remote-State-Konfiguration
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket"
    key    = "projekt/terraform.tfstate"
    region = "eu-central-1"
  }
}
```

Diese Kernkonzepte bilden das Fundament für alles, was du mit Terraform tun wirst.

## 2. Terraform-Architektur

### 2.1 HashiCorp Configuration Language (HCL)

Terraform-Konfigurationen werden in HashiCorp Configuration Language (HCL) geschrieben – einer von HashiCorp entwickelten Sprache, die speziell für Infrastrukturkonfigurationen optimiert ist.

**Grundelemente von HCL:**

1. **Blöcke und Attribute**:
   ```hcl
   resource "aws_instance" "beispiel" {  # Block
     ami           = "ami-0c55b159cbfafe1f0"  # Attribut
     instance_type = "t2.micro"  # Attribut
   }
   ```

2. **Variablen und Expressions**:
   ```hcl
   variable "instance_type" {
     default = "t2.micro"
   }
   
   resource "aws_instance" "beispiel" {
     instance_type = var.instance_type  # Variable referenzieren
   }
   ```

3. **Listen und Maps**:
   ```hcl
   # Liste
   availability_zones = ["eu-central-1a", "eu-central-1b"]
   
   # Map
   tags = {
     Name        = "WebServer"
     Environment = "Production"
   }
   ```

4. **Interpolation**:
   ```hcl
   name = "${var.projekt}-webserver-${var.umgebung}"
   ```

5. **Kommentare**:
   ```hcl
   # Dies ist ein Kommentar
   
   /* 
      Und dies 
      ist ein mehrzeiliger
      Kommentar 
   */
   ```

HCL ist bewusst einfach gehalten und leicht lesbar – auch für Teammitglieder, die keine Programmierer sind.

### 2.2 Terraform-Workflow: Init, Plan, Apply, Destroy

**Workflow von Terraform**

Der Terraform-Workflow besteht aus vier Hauptschritten, die du dir wie einen Bauplan für ein Haus vorstellen kannst:

**1. `terraform init`** - Der Bauplan wird vorbereitet
Initialisiert ein Terraform-Projekt:
- Lädt Provider-Plugins herunter (die Werkzeuge)
- Richtet Backend für State-Speicherung ein (die Baustellendokumentation)
- Lädt Module herunter (vorgefertigte Bauelemente)

```bash
$ terraform init

Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.19.0...
```

**2. `terraform plan`** - Die Bauplanung wird durchgegangen
Erstellt einen Ausführungsplan:
- Vergleicht aktuellen Zustand mit der Konfiguration
- Zeigt, was sich ändern würde (wie ein Architekt, der Änderungen am Bauplan erklärt)
- Hilft, unerwartete Änderungen zu erkennen

```bash
$ terraform plan

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                          = "ami-0c55b159cbfafe1f0"
      + instance_type                = "t2.micro"
      ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**3. `terraform apply`** - Das Gebäude wird errichtet
Setzt die Konfiguration um:
- Erstellt, aktualisiert oder löscht Ressourcen
- Aktualisiert den State (die Baustellendokumentation)
- Zeigt Outputs an (die Abnahme des Gebäudes)

```bash
$ terraform apply

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Creation complete after 50s [id=i-0123456789abcdef0]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

**4. `terraform destroy`** - Das Gebäude wird abgerissen
Entfernt alle von Terraform verwalteten Ressourcen:
- Löscht alle Ressourcen in umgekehrter Abhängigkeitsreihenfolge
- Fragt nach Bestätigung

```bash
$ terraform destroy

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying...
aws_instance.example: Destruction complete after 35s

Destroy complete! Resources: 1 destroyed.
```

Dieser Workflow macht Terraform-Änderungen vorhersehbar und sicher. Der große Vorteil: Du kannst immer sehen, was passieren wird, bevor du es tatsächlich umsetzt.

### 2.3 State-Management

Der Terraform-State ist entscheidend für das Verständnis, wie Terraform funktioniert. Stell ihn dir wie das Gedächtnis von Terraform vor - ohne ihn würde Terraform bei jedem Aufruf vergessen, was es bereits erstellt hat.

Der State enthält:
- Eine Abbildung zwischen deinen Ressourcen im Code und den realen Ressourcen in AWS
- Metadaten wie Ressourcen-Abhängigkeiten (welche Ressource muss vor welcher anderen erstellt werden)
- Performance-relevante Informationen (um nicht bei jedem Aufruf alles neu abfragen zu müssen)

**State-Speicherung**:

1. **Lokaler State** (Standard):
   - Einfache `.tfstate`-Datei im Projektverzeichnis
   - Gut für persönliche Projekte und erste Schritte
   - Problematisch für Teamarbeit (Wer hat die neueste Version?)

2. **Remote State**:
   - Gespeichert in S3, Azure Blob Storage, GCS, etc.
   - Unterstützt State Locking (verhindert gleichzeitige Änderungen)
   - Notwendig für Teamarbeit

Beispiel für einen S3-Backend:

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "projekt/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

In diesem Beispiel:
- Die State-Datei wird in einem S3-Bucket gespeichert
- DynamoDB wird für Locking verwendet (verhindert, dass zwei Personen gleichzeitig Änderungen vornehmen)
- Die State-Datei wird verschlüsselt gespeichert

**Best Practices für State-Management**:

1. **Remote State verwenden** für Teamarbeit
2. **State Locking aktivieren**, um gleichzeitige Änderungen zu verhindern
3. **State verschlüsseln**, da er sensible Daten enthalten kann
4. **State niemals in Git committen** (`.tfstate`-Dateien in `.gitignore` aufnehmen)
5. **Regelmäßige Backups** des States machen

**Nützliche State-Befehle**:

- `terraform state list` - zeigt alle verwalteten Ressourcen
- `terraform state show [RESSOURCE]` - zeigt Details einer Ressource
- `terraform state mv` - verschiebt Ressourcen im State (z.B. bei Umbenennungen)
- `terraform state rm` - entfernt Ressourcen aus dem State (Vorsicht: löscht nicht die tatsächliche Ressource!)
- `terraform import` - importiert existierende Ressourcen in den State

### 2.4 Terraform-Module

Module sind wiederverwendbare Terraform-Konfigurationen. Sie helfen dir, Code-Wiederholung zu vermeiden und deine Infrastruktur in logische Komponenten zu strukturieren.

**Aufbau eines Moduls**:

Ein Modul besteht typischerweise aus mehreren Dateien:
- `main.tf` - Hauptkonfiguration
- `variables.tf` - Input-Variablen
- `outputs.tf` - Output-Werte
- `README.md` - Dokumentation
- Ggf. weitere `.tf`-Dateien für spezifische Ressourcen

**Beispiel eines einfachen Moduls**:

```hcl
# modules/webserver/variables.tf
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI ID for the EC2 instance"
}

# modules/webserver/main.tf
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = {
    Name = "Webserver"
  }
}

# modules/webserver/outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web.public_ip
}
```

**Verwendung eines Moduls**:

```hcl
module "webserver" {
  source = "./modules/webserver"
  
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.medium"
}

output "webserver_ip" {
  value = module.webserver.public_ip
}
```

**Vorteile von Modulen**:

1. **Wiederverwendbarkeit**: Einmal definieren, überall nutzen
2. **Abstraktion**: Komplexität verbergen hinter einfachen Schnittstellen
3. **Konsistenz**: Gleiche Ressourcen immer gleich konfigurieren
4. **Testbarkeit**: Module können isoliert getestet werden
5. **Zusammenarbeit**: Verschiedene Teams können an verschiedenen Modulen arbeiten

Es gibt drei Haupttypen von Modulen:

1. **Lokale Module**: In deinem Projekt gespeichert
2. **Registry Module**: Aus dem Terraform Registry (public.registry.terraform.io)
3. **Git/Mercurial Module**: Aus einem Git-Repository

Das Terraform Registry enthält viele qualitativ hochwertige Module, die du direkt nutzen kannst, ohne das Rad neu zu erfinden.

## 3. Terraform und AWS

### 3.1 Warum AWS mit Terraform?

AWS ist eine der beliebtesten Cloud-Plattformen und bietet eine enorme Palette an Diensten. Terraform und AWS ergänzen sich aus mehreren Gründen hervorragend:

**Vorteile der Kombination AWS + Terraform**:

1. **Vollständige Abdeckung**: Terraform unterstützt praktisch alle AWS-Services – von EC2 und S3 bis hin zu spezialisierten Diensten wie Lambda, EKS oder AppSync.

2. **Alternative zu CloudFormation**: Im Vergleich zu AWS' eigenem IaC-Tool bietet Terraform eine einheitlichere Syntax, bessere Modularität und eine anbieterübergreifende Lösung.

3. **Umfassendes Ökosystem**: Für AWS existieren zahlreiche vorgefertigte Terraform-Module, die dir viel Arbeit ersparen können.

4. **Infrastruktur-Unabhängigkeit**: Mit Terraform kannst du bei Bedarf leichter zwischen Cloud-Anbietern wechseln oder Multi-Cloud-Architekturen aufbauen.

5. **Bessere Lesbarkeit**: Viele Entwickler finden HCL leichter zu lesen und zu schreiben als das JSON/YAML von CloudFormation.

Bevor Terraform existierte, mussten AWS-Nutzer oft CloudFormation-Templates schreiben oder AWS-Ressourcen manuell über die Konsole oder CLI erstellen. Terraform hat diese Prozesse erheblich vereinfacht und verbessert.

### 3.2 AWS-Provider für Terraform

Der AWS-Provider ist die Schnittstelle zwischen Terraform und den AWS-APIs. Er enthält den Code, der deine Terraform-Konfigurationen in API-Aufrufe an AWS übersetzt.

**Provider-Konfiguration**:

```hcl
provider "aws" {
  region = "eu-central-1"
  # Besser: Umgebungsvariablen oder AWS-Profile verwenden!
}
```

**Beste Praktiken**:

1. **NIEMALS Zugriffsschlüssel im Code speichern!** Verwende stattdessen:
   - Umgebungsvariablen (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
   - AWS-Profile (`profile = "development"`)
   - Instance-Profile (für EC2)
   - AssumeRole für temporäre Credentials

2. **Provider-Version fixieren**:
   ```hcl
   terraform {
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "~> 5.0"
       }
     }
   }
   ```
   Dies stellt sicher, dass dein Code mit einer bestimmten Version des Providers funktioniert und nicht durch unerwartete Updates bricht.

3. **Mehrere AWS-Regionen oder Konten** mit Aliassen:
   ```hcl
   # Haupt-Provider
   provider "aws" {
     region = "eu-central-1"
     alias  = "frankfurt"
   }
   
   # Zusätzlicher Provider
   provider "aws" {
     region = "eu-west-1"
     alias  = "ireland"
     profile = "production"
   }
   
   # Verwendung
   resource "aws_instance" "frankfurt_server" {
     provider = aws.frankfurt
     # ...
   }
   ```

4. **Standardtags für alle Ressourcen** (ab AWS Provider v3.38.0):
   ```hcl
   provider "aws" {
     region = "eu-central-1"
     
     default_tags {
       tags = {
         Environment = "Development"
         Owner       = "DevOps-Team"
         ManagedBy   = "Terraform"
       }
     }
   }
   ```
   Diese Tags werden automatisch auf alle Ressourcen angewendet, die mit diesem Provider erstellt werden.

Der AWS-Provider wird kontinuierlich aktualisiert, um neue AWS-Services und Features zu unterstützen. Die aktuelle Version kannst du in der [Terraform Registry](https://registry.terraform.io/providers/hashicorp/aws/latest) finden.

### 3.3 Berechtigungsmanagement

Bei der Arbeit mit AWS und Terraform ist das Berechtigungsmanagement entscheidend für die Sicherheit. Denke daran: Terraform kann potenziell deine gesamte Cloud-Infrastruktur erstellen und verändern, daher sind korrekte Berechtigungen wichtig.

**IAM-Benutzer für Terraform**:

Terraform benötigt Zugriff auf AWS über Benutzeranmeldedaten. Am besten erstellst du einen speziellen IAM-Benutzer für Terraform:

1. **Programmgesteuerten Zugriff** aktivieren (kein Konsolenzugriff nötig)
2. **Minimale Berechtigungen** nach dem Prinzip der geringsten Berechtigung zuweisen
3. Zugangsdaten **sicher speichern** und regelmäßig rotieren

**Beispiel für eine IAM-Richtlinie mit eingeschränkten Berechtigungen**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:RunInstances",
        "ec2:TerminateInstances",
        "ec2:CreateTags",
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "*"
    }
  ]
}
```

Diese Richtlinie gewährt nur die Berechtigungen, die für bestimmte EC2- und S3-Operationen erforderlich sind, anstatt vollen Zugriff zu gewähren.

**Sicherheitspraktiken**:

1. **Separate AWS-Konten** für verschiedene Umgebungen (Entwicklung, Test, Produktion)
2. **Cross-Account-Rollen** für Terraform, um zwischen Konten zu wechseln
3. **MFA (Multi-Faktor-Authentifizierung)** für sensible Umgebungen aktivieren
4. **Temporäre Anmeldeinformationen** über `assume-role` verwenden
5. **Service Control Policies (SCPs)** für organisationsweite Beschränkungen

**Berechtigungen im Code definieren**:

Terraform kann auch IAM-Ressourcen selbst verwalten – was praktisch, aber auch gefährlich sein kann:

```hcl
resource "aws_iam_user" "app_user" {
  name = "application-user"
}

resource "aws_iam_policy" "app_policy" {
  name        = "app-policy"
  description = "Application specific permissions"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Resource = [aws_s3_bucket.app_data.arn]
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "app_attach" {
  user       = aws_iam_user.app_user.name
  policy_arn = aws_iam_policy.app_policy.arn
}
```

**Wichtig**: Achte besonders auf die Berechtigungen des IAM-Benutzers, der Terraform ausführt. Ein zu mächtiger Benutzer kann versehentlich kritische Ressourcen löschen oder ändern. Ein zu eingeschränkter Benutzer kann erforderliche Ressourcen nicht erstellen.Effect   = "Allow"
        Resource = [aws_s3_bucket.app_data.arn]
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "app_attach" {
  user       = aws_iam_user.app_user.name
  policy_arn = aws_iam_policy.app_policy.arn
}
```

**Wichtig**: Achte besonders auf die Berechtigungen des IAM-Benutzers, der Terraform ausführt. Ein zu mächtiger Benutzer kann versehentlich kritische Ressourcen löschen oder ändern. Ein zu eingeschränkter Benutzer kann erforderliche Ressourcen nicht erstellen.Effect   = "Allow"
        Resource = [aws_s3_bucket.app_data.arn]
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "app_attach" {
  user       = aws_iam_user.app_user.name
  policy_arn = aws_iam_policy.app_policy.arn
}
```

### 3.4 Best Practices

Hier sind einige bewährte Praktiken für die Arbeit mit Terraform und AWS, die dir helfen, sauberen, wartbaren und sicheren Code zu schreiben:

**1. Projektstruktur**:

Eine gute Struktur ist das Fundament für erfolgreiche Terraform-Projekte. So könnte eine typische Projektstruktur aussehen:

```
projekt/
├── main.tf           # Hauptkonfiguration
├── variables.tf      # Eingabevariablen
├── outputs.tf        # Ausgabewerte
├── providers.tf      # Provider-Konfiguration
├── backend.tf        # State-Backend-Konfiguration
├── modules/          # Lokale Module
│   ├── networking/
│   └── computing/
└── environments/     # Umgebungsspezifische Konfigurationen
    ├── dev/
    ├── staging/
    └── prod/
```

Diese Struktur trennt klar zwischen wiederverwendbaren Komponenten (Modulen) und umgebungsspezifischen Konfigurationen.

**2. Aussagekräftige Ressourcenbenennung**:

Namen sollten klar und konsistent sein, um die Lesbarkeit zu verbessern:

```hcl
# Gut: Klar und aussagekräftig
resource "aws_instance" "web_server" {
  # ...
}

# Schlecht: Kryptisch und wenig aussagekräftig
resource "aws_instance" "ws1" {
  # ...
}
```

**3. Konsequente Tagging-Strategie**:

Tags machen deine AWS-Ressourcen besser auffindbar und vereinfachen die Kostenzuordnung:

```hcl
# Gemeinsame Tags für alle Ressourcen
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    Owner       = "DevOps Team"
    ManagedBy   = "Terraform"
  }
}

resource "aws_instance" "example" {
  # ...
  
  tags = merge(
    local.common_tags,
    {
      Name = "WebServer"
    }
  )
}
```

**4. Variablen mit Validierung und Beschreibungen**:

Validierungen helfen, Fehler frühzeitig zu erkennen:

```hcl
variable "instance_type" {
  description = "Der EC2-Instance-Typ"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t3.micro"], var.instance_type)
    error_message = "Nur t2.micro, t2.small oder t3.micro sind erlaubt."
  }
}
```

**5. Terraform- und Provider-Versionen festlegen**:

Fixiere die Versionen, um Überraschungen bei Updates zu vermeiden:

```hcl
terraform {
  required_version = ">= 1.5.0, < 2.0.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**6. Remote State mit Locking**:

Für Teamarbeit ist ein Remote State mit Locking unerlässlich:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**7. Aussagekräftige Outputs definieren**:

Outputs machen wichtige Informationen für Nutzer deiner Infrastruktur zugänglich:

```hcl
output "web_endpoint" {
  description = "Öffentliche DNS-Adresse des Webservers"
  value       = aws_instance.web.public_dns
  
  # Optional: Sensitive-Flag für vertrauliche Daten
  sensitive   = false
}
```

**8. Code-Tests implementieren**:

Teste deine Infrastruktur-Code mit Tools wie Terratest oder checkov:

```bash
# Beispiel für ein einfaches Syntax-Check-Skript
for file in $(find . -name "*.tf"); do
  terraform fmt -check=true $file
  if [ $? -ne 0 ]; then
    echo "File $file is not properly formatted"
    exit 1
  fi
done
```

**9. CI/CD-Integration**:

Automatisiere Terraform-Ausführungen über CI/CD-Pipelines (z.B. GitHub Actions, GitLab CI, Jenkins):

```yaml
# Beispiel für GitHub Actions Workflow
name: Terraform

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
    - name: Terraform Init
      run: terraform init
    - name: Terraform Validate
      run: terraform validate
    - name: Terraform Plan
      run: terraform plan
```

**10. Secrets Management**:

Verwende AWS Secrets Manager, Parameter Store oder HashiCorp Vault für sensible Daten:

```hcl
data "aws_secretsmanager_secret" "db_password" {
  name = "production/db/password"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = data.aws_secretsmanager_secret.db_password.id
}

resource "aws_db_instance" "database" {
  # ...
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}
```

Diese Best Practices helfen dir, Terraform-Code zu schreiben, der robust, wartbar und sicher ist - und gleichzeitig gut in Teams funktioniert.
