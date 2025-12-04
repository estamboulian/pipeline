# Projet déploiement CI/CD dans Kubernetes


## 1. Architecture du Projet (Kustomize)

Le dépôt est structuré selon le modèle standard **Base / Overlays** pour éviter la duplication de code.

```text
.
├── .github/workflows/
│   └── security.yml       # Pipeline CI (Trivy : Scan Secrets & Images)
├── base/                  # Configuration de base (Commune)
│   ├── deployment.yaml    # Déploiement de l'application Nginx
│   ├── service.yaml       # Service pour exposer l'app
│   └── kustomization.yaml # Point d'entrée Kustomize Base
├── overlays/              # Environnements spécifiques
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── nginx-dev.yaml   # Patch : Configuration légère pour le dév
│   │   ├── secret-dev.yaml  # Secrets pour le dév (Test détection Trivy)
│   │   └── presync.yaml     # Hook ArgoCD : Job exécuté avant la synchro
│   └── prod/
│       ├── kustomization.yaml
│       ├── nginx-prod.yaml  # Patch : Haute dispo pour la prod
│       └── secret-prod.yaml # Secrets pour la prod
└── README.md
```
## 2. Le Déploiement Continu (CD) avec ArgoCD

Création de deux applications qui respect le CRD d'ArgoCD pour un déploiement de Dev et de Prod,
```sh
root@control-plane:~/devsecops/pipeline/overlays/prod# kubectl get application -n argocd
NAME         SYNC STATUS   HEALTH STATUS
nginx-dev    Synced        Healthy
nginx-prod   Synced        Healthy
```
![image](images/image_namespace_dev_prod.png)

Pour le déploiement de notre application Nginx, nous utilisons un fichier de customisation (kustomization.yaml) pour isoler l'environnement de développement. Cela rend possible l'application de paramètres dédiés, tels que PreSync et la gestion de secrets spécifiques à la dev.

![image](images/architecture_1.png)

Pour la production, l'isolation est assurée par un fichier de customisation dédié. Celui-ci déploie l'application sur 3 réplicas pour assurer la résilience et intègre des secrets spécifiques, distincts de ceux du développement.

![image](images/architecture_prod.png)

Pour tester la robustesse du déploiement, nous avons altéré le cluster manuellement. ArgoCD, qui surveille l'état en permanence, a identifié que le nombre de réplicas ne correspondait plus au manifeste Git. Il a donc automatiquement écrasé nos modifications manuelles pour revenir à l'état stable.

![image](images/image_namespace_dev.png)

