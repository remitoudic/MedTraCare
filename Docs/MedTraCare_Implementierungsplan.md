# Implementierungsplan: MedTraCare – Digitales Patientenportal & CRM-Integration

**Erstellt von:** [Ihr Name]  
**Datum:** 20. April 2026  
**Version:** 1.0  
**Projekttyp:** Webbasiertes Patientenportal mit Odoo CRM, DSGVO-konform  

---

## 1. Projektzusammenfassung

Ziel dieses Projektes ist die Entwicklung einer vollständig digitalen, DSGVO-konformen Lösung für MedTraCare, die:

- Patienten eine strukturierte Möglichkeit bietet, ihre Klinikdokumente hochzuladen und Daten zu erfassen
- alle Patientenvorgänge automatisch in ein zentrales Odoo-CRM überträgt
- Mitarbeiter durch einen klaren, nachvollziehbaren Workflow unterstützt
- Treueprogramme (Finanzierung, Cashback, Kreditkarte) abwickelt
- automatisierte, rechtssichere Kommunikation mit dem Patienten sicherstellt
- jederzeit erweiterbar ist (API-Anbindungen, Partner-Schnittstellen)

Die gesamte Infrastruktur wird auf einem **IONOS VPS (Ubuntu)** betrieben und entspricht vollständig der **DSGVO**.

---

## 2. Gesamtarchitektur

```mermaid
graph TB
    subgraph FE["Öffentliches Frontend"]
        A["Patient"] --- A1["Formular"]
        A --- A2["Upload"]
        A --- A3["DSGVO"]
    end

    subgraph SRV["IONOS VPS Ubuntu Deutschland"]
        B["Nginx Proxy + SSL"]
        C["Backend API FastAPI"]
        D["Odoo 17 CRM"]
        E["PostgreSQL 15"]
        F["Filestore verschlüsselt"]
        G["E-Mail Brevo"]
    end

    subgraph EXT["Externe Partner"]
        H["Partnerkliniken"]
        I["Versicherung"]
        J["FastTrack"]
        K["Finanzierung"]
        L["Kreditkarten"]
    end

    A --> B --> C --> D --> E
    C --> F
    D --> G --> A
    D <--> H
    D -.-> I
    D -.-> J
    D --> K
    D --> L
```

---

## 3. Patient Journey – Prozessablauf (7 Phasen)

```mermaid
graph TD
    subgraph PHASE1["PHASE 1 – Datenerfassung"]
        P1A["Patient öffnet Portal"]
        P1B["Formular ausfüllen"]
        P1C["Dokumente hochladen"]
        P1D["DSGVO aktiv bestätigen"]
        P1E["Backend: Validierung + Virenscan"]
        P1F["Odoo: Lead erstellen"]
        P1A --> P1B --> P1C --> P1D --> P1E --> P1F
    end

    subgraph PHASE2["PHASE 2 – Vollständigkeitsprüfung"]
        P2A["Mitarbeiter prüft Daten"]
        P2B{"Vollständig?"}
        P2C["E-Mail: Optionsauswahl A/B/C"]
        P2D["Nachfrage an Patient"]
        P2A --> P2B
        P2B -->|Ja| P2C
        P2B -->|Nein| P2D
    end

    subgraph PHASE3["PHASE 3 – Treueprogramm"]
        P3A["Patient wählt Option"]
        P3B["A: 0% Finanzierung"]
        P3C["B: 5% Banküberweisung"]
        P3D["C: 10% Kreditkarte"]
        P3A --> P3B
        P3A --> P3C
        P3A --> P3D
    end

    subgraph PHASE4["PHASE 4 – Zusatzleistungen"]
        P4A["Mitarbeiter bucht Services"]
        P4B["Versicherung / FastTrack / Lounge"]
        P4A --> P4B
    end

    subgraph PHASE5["PHASE 5 – Zahlung"]
        P5A["Klinik sendet Zahlungslink"]
        P5B["Link an Patient senden"]
        P5C["Patient zahlt direkt an Klinik"]
        P5A --> P5B --> P5C
    end

    subgraph PHASE6["PHASE 6 – Bestätigung"]
        P6A["Zahlung bestätigt"]
        P6B["Buchungsbestätigung + Anhänge"]
        P6A --> P6B
    end

    subgraph PHASE7["PHASE 7 – Abschluss"]
        P7A["Provision an MedTraCare"]
        P7B["Treuepräme auszahlen"]
        P7C["Abschlussmail + Bewertungslink"]
        P7A --> P7B --> P7C
    end

    P1F --> P2A
    P2C --> P3A
    P3B --> P4A
    P3C --> P4A
    P3D --> P4A
    P4B --> P5A
    P5C --> P6A
    P6B --> P7A
```

