# Keycloak-Konfiguration für das FORT-Projekt

Diese Anleitung beschreibt die lokale Einrichtung von Keycloak für das Projekt – von Grund auf, auch wenn du Keycloak zum ersten Mal verwendest.

***

## 1. Keycloak starten

Im Projektverzeichnis ausführen:

```bash
cd fort-infra/keycloak
docker compose up
```

Danach ist Keycloak lokal unter folgender Adresse erreichbar:

```text
http://localhost:9000
```

Zum Einloggen in das Adminpanel:
* **Nutzername:** `admin`
* **Passwort:** `admin`

***

## 2. Realm anlegen

Ein *Realm* ist ein abgeschlossener Bereich in Keycloak, der Benutzer, Rollen und Clients verwaltet.
Für dieses Projekt wird ein eigener Realm namens `fort` benötigt.

1. Keycloak im Browser öffnen: `http://localhost:9000`
2. Zu **Manage realms** gehen
3. **Create realm** auswählen
4. Als Namen eintragen:

```text
fort
```

***

## 3. Realm-Rollen anlegen

Rollen steuern, welche Berechtigungen ein Benutzer in der Anwendung hat.
Unter **Realm roles → Create role** folgende drei Rollen anlegen:

| Rollenname | Beschreibung |
|---|---|
| `ADMIN` | Vollzugriff: Benutzer verwalten, alles lesen/schreiben |
| `EDITOR` | Dozenten, Vorlesungen, Zuweisungen lesen/schreiben, aber keine Benutzerverwaltung |
| `USER` | Nur Lesezugriff auf Dozenten, Vorlesungen, Zuweisungen |

> **Wichtig:** Die Rollennamen müssen **exakt so in Großbuchstaben** geschrieben sein,
> da die Anwendung diese Namen direkt vergleicht.

***

## 4. Client anlegen

Ein *Client* repräsentiert die Anwendung (hier das BFF – Backend for Frontend),
die sich bei Keycloak authentifiziert.

1. Zu **Clients** gehen
2. **Create client** auswählen
3. Als **Client ID** eintragen:

```text
fort-bff
```

4. **Next** klicken

### 4.1 Capability config

Folgende Optionen setzen:

| Einstellung | Wert |
|---|---|
| Client authentication | **On** (confidential client) |
| Authorization | Off |
| Standard flow | **On** (für OAuth2 Login) |
| Service accounts roles | **On** (für Admin-API-Zugriff per `client_credentials`) |

Danach **Next** klicken.

### 4.2 Client-URLs konfigurieren

Folgende Werte eintragen:

**Root URL**

```text
http://localhost:8084
```

**Home URL**

```text
http://localhost:8084
```

**Valid redirect URIs**

```text
http://localhost:8084/login/oauth2/code/fort
```

**Valid post logout redirect URIs**

```text
http://localhost:8084
http://localhost:5173/login
http://localhost:5173/login-blocked
```

**Web Origins**

Für lokales Testen:

```text
http://localhost:8084
```

Wenn ein separates Frontend verwendet wird, stattdessen den Frontend-Port eintragen, zum Beispiel:

```text
http://localhost:5173
```

***

## 5. Client Secret kopieren

1. Im angelegten Client auf **Credentials** gehen
2. Das **Client Secret** kopieren
3. In die `.env`-Datei des BFFs einfügen

### Für IntelliJ-Nutzer

In manchen Fällen erkennt IntelliJ die `.env`-Datei nicht korrekt, was zu einem falschen
oder fehlenden Secret führt, sodass der Login fehlschlägt.

Als Lösung lässt sich das Secret direkt in der Spring-Konfiguration des BFFs in IntelliJ setzen:

1. Klicke unter **Run** auf die Option **Edit Configurations...**


2. Wähle in der linken Seitenleiste unter **Spring Boot** die Konfiguration **FortBffApplication**


3. Ggf. muss unter **Modify Options** die Sichtbarkeit der Option **Environment Variables** aktiviert werden


4. Das Secret lässt sich in das neue Feld einfügen:
   ```text
   OIDC_CLIENT_SECRET=dein-secret-hier
   ```


### Für Nutzer in einem Linux Terminal

```bash
$ set -a
$ source .env
$ set +a
```

***

## 6. Service Account Rollen zuweisen

Da der Client mit aktivierten Service Account Roles angelegt wurde, kann das BFF die
Keycloak Admin-API aufrufen (z. B. zum Anlegen oder Verwalten von Benutzern).
Dafür müssen dem Service Account die passenden Rechte gegeben werden.

Navigiere zu **Clients → fort-bff → Service accounts roles**:

### Client-Rollen aus `realm-management` zuweisen

1. **Assign role** → **Client roles** klicken
2. Den Client `realm-management` auswählen
3. Folgende Rollen auswählen und zuweisen:

| Rolle | Wozu benötigt |
|---|---|
| `manage-users` | Benutzer erstellen, bearbeiten, löschen |
| `view-users` | Benutzer und Rollen abrufen |
| `manage-realm` | Realm-Einstellungen lesen/schreiben |
| `query-users` | Benutzer suchen |
| `query-realms` | Realm-Informationen abfragen |

### Realm-Rollen zuweisen

1. **Assign role** → **Realm roles** klicken
2. Folgende Rollen zuweisen: `ADMIN`, `USER` & `EDITOR`

***

## 7. Mapper konfigurieren

Damit die Rollen eines Benutzers im JWT-Token der Anwendung sichtbar sind,
muss ein Mapper eingerichtet werden.

Navigiere zu **Clients → fort-bff → Client scopes → fort-bff-dedicated → Add mapper**
und richte ihn wie folgt ein:

| Feld | Wert |
|---|---|
| Mapper type | User Realm Role |
| Name | `realm roles` |
| Realm Role prefix | *(leer lassen)* |
| Multivalued | On |
| Token Claim Name | `realm_access.roles` |
| Claim JSON Type | String |
| Add to ID token | On |
| Add to access token | On |
| Add to lightweight access token | Off |
| Add to userinfo | On |
| Add to token introspection | On |

***

## 8. User Profile konfigurieren

*(Falls noch nicht automatisch gesetzt)*

Navigiere zu **Realm settings → User profile → username → Edit attribute**:

### Validations

Folgende Validatoren müssen eingetragen sein (über **Add validator**):

| Validator name | Config |
|---|---|
| `length` | `{"min": 3, "max": 255}` |
| `username-prohibited-characters` | `{}` |
| `up-username-not-idn-homograph` | `{}` |

### Permissions

| Wer | Kann editieren | Kann ansehen |
|---|---|---|
| User | ✅ | ✅ |
| Admin | ✅ | ✅ |

***

## 9. Realm settings – Login konfigurieren

Navigiere zu **Realm settings → Login** und prüfe/ergänze folgende Einstellungen:

| Einstellung | Wert | Begründung |
|---|---|---|
| User registration | **ON** | |
| Forgot password | **ON** | Das BFF ruft `execute-actions-email` mit `UPDATE_PASSWORD` auf – dafür muss Keycloak E-Mails versenden können |
| Remember me | nach Wunsch | Optional |
| Login with email | **ON** | |
| Email as username | **OFF** | Username und E-Mail sind getrennte Felder |
| Verify email | **OFF** | Das BFF setzt `emailVerified: true` beim Anlegen bereits programmatisch |

***

## 10. Testbenutzer anlegen

1. Zu **Users** gehen
2. **Add User** auswählen
3. Als Benutzername eintragen:

```text
testuser
```

4. Benutzer speichern
5. Danach zu **Credentials** gehen
6. Ein Passwort setzen
7. **Temporary** deaktivieren

***

## 11. Eigenem User Admin-Rechte geben

Um die Anwendung vollständig nutzen zu können, sollte dem eigenen Benutzer
die `ADMIN`-Rolle zugewiesen werden:

1. **Users → deinen Benutzernamen** auswählen
2. **Role mappings → Assign role → Realm roles → `ADMIN`** zuweisen

***

## 12. Login & Logout testen

Nach abgeschlossener Konfiguration kann der Login getestet werden:

```text
http://localhost:8084/login
```

Hinweis: Nach erfolgreichem Login zeigt der Endpoint `http://localhost:8084/auth/me` die Nutzerdaten an.

Logout ist erreichbar unter:

```text
http://localhost:8084/logout
```

***

## 13. Wichtiger Hinweis zu IntelliJ und `.env`

Unbedingt sicherstellen, dass IntelliJ wirklich die richtige `.env`-Datei verwendet.

Für einen möglichen Lösungsansatz siehe: [5. Client Secret kopieren → Für IntelliJ-Nutzer](#für-intellij-nutzer)

Gerade bei Login- oder Logout-Problemen liegt die Ursache oft daran, dass:

* die falsche `.env` geladen wird
* das `client secret` nicht aktuell ist
* die Anwendung ohne die erwarteten Environment-Variablen gestartet wurde

***

## Kurzüberblick

### Keycloak
- URL: `http://localhost:9000`
- Realm: `fort`
- Client ID: `fort-bff`

### Anwendung
- App URL: `http://localhost:8084`
- Login: `http://localhost:8084/login`
- Logout: `http://localhost:8084/logout`

### Redirects
- Redirect URI: `http://localhost:8084/login/oauth2/code/fort`
- Post Logout Redirect URIs: `http://localhost:8084`, `http://localhost:5173/login`, `http://localhost:5173/login-blocked`

### Rollen
- `ADMIN` – Vollzugriff
- `EDITOR` – Lesen/Schreiben ohne Benutzerverwaltung
- `USER` – Nur Lesezugriff

### Testuser
- Username: `testuser`
- Passwort: manuell unter **Credentials** setzen