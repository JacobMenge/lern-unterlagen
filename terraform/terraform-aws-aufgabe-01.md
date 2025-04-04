# Terraform State Management: Grundlegende Übungen

**Was ist der Terraform State und warum ist er wichtig?**

Der Terraform State ist wie das "Gedächtnis" von Terraform. Er speichert Informationen über alle Ressourcen, die Terraform erstellt oder verwaltet hat. Stell dir den State wie eine Datenbank vor, die folgendes enthält:
- Welche Ressourcen wurden erstellt
- Welche Eigenschaften haben diese Ressourcen
- Wie hängen diese Ressourcen zusammen
- Welche Metadaten werden benötigt, um diese Ressourcen zu verwalten

Ohne den State wüsste Terraform nicht, welche Ressourcen es bereits erstellt hat und was es bei jedem `terraform apply` tun muss. Es ist wie ein "Vorher-Nachher"-Vergleich: Der State zeigt den "Ist-Zustand", während deine Terraform-Konfiguration den "Soll-Zustand" definiert.

In diesen Übungen lernst du Schritt für Schritt, wie Terraform den State verwaltet und wie du damit arbeiten kannst.

**Bei Fragen oder Problemen:** Bitte melde dich jederzeit per Slack oder im Classroom bei mir. Ich helfe dir gerne weiter!

**Voraussetzungen:** Für diese Übungen benötigst du Terraform (Version 1.0.0 oder höher) und für die AWS-spezifischen Übungen eine konfigurierte AWS CLI.

## Übung 1: Lokalen State verstehen

**Zusammenfassung:** In dieser Übung lernst du, wie Terraform den State lokal speichert und wie du seinen Inhalt einsehen kannst.

1. Erstelle ein neues Verzeichnis für diese Übung:
   ```
   mkdir terraform-state-basics
   cd terraform-state-basics
   ```

   > **Hinweis:** Jedes Terraform-Projekt sollte in einem eigenen Verzeichnis liegen, um die State-Dateien sauber zu trennen.

2. Erstelle eine einfache `main.tf` Datei:
   ```hcl
   # wir erstellen eine lokale datei als beispiel
   resource "local_file" "beispiel" {
     # hier kommt der inhalt der datei rein
     content  = "Diese Datei wird von Terraform verwaltet."
     # hier geben wir an wo die datei gespeichert werden soll
     filename = "${path.module}/beispiel.txt"
   }
   ```

   > **Erklärung:** Diese Konfiguration erstellt eine einfache Textdatei. Die `path.module` Variable gibt das aktuelle Verzeichnis an.

3. Initialisiere das Terraform-Projekt:
   ```
   terraform init
   ```

   > **Hinweis:** Wenn du eine Fehlermeldung zu fehlendem Provider erhältst, ergänze folgende Zeilen am Anfang der `main.tf`:
   ```hcl
   terraform {
     required_providers {
       local = {
         source = "hashicorp/local"
         version = "~> 2.4.0"
       }
     }
   }
   ```

4. Wende die Konfiguration an:
   ```
   terraform apply -auto-approve
   ```

   > **Erklärung:** Der `-auto-approve` Parameter überspringt die Bestätigungsabfrage, was für Übungszwecke praktisch ist. In Produktionsumgebungen solltest du diesen Parameter weglassen!

5. Schaue dir den erstellten State an:
   ```
   terraform state list
   ```

   > **Wichtig:** Dieser Befehl listet alle Ressourcen auf, die Terraform im State verfolgt. Dies ist einer der ersten Befehle, die du ausführen solltest, wenn du den Zustand eines Projekts verstehen willst.

6. Zeige die Details der Ressource im State an:
   ```
   terraform state show local_file.beispiel
   ```

   > **Erklärung:** Mit diesem Befehl siehst du alle Details einer spezifischen Ressource im State, inklusive aller Attribute, die Terraform verfolgt.

7. Untersuche die Dateistruktur des Projekts:
   ```
   ls -la
   ```

   Du solltest eine `terraform.tfstate` Datei sehen. Diese Datei ist das Herzstück deines Terraform-Projekts.

