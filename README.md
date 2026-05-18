# How To Run [Slidev](https://github.com/slidevjs/slidev)!

To start the slide show:

- `npm install`
- `npm run dev`
- visit <http://localhost:3030>

Edit the [slides.md](./slides.md) to see the changes.

Learn more about Slidev at the [documentation](https://sli.dev/).

---

# Dokumentation: Qantas Supply Chain Breach 2025

Dieses Dokument dient als fachlicher Hintergrund für die Analyse des Cyber-Angriffs auf Qantas Airways im Juli 2025. Es deckt die strategischen, taktischen und technischen Aspekte des Vorfalls ab.

## 1. Strategischer Kontext: Der Supply Chain Angriff

Im modernen Cybersecurity-Umfeld ist die **Supply Chain (Lieferkette)** oft das schwächste Glied. Organisationen wie Qantas verfügen über extrem gehärtete interne Netzwerke. Angreifer wählen daher den Weg über Drittanbieter (hier: ein externes Callcenter), die über legitime, aber oft schlechter überwachte Zugänge zum Zielnetzwerk verfügen.

* **Implicit Trust:** Das Kernproblem war das blinde Vertrauen in die Identität des Partners. Sobald ein Account des Drittanbieters kompromittiert war, wurde dessen Aktivität innerhalb der Qantas-API-Umgebung nicht ausreichend als anomal eingestuft.

## 2. Phase 1: Initial Infiltration (Social Engineering)

Der Angriff begann nicht mit einem Software-Exploit, sondern mit der Manipulation des Faktors Mensch.

### Vishing & Credential Harvesting

Die Gruppe (mutmaßlich **Scattered Spider**) nutzte **Vishing** (Voice Phishing). Dabei gaben sich die Angreifer am Telefon gegenüber Mitarbeitern des philippinischen Callcenters als IT-Support von Qantas aus. Durch psychologischen Druck und Fachjargon brachten sie die Mitarbeiter dazu, ihre Zugangsdaten auf einer präparierten Login-Seite einzugeben (**Credential Harvesting**).

### MFA-Bypass Techniken

Obwohl Konten durch Mehrfaktor-Authentifizierung geschützt waren, wurden zwei Methoden zur Umgehung genutzt:

1. **MFA Fatigue:** Der Mitarbeiter wird mit Push-Benachrichtigungen überflutet, bis er eine davon – oft aus Frustration oder Unachtsamkeit – bestätigt.
2. **AiTM (Adversary-in-the-Middle):** Die Angreifer schalteten einen Proxy-Server zwischen Mitarbeiter und das echte Login-Portal. Dadurch konnten sie neben dem Passwort auch das **Session-Token** abfangen. Ein Session-Token ist ein digitaler Ausweis, der beweist, dass die MFA bereits erfolgreich abgeschlossen wurde. Mit diesem Token konnte der Angreifer die Identität des Opfers ohne erneute MFA-Abfrage übernehmen.

## 3. Phase 2: Technischer Exploit (API Security)

Sobald der Zugriff auf die CRM-Schnittstellen (Customer Relationship Management) des Callcenters etabliert war, identifizierten die Angreifer eine kritische logische Schwachstelle in der API-Architektur.

### BOLA – Broken Object Level Authorization (OWASP API #1)

Dies war der entscheidende Hebel des Angriffs. **BOLA** tritt auf, wenn ein System zwar die **Authentifizierung** (Wer bist du?) prüft, aber die **Autorisierung** (Darfst du das?) auf Datensatz-Ebene vernachlässigt.

* **Mechanik:** Der Angreifer konnte die ID eines Kunden in der API-Anfrage (z.B. `GET /api/v1/customer/1001`) einfach in `1002`, `1003` usw. ändern (**ID Enumeration**).
* **Das Versagen:** Der Server prüfte nicht, ob der Callcenter-Mitarbeiter gerade tatsächlich ein Ticket für diesen spezifischen Kunden bearbeitete. Er lieferte die Daten einfach aus, weil die Anfrage von einem „vertrauenswürdigen“ Partner-Account kam.

### Fehlendes Rate Limiting & Mass Assignment

Normalerweise sollten Sicherheitsmechanismen wie **Rate Limiting** verhindern, dass ein einzelner Account Millionen von Abfragen in kurzer Zeit stellt. Im Fall Qantas fehlte diese Drosselung für Partner-Schnittstellen, was die **Exfiltration** von 6 Millionen Datensätzen in Batches ermöglichte.

## 4. Phase 3: Auswirkungen & Daten-Exfiltration

Die entwendeten Daten (PII - Personally Identifiable Information) umfassen Namen, E-Mails, Telefonnummern und Vielflieger-Informationen.

* **Identity Starter Kits:** Diese Datenkombination ist wertvoll für nachgelagerte Angriffe. Mit Geburtsdatum und vollem Namen können Angreifer bei Banken oder anderen Dienstleistern Identitätsprüfungen (Knowledge-Based Authentication) umgehen.
* **Regulatorik:** Der Vorfall löste Untersuchungen unter dem **Australian Privacy Act** aus. In der Cybersecurity-Vorlesung ist hier der Hinweis wichtig, dass Unternehmen für die Sicherheit ihrer Daten haften, auch wenn sie diese an Dritte auslagern.

## 5. Prävention & Lessons Learned

Der Vorfall erzwingt einen Paradigmenwechsel weg von passiver Sicherheit hin zu einer **Zero-Trust-Architektur**.

### FIDO2 & WebAuthn

Die wichtigste Abwehrmaßnahme gegen das hier genutzte Social Engineering ist **FIDO2**. Im Gegensatz zu SMS oder Push-Codes nutzt FIDO2 Kryptografie mit öffentlichem Schlüssel. Die Authentifizierung ist an die Hardware (z.B. einen YubiKey) und die spezifische URL gebunden. Ein Diebstahl des Tokens per Phishing oder Vishing ist technisch unmöglich, da der private Schlüssel das Gerät nie verlässt.

### Micro-Segmentation & Least Privilege

* **Micro-Segmentation:** Partnernetzwerke müssen isoliert werden, sodass ein kompromittierter Account keinen Zugriff auf die gesamte Datenbank hat.
* **Least Privilege:** Zugriffsberechtigungen müssen so feingranular sein, dass ein Callcenter-Agent nur die Daten sieht, die für ein aktuell offenes Support-Ticket relevant sind.

---

**Anhang: Fachbegriffe für die Diskussion**

* **Attack Surface:** Die Gesamtheit aller Angriffspunkte (durch Drittanbieter massiv vergrößert).
* **Blast Radius:** Das Ausmaß des Schadens, den ein einzelner kompromittierter Account anrichten kann.
* **UEBA (User and Entity Behavior Analytics):** Systeme, die den massiven Datenabzug durch den Partner-Account als anomal hätten erkennen müssen.
