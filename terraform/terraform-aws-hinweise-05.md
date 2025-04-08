# Hinweise zur Aufgabe: Günstige Web-App in AWS

Falls ihr bei der Architektur-Idee etwas Orientierung braucht, hier ein paar mögliche Ansätze und Services, die ihr euch anschauen könnt:

---

## Frontend

- **Amazon S3** zum statischen Hosting von HTML/CSS/JS
- **CloudFront** als Content Delivery Network (CDN) zur Beschleunigung und globalen Auslieferung
- **AWS Amplify** als Komplettlösung für Hosting, CI/CD und ggf. sogar Backend-Funktionen

---

## Backend

- **AWS Lambda** (serverless Funktionen)
- **API Gateway** zur Anbindung eurer Lambda-Funktionen über HTTP-Endpoints
- **AWS Amplify Backend** (GraphQL- oder REST-API)

---

## Datenbank

- **Amazon DynamoDB** (NoSQL, günstig bei wenig Last, On-Demand-Modell)
- Alternativ: **Amazon RDS (Free Tier)**, z. B. für MySQL oder PostgreSQL

---

## Infrastruktur mit Terraform

Wenn ihr an die Terraform-Umsetzung denkt, könnten folgende Ressourcen interessant sein:

- `aws_s3_bucket`
- `aws_lambda_function`
- `aws_api_gateway_rest_api`
- `aws_dynamodb_table`
- `aws_amplify_app` (eher optional und fortgeschritten)

---

💬 Nutzt diese Hinweise nur als Inspiration – eure eigene Lösung muss nicht alle Services verwenden. Es geht darum, einen gangbaren und günstigen Weg zu finden.