8. Öffne die State-Datei mit einem Texteditor:
   ```
   cat terraform.tfstate
   ```

   > **Hinweis:** Die State-Datei ist ein JSON-Dokument. Auch wenn du sie anschauen kannst, solltest du sie niemals manuell bearbeiten - nutze immer die Terraform-Befehle!

**Warum ist der Terraform State wichtig?**

Der Terraform State ist die Brücke zwischen deinem Code und der realen Infrastruktur:

- Er speichert die Zuordnung zwischen deinen Ressourcendefinitionen und den tatsächlichen Ressourcen
- Er ermöglicht Terraform zu erkennen, welche Änderungen notwendig sind
- Er verfolgt Abhängigkeiten zwischen Ressourcen
- Er speichert sensible Daten wie Passwörter und Zugriffsschlüssel

Ohne State müsste Terraform bei jedem Lauf alle Ressourcen neu erstellen, da es keinen Bezug zur existierenden Infrastruktur hätte.

## Übung 2: State-Manipulation mit Terraform-Befehlen

**Zusammenfassung:** In dieser Übung lernst du, wie du den Terraform State mit verschiedenen Befehlen manipulieren kannst.

1. Erweitere deine `main.tf` mit einer weiteren Ressource:
   ```hcl
   # wir erstellen eine lokale datei als beispiel
   resource "local_file" "beispiel" {
     # hier kommt der inhalt der datei rein
     content  = "Diese Datei wird von Terraform verwaltet."
     # hier geben wir an wo die datei gespeichert werden soll
     filename = "${path.module}/beispiel.txt"
   }

   # jetzt machen wir noch eine zweite datei
   resource "local_file" "zweites_beispiel" {
     # bisschen anderer text
     content  = "Dies ist eine zweite Datei."
     # anderer dateiname
     filename = "${path.module}/zweites_beispiel.txt"
   }
   ```

   > **Erklärung:** Wir fügen eine zweite Ressource hinzu, um später zu demonstrieren, wie mehrere Ressourcen im State verwaltet werden.

2. Wende die Konfiguration an:
   ```
   terraform apply -auto-approve
   ```

3. Überprüfe den State:
   ```
   terraform state list
   ```

   > **Hinweis:** Du solltest jetzt beide Ressourcen im State sehen. Dieser Befehl ist sehr hilfreich, um schnell zu überprüfen, welche Ressourcen Terraform verwaltet.

4. Wir werden jetzt simulieren, was passiert, wenn wir eine Ressource umbenennen:

   Ändere die zweite Ressource in `main.tf` wie folgt:
   ```hcl
   # wir erstellen eine lokale datei als beispiel
   resource "local_file" "beispiel" {
     # hier kommt der inhalt der datei rein
     content  = "Diese Datei wird von Terraform verwaltet."
     # hier geben wir an wo die datei gespeichert werden soll
     filename = "${path.module}/beispiel.txt"
   }

   # wir haben nur den namen geändert - von zweites_beispiel zu umbenannt
   # aber der dateiinhalt und pfad bleiben gleich
   resource "local_file" "umbenannt" {
     content  = "Dies ist eine zweite Datei."
     filename = "${path.module}/zweites_beispiel.txt"
   }
   ```

   > **Wichtig:** Wir haben nur den Namen der Ressource in der Konfiguration geändert, nicht den Dateinamen oder Inhalt.

5. Wende die Konfiguration erneut an:
   ```
   terraform apply
   ```
   
   > **Beobachtung:** Terraform wird die alte Ressource löschen und eine neue erstellen, obwohl sich nur der Name geändert hat. Das ist nicht optimal!

6. Stelle die vorherige Version der zweiten Ressource wieder her:
   ```hcl
   # wir erstellen eine lokale datei als beispiel
   resource "local_file" "beispiel" {
     # hier kommt der inhalt der datei rein
     content  = "Diese Datei wird von Terraform verwaltet."
     # hier geben wir an wo die datei gespeichert werden soll
     filename = "${path.module}/beispiel.txt"
   }

   # wir setzen den namen wieder zurück wie er war
   resource "local_file" "zweites_beispiel" {
     content  = "Dies ist eine zweite Datei."
     filename = "${path.module}/zweites_beispiel.txt"
   }
   ```

