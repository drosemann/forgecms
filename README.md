 ForgeCMS

> Flask Blog & CMS mit MariaDB – Auth, Draft/Scheduled/Published Workflow, Markdown und Media Uploads. Gebaut weil ich schon immer bloggen wollte, aber nie Zeit und kein Impressum hatte.

**Stack:** Python 3.8+, Flask, SQLAlchemy, MariaDB 10.5+, Bootstrap 5, Docker

Ich wollte immer einen eigenen Blog statt WordPress – leicht, selbst gehostet, ohne Overhead. Am Ende ist es nie live gegangen, weil Zeit gefehlt hat und in Deutschland ohne Impressum direkt Abmahn-Risiko hast. Deshalb liegt es jetzt als Open-Source hier.

### Was kann ForgeCMS?
- User Registrierung & Login
- Admin Content Verwaltung
- Post Workflow: `draft`, `scheduled`, `published`, `archived`
- Vorschau für unveröffentlichte Posts
- SEO-freundliche Slugs, Bootstrap Responsive UI
- Bild Uploads für Posts

### Repo Aufbau
- `app/` – Flask App (Blueprints, Models, App Factory)
- `templates/` – Jinja2 Templates
- `static/` – CSS & Uploads
- `migrations/` – DB Migrationen inkl. `migrations/manual/` SQL
- `wsgi.py` – WSGI Entrypoint
- `Dockerfile` / `docker-compose.yml` – Single Server Deployment
- `docker-compose.enterprise.yml` – Multi-Instance mit NGINX Loadbalancer
- `docker/` – Container Helper & NGINX Config

## Schnellstart mit Docker (empfohlen)

### Voraussetzungen
- Docker Engine 24+
- Docker Compose v2+

### 1) Environment
```bash
cp .env.example .env
```
Mindestens setzen:
- `SECRET_KEY`
- `DB_PASSWORD`
- `DB_ROOT_PASSWORD`

### 2) Default Mode – ein Linux Server
Ein ForgeCMS Container + ein MariaDB Container.

```bash
docker compose up -d --build
```

- ForgeCMS: `http://localhost:8000`
- Persistente Daten: `mariadb_data` und `uploads_data` Volumes

Stop:
```bash
docker compose down
```

### 3) Enterprise Mode – Multi-Instance
Zwei ForgeCMS Instanzen hinter NGINX + ein MariaDB.

```bash
docker compose -f docker-compose.enterprise.yml up -d --build
```

- Einstieg: `http://localhost` (Port 80)
- Backend: `forgecms-a`, `forgecms-b`
- Shared Uploads Volume für beide Apps + NGINX

Stop:
```bash
docker compose -f docker-compose.enterprise.yml down
```

### Docker Hinweise
- Container warten auf MariaDB bevor die App startet
- Migrationen laufen automatisch beim Start (im Enterprise Mode nur `forgecms-a`)
- `UPLOAD_FOLDER` default `static/uploads`, liegt im Volume

## Lokal ohne Docker

```bash
git clone <repo-url>
cd forgecms
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### MariaDB anlegen
```sql
CREATE DATABASE forgecms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'forgecms'@'localhost' IDENTIFIED BY 'dein_passwort';
GRANT ALL PRIVILEGES ON forgecms.* TO 'forgecms'@'localhost';
FLUSH PRIVILEGES;
```

```bash
cp .env.example .env
flask db upgrade
flask run
```
Läuft auf `http://127.0.0.1:5000`.

## Admin User anlegen
```bash
flask shell
```
```python
from app import db
from app.models import User

admin = User(username='admin', email='admin@example.com', is_admin=True)
admin.set_password('sicheres-passwort')
db.session.add(admin)
db.session.commit()
```

## Warum das Projekt noch relevant ist
Für mich ist es die Basis für `d5niel.de` und den News-Bereich auf `shadowunity.de` – sobald Impressum geklärt ist. Kein WordPress, volle Kontrolle, MariaDB Erfahrung die ich eh täglich nutze. Und Docker-Ready für meinen "pure Docker statt Panel" Ansatz.

## Lizenz & Community
- [Contributing](CONTRIBUTING.md)
- [Code of Conduct](CODE_OF_CONDUCT.md)
- [MIT](LICENSE) — gilt für Releases ab 24.10.2026. Ältere Stände/Commits davor bleiben unter GNU GPLv3.