---

## 3.2 Detaillierter Sequenzieller Datenfluss

Dieses Diagramm zeigt die technischen Interaktionen und Systemaufrufe zwischen den Akteuren in den verschiedenen Phasen des Prozesses.

```mermaid
sequenceDiagram
    autonumber
    actor P as Patient
    participant PT as Patientenportal
    participant API as Backend API
    participant CRM as Odoo CRM
    participant Mail as E-Mail-Dienst
    participant Staff as Mitarbeiter
    participant Klinik as Partnerklinik

    Note over P, Klinik: PHASE 1 - Datenerfassung

    P->>PT: Öffnet Website & füllt Formular aus
    P->>PT: Lädt Klinikangebot hoch
    P->>PT: Lädt Behandlungsvertrag hoch
    P->>PT: Bestätigt DSGVO-Einwilligung (aktiv, Checkbox)
    PT->>API: POST /api/patient/submit (Formulardaten + Dateien)
    API->>API: Eingabevalidierung & Virenscan der Dateien
    API->>CRM: Erstellt Lead / Patientenvorgang via Odoo API
    API->>API: Speichert Dokumente im verschlüsselten Filestore
    CRM-->>P: Bestätigungsseite: "Ihre Daten wurden empfangen"

    Note over P, Klinik: PHASE 2 - Vollständigkeitsprüfung

    CRM->>Staff: Benachrichtigung: Neuer Patientenvorgang
    Staff->>CRM: Prüft Datenvollständigkeit im CRM
    
    alt Daten vollständig
        Staff->>CRM: Markiert Vorgang als "Vollständig"
        CRM->>Mail: Trigger: Automatische E-Mail an Patient
        Mail-->>P: "Wir buchen Ihre Zusatzleistungen" + Optionsauswahl
    else Daten unvollständig
        Staff->>CRM: Markiert fehlende Felder
        CRM->>Mail: Nachfrage-E-Mail an Patient
        Mail-->>P: Nachfrage-E-Mail
    end

    Note over P, Klinik: PHASE 3 - Treueprogramm-Auswahl

    P->>PT: Wählt Option A, B oder C
    PT->>CRM: Speichert Treueprogramm-Auswahl im CRM

    Note over P, Klinik: PHASE 4 - Zusatzleistungen buchen

    Staff->>CRM: Markiert gewählte Option (A/B/C)
    Staff->>Klinik: Bucht Versicherung, FastTrack, Lounge etc.
    Staff->>CRM: Speichert Partner-Bestätigungen als Anhang

    Note over P, Klinik: PHASE 5 - Zahlungslink

    Klinik->>Staff: Sendet externen Zahlungslink
    Staff->>CRM: Trägt Zahlungslink ein
    CRM->>Mail: Automatische E-Mail mit Zahlungslink
    Mail-->>P: E-Mail mit klinikeigenem Zahlungslink

    Note over P, Klinik: PHASE 6 - Buchungsbestätigung

    Klinik->>Staff: Bestätigt Zahlungseingang
    Staff->>CRM: Setzt Status auf "Zahlung eingegangen"
    CRM->>Mail: Trigger: Finale Buchungsbestätigung
    Mail-->>P: Buchungsbestätigung + alle Anhänge

    Note over P, Klinik: PHASE 7 - Nach der Behandlung

    Klinik->>Staff: Überweist Provision an MedTraCare
    Staff->>CRM: Prüft Unterlagen, markiert "Abgeschlossen"
    
    alt Option B gewählt
        Staff->>Mail: Auszahlung 5% Treuepräme per Banküberweisung
    else Option C gewählt
        Staff->>Mail: Ausgabe Kreditkarte mit 10% Startguthaben
    end
    
    Staff->>Mail: Abschlussmail + Bewertungslink (Google / Trustpilot)
    Mail-->>P: Abschlussmail
```

---

## 4. Technischer Stack

