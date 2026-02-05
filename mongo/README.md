# MongoDB Infrastructure

This directory contains the Docker setup for the MongoDB instance used by the Fort project.

## Services

- **MongoDB**: The database server (Port `27017`)
  - Username: `${MONGO_ROOT_USER}` (from `.env`)
  - Password: `${MONGO_ROOT_PASSWORD}` (from `.env`)
- **Mongo Express**: Web-based administrative interface (Port `8083`)
  - URL: http://localhost:8083

## Configuration

The setup uses a `.env` file for secrets.
Default values:
- User: `root`
- Password: `examplepassword`

## Usage

Start the services:

```bash
docker compose up -d
```

Stop the services:

```bash
docker compose down
```
