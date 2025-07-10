# Service de Métriques - Prometheus & Grafana

Ce service permet de collecter, stocker et visualiser les métriques de vos microservices.

## Architecture

- **Prometheus** : Collecte et stockage des métriques
- **Grafana** : Visualisation et tableaux de bord
- **Service d'authentification** : Instrumenté avec des métriques Prometheus
- **Backend CV/Quiz** : Instrumenté avec des métriques Prometheus

## Métriques collectées

### Métriques HTTP
- Nombre total de requêtes par endpoint
- Temps de réponse par endpoint
- Codes de statut HTTP

### Métriques d'authentification
- Tentatives de connexion/inscription (succès/échec)
- Durée des opérations d'authentification
- Taux de succès d'authentification

### Métriques du Backend CV/Quiz
- Requêtes de génération et correction de quiz
- Durée de génération de quiz
- Requêtes OpenAI et temps de réponse
- Taux de succès des opérations

### Métriques système
- Utilisation CPU
- Utilisation mémoire
- Temps de fonctionnement

## Installation et démarrage

### 1. Installer les dépendances du service d'authentification

```bash
cd ../autenthication-service-NegYanis16
npm install
```

### 2. Installer les dépendances du backend CV/Quiz

```bash
cd ../Backend-cv
npm install
```

### 3. Démarrer le service d'authentification

```bash
cd autenthication-service-NegYanis16
npm run dev
```

Le service sera disponible sur `http://localhost:3001`
Les métriques seront exposées sur `http://localhost:3001/metrics`

### 4. Démarrer le backend CV/Quiz

```bash
cd Backend-cv
npm run dev
```

Le service sera disponible sur `http://localhost:5000`
Les métriques seront exposées sur `http://localhost:5000/metrics`

### 5. Démarrer Prometheus et Grafana

```bash
cd metrics-service-NegYanis16
docker-compose up -d
```

- **Prometheus** : http://localhost:9090
- **Grafana** : http://localhost:3010 (admin/admin)

## Configuration

### Prometheus
Le fichier `prometheus.yml` configure la collecte des métriques :
- Service d'authentification : `host.docker.internal:3001`
- Backend CV/Quiz : `host.docker.internal:5000`
- Intervalle de collecte : 10 secondes

### Grafana
- **Source de données** : Prometheus (automatiquement configurée)
- **Tableaux de bord** :
  - Service d'Authentification - Métriques (chargé automatiquement)
  - Backend CV/Quiz - Métriques (chargé automatiquement)

## Utilisation

### 1. Vérifier que les métriques sont collectées

Accédez aux endpoints métriques :
- Service d'authentification : http://localhost:3001/metrics
- Backend CV/Quiz : http://localhost:5000/metrics

### 2. Visualiser dans Prometheus

Accédez à `http://localhost:9090` et utilisez l'onglet "Graph" pour tester des requêtes :

**Métriques d'authentification :**
- `auth_attempts_total` : Nombre total de tentatives d'authentification
- `rate(auth_attempts_total[5m])` : Taux de tentatives d'authentification

**Métriques du backend :**
- `quiz_requests_total` : Nombre total de requêtes de quiz
- `openai_requests_total` : Nombre total de requêtes OpenAI
- `rate(quiz_generation_duration_seconds[5m])` : Durée moyenne de génération de quiz

### 3. Visualiser dans Grafana

Accédez à `http://localhost:3010` avec les identifiants `admin/admin`.

Deux tableaux de bord seront automatiquement disponibles :

**Service d'Authentification - Métriques :**
- Tentatives d'authentification
- Taux de succès
- Durée des opérations
- Métriques HTTP
- Utilisation système

**Backend CV/Quiz - Métriques :**
- Requêtes de quiz par opération
- Taux de succès des requêtes
- Durée de génération de quiz
- Requêtes OpenAI et temps de réponse
- Métriques HTTP
- Utilisation système

## Tests

### Service d'authentification

```bash
# Test d'inscription
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password123","nom":"Test","prenom":"User","niveau":"M2","classe":"INFO"}'

# Test de connexion
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password123"}'
```

### Backend CV/Quiz

```bash
# Test de génération de quiz (nécessite un token d'authentification)
curl -X GET "http://localhost:5000/api/quiz/generate?level=M2&subject=Mathématiques" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"

# Test de correction de quiz (nécessite un token d'authentification)
curl -X POST http://localhost:5000/api/quiz/correct \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{"quizId":"quiz_id_here","answers":["réponse1","réponse2"]}'
```

## Métriques disponibles

### Métriques d'authentification
- `auth_attempts_total{method,success,provider}` : Tentatives d'authentification
- `auth_duration_seconds{method,provider}` : Durée des opérations d'authentification

### Métriques du backend CV/Quiz
- `quiz_requests_total{operation,status}` : Requêtes de quiz
- `quiz_generation_duration_seconds{type}` : Durée de génération de quiz
- `openai_requests_total{operation,status}` : Requêtes OpenAI
- `openai_response_time_seconds{operation}` : Temps de réponse OpenAI

### Métriques HTTP
- `http_requests_total{method,route,status_code}` : Requêtes HTTP
- `http_request_duration_seconds{method,route,status_code}` : Durée des requêtes

### Métriques système
- `process_cpu_usage_percent` : Utilisation CPU
- `process_memory_usage_bytes` : Utilisation mémoire

## Prochaines étapes

1. **Instrumenter les autres services** : Ajouter des métriques aux autres microservices
2. **Alertes** : Configurer des alertes Prometheus pour détecter les problèmes
3. **Métriques métier** : Ajouter des métriques spécifiques à votre domaine
4. **Monitoring avancé** : Ajouter des métriques de base de données, cache, etc. 