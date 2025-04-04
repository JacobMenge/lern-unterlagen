# Deine Terraform-Reise: Schritt für Schritt

Wir bauen Schritt für Schritt dein Wissen auf und du wirst sehen, wie einfach und mächtig Infrastructure as Code sein kann.

## Übung 1: Deine ersten Schritte in die Terraform-Welt

Fangen wir ganz von vorne an:

1. Installiere Terraform auf deinem Rechner https://github.com/BrianR-Back2Code/Terraform
2. Erstelle dir einen neuen Projektordner – hier wird deine Reise beginnen
3. Leg eine Datei namens `main.tf` an und füge diesen Code ein:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

resource "local_file" "hallo_welt" {
  content  = "Hallo Terraform-Welt! Meine erste selbsterstellte Datei."
  filename = "${path.module}/hallo_welt.txt"
}
```

4. Öffne dein Terminal im Projektordner und tippe `terraform init` – Terraform richtet nun deine Arbeitsumgebung ein
5. Gib `terraform plan` ein und schau dir an, was Terraform vorhat
6. Mit `terraform apply` (und einem "yes" zur Bestätigung) setzt du deinen Plan in die Tat um
7. Schau in deinem Ordner nach – die Datei sollte dort erschienen sein!
8. Wenn du aufräumen möchtest, nutze `terraform destroy` – und alles ist wieder wie vorher

## Übung 2: Mit Variablen das Leben leichter machen

Jetzt wird's flexibler. Wir nutzen Variablen, um unseren Code anpassungsfähiger zu machen:

1. Erstelle eine neue Datei `variables.tf` mit diesem Inhalt:

```hcl
variable "dateiname" {
  description = "Wie soll deine Datei heißen?"
  type        = string
  default     = "meine_nachricht.txt"
}

variable "dateiinhalt" {
  description = "Was soll in deiner Datei stehen?"
  type        = string
  default     = "Dies ist meine persönliche Terraform-Nachricht."
}
```

2. Leg eine Datei `outputs.tf` an, um dir wichtige Infos anzeigen zu lassen:

```hcl
output "datei_pfad" {
  value       = local_file.beispiel.filename
  description = "Hier findest du deine erstellte Datei"
}
```

3. Passe nun deine `main.tf` an:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

resource "local_file" "beispiel" {
  content  = var.dateiinhalt
  filename = "${path.module}/${var.dateiname}"
}
```

4. Führe den bekannten Terraform-Workflow aus (`init`, `plan`, `apply`)
5. Probier mal aus, eigene Werte einzugeben: `terraform apply -var="dateiname=geheim.txt" -var="dateiinhalt=Mein geheimer Text!"`

## Übung 3: Module – Der Baukasten für deine Infrastruktur

Zeit, deinen Code modularer zu gestalten:

1. Erstelle einen Ordner namens `module` in deinem Projekt
2. Darin erstellst du eine `main.tf`:

```hcl
resource "local_file" "modulfile" {
  content  = var.inhalt
  filename = "${path.module}/${var.name}"
}
```

3. Und eine `variables.tf` im selben Modulordner:

```hcl
variable "name" {
  description = "Dateiname für deine Moduldatei"
  type        = string
}

variable "inhalt" {
  description = "Was soll in deiner Moduldatei stehen?"
  type        = string
}
```

4. Jetzt passt du deine Haupt-`main.tf` im Wurzelverzeichnis an:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

module "datei_erstellen" {
  source = "./module"
  
  name   = "modul_nachricht.txt"
  inhalt = "Wow! Diese Datei kommt aus einem eigenen Modul!"
}
```

5. Führe den Terraform-Workflow aus und schau, was passiert
6. Du solltest nun eine neue Datei haben, die dein Modul erstellt hat

## Übung 4: Ab in die Cloud – AWS S3 Bucket erstellen

Jetzt wird's ernst – wir gehen in die echte Cloud (hierfür brauchst du ein AWS-Konto):

1. Erstelle eine `provider.tf`:

```hcl
provider "aws" {
  region = "eu-central-1"  # Die Frankfurt-Region, oder wähle deine bevorzugte Region
}
```

2. Passe deine `main.tf` an:

```hcl
resource "aws_s3_bucket" "mein_bucket" {
  bucket = "mein-terraform-bucket-${random_string.bucketname.result}"
}