| Komponente | Technologie | Begründung |
|---|---|---|
| **Webserver** | Nginx | Reverse Proxy, SSL-Terminierung, hohe Performance |
| **SSL-Zertifikat** | Let's Encrypt (Certbot) | Kostenfrei, automatisch erneuert, DSGVO-konform |
| **Backend API** | Python (FastAPI) | Schnell, modern, ideal für Odoo-Integration |
| **CRM** | Odoo 17 Community | Open Source, vollständig anpassbar, DSGVO-konform |
| **Datenbank** | PostgreSQL 15 | Enterprise-grade, von Odoo nativ verwendet |
| **Datei-Speicher** | Odoo Filestore (verschlüsselt) | Dokumente direkt im CRM verknüpft |
| **E-Mail** | Brevo / SendGrid + SMTP | Zuverlässige Zustellung, DSGVO-konform |
| **Server** | IONOS VPS (Ubuntu 22.04 LTS) | Rechenzentrum in Deutschland, DSGVO-konform |
| **Backup** | Tägliches Backup via IONOS Snapshots + pg_dump | Datensicherheit |
| **Monitoring** | Uptime Kuma oder Netdata | Systemüberwachung, Alarmierung |

---

## 5. Detaillierte Implementierungsschritte

### Phase 0 – Infrastruktur & Setup (2 Tage)

- [ ] IONOS VPS bestellen und Ubuntu 22.04 LTS installieren
- [ ] Server-Hardening (SSH-Key-Only, Fail2Ban, UFW Firewall)
- [ ] Nginx installieren und konfigurieren
- [ ] SSL-Zertifikat via Let's Encrypt einrichten (automatische Erneuerung)
- [ ] PostgreSQL 15 installieren, Benutzer und Datenbank für Odoo anlegen
- [ ] Odoo 17 Community installieren und Grundkonfiguration vornehmen
- [ ] Backup-Strategie einrichten (täglich, Off-Site via IONOS Object Storage)

---

### Phase 1 – Odoo CRM Konfiguration (2 Tage)

#### 1.1 Benutzerdefiniertes Patientenmodell

Ein erweitertes CRM-Lead-Modell wird in einem benutzerdefinierten Odoo-Modul (`medtracare_patient`) implementiert:

| Feld | Typ | Beschreibung |
|---|---|---|
| `patient_first_name` | Char | Vorname (Pflicht) |
| `patient_last_name` | Char | Nachname (Pflicht) |
| `patient_address` | Text | Anschrift (Pflicht) |
| `patient_email` | Char | E-Mail (Pflicht, validiert) |
| `patient_phone` | Char | Telefon (Pflicht) |
| `clinic_id` | Many2one → res.partner | Behandelnde Klinik (Dropdown, nur Partnerkliniken) |
| `treatment_ids` | Many2many → med.treatment | Behandlungsarten (max. 3 auswählbar) |
| `surgery_date` | Date | OP-Datum (Pflicht) |
| `clinic_offer` | Binary / Attachment | Upload Klinikangebot |
| `treatment_contract` | Binary / Attachment | Upload Behandlungsvertrag |
| `gdpr_consent` | Boolean | DSGVO-Einwilligung (Pflicht, protokolliert) |
| `gdpr_consent_date` | Datetime | Zeitstempel der Einwilligung |
| `loyalty_choice` | Selection (A/B/C) | Treueprogramm-Auswahl |
| `payment_link` | Char | Externer Zahlungslink der Klinik |
| `passport_data` | Text | Reispassdaten (verschlüsselt) |
| `flight_data` | Text | Flugdaten (für FastTrack) |
| `stage_id` | Many2one → crm.stage | Status-Workflow |

#### 1.2 CRM Status-Workflow (Kanban-Phasen)

```
[Neu eingegangen] → [In Prüfung] → [Vollständig] → [Zusatzleistungen gebucht]
→ [Zahlungslink gesendet] → [Zahlung eingegangen] → [Behandlung abgeschlossen]
→ [Treueprogramm ausgezahlt] → [Abgeschlossen]
```

#### 1.3 Automatisierte E-Mail-Vorlagen in Odoo

1. **Eingangsbestätigung** – "Ihre Daten wurden empfangen"
2. **Zusatzleistungs-E-Mail** – Mit Auswahloptionen A/B/C
3. **Zahlungslink-E-Mail** – Mit externem Klinik-Zahlungslink
4. **Buchungsbestätigung** – Mit allen Anhängen
5. **Abschluss-E-Mail** – Treueprogramm + Google/Trustpilot Bewertungslink
6. **Nachfrage-E-Mail** – Bei unvollständigen Daten

---

### Phase 2 – Patientenportal (Frontend) (zu verhandeln)

#### 2.1 Formular-Seite (Schritt-für-Schritt, mehrstufig)

