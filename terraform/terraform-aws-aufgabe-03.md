# Terraform-Module: Schritt-für-Schritt-Übung

In dieser praktischen Übung lernst du, wie du mit Terraform-Modulen arbeitest. Solltest du während der Übung Probleme haben oder Fragen aufkommen, melde dich gerne jederzeit per Slack bei mir. Ich helfe dir gerne weiter!

> **⚠️ Wichtiger Hinweis zu AWS-Kosten:**  
> Diese Übung verwendet AWS-Ressourcen, die Kosten verursachen können. Wir haben die Ressourcen so kostengünstig wie möglich gewählt, aber es ist wichtig, dass du nach Abschluss der Übung alle Ressourcen mit `terraform destroy` wieder löschst. Selbst "kleine" Ressourcen können über Zeit Kosten verursachen!

## Was sind Terraform-Module und warum sind sie wichtig?

Terraform-Module sind wie Bausteine für deine Infrastruktur. Stell dir vor, du baust immer wieder ähnliche Dinge - Webserver, Datenbanken, Netzwerke. Statt den Code jedes Mal neu zu schreiben, packst du ihn in ein Modul und verwendest dieses immer wieder.

**Warum Module wichtig sind:**
- **Wiederverwendbarkeit**: Einmal schreiben, überall nutzen
- **Wartbarkeit**: Fehler nur an einer Stelle beheben, nicht in 20 verschiedenen Dateien
- **Konsistenz**: Alle Teams nutzen die gleichen, gut getesteten Module
- **Abstraktion**: Komplexe Infrastruktur hinter einfachen Schnittstellen verstecken
- **Teamarbeit**: Verschiedene Teams können an verschiedenen Modulen arbeiten

Im Prinzip sind Module einfach eine Gruppe von Terraform-Dateien in einem Ordner. Aber die Art, wie wir sie nutzen, macht sie so mächtig.

## Übungsaufbau

Wir werden ein einfaches, wiederverwendbares Terraform-Modul erstellen und es in zwei verschiedenen Umgebungen nutzen. So verstehst du das Konzept von Modulen ganz praktisch.

## Schritt 1: Projektstruktur erstellen

**Was wir machen:** Wir legen die Grundstruktur für unser Projekt an.

**Warum:** Eine klare Ordnerstruktur hilft dir, den Überblick zu behalten und macht es anderen im Team leichter, sich zurechtzufinden.

```bash
mkdir -p terraform-module-workshop/{modules/webserver,environments/{dev,prod,shared}}
cd terraform-module-workshop
```

**Was du lernst:** Du verstehst, wie man Terraform-Projekte mit Modulen und unterschiedlichen Umgebungen organisiert - ein echter Best-Practice-Ansatz.

## Schritt 2: Das Webserver-Modul erstellen

**Was wir machen:** Wir erstellen die Grunddateien für unser Webserver-Modul.

### Schritt 2.1: variables.tf erstellen

Erstelle die Datei `modules/webserver/variables.tf`:

```hcl
variable "environment" {
  description = "Umgebungsname (z.B. dev, prod)"
  type        = string
}

variable "instance_type" {
  description = "EC2 Instance-Typ"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI ID für die EC2-Instance"
  type        = string
}

variable "vpc_id" {
  description = "ID des VPC, in dem die Ressourcen erstellt werden"
  type        = string
}

variable "subnet_id" {
  description = "ID des Subnets für die EC2-Instance"
  type        = string
}
```

**Warum:** Variablen sind das Herzstück eines flexiblen Moduls. Sie ermöglichen es, das gleiche Modul in verschiedenen Situationen unterschiedlich zu konfigurieren - ohne den Code anzufassen.

**Was du lernst:** Du lernst, wie man Input-Variablen richtig definiert, mit Beschreibungen und Standardwerten versieht und damit die Grundlage für ein wiederverwendbares Modul schafft.

### Schritt 2.2: main.tf erstellen

Erstelle die Datei `modules/webserver/main.tf`:

```hcl
resource "aws_security_group" "web" {
  name        = "web-sg-${var.environment}"
  description = "Security Group für Webserver (${var.environment})"
  vpc_id      = var.vpc_id

  # SSH zugriff
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "SSH"
  }

  # HTTP zugriff
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP"
  }

  # Ausgehenden Traffic erlauben
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "web-sg-${var.environment}"
    Environment = var.environment
  }
}

resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  # Einfacher Webserver für Demo-Zwecke
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello from ${var.environment} environment!" > index.html
              nohup python3 -m http.server 80 &
              EOF

  tags = {
    Name        = "webserver-${var.environment}"
    Environment = var.environment
  }
}
```

**Warum:** Hier definieren wir, was unser Modul tatsächlich tut: eine EC2-Instance und eine Security Group erstellen. Beachte, wie wir die Variablen nutzen, um verschiedene Aspekte zu konfigurieren.

**Was du lernst:** Du lernst, wie man innerhalb eines Moduls Ressourcen definiert und dabei die übergebenen Variablen nutzt, um die Konfiguration anzupassen.

### Schritt 2.3: outputs.tf erstellen

Erstelle die Datei `modules/webserver/outputs.tf`:

```hcl
output "instance_id" {
  description = "ID der erstellten EC2-Instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Öffentliche IP-Adresse der EC2-Instance"
  value       = aws_instance.web.public_ip
}
```

**Warum:** Outputs sind wie Rückgabewerte einer Funktion - sie geben wichtige Informationen aus dem Modul an den Aufrufer zurück. So können wir später z.B. die IP-Adresse des Servers anzeigen.

**Was du lernst:** Du verstehst, wie man wichtige Informationen aus einem Modul heraus verfügbar macht, damit sie woanders genutzt werden können.

## Schritt 3: Umgebungsspezifische Konfigurationen erstellen

### Schritt 3.1: Gemeinsame VPC erstellen

Um Kosten zu sparen, erstellen wir eine gemeinsame VPC für alle Umgebungen. Erstelle die Datei `environments/shared/main.tf`:

```hcl
provider "aws" {
  region = "eu-central-1"
}

# Gemeinsame VPC für alle Umgebungen
resource "aws_vpc" "shared" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  
  tags = {
    Name = "shared-vpc"
    Purpose = "Terraform-Module-Workshop"
  }
}

# Shared subnet
resource "aws_subnet" "shared" {
  vpc_id                  = aws_vpc.shared.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "shared-subnet"
    Purpose = "Terraform-Module-Workshop"
  }
}

# Outputs
output "vpc_id" {
  value = aws_vpc.shared.id
}

output "subnet_id" {
  value = aws_subnet.shared.id
}
```

Erstelle dann die Datei `environments/shared/outputs.tf` um auch einen Remote State Zugriff zu ermöglichen:

```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

### Schritt 3.2: Entwicklungsumgebung konfigurieren

Erstelle die Datei `environments/dev/main.tf`:

```hcl
provider "aws" {
  region = "eu-central-1"
}

# Zugriff auf die gemeinsame VPC
data "terraform_remote_state" "shared" {
  backend = "local"
  config = {
    path = "../shared/terraform.tfstate"
  }
}

# Verwendung des Webserver-Moduls
module "dev_webserver" {
  source = "../../modules/webserver"
  
  environment   = "dev"
  instance_type = "t2.micro"  # Free Tier eligible
  ami_id        = "ami-0faab6bdbac9486fb"  # Amazon Linux 2023 AMI in eu-central-1
  vpc_id        = data.terraform_remote_state.shared.outputs.vpc_id
  subnet_id     = data.terraform_remote_state.shared.outputs.subnet_id
}

# Outputs
output "dev_instance_ip" {
  value = module.dev_webserver.public_ip
}
```

**Warum:** Jetzt wird's spannend - wir verwenden unser selbst erstelltes Modul! Dabei übergeben wir die entwicklungsspezifischen Werte und rufen das Modul wie eine Funktion auf.

**Was du lernst:** Du siehst, wie man ein Modul in einem realen Kontext verwendet und welche Werte man übergeben muss. Der Trick liegt darin, dass wir Module mit unterschiedlichen Parametern aufrufen können.

### Schritt 3.3: Produktionsumgebung konfigurieren

Erstelle die Datei `environments/prod/main.tf`:

```hcl
provider "aws" {
  region = "eu-central-1"
}