7. Wende die Änderungen an:
   ```
   terraform apply -auto-approve
   ```

8. **Jetzt kommt der Schlüsselpunkt dieser Übung:**
   Verwende den `terraform mv` Befehl, um die Ressource im State umzubenennen:
   ```
   terraform state mv local_file.zweites_beispiel local_file.umbenannt
   ```

   > **Warum ist das wichtig?** Der `terraform mv` Befehl ändert den Namen im State, ohne die Ressource neu zu erstellen. In Produktionsumgebungen kann das entscheidend sein, um Ausfallzeiten zu vermeiden!

9. Ändere jetzt den Namen in der Konfiguration:
   ```hcl
   # wir erstellen eine lokale datei als beispiel
   resource "local_file" "beispiel" {
     # hier kommt der inhalt der datei rein
     content  = "Diese Datei wird von Terraform verwaltet."
     # hier geben wir an wo die datei gespeichert werden soll
     filename = "${path.module}/beispiel.txt"
   }

   # jetzt können wir den namen in der config ändern 
   # weil wir vorher den state aktualisiert haben
   resource "local_file" "umbenannt" {
     content  = "Dies ist eine zweite Datei."
     filename = "${path.module}/zweites_beispiel.txt"
   }
   ```

10. Wende die Konfiguration erneut an:
    ```
    terraform apply
    ```
    
    > **Beobachtung:** Diesmal sollte keine Ressource neu erstellt werden, da wir den State bereits aktualisiert haben. Die Ausgabe sollte "No changes" zeigen.

11. Probieren wir einen weiteren wichtigen State-Befehl aus:
    ```
    terraform state rm local_file.beispiel
    ```

    > **Hinweis:** Dieser Befehl entfernt die Ressource aus dem State, ohne sie physisch zu löschen. Das ist nützlich, wenn du eine Ressource von der Terraform-Verwaltung ausschließen möchtest.

12. Überprüfe den State und die tatsächlichen Dateien:
    ```
    terraform state list
    ls -la
    ```
    
    > **Beobachtung:** Die Ressource ist nicht mehr im State, aber die Datei existiert noch physisch.

13. Wende die Konfiguration erneut an:
    ```
    terraform apply
    ```
    
    > **Erklärung:** Da die Ressource noch in der Konfiguration, aber nicht mehr im State ist, wird Terraform sie als "neu" betrachten und wieder erstellen.

**Die wichtigsten State-Befehle im Überblick:**

- `terraform state list` - Zeigt alle Ressourcen im State
- `terraform state show RESSOURCE` - Zeigt Details zu einer bestimmten Ressource
- `terraform state mv QUELLE ZIEL` - Benennt Ressourcen um oder verschiebt sie
- `terraform state rm RESSOURCE` - Entfernt eine Ressource aus dem State (löscht sie nicht physisch)
- `terraform state pull` - Gibt den gesamten State als JSON aus
- `terraform state push` - Überschreibt den State (sehr gefährlich, nur für Notfälle!)

Diese Befehle sind unverzichtbare Werkzeuge im Umgang mit komplexeren Terraform-Projekten, besonders wenn Ressourcen umbenannt oder zwischen Modulen verschoben werden müssen.

## Übung 3: Remote State einrichten

**Zusammenfassung:** In dieser Übung richtest du einen Remote State in AWS S3 mit DynamoDB-Locking ein - eine essentielle Grundlage für Teamarbeit mit Terraform.

> **Wichtiger Hinweis:** Für die Arbeit mit AWS benötigst du eine korrekt konfigurierte AWS CLI. Eine ausführliche Anleitung findest du hier: https://github.com/BrianR-Back2Code/Terraform/blob/main/aws_connect_vscode.md

1. Erstelle ein neues Verzeichnis für den State-Bucket:
   ```
   mkdir terraform-remote-state
   cd terraform-remote-state
   ```

   > **Hinweis:** Wir erstellen zuerst die Infrastruktur für den Remote State (S3-Bucket und DynamoDB-Tabelle) und verwenden sie dann in einem separaten Projekt.

