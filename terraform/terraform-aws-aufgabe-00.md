# Terraform Core Fundamentals: Essenzielle Übungen für Infrastructure as Code

Diese Übungen führen dich durch die grundlegendsten Terraform-Konzepte. Jede Übung baut auf der vorherigen auf und enthält nicht nur eine Anleitung, sondern auch Erklärungen, warum die jeweiligen Schritte wichtig sind.

## Übung 1: Terraform Installation und Verifizierung

**Zusammenfassung:** In dieser Übung installierst du Terraform und validierst die korrekte Installation anhand der Version.

Für eine Installationsanleitung empfehle ich dir das Tutorial von Brian: https://github.com/BrianR-Back2Code/Terraform

**Warum ist dieser Schritt wichtig?**  
Terraform ist ein Command-Line-Tool, das auf deinem lokalen System installiert sein muss. Die Verifizierung stellt sicher, dass du Terraform korrekt ausführen kannst und die von dir verwendete Version kennst, was später bei Kompatibilitätsfragen wichtig sein kann.

Nach der Installation solltest du die Installation mit folgendem Befehl verifizieren:
```
terraform version
```

Die Versionsprüfung ist wichtig, da Terraform ständig weiterentwickelt wird und manche Funktionalitäten nur in neueren Versionen verfügbar sind.

## Übung 2: Implementierung einer lokalen Ressource

**Zusammenfassung:** In dieser Übung erstellst du deine erste Terraform-Ressource durch Definition einer lokalen Datei, wobei du den vollständigen Terraform-Workflow kennenlernst.

1. Erstelle ein neues Projektverzeichnis und navigiere hinein:
   ```
   mkdir terraform-local-resource
   cd terraform-local-resource
   ```

2. Erstelle eine `main.tf` Konfigurationsdatei mit folgendem Inhalt:

```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.4.0"
    }
  }
}

resource "local_file" "beispiel_datei" {
  content         = "Dies ist eine mit Terraform verwaltete Datei."
  filename        = "${path.module}/terraform-managed.txt"
  file_permission = "0644"
}
```

3. Initialisiere das Terraform-Projekt:
   ```
   terraform init
   ```

4. Führe eine Planungsphase durch, um die beabsichtigten Änderungen zu validieren:
   ```
   terraform plan
   ```

5. Wende die Konfiguration an, um die Ressource zu erstellen:
   ```
   terraform apply
   ```

6. Überprüfe das Ergebnis, indem du die erstellte Datei öffnest:
   ```
   cat terraform-managed.txt
   ```

7. Experimentiere mit einer Änderung des Dateiinhalts im Terraform-Code und führe `terraform apply` erneut aus, um den Änderungsprozess zu beobachten

8. Entferne die erstellten Ressourcen:
   ```
   terraform destroy
   ```

**Warum ist diese Übung wichtig?**

Diese Übung zeigt dir den grundlegenden Terraform-Workflow, der das Herzstück aller Terraform-Projekte bildet:

- **required_providers Block:** Hier definierst du, welche Provider (in diesem Fall "local") Terraform verwenden soll. Die Versions-Angabe stellt sicher, dass dein Code auch bei Updates kompatibel bleibt.

- **terraform init:** Dieser Befehl initialisiert ein Terraform-Projekt. Er lädt die benötigten Provider herunter, richtet die Backend-Konfiguration ein und prüft deine Konfigurationsdateien auf grundlegende Syntax-Fehler. Du musst diesen Befehl nur einmal am Anfang eines Projekts ausführen oder wenn du Provider oder das Backend änderst.

- **terraform plan:** Dieser Schritt ist entscheidend, da Terraform hier einen "Ausführungsplan" erstellt. Es analysiert den aktuellen Zustand der Infrastruktur, vergleicht ihn mit deinem Code und zeigt dir genau, welche Änderungen vorgenommen werden müssen. Dies gibt dir die Chance, mögliche Probleme zu erkennen, bevor sie auftreten.

- **terraform apply:** Hier setzt Terraform den Plan in die Tat um und erstellt, ändert oder löscht Ressourcen gemäß deiner Konfiguration. Terraform verfolgt dabei den Zustand deiner Infrastruktur in einer State-Datei.

- **terraform destroy:** Diese Operation entfernt alle von Terraform verwalteten Ressourcen. In Produktionsumgebungen sollte dieser Befehl mit Vorsicht verwendet werden!

Das Experimentieren mit Änderungen zeigt dir, wie Terraform den Zustand der Infrastruktur verwaltet und nur die notwendigen Änderungen vornimmt, um den gewünschten Zustand zu erreichen - ein Kernprinzip von Infrastructure as Code.

## Übung 3: Implementierung von Variablen und Outputs

**Zusammenfassung:** In dieser Übung lernst du, wie du Konfigurationen durch Variablen parametrisierst und Ergebnisse über Outputs verfügbar machst.

