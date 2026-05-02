# POC DevOps — Jenkins + Harbor + K3s

Pipeline CI/CD complet : GitHub → Jenkins → Harbor → K3s

## Architecture

```
GitHub push
    ↓
Jenkins (webhook)
    ↓
Build image Docker
    ↓
Push sur Harbor (registry privé)
    ↓
Deploy sur K3s
```

## Structure du projet

```
poc-devops/
├── app/
│   ├── index.js          # App Node.js Express
│   └── package.json
├── k8s/
│   ├── namespace.yaml    # Namespace "poc"
│   ├── deployment.yaml   # Deployment K3s
│   └── service.yaml      # Service LoadBalancer
├── Dockerfile
├── Jenkinsfile
└── README.md
```

## Prérequis

- K3s installé et fonctionnel
- Harbor installé sur K3s
- Jenkins installé sur K3s
- kubectl configuré

## Setup

### 1. Remplacer l'IP dans les fichiers

Remplace `192.168.1.X` par l'IP de ta VM K3s dans :
- `Jenkinsfile` → variable `HARBOR_URL`
- `k8s/deployment.yaml` → champ `image`

### 2. Créer un projet Harbor

- Connecte-toi à Harbor
- Créer un projet nommé `poc`
- Le rendre privé

### 3. Configurer Jenkins

Ajouter ces credentials dans Jenkins :

| ID | Type | Valeur |
|---|---|---|
| `harbor-credentials` | Username/Password | Login Harbor |
| `kubeconfig` | Secret file | Contenu de ~/.kube/config |

### 4. Créer le pipeline Jenkins

- New Item → Pipeline
- Pipeline script from SCM
- SCM : Git → URL de ce repo
- Branch : main
- Script Path : Jenkinsfile

### 5. Configurer le webhook GitHub

Dans GitHub → Settings → Webhooks :
- URL : `http://<jenkins-ip>/github-webhook/`
- Content type : `application/json`
- Trigger : `push`

## Test

```bash
# Vérifier le déploiement
kubectl get pods -n poc
kubectl get svc -n poc

# Tester l'app
curl http://<external-ip>
```

Réponse attendue :
```json
{
  "message": "POC DevOps - Hello from K3s!",
  "version": "1",
  "hostname": "poc-devops-xxx"
}
```