# Zugriff auf die gemeinsame VPC
data "terraform_remote_state" "shared" {
  backend = "local"
  config = {
    path = "../shared/terraform.tfstate"
  }
}

# Verwendung des Webserver-Moduls
module "prod_webserver" {
  source = "../../modules/webserver"
  
  environment   = "prod"
  instance_type = "t3a.micro"  # AMD-basierte Instance, kostengünstiger als Intel
  ami_id        = "ami-0faab6bdbac9486fb"  # Amazon Linux 2023 AMI in eu-central-1
  vpc_id        = data.terraform_remote_state.shared.outputs.vpc_id
  subnet_id     = data.terraform_remote_state.shared.outputs.subnet_id
}

# Outputs
output "prod_instance_ip" {
  value = module.prod_webserver.public_ip
}
```

**Warum:** Hier zeigt sich die Magie der Module - wir nutzen genau denselben Code, aber mit anderen Parametern für die Produktionsumgebung.

**Was du lernst:** Du erkennst, wie du mit demselben Modul verschiedene Umgebungen aufbauen kannst, indem du einfach andere Parameter übergibst. Das spart enorm viel doppelten Code!

## Schritt 4: Testing und Deployment

### Schritt 4.1: Shared-Umgebung erstellen

```bash
cd environments/shared
terraform init
terraform apply
```

Bestätige mit "yes", wenn du gefragt wirst.

**Warum:** Wir erstellen zuerst die gemeinsame VPC, die von beiden Umgebungen verwendet wird.

**Was du lernst:** Wie man gemeinsame Ressourcen erstellt und über Remote State teilt.

> **💡 AWS-Tipp:**  
> Eine gemeinsame VPC für mehrere Umgebungen zu verwenden ist im Produktionseinsatz nicht empfehlenswert (schlechte Isolation), spart aber bei dieser Übung Kosten. Eine VPC selbst kostet nichts, aber zusätzliche Komponenten wie NAT Gateways können teuer werden.

### Schritt 4.2: Entwicklungsumgebung initialisieren und testen

```bash
cd ../dev
terraform init
terraform plan
```

**Warum:** Bevor Terraform etwas erstellen kann, müssen wir es initialisieren und können dann einen Plan erstellen, der uns zeigt, was passieren würde - wie eine Vorschau.

**Was du lernst:** Du lernst den Workflow kennen: Erst initialisieren, dann planen, und verstehst, wie Terraform mit Modulen arbeitet.

### Schritt 4.3: Entwicklungsumgebung anwenden (optional)

```bash
terraform apply
```

Bestätige mit "yes", wenn du gefragt wirst.

**Warum:** Jetzt wird's ernst - Terraform erstellt tatsächlich die Ressourcen in AWS.

**Was du lernst:** Du siehst, wie Terraform die modulbasierte Infrastruktur in der echten Welt umsetzt.

### Schritt 4.4: Produktionsumgebung testen (optional)

```bash
cd ../prod
terraform init
terraform plan
```

**Warum:** Auch für die Produktionsumgebung machen wir erst eine Vorschau - in der Praxis würdest du das vielleicht erst nach erfolgreichen Tests in der Dev-Umgebung machen.

**Was du lernst:** Du verstehst, wie das gleiche Modul unterschiedlich konfiguriert werden kann und siehst die Unterschiede im Plan.

> **⚠️ AWS-Kostenmerkblatt:**
> - EC2-Instanzen erzeugen laufende Kosten, selbst wenn sie nicht verwendet werden
> - t2.micro/t3a.micro sind im Free Tier kostenlos (für 12 Monate, mit Limits)
> - Öffentliche IP-Adressen können Kosten verursachen
> - Nach dem Üben unbedingt alle Ressourcen mit `terraform destroy` in jeder Umgebung löschen!

## Schritt 5: Erweiterung des Moduls (einfache Variante)

### Schritt 5.1: variables.tf erweitern

Füge in `modules/webserver/variables.tf` hinzu:

```hcl
variable "enable_monitoring" {
  description = "CloudWatch detailliertes Monitoring aktivieren"
  type        = bool
  default     = false
}