**Stufe 1:** Persönliche Daten (Vorname, Nachname, Anschrift, E-Mail, Telefon)  
**Stufe 2:** Behandlungsdaten (Klinik-Dropdown, Behandlungsarten max. 3, OP-Datum)  
**Stufe 3:** Dokument-Upload (Klinikangebot + Behandlungsvertrag, max. 10 MB)  
**Stufe 4:** DSGVO & Bestätigung (aktive Checkbox, nicht vorausgefüllt)

#### 2.2 UX / Design-Anforderungen

- Mobil-optimiert (Responsive Design)
- Klare Fehlerhinweise bei Pflichtfeldern
- Fortschrittsbalken (Schritt 1 von 4)
- Ladeanimation beim Absenden
- Erfolgsmeldung nach Formular-Übermittlung

---

### Phase 3 – Backend API & Odoo-Integration (3 Tage)

#### 3.1 REST API Endpunkte

| Endpunkt | Methode | Beschreibung |
|---|---|---|
| `/api/patient/submit` | POST | Übermittelt Patientendaten + Dateien |
| `/api/patient/clinics` | GET | Gibt Liste der Partnerkliniken zurück |
| `/api/patient/treatments` | GET | Gibt Liste der Behandlungsarten zurück |
| `/api/patient/loyalty` | POST | Speichert Treueprogramm-Wahl |
| `/api/patient/status/{id}` | GET | (Optional) Patientenstatus abfragen |

#### 3.2 Datei-Upload-Sicherheit

- Virenscan aller hochgeladenen Dateien (ClamAV Integration)
- Strikte MIME-Type-Prüfung (nur PDF, JPG, PNG)
- Maximale Dateigröße: 10 MB
- Dateien werden nur in Odoo Filestore gespeichert
- Zugriffsschutz: Dateien nur für authentifizierte Mitarbeiter einsehbar

---

### Phase 4 – DSGVO-Compliance (Durchgängig)

| Maßnahme | Implementierung |
|---|---|
| **Einwilligung** | Aktive Checkbox, konserviert mit Zeitstempel und IP-Adresse |
| **Datenminimierung** | Nur notwendige Felder werden erfasst |
| **Verschlüsselung** | HTTPS (TLS 1.3), verschlüsselter Datei-Speicher |
| **Serverstandort** | IONOS Rechenzentrum Deutschland (Frankfurt / Karlsruhe) |
| **Datenlöschung** | Löschkonzept: Inaktive Datensätze nach gesetzlicher Frist löschbar |
| **Zugriffskontrolle** | Rollenbasierter Zugriff in Odoo (nur autorisierte Mitarbeiter) |
| **Audit-Log** | Vollständige Nachverfolgung aller Statusänderungen in Odoo Chatter |
| **Datenportabilität** | Export von Patientendaten auf Anfrage möglich |
| **Auftragsverarbeitung** | AVV-Vertrag mit IONOS und allen API-Partnern |
| **Datenschutzerklärung** | Rechtskonforme Datenschutzerklärung im Portal verlinkt |

---

### Phase 5 – Treueprogramm-Logik (Woche 4)

#### Option A – Finanzierung mit 0% Zinsen
- Weiterleitung zu Finanzierungspartner-Antragsstrecke per E-Mail-Link

#### Option B – 5% Treuepräme per Banküberweisung
- Nach Behandlungsabschluss und Provisionseingang der Klinik

#### Option C – 10% Startguthaben auf Kreditkarte
- Prepaid-Kreditkarte über Kreditkarten-Partner

---

### Phase 6 – Erweiterbarkeit (API-Schnittstellen)

| Partner | Schnittstelle | Status |
|---|---|---|
| Versicherungspartner | REST API | Geplant |
| FastTrack-Anbieter | REST API | Geplant |
| Flughafen-Lounge | REST API | Geplant |
| Finanzierungspartner | REST API / OAuth2 | Geplant |
| Kreditkarten-Anbieter | REST API | Geplant |
| Google Reviews | REST API | Geplant |
| Trustpilot | REST API | Geplant |

---

## 6. Projektplan & Zeitstrahl