2. Erstelle eine `main.tf` Datei für den State-Bucket:
   ```hcl
   provider "aws" {
     region = "eu-central-1"  # passe die region an
   }

   # für eindeutige namen brauchen wir eine zufallszahl
   resource "random_id" "suffix" {
     byte_length = 4
   }

   # hier erstellen wir den s3 bucket für unseren state
   resource "aws_s3_bucket" "terraform_state" {
     bucket = "terraform-state-${random_id.suffix.hex}"
   }

   # wir sollten immer versionierung aktivieren
   # so können wir zu alten versionen zurückkehren wenn was schiefgeht
   resource "aws_s3_bucket_versioning" "versioning" {
     bucket = aws_s3_bucket.terraform_state.id
     versioning_configuration {
       status = "Enabled"
     }
   }

   # verschlüsselung ist wichtig weil im state sensible daten sein können
   resource "aws_s3_bucket_server_side_encryption_configuration" "encryption" {
     bucket = aws_s3_bucket.terraform_state.id
     rule {
       apply_server_side_encryption_by_default {
         sse_algorithm = "AES256"
       }
     }
   }

   # wir wollen auf keinen fall dass der bucket öffentlich ist
   resource "aws_s3_bucket_public_access_block" "public_access" {
     bucket = aws_s3_bucket.terraform_state.id

     block_public_acls       = true
     block_public_policy     = true
     ignore_public_acls      = true
     restrict_public_buckets = true
   }

   # dynamodb tabelle für state locking
   # verhindert dass zwei leute gleichzeitig änderungen machen
   resource "aws_dynamodb_table" "terraform_locks" {
     name         = "terraform-locks-${random_id.suffix.hex}"
     billing_mode = "PAY_PER_REQUEST"  # wir zahlen nur was wir nutzen
     hash_key     = "LockID"  # muss so heißen für terraform

     attribute {
       name = "LockID"
       type = "S"  # string typ
     }
   }

   # wir geben die namen aus damit wir sie später nutzen können
   output "state_bucket_name" {
     value = aws_s3_bucket.terraform_state.bucket
   }

   output "dynamodb_table_name" {
     value = aws_dynamodb_table.terraform_locks.name
   }
   ```

   > **Wichtig:** Der Code oben könnte abhängig von deiner AWS-Provider-Version minimal angepasst werden müssen. Falls du Fehler bekommst, überprüfe die aktuelle AWS-Provider-Dokumentation für S3-Buckets.

3. Initialisiere und wende die Konfiguration an:
   ```
   terraform init
   terraform apply
   ```

   > **Wichtig:** Notiere dir die Ausgaben (Bucket-Name und DynamoDB-Tabelle), die du im nächsten Schritt benötigst!
   > **Fehlerbehebung:** Falls du einen Fehler erhältst, dass der Provider "random" nicht gefunden wurde, ergänze folgende Zeilen am Anfang der `main.tf`:
   ```hcl
   terraform {
     required_providers {
       aws = {
         source = "hashicorp/aws"
       }
       random = {
         source = "hashicorp/random"
       }
     }
   }
   ```

4. Erstelle ein neues Verzeichnis für ein Projekt, das den Remote State verwendet:
   ```
   cd ..
   mkdir remote-state-projekt
   cd remote-state-projekt
   ```

5. Erstelle eine `backend.tf` Datei mit den Werten aus Schritt 3:
   ```hcl
   terraform {
     # hier sagen wir dass unser state in s3 gespeichert werden soll
     backend "s3" {
       bucket         = "DEIN-BUCKET-NAME"  # trag hier deinen bucket namen ein
       key            = "terraform.tfstate"  # pfad zum state im bucket
       region         = "eu-central-1"       # muss die gleiche region sein
       dynamodb_table = "DEIN-DYNAMODB-NAME" # trag hier deine dynamodb tabelle ein
       encrypt        = true                 # verschlüsselung ist wichtig
     }
   }
   ```

   > **Wichtig:** Ersetze DEIN-BUCKET-NAME und DEIN-DYNAMODB-NAME mit den Werten aus der Ausgabe von Schritt 3.
   > **Erklärung:** Der `key`-Parameter gibt den Pfad innerhalb des Buckets an, wo der State gespeichert wird.

