# Secure Event Ticketing Platform

## Arhitektura

Aplikacija se sastoji od 5 servisa:

- **Frontend** — Express.js web sučelje (port 3000)
- **API** — Express.js backend (port 8080)
- **Worker** — pozadinski processor ticket narudžbi
- **PostgreSQL** — baza podataka
- **Redis** — queue i cache

## 1. dio — Lokalni razvoj (Docker Compose)

### Preduvjeti

- Docker Desktop
- Git

### Pokretanje

```bash
# Kloniraj repozitorij
git clone https://github.com/kristijanbobovec/devops-project.git
cd devops-project

# Postavi environment varijable
cp .env.example .env

# Pokreni cijeli stack
docker compose up --build
```

### Validacija

```bash
# Health endpoint
curl http://localhost:8080/healthz

# Frontend
open http://localhost:3000
```

### Zaustavljanje

```bash
# Zaustavi (podaci ostaju)
docker compose down

# Zaustavi i obrisi podatke
docker compose down -v
```

## 2. dio — Produkcijski deployment (Kubernetes)

### Preduvjeti

- Minikube
- kubectl

### Pokretanje

```bash
# Pokreni Minikube
minikube start --driver=docker

# Omogući Ingress
minikube addons enable ingress

# Postavi slike u Minikube
eval $(minikube docker-env)
docker build -t ticketing-api:v1 ./api
docker build -t ticketing-frontend:v1 ./frontend
docker build -t ticketing-worker:v1 ./worker

# Primijeni sve manifeste
kubectl apply -f k8s

# Provjeri status
kubectl get pods -n ticketing
```

### Validacija

```bash
# Health endpointi
minikube service api -n ticketing --url
minikube service frontend -n ticketing --url

# Provjeri sve resurse
kubectl get all -n ticketing
```

### Rolling update i rollback

```bash
# Rolling update
kubectl set image deployment/api api=ticketing-api:v2 -n ticketing
kubectl rollout status deployment/api -n ticketing

# Rollback
kubectl rollout undo deployment/api -n ticketing
```

## CI/CD Pipeline

GitHub Actions automatski:

1. Gradi Docker slike za sve servise
2. Skenira slike s Trivyjem (CRITICAL i HIGH ranjivosti)
3. Sprema sigurnosne izvještaje kao artifacts
4. Pusha slike na GitHub Container Registry

## Sigurnost

- Multi-stage Docker build (manje slike)
- Non-root korisnik u svim kontejnerima
- Secrets odvojeni od konfiguracije
- RBAC i ServiceAccount s minimalnim pravima
- NetworkPolicy segmentacija prometa
- Trivy skeniranje u CI/CD pipelinu
