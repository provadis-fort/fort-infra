# Docker-Deployment (GHCR-Images)


## Voraussetzungen

- Docker Engine und Docker Compose

## Dateien im Ordner /docker/

- `docker-compose.yml` (zieht alle Images von GHCR)
- fort-bff.env
- Diese Anleitung

## Schritt-für-Schritt: Installation und Start

0)Herunterladen
Die folgenden Dateien müssen im gleichen Ordner liegen:
- docker-comopose.yml
- fort-bff.env

1)Stack starten:

```bash
docker compose up -d
```

2)Keycloak konfigurieren (nur beim ersten Mal):

- Anleitung in `fort-infra/keycloak/ConfigHelp.md` befolgen
- Client für die UI `http://localhost:8083` konfigurieren
- `OIDC_CLIENT_SECRET` aus den `fort-bff`-Client-Credentials kopieren
- `fort-bff.env` mit dem echten Secret aktualisieren

3)BFF neu starten, damit das Secret übernommen wird:

```bash
docker compose restart fort-bff
```

4)UI öffnen:

- `http://localhost:8083`

## Ports (localhost)

- UI: `http://localhost:8083`
- BFF: `http://localhost:8084`
- Lecture API: `http://localhost:8080`
- Instructor API: `http://localhost:8081`
- Assignment API: `http://localhost:8082`
- Keycloak: `http://localhost:9000`

## Stoppen / Entfernen

```bash
docker compose down
```

## Fehlerbehebung

- Container prüfen: `docker compose ps`
- Logs: `docker compose logs --tail=200`