6. Erstelle eine einfache `main.tf` Datei:
   ```hcl
   provider "aws" {
     region = "eu-central-1"
   }

   # für eindeutige namen
   resource "random_id" "suffix" {
     byte_length = 4
   }

   # wir erstellen einen einfachen s3 bucket als test
   resource "aws_s3_bucket" "beispiel" {
     bucket = "remote-state-beispiel-${random_id.suffix.hex}"
   }
   ```

7. Initialisiere das Projekt mit dem Remote-Backend:
   ```
   terraform init
   ```

   > **Beobachtung:** Terraform fragt, ob der State migriert werden soll. Bei einem neuen Projekt ist das nicht relevant, aber bei bestehenden Projekten wird so der lokale State in den Remote State übertragen.


8. Wende die Konfiguration an:
   ```
   terraform apply
   ```

   > **Beobachtung:** Terraform erstellt jetzt einen S3-Bucket und speichert den State im Remote-Backend.
   > **Wichtig:** Du kannst überprüfen, dass der State tatsächlich im S3-Bucket gespeichert wird, indem du in der AWS Console nachschaust oder folgenden Befehl ausführst:
   ```
   aws s3 ls s3://DEIN-BUCKET-NAME/
   ```

9. Der entscheidende Test - State-Locking demonstrieren:
   
    a) Starte in einem Terminal einen `terraform apply` Befehl, aber breche ihn noch nicht ab:
      ```
      terraform apply
      ```
      
    b) Öffne ein zweites Terminal, wechsle in dasselbe Verzeichnis und führe aus:
      ```
      terraform apply
      ```
      
   > **Beobachtung:** Der zweite Befehl wartet oder schlägt fehl, da der State durch den ersten Befehl gesperrt ist. Dies ist der DynamoDB-Lock in Aktion und verhindert Konflikte bei Teamarbeit!

10. Schließe die laufenden Terraform-Prozesse und räume auf:
    ```
    terraform destroy
    ```
    
    Wechsle zurück zum State-Bucket-Verzeichnis und räume auch dort auf:
    ```
    cd ../terraform-remote-state
    terraform destroy
    ```

**Warum ist Remote State so wichtig?**

Der Remote State löst mehrere kritische Probleme bei der Arbeit mit Terraform:

1. **Teamarbeit:** Mehrere Entwickler können am selben Projekt arbeiten, ohne lokale State-Dateien austauschen zu müssen.

2. **Sicherheit:** 
   - Verschlüsselung schützt sensible Daten im State
   - Keine State-Dateien auf lokalen Geräten, die verloren gehen können

3. **Nebenläufigkeit:** Das DynamoDB-Locking verhindert, dass mehrere Personen gleichzeitig Änderungen vornehmen und sich gegenseitig überschreiben.

4. **Versionierung:** Frühere Zustände bleiben durch S3-Versionierung erhalten, was bei Fehlern lebensrettend sein kann.

In einem echten Unternehmensumfeld ist Remote State nicht optional, sondern eine absolute Notwendigkeit für professionelles Arbeiten mit Terraform!

## Übung 4: State importieren und exportieren

**Zusammenfassung:** In dieser Übung lernst du, wie du bestehende Ressourcen in den Terraform State importierst und wie du den State für Analysezwecke exportierst.

1. Erstelle ein neues Verzeichnis für diese Übung:
   ```
   mkdir terraform-state-import
   cd terraform-state-import
   ```

   > **Erklärung:** In der Praxis musst du oft bestehende Ressourcen in Terraform übernehmen, die nicht ursprünglich mit Terraform erstellt wurden.

2. Erstelle zuerst manuell eine Ressource, die wir später importieren werden. 
   Hier erstellen wir einen S3-Bucket mit der AWS CLI:
   ```
   aws s3 mb s3://terraform-import-example-$(date +%s)
   ```

   > **Hinweis:** Der `date +%s` Teil fügt einen Zeitstempel hinzu, um einen eindeutigen Bucket-Namen zu gewährleisten.
   > **Wichtig:** Notiere dir den genauen Bucket-Namen, den du siehst!
   > **Fehlerbehebung:** Wenn du eine Fehlermeldung erhältst, dass der Bucket-Name ungültig ist, versuche einen anderen Namen ohne Unterstriche oder Großbuchstaben.