```mermaid
gantt
    title MedTraCare Projektplan
    dateFormat  YYYY-MM-DD
    axisFormat  %d. %b

    section Phase 0
    IONOS VPS Setup          :p0a, 2026-05-05, 1d
    Nginx SSL PostgreSQL     :p0b, after p0a, 1d
    Odoo 17 Installation     :p0c, after p0b, 1d

    section Phase 1
    Patientenmodul           :p1a, after p0c, 1d
    Workflow & Automationen  :p1b, after p1a, 1d

    section Phase 2
    Frontend (zu verhandeln) :p2a, after p0c, 10d

    section Phase 3
    REST API & Integration   :p3a, after p2a, 3d
    ClamAV Sicherheit        :p3b, after p3a, 1d

    section Phase 4
    DSGVO Review             :p4a, after p3b, 1d
    E2E Tests                :p4b, after p4a, 2d
    Sicherheitsaudit         :p4c, after p4b, 1d

    section Phase 5
    Abnahme & Go-Live        :p5a, after p4c, 3d
    Go-Live                  :milestone, after p5a, 0d
```

**Gesamtdauer: ca. 4–6 Wochen** (stark abhängig von Phase 2)

---

## 7. Liefergegenstände

| # | Liefergegenstand | Beschreibung |
|---|---|---|
| 1 | Produktive Odoo-Instanz | Vollständig konfiguriert auf IONOS VPS |
| 2 | Benutzerdefiniertes Odoo-Modul | medtracare_patient |
| 3 | Patientenportal | Responsives Webformular |
| 4 | Backend API | FastAPI-Service |
| 5 | E-Mail-Vorlagen | 6 automatisierte Vorlagen |
| 6 | DSGVO-Dokumentation | Verarbeitungsverzeichnis |
| 7 | Systemdokumentation | Technische und Benutzer-Doku |
| 8 | Mitarbeiter-Schulung | 2-stündige Online-Schulung |
| 9 | Support-Phase | 4 Wochen Nachbetreuung |

---

## 8. Datensicherheit & Backup

- **RTO:** < 2 Stunden
- **RPO:** < 24 Stunden
- Alle Backups auf deutschen IONOS-Servern

---

## 9. Rollen & Zugriffsrechte

| Rolle | Zugriffsrechte |
|---|---|
| Patient (extern) | Nur Formular-Übermittlung |
| Mitarbeiter | Eigene Vorgänge, E-Mails, Status |
| Manager | Voller Zugriff, Berichte |
| Administrator | Systemkonfiguration |

---

## 10. Mitwirkungspflichten des Auftraggebers

- [ ] Zugangsdaten zum IONOS VPS
- [ ] Liste aller Partnerkliniken
- [ ] Liste aller Behandlungsarten
- [ ] Logo, Farben und Design-Vorgaben
- [ ] Datenschutzerklärung und AGB (vom Rechtsanwalt)
- [ ] Kontakt zu Finanzierungspartner / Kreditkarten-Anbieter
- [ ] E-Mail-Texte für Automationen

---

## 11. Risiken & Gegenmaßnahmen

| Risiko | Wahrscheinlichkeit | Maßnahme |
|---|---|---|
| Verzögerung durch Drittanbieter-APIs | Mittel | Manuelle Fallback-Prozesse |
| Serverausfall | Gering | Monitoring + IONOS SLA |
| Missbrauch Upload-Formular | Mittel | reCAPTCHA, Virenscan, Rate Limiting |
| Scope Creep | Mittel | Change-Management-Verfahren |

---

## 12. Kostenübersicht

| Position | Monatlich |
|---|---|
| IONOS VPS | ~20–30 € |
| SSL (Let's Encrypt) | 0 € |
| Odoo 17 Community | 0 € |
| E-Mail (Brevo) | 0–15 € |
| Backup (Object Storage) | ~5 € |
| **Gesamt laufend** | **ca. 25–50 €/Monat** |

---

## 13. Warum ich der richtige Partner bin

- **Odoo-Expertise:** Mehrjährige Erfahrung in Customizing und Modulentwicklung
- **DSGVO-Kenntnisse:** Datenschutzkonforme Webapplikationen für Deutschland
- **Full-Stack:** Von Backend-API bis responsivem Frontend
- **Linux-Administration:** IONOS/Ubuntu, Nginx, SSL, Backup
- **Kommunikation:** Regelmäßige Updates, transparente Arbeitsweise

---

## 14. Nächste Schritte

1. **Kick-off-Gespräch** – Offene Fragen klären
2. **Anforderungs-Workshop** – Kliniken, Behandlungsarten, E-Mail-Texte definieren
3. **Vertragsabschluss** – Klarer Leistungsumfang
4. **Server-Bestellung** – IONOS VPS
5. **Projektstart** – Phase 0 beginnt

---

*Kontakt:* **[Ihr Name]** | [Ihre E-Mail] | [Ihr Malt-Profil-Link]