# Moderne S3-Einstellungen für mehr Sicherheit
resource "aws_s3_bucket_versioning" "bucket_versioning" {
  bucket = aws_s3_bucket.mein_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "bucket_encryption" {
  bucket = aws_s3_bucket.mein_bucket.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Zufallsstring für einen eindeutigen Bucketnamen
resource "random_string" "bucketname" {
  length  = 8
  special = false
  upper   = false
}
```

3. Füge eine nützliche Ausgabe in `outputs.tf` hinzu:

```hcl
output "bucket_name" {
  value       = aws_s3_bucket.mein_bucket.bucket
  description = "Name deines erstellten S3-Buckets"
}

output "bucket_arn" {
  value       = aws_s3_bucket.mein_bucket.arn
  description = "ARN deines erstellten S3-Buckets"
}
```

4. Führe `terraform init` aus, um die AWS-Provider zu laden
5. Dann `terraform apply` und beobachte, wie dein Bucket in der Cloud entsteht
6. Schau in der AWS-Konsole nach deinem neuen Bucket
7. Denk daran, mit `terraform destroy` wieder aufzuräumen, damit keine unerwarteten Kosten entstehen

## Übung 5: Verschiedene Umgebungen mit Workspaces

In der realen Welt brauchst du oft verschiedene Umgebungen:

1. Erstelle eine neue `main.tf` mit diesem Code:

```hcl
locals {
  umgebung = terraform.workspace
  prefix   = terraform.workspace == "produktion" ? "prod" : "dev"
}

resource "local_file" "umgebungs_datei" {
  content  = "Willkommen in der ${local.umgebung}-Umgebung! Hier gelten besondere Regeln."
  filename = "${path.module}/${local.prefix}_config.txt"
}
```

2. Zeit für deine ersten Workspaces:
   - Tippe `terraform workspace new entwicklung`
   - Und dann `terraform workspace new produktion`

  > **Hinweis:** `terraform workspace show` Dieser Befehl gibt einfach den Namen des Workspaces aus, in dem du dich gerade befindest (z.B. default, staging, production).


3. Wechsle zwischen den Umgebungen und sieh, was passiert:
   - `terraform workspace select entwicklung`
   - `terraform apply`
   - `terraform workspace select produktion`
   - `terraform apply`

4. Du solltest nun zwei verschiedene Dateien haben – für jede Umgebung eine!

## Übung 6: Dynamische Ressourcen mit for_each

Fortgeschrittene Terraform-Nutzer lieben for_each – so erstellst du mehrere ähnliche Ressourcen auf einmal:

1. Erstelle eine neue `main.tf`:

```hcl
locals {
  dateien = {
    begruessung = "Hallo und willkommen zu Terraform!"
    tipp        = "Nutze for_each für elegante Wiederholungen."
    zitat       = "Infrastructure as Code verändert die Art, wie wir arbeiten."
  }
}

resource "local_file" "mehrere_dateien" {
  for_each = local.dateien
  
  content  = each.value
  filename = "${path.module}/${each.key}.txt"
}
```

2. Führe die bekannten Terraform-Befehle aus
3. Schau in deinem Verzeichnis nach – du solltest mehrere Dateien mit unterschiedlichem Inhalt haben

## Bonus-Übung: Terraform State Remote speichern

Im Team-Kontext ist ein gemeinsamer, remote gespeicherter State wichtig:

1. Erstelle zuerst einen S3-Bucket für deinen State (hier vereinfacht):

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "mein-terraform-state-bucket-${random_string.suffx.result}"
}

resource "random_string" "suffx" {
  length  = 6
  special = false
  upper   = false
}
```

2. Nach dem Anlegen des Buckets, konfiguriere ein Backend in einer neuen `backend.tf`:

```hcl
terraform {
  backend "s3" {
    bucket = "DEIN-BUCKET-NAME"  # Ersetze mit deinem tatsächlichen Bucket-Namen
    key    = "terraform.tfstate"
    region = "eu-central-1"
  }
}
```

3. Initialisiere Terraform neu mit `terraform init` und bestätige die State-Migration

Das war's! Du hast gerade eine komplette Terraform-Reise absolviert – von den ersten Schritten bis hin zu fortgeschrittenen Techniken. Jede Übung baut auf der vorherigen auf, und zusammen geben sie dir ein solides Fundament für deine eigenen Infrastructure-as-Code-Projekte. Viel Spaß beim Experimentieren und Erweitern!