3. Erstelle eine `main.tf` Datei, die diese Ressource in Terraform beschreibt:
   ```hcl
   provider "aws" {
     region = "eu-central-1"  # passe die region an
   }

   # hier beschreiben wir den bucket den wir manuell erstellt haben
   resource "aws_s3_bucket" "importiert" {
     bucket = "DEIN-BUCKET-NAME"  # trag hier den namen deines buckets ein
   }
   ```

   > **Warum ist das wichtig?** Die Ressource muss in der Terraform-Konfiguration exakt so beschrieben sein, wie sie in der realen Welt existiert, damit Terraform sie richtig verwalten kann.

4. Initialisiere das Terraform-Projekt:
   ```
   terraform init
   ```

5. Versuche einen Plan anzuwenden:
   ```
   terraform plan
   ```
   
   > **Beobachtung:** Terraform will den Bucket erstellen, obwohl er bereits existiert. Das liegt daran, dass er nicht im State ist!

6. Jetzt kommt der Import-Befehl zum Einsatz:
   ```
   terraform import aws_s3_bucket.importiert DEIN-BUCKET-NAME
   ```
   
   > **Erklärung:** Dieser Befehl teilt Terraform mit, dass die Ressource bereits existiert und im State verfolgt werden soll. Das Format ist `terraform import RESSOURCE_ADRESSE RESSOURCE_ID`.
   > **Fehlerbehebung:** Wenn der Import fehlschlägt, überprüfe nochmals, ob der S3-Bucket-Name exakt mit dem in der `main.tf` übereinstimmt.

7. Überprüfe den State:
   ```
   terraform state list
   terraform state show aws_s3_bucket.importiert
   ```

   > **Wichtig:** Mit `terraform state show` kannst du alle Details der importierten Ressource sehen. Das hilft dir, die vollständige Konfiguration zu erstellen.

8. Führe jetzt einen Plan aus:
   ```
   terraform plan
   ```
   
   > **Beobachtung:** Jetzt sollte Terraform keine Änderungen mehr anzeigen, da die Ressource bereits im State ist.
   > **Hinweis:** Es könnte sein, dass Terraform trotzdem kleinere Änderungen anzeigt, weil die importierte Ressource möglicherweise Attribute hat, die in deiner Konfiguration nicht definiert sind.

9. Exportiere den State als JSON-Datei:
   ```
   terraform show -json > terraform-state.json
   ```

   > **Anwendungsfall:** Dieser Export ist nützlich für Audits, Dokumentation oder um den State mit eigenen Skripten zu analysieren.

10. Untersuche die JSON-Datei:
    ```
    cat terraform-state.json | grep bucket
    ```

11. Nachdem du fertig bist, räume alles auf:
    ```
    terraform destroy
    ```
    
    > **Wichtig:** Bestätige mit "yes", dass du den Bucket löschen möchtest.
    > **Fehlerbehebung:** Falls der Destroy-Vorgang fehlschlägt, kann es sein, dass der Bucket nicht leer ist. In diesem Fall musst du ihn zuerst über die AWS Console oder CLI leeren.

**Warum sind Import und Export im Terraform-Workflow wichtig?**

- **State-Import:** In der realen Welt beginnt man selten auf der "grünen Wiese". Der Import ermöglicht es dir, bestehende Infrastruktur in Terraform zu integrieren. Dies ist entscheidend für:
  - Migration zu Infrastructure as Code
  - Übernahme von manuell erstellten Ressourcen
  - Integration von Ressourcen aus anderen Systemen

- **State-Export:** Der Export des States in maschinenlesbare Formate bietet dir:
  - Möglichkeiten zur tieferen Analyse der Infrastruktur
  - Erstellung von benutzerdefinierten Reports
  - Integration mit anderen Tools und Systemen

Für echte Produktionsumgebungen sind diese Fähigkeiten nicht optional, sondern essentielle Werkzeuge im Terraform-Toolset!
