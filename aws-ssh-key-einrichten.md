# Anhang: Einrichtung des SSH-Schlüssels für AWS

Für die AWS-Verbindung in dieser Übung wird ein SSH-Schlüsselpaar benötigt. Diese Anleitung führt dich durch die Erstellung und Einrichtung eines solchen Schlüsselpaars.

## A. Erstellen eines neuen SSH-Schlüsselpaars in AWS

1. **Melde dich in der AWS Management Console an**:
   - Gehe zu [https://console.aws.amazon.com/](https://sandboxes.techstarter.de/)
   - Melde dich mit deinen AWS-Zugangsdaten an

2. **Navigiere zum EC2-Dashboard**:
   - Suche nach "EC2" im Suchfeld oder finde es unter "Services"
   - Wähle "EC2" aus

3. **Erstelle ein neues Schlüsselpaar**:
   - Klicke im linken Menü auf "Schlüsselpaare" (unter "Netzwerk & Sicherheit")
   - Klicke auf "Schlüsselpaar erstellen"
   - Gib einen Namen für dein Schlüsselpaar ein (z.B. "terraform-ansible-demo")
   - Wähle als Schlüsselpaartyp "RSA"
   - Wähle als Dateiformat ".pem" für Linux/macOS/WSL oder ".ppk" für PuTTY (Windows)
   - Klicke auf "Erstellen"
   - Die private Schlüsseldatei wird automatisch heruntergeladen - bewahre sie sicher auf!

## B. Speichern des Schlüssels in der richtigen Umgebung

### Für Linux oder macOS:

1. **Verschiebe den Schlüssel in das SSH-Verzeichnis**:
   ```bash
   mkdir -p ~/.ssh
   mv ~/Downloads/terraform-ansible-demo.pem ~/.ssh/
   ```

2. **Setze die korrekten Berechtigungen**:
   ```bash
   chmod 600 ~/.ssh/terraform-ansible-demo.pem
   ```

### Für WSL:

1. **Option 1: Schlüssel direkt in WSL speichern** (empfohlen):
   ```bash
   # Erstelle das SSH-Verzeichnis, falls es nicht existiert
   mkdir -p ~/.ssh
   
   # Verschiebe den Schlüssel aus dem Windows-Downloads-Ordner nach WSL
   cp /mnt/c/Users/DeinWindowsNutzer/Downloads/terraform-ansible-demo.pem ~/.ssh/
   
   # Setze die korrekten Berechtigungen
   chmod 600 ~/.ssh/terraform-ansible-demo.pem
   ```

2. **Option 2: Schlüssel im Windows-Dateisystem verwenden**:
   ```bash
   # Setze Berechtigungen (funktioniert möglicherweise eingeschränkt für Windows-Dateien)
   chmod 600 /mnt/c/Users/DeinWindowsNutzer/Downloads/terraform-ansible-demo.pem
   ```

## c. Überprüfung der SSH-Konfiguration

Vergewissere dich, dass:
1. Der Schlüsselname in `key_name` exakt mit dem Namen des in AWS erstellten Schlüssels übereinstimmt
2. Der Pfad in `private_key_path` zum tatsächlichen Speicherort der Schlüsseldatei führt
3. Die Berechtigungen der Schlüsseldatei korrekt sind (600)

Mit dieser Konfiguration kann Terraform den Schlüssel für die EC2-Instanz verwenden, und Ansible kann sich über SSH mit der Instanz verbinden.