variable "instance_tags" {
  description = "Zusätzliche Tags für die EC2 Instance"
  type        = map(string)
  default     = {}
}
```

### Schritt 5.2: main.tf anpassen

Passe in `modules/webserver/main.tf` die EC2-Instance an:

```hcl
resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = [aws_security_group.web.id]
  monitoring             = var.enable_monitoring  # Neue Option
  
  # Rest bleibt gleich...
  
  # Besseres Tagging
  tags = merge(
    {
      Name        = "webserver-${var.environment}"
      Environment = var.environment
    },
    var.instance_tags
  )
}
```

**Warum:** Jetzt wird's richtig cool - wir erweitern ein bestehendes Modul, ohne dass bestehende Anwendungen kaputtgehen. Das geht, weil wir einen Standardwert gesetzt haben.

**Was du lernst:** Du lernst, wie du ein Modul weiterentwickeln kannst, ohne dass bestehende Nutzer etwas ändern müssen - ein wichtiger Aspekt nachhaltiger Infrastruktur.

> **💡 AWS-Kostentipp:**
> Das detaillierte CloudWatch-Monitoring verursacht zusätzliche Kosten. Für Lernzwecke solltest du es deaktiviert lassen (der Standardwert `false` tut genau das). Aber in einer Produktionsumgebung kann es trotz der Kosten sinnvoll sein, um Performance-Probleme zu erkennen.

## Schritt 6: Aufräumen nach der Übung

> **⚠️ WICHTIG:**  
> Dieser Schritt ist notwendig, um unerwartete AWS-Kosten zu vermeiden!

Nach Abschluss der Übung solltest du alle erstellten Ressourcen wieder löschen:

```bash
# Zuerst die Produktionsumgebung aufräumen
cd environments/prod
terraform destroy

# Dann die Entwicklungsumgebung aufräumen
cd ../dev
terraform destroy

# Zuletzt die gemeinsame Infrastruktur aufräumen
cd ../shared
terraform destroy
```

Bestätige jeweils mit "yes", wenn du gefragt wirst.

**Warum:** Selbst wenn die einzelnen Ressourcen günstig erscheinen, können sie über die Zeit Kosten verursachen. Besonders EC2-Instanzen erzeugen laufende Kosten, solange sie aktiv sind.

**Was du lernst:** Den verantwortungsvollen Umgang mit Cloud-Ressourcen und wie wichtig es ist, nicht benötigte Infrastruktur aufzuräumen.

## Fortgeschrittene Konzepte (optional)

Hier sind einige weiterführende Konzepte, die du nach dieser Grundübung erkunden könntest:

1. **Module aus dem Terraform Registry** verwenden (z.B. das offizielle AWS VPC-Modul)
2. **Remote State mit S3 und DynamoDB** für Teamarbeit einrichten
3. **Workspaces** für Umgebungsmanagement nutzen
4. **Terragrunt** als Wrapper für DRY-Infrastrukturcode verwenden
5. **CI/CD-Pipelines** für Terraform-Deployments erstellen

> **💡 Kostentipp für weiterführendes Lernen:**  
> Für fortgeschrittene Übungen kannst du die AWS-Kosten niedrig halten durch:
> - Nutzung von Spot-Instances statt On-Demand
> - Einrichtung von AWS Budget Alerts (um Kostenalarme zu erhalten)
> - Verwendung einer Sandbox-AWS-Organisation mit Kostenlimits
> - Automatisches Abschalten von Ressourcen außerhalb der Arbeitszeiten

Falls du Fragen zur Übung hast oder Hilfe benötigst, melde dich gerne per Slack bei mir!