1. Erstelle ein neues Projekt oder erweitere das vorherige:

2. Erstelle eine `variables.tf` Datei für die Definition der Variablen:

```hcl
variable "datei_inhalt" {
  description = "Der Inhalt der zu erstellenden Datei"
  type        = string
  default     = "Standard-Konfigurationsinhalt"
}

variable "datei_name" {
  description = "Name der zu erstellenden Datei"
  type        = string
  default     = "konfiguration.txt"
}
```

3. Erstelle eine `outputs.tf` Datei für die Ausgabedefinitionen:

```hcl
output "datei_pfad" {
  description = "Absoluter Pfad zur erstellten Datei"
  value       = abspath(local_file.konfigurationsdatei.filename)
}

output "datei_hash" {
  description = "MD5-Hash des Dateiinhalts"
  value       = md5(var.datei_inhalt)
}
```

4. Aktualisiere die `main.tf` Datei:

```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.4.0"
    }
  }
}

resource "local_file" "konfigurationsdatei" {
  content         = var.datei_inhalt
  filename        = "${path.module}/${var.datei_name}"
  file_permission = "0644"
}
```

5. Initialisiere und wende die Konfiguration an:
   ```
   terraform init
   terraform apply
   ```

6. Teste die Variable mittels Kommandozeile:
   ```
   terraform apply -var="datei_inhalt=Neuer Inhalt über CLI übergeben" -var="datei_name=benutzerdefiniert.txt"
   ```

7. Teste die Variable mittels Umgebungsvariable:
   ```
   export TF_VAR_datei_inhalt="Inhalt über Umgebungsvariable definiert"
   terraform apply
   ```

8. Experimentiere mit einer `.tfvars` Datei. Erstelle `terraform.tfvars`:
   ```
   datei_inhalt = "Inhalt aus der Variablendatei"
   datei_name   = "aus_varfile.txt"
   ```
   
   Und führe aus:
   ```
   terraform apply
   ```

**Warum sind Variablen und Outputs wichtig?**

Variablen und Outputs sind entscheidende Konzepte in Terraform, die deine Infrastrukturkonfiguration flexibler, wiederverwendbarer und benutzerfreundlicher machen:

- **Variablen:**
  - **Wiederverwendbarkeit:** Du kannst denselben Code für verschiedene Umgebungen (Entwicklung, Staging, Produktion) verwenden, indem du nur die Variablenwerte änderst.
  - **Zentralisierung von Werten:** Indem du Werte in Variablen speicherst, musst du Änderungen nur an einer Stelle vornehmen.
  - **Sicherheit:** Sensible Daten können durch verschiedene Methoden (Umgebungsvariablen, verschlüsselte Dateien) übergeben werden, ohne sie im Code zu speichern.

- **Die verschiedenen Methoden zur Variablenübergabe haben unterschiedliche Anwendungsfälle:**
  - **Kommandozeile (-var):** Gut für schnelle Tests oder einmalige Änderungen.
  - **Umgebungsvariablen (TF_VAR_*):** Nützlich für CI/CD-Pipelines und für sensible Daten.
  - **.tfvars Dateien:** Ideal für umgebungsspezifische Konfigurationen, die wiederverwendet werden.

- **Outputs:**
  - **Informationsrückgabe:** Machen wichtige Informationen über die erstellte Infrastruktur zugänglich.
  - **Verkettung von Modulen:** Ermöglichen die Weitergabe von Informationen zwischen verschiedenen Terraform-Modulen.
  - **Dokumentation:** Bieten eine klare Übersicht über die wichtigsten Aspekte der erstellten Infrastruktur.

Diese Konzepte sind grundlegend für die Erstellung wiederverwendbarer und wartbarer Infrastrukturcode.

## Übung 4: AWS Ressourcenbereitstellung

**Zusammenfassung:** In dieser Übung validierst du deine AWS-Konfiguration und erstellst einen konformen S3-Bucket mit aktuellen Best Practices für Sicherheit und Compliance.

> **Wichtiger Hinweis:** Für die Arbeit mit AWS benötigst du eine korrekt konfigurierte AWS CLI. Eine ausführliche Anleitung zur Einrichtung der AWS CLI findest du in Brians Tutorial: https://github.com/BrianR-Back2Code/Terraform/blob/main/aws_connect_vscode.md

1. Stelle sicher, dass die AWS CLI installiert und konfiguriert ist:
   ```
   aws --version
   aws sts get-caller-identity
   ```

2. Erstelle ein neues Projekt für die AWS-Ressourcen:
   ```
   mkdir terraform-aws-test
   cd terraform-aws-test
   ```

3. Erstelle eine `provider.tf` Datei:

```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

provider "aws" {
  region = "eu-central-1"  # Anpassen an deine bevorzugte Region
  
  # Empfohlene Provider-Konfigurationen für Produktionsumgebungen
  # default_tags {
  #   tags = {
  #     Environment = "Test"
  #     ManagedBy   = "Terraform"
  #   }
  # }
}
```

4. Erstelle eine `main.tf` Datei für die S3-Ressource:

```hcl
# Eindeutiger Suffix für den Bucket-Namen
resource "random_id" "bucket_suffix" {
  byte_length = 4
}

# S3-Bucket mit aktuellen Best Practices
resource "aws_s3_bucket" "test_bucket" {
  bucket = "terraform-test-${random_id.bucket_suffix.hex}"
}

# Bucket-Eigentum festlegen (empfohlen)
resource "aws_s3_bucket_ownership_controls" "bucket_ownership" {
  bucket = aws_s3_bucket.test_bucket.id
  rule {
    object_ownership = "BucketOwnerEnforced"
  }
}

# Public Access blockieren (sicherheitsempfehlung)
resource "aws_s3_bucket_public_access_block" "block_public_access" {
  bucket = aws_s3_bucket.test_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Server-seitige Verschlüsselung aktivieren
resource "aws_s3_bucket_server_side_encryption_configuration" "bucket_encryption" {
  bucket = aws_s3_bucket.test_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

5. Erstelle eine `outputs.tf` Datei:

```hcl
output "bucket_name" {
  description = "Name des erstellten S3-Buckets"
  value       = aws_s3_bucket.test_bucket.bucket
}

output "bucket_arn" {
  description = "ARN des erstellten S3-Buckets"
  value       = aws_s3_bucket.test_bucket.arn
}

output "bucket_region" {
  description = "Region des erstellten S3-Buckets"
  value       = aws_s3_bucket.test_bucket.region
}
```

6. Initialisiere das Projekt:
   ```
   terraform init
   ```

7. Validiere die Konfiguration:
   ```
   terraform validate
   ```

8. Führe eine Planung durch:
   ```
   terraform plan
   ```

9. Wende die Konfiguration an:
   ```
   terraform apply
   ```

10. Überprüfe die Erstellung in der AWS Management Console oder über die AWS CLI:
    ```
    aws s3 ls
    ```

11. Bereinige die Ressourcen, um Kosten zu vermeiden:
    ```
    terraform destroy
    ```
12. Wenn du bis hierhin gelesen hast und die Übung erfolgreich durchgeführt hast, dann schreib mir den folgenden Code als Direktnachricht auf Slack ;)
    ```
    X34t5
    ```

**Warum ist dieser Ansatz wichtig?**

Diese Übung führt dich in die Erstellung von echten Cloud-Ressourcen mit Terraform ein und zeigt Best Practices für Sicherheit und Compliance:

- **AWS Provider Konfiguration:**
  - Die Definition der Provider-Version (~> 5.0) stellt sicher, dass du mit einer aktuellen, aber kompatiblen Version arbeitest.
  - Die Region-Angabe ist wichtig, da AWS-Ressourcen immer in einer bestimmten Region erstellt werden.
  - Die auskommentierten Tags zeigen bewährte Methoden für die Ressourcen-Organisation in Produktionsumgebungen.

- **Eindeutige Ressourcennamen:**
  - Der random_id-Provider erzeugt einen zufälligen Suffix, der sicherstellt, dass deine Bucket-Namen global eindeutig sind (ein S3-Bucket-Name muss AWS-weit einzigartig sein).

- **Sicherheitspraktiken für S3:**
  - **Bucket Ownership Controls:** Klarheit über das Eigentum an Objekten im Bucket.
  - **Public Access Block:** Verhindert versehentliche öffentliche Exposition des Buckets und seiner Daten - einer der häufigsten Sicherheitsfehler in AWS.
  - **Verschlüsselung:** Server-seitige Verschlüsselung schützt die Daten im Ruhezustand.

- **Outputs für wichtige Ressourceninformationen:**
  - Der ARN (Amazon Resource Name) ist ein eindeutiger Identifier, der für Berechtigungen und Cross-Service-Referenzen benötigt wird.
  - Name und Region sind grundlegende Informationen, die du für den Zugriff auf den Bucket benötigst.

- **terraform validate:**
  - Dieser Befehl prüft deine Konfiguration auf syntaktische Korrektheit und interne Konsistenz, bevor du Zeit mit der Ausführung verbringst.

- **Aufräumen:**
  - Der destroy-Befehl ist entscheidend, um unerwartete Kosten zu vermeiden, insbesondere bei Cloud-Diensten, die kontinuierlich Gebühren verursachen können.

Dieser Ansatz zeigt, wie du mit Terraform Cloud-Ressourcen nicht nur erstellen, sondern auch entsprechend aktueller Best Practices für Sicherheit und Betrieb konfigurieren kannst.
