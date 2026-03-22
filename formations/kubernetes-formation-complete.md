# Formation Complète Kubernetes — Du Débutant au Certifié

> **Objectif** : Maîtriser Kubernetes de A à Z, comprendre les concepts en profondeur, et être prêt pour les certifications CKA (Certified Kubernetes Administrator) et CKAD (Certified Kubernetes Application Developer).
>
> **Prérequis** : Connaissances de base en Linux, Docker, et réseau TCP/IP.
>
> **Durée estimée** : 80-120 heures de travail (théorie + pratique)

---

## Table des matières

1. [Module 1 — Fondations : Comprendre l'orchestration de conteneurs](#module-1--fondations--comprendre-lorchestration-de-conteneurs)
2. [Module 2 — Architecture de Kubernetes](#module-2--architecture-de-kubernetes)
3. [Module 3 — Installation et configuration de l'environnement](#module-3--installation-et-configuration-de-lenvironnement)
4. [Module 4 — kubectl : votre couteau suisse](#module-4--kubectl--votre-couteau-suisse)
5. [Module 5 — Les Pods en profondeur](#module-5--les-pods-en-profondeur)
6. [Module 6 — Workloads Controllers](#module-6--workloads-controllers)
7. [Module 7 — Services et Networking](#module-7--services-et-networking)
8. [Module 8 — Ingress et routage HTTP](#module-8--ingress-et-routage-http)
9. [Module 9 — Stockage et Persistance](#module-9--stockage-et-persistance)
10. [Module 10 — Configuration : ConfigMaps et Secrets](#module-10--configuration--configmaps-et-secrets)
11. [Module 11 — Namespaces et multi-tenancy](#module-11--namespaces-et-multi-tenancy)
12. [Module 12 — Scheduling avancé](#module-12--scheduling-avancé)
13. [Module 13 — Sécurité (RBAC, Network Policies, Pod Security)](#module-13--sécurité-rbac-network-policies-pod-security)
14. [Module 14 — Observabilité : Monitoring, Logging, Tracing](#module-14--observabilité--monitoring-logging-tracing)
15. [Module 15 — Helm : le gestionnaire de packages](#module-15--helm--le-gestionnaire-de-packages)
16. [Module 16 — CI/CD et GitOps avec Kubernetes](#module-16--cicd-et-gitops-avec-kubernetes)
17. [Module 17 — Autoscaling](#module-17--autoscaling)
18. [Module 18 — Patterns avancés et bonnes pratiques de production](#module-18--patterns-avancés-et-bonnes-pratiques-de-production)
19. [Module 19 — Troubleshooting (essentiel CKA)](#module-19--troubleshooting-essentiel-cka)
20. [Module 20 — Préparation aux certifications CKA et CKAD](#module-20--préparation-aux-certifications-cka-et-ckad)
21. [Annexes](#annexes)

---

## Module 1 — Fondations : Comprendre l'orchestration de conteneurs

### 1.1 Pourquoi Kubernetes ?

Avant Docker, déployer une application signifiait configurer un serveur manuellement : installer les dépendances, gérer les conflits de versions, et prier pour que la production se comporte comme le dev. Docker a résolu le problème du packaging en créant des conteneurs portables et reproductibles.

Mais Docker seul pose de nouvelles questions :

- **Comment distribuer les conteneurs sur plusieurs machines ?** Un seul serveur ne suffit pas en production.
- **Comment gérer la haute disponibilité ?** Si un conteneur crash, qui le redémarre ?
- **Comment scaler automatiquement ?** Quand le trafic augmente, comment ajouter des instances ?
- **Comment faire des mises à jour sans downtime ?** Rolling updates, canary deployments...
- **Comment gérer le réseau entre conteneurs ?** Service discovery, load balancing...

Kubernetes (K8s) répond à toutes ces questions. C'est un **orchestrateur de conteneurs** qui automatise le déploiement, le scaling et la gestion d'applications conteneurisées.

### 1.2 L'analogie du chef d'orchestre

Imaginons un orchestre symphonique :

| Orchestre | Kubernetes |
|-----------|-----------|
| Le chef d'orchestre | Le Control Plane (décide quoi jouer) |
| Les musiciens | Les Nodes workers (exécutent les conteneurs) |
| La partition | Les manifestes YAML (décrivent l'état souhaité) |
| Chaque note jouée | Un Pod (plus petite unité exécutable) |
| Les sections (cordes, vents...) | Les Namespaces (isolation logique) |

Kubernetes fonctionne sur un modèle **déclaratif** : vous décrivez l'état souhaité ("je veux 3 répliques de mon API"), et K8s se charge de l'atteindre et de le maintenir. C'est la différence fondamentale avec une approche impérative ("lance ce conteneur sur ce serveur").

### 1.3 Concepts clés à retenir

**Desired State vs Current State** : Kubernetes compare en permanence l'état actuel du cluster avec l'état souhaité décrit dans les manifestes. Si un Pod crash, K8s en relance un automatiquement. C'est la **reconciliation loop**.

**Tout est une ressource API** : Chaque objet K8s (Pod, Service, Deployment...) est une ressource manipulable via l'API Server. Vous créez, lisez, mettez à jour et supprimez des ressources — c'est du CRUD appliqué à l'infrastructure.

**Labels et Selectors** : Le mécanisme central de K8s pour associer des ressources entre elles. Un Service trouve ses Pods via des labels, pas via des IPs.

---

## Module 2 — Architecture de Kubernetes

### 2.1 Vue d'ensemble du cluster

Un cluster Kubernetes se compose de deux types de machines :

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                             │
│  ┌──────────────┐ ┌────────────┐ ┌──────────┐ ┌──────────────┐ │
│  │  API Server   │ │   etcd     │ │Scheduler │ │  Controller  │ │
│  │  (kube-api)   │ │ (key-value)│ │          │ │   Manager    │ │
│  └──────────────┘ └────────────┘ └──────────┘ └──────────────┘ │
│        │               │              │              │           │
│        └───────────────┴──────────────┴──────────────┘           │
└────────────────────────────┬────────────────────────────────────┘
                             │ API calls
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Worker 1   │    │   Worker 2   │    │   Worker 3   │
│ ┌──────────┐ │    │ ┌──────────┐ │    │ ┌──────────┐ │
│ │  kubelet │ │    │ │  kubelet │ │    │ │  kubelet │ │
│ ├──────────┤ │    │ ├──────────┤ │    │ ├──────────┤ │
│ │kube-proxy│ │    │ │kube-proxy│ │    │ │kube-proxy│ │
│ ├──────────┤ │    │ ├──────────┤ │    │ ├──────────┤ │
│ │ Container│ │    │ │ Container│ │    │ │ Container│ │
│ │ Runtime  │ │    │ │ Runtime  │ │    │ │ Runtime  │ │
│ └──────────┘ │    │ └──────────┘ │    │ └──────────┘ │
│  [Pod][Pod]  │    │  [Pod][Pod]  │    │  [Pod][Pod]  │
└──────────────┘    └──────────────┘    └──────────────┘
```

### 2.2 Composants du Control Plane

**kube-apiserver** — Le point d'entrée unique du cluster. Toutes les communications passent par lui. Il valide et traite les requêtes API REST, puis stocke l'état dans etcd. Quand vous tapez `kubectl get pods`, la requête arrive ici.

**etcd** — La base de données distribuée (key-value store) qui stocke TOUT l'état du cluster. C'est la "source de vérité". Si etcd est perdu sans backup, le cluster est perdu. En production, on déploie etcd en cluster de 3 ou 5 nœuds pour la haute disponibilité.

**kube-scheduler** — Décide sur quel node placer un Pod nouvellement créé. Il prend en compte les ressources disponibles (CPU, mémoire), les contraintes d'affinité/anti-affinité, les taints/tolerations, et les topologies.

**kube-controller-manager** — Exécute les boucles de contrôle (controllers) qui surveillent l'état du cluster et agissent pour atteindre l'état souhaité. Exemples : le ReplicaSet controller s'assure que le bon nombre de Pods tourne, le Node controller détecte les nodes défaillants, le Job controller gère les tâches ponctuelles.

**cloud-controller-manager** (optionnel) — Fait le lien avec l'API du cloud provider (AWS, GCP, Azure) pour gérer les load balancers, les volumes de stockage cloud, et les routes réseau.

### 2.3 Composants des Worker Nodes

**kubelet** — L'agent qui tourne sur chaque node. Il reçoit les instructions de l'API Server ("lance ce Pod"), s'assure que les conteneurs tournent correctement, et rapporte l'état au Control Plane. Il gère aussi les health checks (liveness/readiness probes).

**kube-proxy** — Gère les règles réseau sur chaque node. Il implémente le concept de "Service" en routant le trafic vers les bons Pods, en utilisant iptables ou IPVS. C'est lui qui fait le load balancing côté réseau.

**Container Runtime** — Le moteur qui exécute les conteneurs. Kubernetes supporte tout runtime compatible CRI (Container Runtime Interface) : containerd (le plus courant), CRI-O, ou d'autres. Docker en tant que runtime a été déprécié depuis K8s 1.24 au profit de containerd.

### 2.4 Communication entre composants

```
Utilisateur
    │
    ▼
kubectl ──HTTP/REST──► kube-apiserver ──► etcd
                            │
                    ┌───────┼───────┐
                    ▼       ▼       ▼
              scheduler  ctrl-mgr  kubelet (sur chaque node)
                                      │
                                      ▼
                                  containerd ──► Pods
```

Toutes les communications passent par l'API Server. Les composants ne se parlent jamais directement entre eux. C'est un principe architectural fondamental : l'API Server est le **hub central**.

### 2.5 Exercice pratique

> **Question CKA** : Si etcd crash sur un cluster, quels sont les impacts immédiats ? Les Pods en cours d'exécution continuent-ils de tourner ?
>
> **Réponse** : Les Pods existants continuent de fonctionner (le kubelet les maintient). Mais aucune nouvelle opération n'est possible : pas de nouveaux Pods, pas de scaling, pas de mise à jour. Le cluster est en mode "lecture seule" dégradé. C'est pourquoi le backup etcd est critique.

---

## Module 3 — Installation et configuration de l'environnement

### 3.1 Options pour un cluster local de développement

| Outil | Avantages | Inconvénients | Recommandé pour |
|-------|-----------|---------------|-----------------|
| **Minikube** | Simple, multi-drivers (VirtualBox, Docker, KVM) | Mono-node par défaut | Apprentissage, CKA prep |
| **kind** (Kubernetes in Docker) | Léger, multi-nodes, CI/CD | Moins de fonctionnalités réseau | Tests, CI/CD |
| **k3s** | Ultra-léger (~50MB), production-ready | Simplifié (pas de etcd natif) | Edge, IoT, Raspberry Pi |
| **Rancher Desktop** | GUI, Docker+K8s intégré, multi-OS | Plus lourd | Développeurs qui veulent du GUI |
| **kubeadm** | Installation "vraie", identique à la prod | Plus complexe | Préparation CKA |

### 3.2 Installation de Minikube (recommandé pour la formation)

```bash
# --- Linux (Ubuntu/Debian) ---

# Installer les dépendances
sudo apt-get update
sudo apt-get install -y curl wget apt-transport-https virtualbox

# Installer Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Démarrer le cluster
minikube start --driver=docker --cpus=2 --memory=4096

# Vérifier
minikube status
```

### 3.3 Installation de kubectl

```bash
# Télécharger kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Vérifier la connexion au cluster
kubectl cluster-info
kubectl get nodes

# Activer l'autocomplétion (INDISPENSABLE pour le CKA)
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc
```

### 3.4 Installation avec kubeadm (méthode CKA)

C'est la méthode que vous devez connaître pour la certification CKA :

```bash
# --- Sur TOUS les nodes (master + workers) ---

# Désactiver le swap (obligatoire pour K8s)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Charger les modules kernel nécessaires
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Configurer sysctl
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# Installer containerd
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
# Activer SystemdCgroup dans le fichier config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd

# Installer kubeadm, kubelet, kubectl
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# --- Sur le MASTER uniquement ---

# Initialiser le cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Configurer kubectl pour l'utilisateur courant
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Installer un plugin réseau (ici Calico)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# --- Sur les WORKERS ---
# Utiliser la commande kubeadm join affichée par le master
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 3.5 Le fichier kubeconfig

Le fichier `~/.kube/config` est votre passeport pour accéder au cluster. Il contient trois éléments clés :

```yaml
apiVersion: v1
kind: Config
# Les clusters auxquels vous pouvez vous connecter
clusters:
- cluster:
    server: https://192.168.49.2:8443       # URL de l'API Server
    certificate-authority: /path/to/ca.crt    # Certificat CA du cluster
  name: minikube

# Les identités disponibles
users:
- name: minikube-user
  user:
    client-certificate: /path/to/client.crt
    client-key: /path/to/client.key

# Les contextes lient un cluster à un user + namespace
contexts:
- context:
    cluster: minikube
    user: minikube-user
    namespace: default          # Namespace par défaut
  name: minikube

# Le contexte actif
current-context: minikube
```

```bash
# Commandes essentielles pour gérer les contextes
kubectl config get-contexts              # Lister les contextes
kubectl config current-context           # Afficher le contexte actif
kubectl config use-context <name>        # Changer de contexte
kubectl config set-context --current --namespace=dev  # Changer le namespace par défaut
```

---

## Module 4 — kubectl : votre couteau suisse

### 4.1 Syntaxe générale

```bash
kubectl [commande] [TYPE] [NOM] [flags]
#        verbe      quoi   lequel  options
```

### 4.2 Commandes essentielles (à connaître par cœur pour le CKA)

```bash
# ===== LIRE =====
kubectl get pods                          # Lister les pods
kubectl get pods -o wide                  # Avec plus de détails (IP, node)
kubectl get pods -o yaml                  # Sortie YAML complète
kubectl get pods -o json                  # Sortie JSON
kubectl get pods --all-namespaces         # Tous les namespaces (ou -A)
kubectl get pods -l app=nginx             # Filtrer par label
kubectl get pods --field-selector status.phase=Running  # Filtrer par champ
kubectl get all                           # Tous les types de ressources

# ===== DÉCRIRE =====
kubectl describe pod <name>               # Détails complets + événements
kubectl describe node <name>              # État du node, ressources, pods

# ===== CRÉER =====
kubectl apply -f manifest.yaml            # Créer/mettre à jour (déclaratif, RECOMMANDÉ)
kubectl create -f manifest.yaml           # Créer uniquement (impératif)
kubectl create deployment nginx --image=nginx --replicas=3  # Création impérative

# ===== MODIFIER =====
kubectl edit deployment nginx             # Éditer en live (ouvre vi/vim)
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'  # Patch JSON
kubectl replace -f manifest.yaml          # Remplacer complètement
kubectl apply -f manifest.yaml            # Appliquer les changements

# ===== SUPPRIMER =====
kubectl delete pod <name>                 # Supprimer un pod
kubectl delete -f manifest.yaml           # Supprimer tout ce qui est dans le fichier
kubectl delete pods --all                 # Supprimer tous les pods
kubectl delete pod <name> --force --grace-period=0  # Suppression immédiate

# ===== DÉBUGGER =====
kubectl logs <pod>                        # Logs du pod
kubectl logs <pod> -c <container>         # Logs d'un conteneur spécifique
kubectl logs <pod> -f                     # Follow (comme tail -f)
kubectl logs <pod> --previous             # Logs du conteneur précédent (crash)
kubectl exec -it <pod> -- /bin/bash       # Shell dans le conteneur
kubectl exec -it <pod> -c <container> -- sh  # Shell dans un conteneur spécifique
kubectl port-forward pod/<name> 8080:80   # Tunnel local vers le pod
kubectl port-forward svc/<name> 8080:80   # Tunnel local vers le service
kubectl top pods                          # Consommation CPU/mémoire des pods
kubectl top nodes                         # Consommation CPU/mémoire des nodes

# ===== GÉNÉRATION RAPIDE DE YAML (CRUCIAL POUR CKA/CKAD) =====
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml > svc.yaml
kubectl create configmap myconfig --from-literal=key=value --dry-run=client -o yaml
kubectl create secret generic mysecret --from-literal=password=admin --dry-run=client -o yaml
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml
```

### 4.3 Astuces de productivité

```bash
# Alias essentiels (à mettre dans .bashrc)
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias ke='kubectl exec -it'

# Changer de namespace rapidement
alias kns='kubectl config set-context --current --namespace'
# Usage: kns kube-system

# Export YAML rapide
alias ky='kubectl --dry-run=client -o yaml'

# Visualiser les ressources en temps réel
watch kubectl get pods    # Rafraîchit toutes les 2 secondes
```

### 4.4 JSONPath (pour les questions CKA avancées)

```bash
# Extraire des informations spécifiques
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Trier les résultats
kubectl get pods --sort-by='.metadata.creationTimestamp'
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

---

## Module 5 — Les Pods en profondeur

### 5.1 Qu'est-ce qu'un Pod ?

Un Pod est la plus petite unité déployable dans Kubernetes. Il encapsule un ou plusieurs conteneurs qui partagent :

- Le même espace réseau (même IP, mêmes ports)
- Le même espace de stockage (volumes partagés)
- Les mêmes namespaces Linux (PID, IPC)

Dans 90% des cas, un Pod = un conteneur. Les Pods multi-conteneurs sont réservés à des patterns spécifiques (sidecar, init containers).

### 5.2 Anatomie d'un Pod — Manifeste YAML commenté

```yaml
apiVersion: v1              # Version de l'API (v1 pour les ressources de base)
kind: Pod                   # Type de ressource
metadata:
  name: webapp              # Nom unique dans le namespace
  namespace: default        # Namespace (default si omis)
  labels:                   # Labels = mécanisme d'association (clé/valeur)
    app: webapp
    version: "1.0"
    environment: production
  annotations:              # Métadonnées informatives (pas utilisées pour le sélection)
    description: "Application web principale"
    maintainer: "equipe-dev@example.com"
spec:
  # --- Init Containers (s'exécutent AVANT les containers principaux, dans l'ordre) ---
  initContainers:
  - name: init-db-check
    image: busybox:1.36
    command: ['sh', '-c', 'until nslookup db-service; do echo "En attente de la BDD..."; sleep 2; done']

  # --- Containers principaux ---
  containers:
  - name: webapp                    # Nom du conteneur
    image: nginx:1.25               # Image Docker (TOUJOURS mettre un tag, JAMAIS :latest en prod)
    imagePullPolicy: IfNotPresent   # Always | IfNotPresent | Never

    # --- Ports exposés ---
    ports:
    - containerPort: 80             # Port exposé par le conteneur
      name: http                    # Nom du port (pour les services)
      protocol: TCP

    # --- Variables d'environnement ---
    env:
    - name: APP_ENV
      value: "production"
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:            # Depuis un ConfigMap
          name: app-config
          key: database_host
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:               # Depuis un Secret
          name: app-secret
          key: db-password

    # --- Ressources (CPU/Mémoire) ---
    resources:
      requests:                     # Minimum garanti (pour le scheduling)
        cpu: "100m"                 # 100 millicores = 0.1 CPU
        memory: "128Mi"             # 128 Mebibytes
      limits:                       # Maximum autorisé
        cpu: "500m"                 # Le conteneur sera throttled au-delà
        memory: "256Mi"             # Le conteneur sera OOMKilled au-delà

    # --- Health Checks (CRITIQUES en production) ---
    livenessProbe:                  # "Le conteneur est-il vivant ?"
      httpGet:                      # GET HTTP sur /healthz
        path: /healthz
        port: 80
      initialDelaySeconds: 15       # Attendre 15s avant le premier check
      periodSeconds: 10             # Vérifier toutes les 10s
      failureThreshold: 3           # 3 échecs = redémarrage du conteneur

    readinessProbe:                 # "Le conteneur est-il prêt à recevoir du trafic ?"
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3           # 3 échecs = retiré du Service (plus de trafic)

    startupProbe:                   # "L'application a-t-elle fini de démarrer ?"
      httpGet:
        path: /healthz
        port: 80
      failureThreshold: 30          # 30 x 10s = 5 minutes max pour démarrer
      periodSeconds: 10

    # --- Volumes montés ---
    volumeMounts:
    - name: app-data
      mountPath: /data              # Chemin dans le conteneur
    - name: config-volume
      mountPath: /etc/config
      readOnly: true

    # --- Sécurité ---
    securityContext:
      runAsNonRoot: true            # Interdire l'exécution en root
      runAsUser: 1000               # UID de l'utilisateur
      readOnlyRootFilesystem: true  # Filesystem en lecture seule
      allowPrivilegeEscalation: false

  # --- Volumes ---
  volumes:
  - name: app-data
    persistentVolumeClaim:
      claimName: webapp-pvc         # Référence à un PVC
  - name: config-volume
    configMap:
      name: app-config              # Montage d'un ConfigMap comme fichiers

  # --- Paramètres du Pod ---
  restartPolicy: Always             # Always | OnFailure | Never
  terminationGracePeriodSeconds: 30 # Temps accordé pour un arrêt propre
  serviceAccountName: webapp-sa     # Compte de service pour RBAC
  dnsPolicy: ClusterFirst           # Résolution DNS
```

### 5.3 Cycle de vie d'un Pod

```
Pending ──► Running ──► Succeeded
    │           │
    │           └──► Failed
    │
    └── (impossible à scheduler : pas assez de ressources, image introuvable...)
```

**Pending** : Le Pod est accepté par l'API Server mais n'est pas encore schedulé sur un node, ou les images sont en cours de téléchargement.

**Running** : Au moins un conteneur est en cours d'exécution.

**Succeeded** : Tous les conteneurs se sont terminés avec succès (code 0). Typique pour les Jobs.

**Failed** : Au moins un conteneur s'est terminé avec un code d'erreur.

**CrashLoopBackOff** : Le conteneur crash et redémarre en boucle. K8s augmente le délai entre chaque redémarrage (back-off exponentiel : 10s, 20s, 40s... jusqu'à 5 minutes).

### 5.4 Multi-conteneurs : Patterns

**Sidecar** : Un conteneur auxiliaire qui étend les fonctionnalités du conteneur principal sans le modifier. Exemple : un collecteur de logs (Fluentd) qui lit les fichiers de log du conteneur principal via un volume partagé.

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  - name: log-collector          # Sidecar
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
      readOnly: true
  volumes:
  - name: logs
    emptyDir: {}                 # Volume éphémère partagé
```

**Ambassador** : Un conteneur proxy qui simplifie l'accès au réseau extérieur pour le conteneur principal.

**Adapter** : Un conteneur qui transforme les sorties du conteneur principal dans un format standard.

### 5.5 Exercices pratiques

```bash
# Exercice 1 : Créer un Pod nginx et vérifier qu'il fonctionne
kubectl run ex1-nginx --image=nginx:1.25
kubectl get pod ex1-nginx -o wide
kubectl port-forward pod/ex1-nginx 8080:80
# Ouvrir http://localhost:8080 dans le navigateur

# Exercice 2 : Créer un Pod avec des resource limits
kubectl run ex2-limited --image=nginx:1.25 --dry-run=client -o yaml > ex2.yaml
# Éditer ex2.yaml pour ajouter resources.requests et resources.limits
kubectl apply -f ex2.yaml
kubectl describe pod ex2-limited | grep -A 10 "Limits"

# Exercice 3 : Débugger un Pod qui crash
kubectl run ex3-crash --image=busybox -- /bin/sh -c "exit 1"
kubectl get pod ex3-crash        # Observer le CrashLoopBackOff
kubectl describe pod ex3-crash   # Lire les événements
kubectl logs ex3-crash --previous  # Logs du conteneur crashé
```

---

## Module 6 — Workloads Controllers

### 6.1 ReplicaSet

Un ReplicaSet garantit qu'un nombre spécifié de répliques de Pod tournent à tout moment. Si un Pod crash ou est supprimé, le ReplicaSet en recrée un. En pratique, on ne crée jamais un ReplicaSet directement — on utilise un Deployment qui le gère.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp            # Le RS gère les Pods ayant ce label
  template:                  # Template du Pod (identique à spec d'un Pod)
    metadata:
      labels:
        app: webapp          # DOIT correspondre au selector
    spec:
      containers:
      - name: webapp
        image: nginx:1.25
```

### 6.2 Deployment (LE contrôleur principal)

Le Deployment gère les ReplicaSets et ajoute les fonctionnalités de mise à jour (rolling update, rollback).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 3
  revisionHistoryLimit: 10        # Nombre de ReplicaSets conservés pour rollback
  selector:
    matchLabels:
      app: webapp
  strategy:
    type: RollingUpdate           # RollingUpdate (défaut) ou Recreate
    rollingUpdate:
      maxUnavailable: 1           # Max 1 Pod indisponible pendant le rolling update
      maxSurge: 1                 # Max 1 Pod en plus pendant le rolling update
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

```bash
# Opérations courantes sur les Deployments
kubectl apply -f deployment.yaml              # Déployer
kubectl scale deployment webapp --replicas=5  # Scaler
kubectl set image deployment/webapp webapp=nginx:1.26  # Mettre à jour l'image

# Rolling update et rollback
kubectl rollout status deployment/webapp      # Suivre le déploiement
kubectl rollout history deployment/webapp     # Historique des versions
kubectl rollout undo deployment/webapp        # Rollback à la version précédente
kubectl rollout undo deployment/webapp --to-revision=2  # Rollback à une version spécifique
kubectl rollout pause deployment/webapp       # Mettre en pause le rolling update
kubectl rollout resume deployment/webapp      # Reprendre le rolling update
```

### 6.3 DaemonSet

Un DaemonSet garantit qu'un Pod tourne sur **chaque node** du cluster (ou un sous-ensemble). Cas d'usage typiques : agents de monitoring, collecteurs de logs, kube-proxy.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.16
        resources:
          requests:
            cpu: "100m"
            memory: "200Mi"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      tolerations:                # Permet de tourner aussi sur les nodes master
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
```

### 6.4 StatefulSet

Pour les applications stateful (bases de données, message queues) qui nécessitent :

- Une identité réseau stable (nom DNS persistant)
- Un stockage persistant lié à chaque Pod
- Un ordre de déploiement/suppression garanti

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"        # Headless Service obligatoire
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:           # Crée un PVC par Pod automatiquement
  - metadata:
      name: pgdata
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

Les Pods d'un StatefulSet ont des noms prévisibles : `postgres-0`, `postgres-1`, `postgres-2`. Ils sont créés dans l'ordre (0, puis 1, puis 2) et supprimés dans l'ordre inverse.

### 6.5 Job et CronJob

**Job** — Exécute une tâche ponctuelle jusqu'à complétion :

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1               # Nombre d'exécutions réussies requises
  parallelism: 1               # Nombre de Pods en parallèle
  backoffLimit: 3              # Nombre max de tentatives en cas d'échec
  activeDeadlineSeconds: 300   # Timeout global (5 minutes)
  template:
    spec:
      containers:
      - name: migration
        image: myapp:migrate
        command: ["python", "manage.py", "migrate"]
      restartPolicy: OnFailure  # Never ou OnFailure (pas de Always pour un Job)
```

**CronJob** — Planifie des Jobs récurrents (syntaxe cron standard) :

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"          # Tous les jours à 2h du matin
  concurrencyPolicy: Forbid      # Allow | Forbid | Replace
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:16
            command: ["pg_dump", "-h", "db-service", "-U", "admin", "mydb"]
          restartPolicy: OnFailure
```

### 6.6 Tableau récapitulatif

| Controller | Cas d'usage | Identité | Stockage | Ordre |
|-----------|------------|----------|----------|-------|
| **Deployment** | Apps stateless (API, frontend) | Aléatoire | Partagé | Non garanti |
| **StatefulSet** | Apps stateful (BDD, cache) | Stable (pod-0, pod-1...) | Dédié par pod | Garanti |
| **DaemonSet** | Agents système (monitoring, logs) | Un par node | Via hostPath | N/A |
| **Job** | Tâches ponctuelles (migration, batch) | Temporaire | N/A | N/A |
| **CronJob** | Tâches planifiées (backup, nettoyage) | Temporaire | N/A | N/A |

---

## Module 7 — Services et Networking

### 7.1 Le problème que les Services résolvent

Les Pods sont éphémères : ils peuvent être recréés à tout moment avec une nouvelle adresse IP. Impossible de se connecter directement à l'IP d'un Pod de manière fiable. Les Services fournissent une adresse IP stable et un DNS pour accéder à un groupe de Pods.

### 7.2 Les types de Services

```
                                    Internet
                                       │
                            ┌──────────┴──────────┐
                            │    LoadBalancer      │ (Type 3)
                            │   IP externe cloud   │
                            └──────────┬──────────┘
                                       │
                            ┌──────────┴──────────┐
                            │     NodePort         │ (Type 2)
                            │   Port sur le node   │
                            └──────────┬──────────┘
                                       │
                            ┌──────────┴──────────┐
                            │    ClusterIP         │ (Type 1)
                            │   IP interne cluster │
                            └──────────┬──────────┘
                                       │
                          ┌────────────┼────────────┐
                          ▼            ▼            ▼
                      [Pod A]      [Pod B]      [Pod C]
                     app=webapp   app=webapp   app=webapp
```

**ClusterIP (défaut)** — Accessible uniquement depuis l'intérieur du cluster :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: ClusterIP               # Défaut si omis
  selector:
    app: webapp                 # Cible les Pods avec le label app=webapp
  ports:
  - port: 80                   # Port du Service (ce que les clients utilisent)
    targetPort: 8080           # Port du conteneur (ce que l'app écoute)
    protocol: TCP
```

Le Service est accessible via :
- `webapp-service` (dans le même namespace)
- `webapp-service.default.svc.cluster.local` (FQDN complet)

**NodePort** — Expose le service sur un port de chaque node (30000-32767) :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080             # Port sur le node (optionnel, K8s en assigne un sinon)
```

Accessible via `http://<n'importe-quel-node-ip>:30080`

**LoadBalancer** — Provisionne un load balancer cloud (AWS ELB, GCP LB, Azure LB) :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-lb
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
```

**Headless Service** — Pas de ClusterIP, le DNS renvoie directement les IPs des Pods :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None              # ← C'est ce qui le rend "headless"
  selector:
    app: postgres
  ports:
  - port: 5432
```

Utilisé avec les StatefulSets. Chaque Pod obtient un enregistrement DNS : `postgres-0.postgres.default.svc.cluster.local`

**ExternalName** — Alias DNS vers un service extérieur :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.production.example.com  # CNAME DNS
```

### 7.3 Service Discovery

Kubernetes offre deux mécanismes de découverte de services :

1. **DNS (recommandé)** : CoreDNS crée automatiquement des entrées DNS pour chaque Service.
   - `<service-name>` — dans le même namespace
   - `<service-name>.<namespace>` — cross-namespace
   - `<service-name>.<namespace>.svc.cluster.local` — FQDN

2. **Variables d'environnement** : K8s injecte les variables `<SERVICE_NAME>_SERVICE_HOST` et `<SERVICE_NAME>_SERVICE_PORT` dans chaque Pod. Moins flexible car les variables sont créées au démarrage du Pod.

### 7.4 Endpoints et EndpointSlices

Quand un Service est créé, Kubernetes crée un objet Endpoints qui liste les IPs des Pods correspondant au selector. Les EndpointSlices sont la version scalable (pour les clusters avec des milliers de Pods).

```bash
kubectl get endpoints webapp-service
kubectl get endpointslices -l kubernetes.io/service-name=webapp-service
```

### 7.5 Le réseau Kubernetes — CNI

Kubernetes ne fournit pas de solution réseau intégrée. Il délègue à un plugin CNI (Container Network Interface). Les plus courants :

| Plugin | Avantages | Network Policies |
|--------|-----------|------------------|
| **Calico** | Performant, riche en fonctionnalités | Oui |
| **Cilium** | eBPF-based, très performant, observabilité | Oui |
| **Flannel** | Simple, léger | Non (basique) |
| **Weave** | Simple, chiffrement intégré | Oui |

Règles réseau fondamentales de K8s :

1. Tous les Pods peuvent communiquer avec tous les autres Pods sans NAT
2. Tous les nodes peuvent communiquer avec tous les Pods sans NAT
3. L'IP qu'un Pod voit pour lui-même est la même que les autres Pods voient

---

## Module 8 — Ingress et routage HTTP

### 8.1 Pourquoi Ingress ?

Sans Ingress, exposer 10 applications web nécessiterait 10 LoadBalancers (= 10 IP publiques = coûteux). Ingress fournit un point d'entrée unique avec du routage basé sur l'URL ou le hostname.

```
                    Internet
                       │
                ┌──────┴──────┐
                │   Ingress   │     ← Un seul LoadBalancer
                │  Controller │
                └──────┬──────┘
                       │
          ┌────────────┼────────────┐
          │ /api/*     │ /web/*     │ blog.example.com
          ▼            ▼            ▼
     [api-svc]    [web-svc]    [blog-svc]
```

### 8.2 Installation d'un Ingress Controller

```bash
# NGINX Ingress Controller (le plus courant)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

# Vérifier l'installation
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

### 8.3 Configuration d'Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx          # Référence à l'Ingress Controller
  tls:                             # Configuration HTTPS
  - hosts:
    - app.example.com
    secretName: tls-secret         # Secret contenant le certificat TLS
  rules:
  - host: app.example.com         # Routage par hostname
    http:
      paths:
      - path: /api                 # Routage par chemin
        pathType: Prefix           # Prefix | Exact | ImplementationSpecific
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: blog.example.com        # Deuxième hostname
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-service
            port:
              number: 80
```

### 8.4 Gestion automatique des certificats TLS avec cert-manager

```bash
# Installer cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.0/cert-manager.yaml
```

```yaml
# ClusterIssuer pour Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: nginx

---
# Ingress avec certificat automatique
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # ← cert-manager gère le TLS
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: webapp-tls         # cert-manager crée et renouvelle ce Secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
```

---

## Module 9 — Stockage et Persistance

### 9.1 Le problème du stockage éphémère

Par défaut, le filesystem d'un conteneur est éphémère. Si le conteneur redémarre, les données sont perdues. Kubernetes propose plusieurs mécanismes de stockage persistant.

### 9.2 Volumes éphémères

**emptyDir** — Volume temporaire partagé entre conteneurs d'un même Pod :

```yaml
volumes:
- name: cache
  emptyDir:
    sizeLimit: "500Mi"         # Limite optionnelle
    medium: Memory             # "Memory" pour tmpfs (RAM), vide pour disque
```

**hostPath** — Monte un répertoire du node dans le Pod (à éviter en prod) :

```yaml
volumes:
- name: docker-sock
  hostPath:
    path: /var/run/docker.sock
    type: Socket               # Directory | File | Socket | CharDevice | BlockDevice
```

### 9.3 Stockage persistant : PV, PVC, StorageClass

Le modèle de stockage K8s sépare les rôles :

```
Admin cluster                                 Développeur
     │                                              │
     ▼                                              ▼
StorageClass ◄─── Provisionnement ───► PersistentVolumeClaim (PVC)
     │              dynamique                       │
     ▼                                              │
PersistentVolume (PV) ◄───── Binding ──────────────┘
     │
     ▼
Stockage physique (AWS EBS, GCE PD, NFS, Ceph...)
```

**StorageClass** — Définit un type de stockage et son provisionneur :

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs      # ou pd.csi.storage.gke.io, etc.
parameters:
  type: gp3
reclaimPolicy: Delete                    # Delete | Retain
volumeBindingMode: WaitForFirstConsumer  # Attend qu'un Pod utilise le PVC
allowVolumeExpansion: true               # Permet de redimensionner
```

**PersistentVolume (PV)** — Provisionné manuellement (ou dynamiquement via StorageClass) :

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteMany                        # ReadWriteOnce | ReadOnlyMany | ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    server: 192.168.1.100
    path: /exports/data
```

**PersistentVolumeClaim (PVC)** — La demande du développeur :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: webapp-data
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd             # Référence à la StorageClass
  resources:
    requests:
      storage: 10Gi
```

**Utilisation dans un Pod** :

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: webapp-data
```

### 9.4 Access Modes

| Mode | Abréviation | Description |
|------|-------------|-------------|
| ReadWriteOnce | RWO | Un seul node en lecture/écriture |
| ReadOnlyMany | ROX | Plusieurs nodes en lecture seule |
| ReadWriteMany | RWX | Plusieurs nodes en lecture/écriture |
| ReadWriteOncePod | RWOP | Un seul Pod en lecture/écriture (K8s 1.27+) |

---

## Module 10 — Configuration : ConfigMaps et Secrets

### 10.1 ConfigMaps

Les ConfigMaps stockent des données de configuration non sensibles sous forme de paires clé-valeur.

```bash
# Création impérative
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info \
  --from-file=nginx.conf=/path/to/nginx.conf
```

```yaml
# Création déclarative
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  DATABASE_URL: "postgres://db-service:5432/mydb"
  nginx.conf: |                  # Fichier de configuration complet
    server {
        listen 80;
        server_name example.com;
        location / {
            proxy_pass http://localhost:8080;
        }
    }
```

**Injection en tant que variables d'environnement** :

```yaml
spec:
  containers:
  - name: app
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    # OU injecter TOUTES les clés d'un coup :
    envFrom:
    - configMapRef:
        name: app-config
```

**Montage en tant que fichiers** :

```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
      items:                     # Sélectionner des clés spécifiques
      - key: nginx.conf
        path: nginx.conf         # Sera monté comme /etc/config/nginx.conf
```

### 10.2 Secrets

Les Secrets stockent des données sensibles (mots de passe, tokens, certificats). Ils sont encodés en base64 (pas chiffrés par défaut !).

```bash
# Création impérative
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='S3cur3P@ss!'

# Pour TLS
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

```yaml
# Création déclarative (les valeurs doivent être en base64)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=           # echo -n "admin" | base64
  password: UzNjdXIzUEBzcyE=  # echo -n "S3cur3P@ss!" | base64
---
# Alternative avec stringData (pas besoin de base64)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:                     # Sera converti en base64 automatiquement
  username: admin
  password: "S3cur3P@ss!"
```

**Sécuriser les Secrets** :

- Activer le chiffrement at-rest (EncryptionConfiguration dans l'API Server)
- Utiliser un gestionnaire de secrets externe (HashiCorp Vault, AWS Secrets Manager)
- Limiter l'accès via RBAC
- Ne jamais committer les Secrets dans Git (utiliser Sealed Secrets ou SOPS)

### 10.3 Immutabilité

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true                 # Empêche toute modification après création
data:
  APP_ENV: "production"
```

Les ConfigMaps et Secrets immutables protègent contre les modifications accidentelles et améliorent les performances du cluster (K8s n'a pas besoin de surveiller les changements).

---

## Module 11 — Namespaces et multi-tenancy

### 11.1 Concept

Les Namespaces sont des espaces de noms logiques qui divisent un cluster en environnements isolés. Ils permettent de séparer les ressources, d'appliquer des quotas, et de gérer les permissions.

```bash
# Namespaces par défaut
kubectl get namespaces
# default          - Namespace par défaut
# kube-system      - Composants système de K8s
# kube-public      - Ressources publiques (ConfigMap cluster-info)
# kube-node-lease  - Heartbeats des nodes
```

### 11.2 Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    pods: "20"                   # Max 20 pods
    requests.cpu: "10"           # Max 10 CPU demandés
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    persistentvolumeclaims: "10"
    services.loadbalancers: "2"
```

### 11.3 LimitRange

Définit des valeurs par défaut et des limites pour les conteneurs d'un namespace :

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - type: Container
    default:                     # Limits par défaut si non spécifiées
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:              # Requests par défaut si non spécifiées
      cpu: "100m"
      memory: "128Mi"
    max:                         # Valeurs maximum autorisées
      cpu: "2"
      memory: "2Gi"
    min:                         # Valeurs minimum autorisées
      cpu: "50m"
      memory: "64Mi"
```

---

## Module 12 — Scheduling avancé

### 12.1 Comment le Scheduler choisit un node ?

1. **Filtrage** : Élimine les nodes incompatibles (pas assez de ressources, taints incompatibles, affinity rules non respectées)
2. **Scoring** : Classe les nodes restants par score (meilleure répartition des ressources, localité des images, etc.)
3. **Binding** : Assigne le Pod au node avec le meilleur score

### 12.2 Node Selector (simple)

```yaml
spec:
  nodeSelector:
    disktype: ssd               # Le Pod sera placé sur un node avec ce label
    region: eu-west-1
```

### 12.3 Node Affinity (avancé)

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:    # OBLIGATOIRE
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In                                 # In | NotIn | Exists | DoesNotExist | Gt | Lt
            values:
            - eu-west-1a
            - eu-west-1b
      preferredDuringSchedulingIgnoredDuringExecution:   # PRÉFÉRENCE (soft)
        - weight: 80                                     # 1-100, plus c'est haut plus c'est prioritaire
          preference:
            matchExpressions:
            - key: node-type
              operator: In
              values:
              - high-memory
```

### 12.4 Pod Affinity et Anti-Affinity

```yaml
spec:
  affinity:
    podAffinity:                    # "Placer près de..."
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - cache
        topologyKey: kubernetes.io/hostname   # Même node

    podAntiAffinity:                # "Placer loin de..."
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - webapp
          topologyKey: topology.kubernetes.io/zone   # Zones différentes
```

### 12.5 Taints et Tolerations

Les **taints** sont appliquées aux nodes pour repousser les Pods. Les **tolerations** sont appliquées aux Pods pour tolérer les taints.

```bash
# Appliquer une taint à un node
kubectl taint nodes node1 gpu=true:NoSchedule
# Effets : NoSchedule | PreferNoSchedule | NoExecute
```

```yaml
# Toleration dans un Pod
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"           # Equal | Exists
    value: "true"
    effect: "NoSchedule"
```

### 12.6 Topology Spread Constraints

Distribue les Pods uniformément sur les zones, régions ou nodes :

```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1                              # Écart maximum entre les zones
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule        # DoNotSchedule | ScheduleAnyway
    labelSelector:
      matchLabels:
        app: webapp
```

---

## Module 13 — Sécurité (RBAC, Network Policies, Pod Security)

### 13.1 RBAC (Role-Based Access Control)

RBAC contrôle qui peut faire quoi dans le cluster. Quatre objets clés :

```
        Namespaced                          Cluster-wide
    ┌─────────────────┐              ┌─────────────────────┐
    │      Role       │              │    ClusterRole       │
    │ (permissions)   │              │ (permissions globales)│
    └────────┬────────┘              └────────┬────────────┘
             │ lié par                        │ lié par
    ┌────────┴────────┐              ┌────────┴────────────┐
    │  RoleBinding    │              │ ClusterRoleBinding   │
    │ (qui a le rôle) │              │ (qui a le rôle)      │
    └─────────────────┘              └─────────────────────┘
```

```yaml
# Role : permissions dans un namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development
rules:
- apiGroups: [""]               # "" = core API group (pods, services, configmaps...)
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]       # Sous-ressource : logs des pods
  verbs: ["get"]

---
# RoleBinding : lie le Role à un utilisateur/groupe/service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: development
subjects:
- kind: User
  name: mathieu
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: ci-cd-sa
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# ClusterRole pour les ressources non-namespaced (nodes, PV...)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list"]
```

```bash
# Tester les permissions
kubectl auth can-i create pods --namespace=development --as=mathieu
kubectl auth can-i delete nodes --as=mathieu
kubectl auth can-i --list --as=mathieu  # Lister toutes les permissions
```

### 13.2 Service Accounts

Chaque Pod tourne avec un Service Account qui définit ses permissions API :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
automountServiceAccountToken: false  # Bonne pratique : désactiver si pas besoin

---
# Utilisation dans un Pod
spec:
  serviceAccountName: app-sa
```

### 13.3 Network Policies

Les Network Policies contrôlent le trafic réseau entre Pods (nécessite un CNI compatible : Calico, Cilium).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api                   # S'applique aux Pods avec ce label

  policyTypes:
  - Ingress                      # Contrôle le trafic entrant
  - Egress                       # Contrôle le trafic sortant

  ingress:
  - from:
    - podSelector:               # Autoriser le trafic depuis les Pods avec app=frontend
        matchLabels:
          app: frontend
    - namespaceSelector:         # ET depuis le namespace monitoring
        matchLabels:
          name: monitoring
    ports:
    - port: 8080
      protocol: TCP

  egress:
  - to:
    - podSelector:               # Autoriser le trafic vers les Pods avec app=database
        matchLabels:
          app: database
    ports:
    - port: 5432
      protocol: TCP
  - to:                          # Autoriser le DNS (obligatoire !)
    - namespaceSelector: {}
    ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

**Deny All** — Politique de base (zero-trust) :

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}               # S'applique à tous les Pods
  policyTypes:
  - Ingress
  - Egress
```

### 13.4 Pod Security Standards

Depuis K8s 1.25, les Pod Security Standards remplacent les PodSecurityPolicies :

```yaml
# Appliquer au namespace via des labels
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted     # Bloque les Pods non conformes
    pod-security.kubernetes.io/warn: restricted         # Affiche un warning
    pod-security.kubernetes.io/audit: restricted         # Log pour audit

# Niveaux :
# - privileged : pas de restrictions
# - baseline : minimise les vecteurs d'attaque connus
# - restricted : bonnes pratiques de sécurité strictes
```

---

## Module 14 — Observabilité : Monitoring, Logging, Tracing

### 14.1 Les trois piliers de l'observabilité

| Pilier | Question | Outil principal |
|--------|----------|-----------------|
| **Metrics** | "Comment se porte mon système ?" | Prometheus + Grafana |
| **Logs** | "Que s'est-il passé ?" | Loki / EFK Stack |
| **Traces** | "Quel chemin a pris cette requête ?" | Jaeger / Tempo |

### 14.2 Metrics Server

```bash
# Installation du Metrics Server (nécessaire pour kubectl top et HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Vérifier
kubectl top nodes
kubectl top pods --all-namespaces
```

### 14.3 Prometheus + Grafana

```bash
# Installation via Helm (méthode recommandée)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

Le kube-prometheus-stack installe automatiquement :

- Prometheus (collecte de métriques)
- Grafana (dashboards de visualisation)
- AlertManager (gestion des alertes)
- Node Exporter (métriques système des nodes)
- kube-state-metrics (métriques de l'état des objets K8s)

### 14.4 Logging avec Loki

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set grafana.enabled=false  # Utiliser le Grafana déjà installé
```

### 14.5 Commandes de diagnostic

```bash
# État global du cluster
kubectl get componentstatuses        # Santé des composants (déprécié mais utile)
kubectl get nodes -o wide            # État des nodes
kubectl get events --sort-by='.lastTimestamp'  # Événements récents

# Diagnostic d'un Pod problématique
kubectl describe pod <name>          # Événements et détails
kubectl logs <name> --previous       # Logs du crash précédent
kubectl exec -it <name> -- sh        # Shell dans le conteneur
kubectl get pod <name> -o yaml       # Spec complète

# Diagnostic réseau
kubectl run debug --image=nicolaka/netshoot -it --rm -- bash  # Pod de debug réseau
# Dans le pod : nslookup, curl, ping, traceroute, tcpdump...
```

---

## Module 15 — Helm : le gestionnaire de packages

### 15.1 Concepts

Helm est le gestionnaire de packages pour Kubernetes. Un **chart** est un package Helm contenant tous les manifestes K8s d'une application, avec des valeurs paramétrables.

```
Chart/
├── Chart.yaml          # Métadonnées du chart
├── values.yaml         # Valeurs par défaut (surchargeables)
├── templates/          # Manifestes YAML avec des variables Go
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl    # Fonctions réutilisables
└── charts/             # Dépendances (sous-charts)
```

### 15.2 Commandes essentielles

```bash
# Gestion des repos
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm search hub wordpress        # Recherche sur Artifact Hub

# Installation
helm install my-release bitnami/nginx                          # Installer
helm install my-release bitnami/nginx -f custom-values.yaml   # Avec valeurs custom
helm install my-release bitnami/nginx --set replicaCount=3    # Override inline
helm install my-release bitnami/nginx --namespace prod --create-namespace

# Gestion
helm list                        # Lister les releases
helm status my-release           # État d'une release
helm upgrade my-release bitnami/nginx --set replicaCount=5   # Mettre à jour
helm rollback my-release 1       # Rollback à la révision 1
helm uninstall my-release        # Désinstaller

# Debug
helm template my-release bitnami/nginx -f values.yaml  # Rendu sans installer
helm get values my-release       # Valeurs actuelles
helm get manifest my-release     # Manifestes générés
```

### 15.3 Créer son propre Chart

```bash
# Scaffolding
helm create my-chart
```

Exemple de template avec des variables :

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-chart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-chart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.targetPort }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
```

```yaml
# values.yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

---

## Module 16 — CI/CD et GitOps avec Kubernetes

### 16.1 Stratégies de déploiement

| Stratégie | Downtime | Risque | Complexité |
|-----------|----------|--------|------------|
| **Rolling Update** | Non | Moyen | Faible (natif K8s) |
| **Blue/Green** | Non | Faible | Moyenne |
| **Canary** | Non | Très faible | Élevée |
| **Recreate** | Oui | Élevé | Très faible |

### 16.2 GitOps avec ArgoCD

Le principe GitOps : Git est la source de vérité. Toute modification de l'infrastructure passe par un commit Git. ArgoCD surveille le repo Git et synchronise le cluster automatiquement.

```bash
# Installation d'ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Accéder à l'UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Mot de passe initial :
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

```yaml
# Application ArgoCD
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/k8s-manifests.git
    targetRevision: main
    path: apps/webapp              # Dossier contenant les manifestes
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:                     # Synchronisation automatique
      prune: true                  # Supprimer les ressources absentes de Git
      selfHeal: true               # Corriger les drifts manuels
```

### 16.3 Pipeline CI/CD type (GitLab CI)

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t registry.example.com/webapp:$CI_COMMIT_SHA .
    - docker push registry.example.com/webapp:$CI_COMMIT_SHA

deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/webapp webapp=registry.example.com/webapp:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/webapp -n staging --timeout=300s
  environment:
    name: staging

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/webapp webapp=registry.example.com/webapp:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/webapp -n production --timeout=300s
  environment:
    name: production
  when: manual                     # Déploiement en prod = validation manuelle
```

---

## Module 17 — Autoscaling

### 17.1 Horizontal Pod Autoscaler (HPA)

Scale le nombre de répliques en fonction de la charge (CPU, mémoire, métriques custom).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70         # Scale si CPU moyen > 70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60   # Attendre 60s avant de scaler up
      policies:
      - type: Percent
        value: 100                     # Doubler le nombre de pods max
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # Attendre 5 min avant de scaler down
      policies:
      - type: Percent
        value: 10                      # Réduire de 10% max par période
        periodSeconds: 60
```

### 17.2 Vertical Pod Autoscaler (VPA)

Ajuste les requests/limits des conteneurs en fonction de l'utilisation réelle :

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: "Auto"               # Off | Initial | Recreate | Auto
  resourcePolicy:
    containerPolicies:
    - containerName: webapp
      minAllowed:
        cpu: "50m"
        memory: "64Mi"
      maxAllowed:
        cpu: "2"
        memory: "4Gi"
```

### 17.3 Cluster Autoscaler

Scale le nombre de nodes du cluster quand les Pods sont en Pending par manque de ressources. Configuré au niveau du cloud provider (EKS, GKE, AKS).

---

## Module 18 — Patterns avancés et bonnes pratiques de production

### 18.1 Labels standards

```yaml
metadata:
  labels:
    app.kubernetes.io/name: webapp
    app.kubernetes.io/instance: webapp-production
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: ecommerce
    app.kubernetes.io/managed-by: helm
```

### 18.2 Probes en production

```yaml
# Règle d'or des probes
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30        # Laisser le temps au démarrage
  periodSeconds: 15
  failureThreshold: 3            # 3 x 15s = 45s avant redémarrage
  timeoutSeconds: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5         # Plus court que liveness
  periodSeconds: 5
  failureThreshold: 3

startupProbe:                    # Pour les apps lentes au démarrage
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10              # 30 x 10s = 5 min max
```

### 18.3 Graceful Shutdown

```yaml
spec:
  terminationGracePeriodSeconds: 60    # Temps pour s'arrêter proprement
  containers:
  - name: app
    lifecycle:
      preStop:                          # Hook exécuté avant l'envoi du SIGTERM
        exec:
          command: ["/bin/sh", "-c", "sleep 10"]  # Attendre que le LB désenregistre le Pod
```

### 18.4 Pod Disruption Budget (PDB)

Garantit un nombre minimum de Pods disponibles pendant les opérations de maintenance :

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb
spec:
  selector:
    matchLabels:
      app: webapp
  minAvailable: 2                # OU maxUnavailable: 1 (pas les deux)
```

### 18.5 Priority Classes

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000                   # Plus c'est haut, plus c'est prioritaire
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "Pour les services critiques"
```

### 18.6 Checklist de production

- [ ] Images avec tags spécifiques (pas `:latest`)
- [ ] Resource requests ET limits définis
- [ ] Liveness, readiness ET startup probes configurées
- [ ] Security context non-root, read-only filesystem
- [ ] Network Policies en place (deny-all par défaut)
- [ ] RBAC avec principe du moindre privilège
- [ ] PDB définis pour les services critiques
- [ ] Monitoring et alerting configurés
- [ ] Logging centralisé
- [ ] Backup etcd automatisé
- [ ] Stratégie de disaster recovery documentée
- [ ] Secrets chiffrés (pas juste encodés en base64)
- [ ] Ingress avec TLS (Let's Encrypt via cert-manager)

---

## Module 19 — Troubleshooting (essentiel CKA)

### 19.1 Méthodologie systématique

```
1. Identifier le symptôme
   └─► kubectl get pods / events / describe
2. Vérifier le node
   └─► kubectl get nodes / describe node
3. Vérifier les composants système
   └─► kubectl get pods -n kube-system
4. Vérifier les logs
   └─► kubectl logs / journalctl
5. Vérifier le réseau
   └─► kubectl exec -- nslookup / curl
6. Vérifier le stockage
   └─► kubectl get pv,pvc
```

### 19.2 Problèmes courants et solutions

**Pod en Pending** :
```bash
kubectl describe pod <name>  # Chercher dans "Events"
# Causes fréquentes :
# - Pas assez de ressources (CPU/mémoire) → kubectl describe node
# - PVC non lié → kubectl get pvc
# - NodeSelector/Affinity impossible → vérifier les labels des nodes
# - Taint sans toleration
```

**Pod en CrashLoopBackOff** :
```bash
kubectl logs <pod> --previous           # Logs du crash
kubectl describe pod <pod>              # Exit code dans containerStatuses
# Exit code 1 = erreur applicative
# Exit code 137 = OOMKilled (manque de mémoire)
# Exit code 139 = Segfault
```

**Pod en ImagePullBackOff** :
```bash
kubectl describe pod <pod>  # Vérifier le message d'erreur
# Causes : image inexistante, tag incorrect, registry privé sans credentials
# Solution : kubectl create secret docker-registry ...
```

**Service ne route pas le trafic** :
```bash
kubectl get endpoints <service>         # Liste vide = pas de Pod correspondant
kubectl get pods --show-labels          # Vérifier que les labels matchent le selector
kubectl describe svc <service>          # Vérifier selector et ports
```

**Node NotReady** :
```bash
kubectl describe node <name>            # Conditions du node
ssh <node> && sudo journalctl -u kubelet -f  # Logs kubelet
sudo systemctl status kubelet           # État du service kubelet
sudo systemctl restart kubelet          # Redémarrer
```

### 19.3 Backup et restauration etcd

```bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Vérifier le backup
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-table

# Restauration
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored

# Mettre à jour le manifeste etcd pour pointer vers le nouveau dossier
sudo vi /etc/kubernetes/manifests/etcd.yaml
# Changer --data-dir=/var/lib/etcd-restored
```

### 19.4 Mise à niveau du cluster (kubeadm)

```bash
# --- Sur le MASTER ---
# 1. Mettre à jour kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.31.x-*

# 2. Planifier la mise à jour
sudo kubeadm upgrade plan

# 3. Appliquer la mise à jour
sudo kubeadm upgrade apply v1.31.x

# 4. Mettre à jour kubelet et kubectl
sudo apt-get install -y kubelet=1.31.x-* kubectl=1.31.x-*
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# --- Sur chaque WORKER ---
# 1. Drainer le node
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# 2. Mettre à jour kubeadm, puis upgrade
sudo apt-get install -y kubeadm=1.31.x-*
sudo kubeadm upgrade node

# 3. Mettre à jour kubelet
sudo apt-get install -y kubelet=1.31.x-*
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# 4. Remettre le node en service
kubectl uncordon <node>
```

---

## Module 20 — Préparation aux certifications CKA et CKAD

### 20.1 Détails des certifications

| | CKA | CKAD |
|---|---|---|
| **Focus** | Administration du cluster | Développement d'applications |
| **Durée** | 2 heures | 2 heures |
| **Questions** | ~17 exercices pratiques | ~16 exercices pratiques |
| **Score min** | 66% | 66% |
| **Validité** | 2 ans | 2 ans |
| **Prix** | ~395 USD | ~395 USD |
| **Environnement** | Terminal Linux avec kubectl | Terminal Linux avec kubectl |

### 20.2 Domaines CKA (poids de l'examen)

1. **Storage** (10%) — PV, PVC, StorageClass, volume modes, access modes
2. **Troubleshooting** (30%) — Logs, monitoring, debugging des applications et du cluster
3. **Workloads & Scheduling** (15%) — Deployments, rolling updates, ConfigMaps, Secrets, scaling, scheduling
4. **Cluster Architecture, Installation & Configuration** (25%) — RBAC, kubeadm, cluster upgrade, etcd backup
5. **Services & Networking** (20%) — Services, Ingress, DNS, Network Policies

### 20.3 Domaines CKAD (poids de l'examen)

1. **Application Design and Build** (20%) — Containers, multi-container pods, init containers, volumes
2. **Application Deployment** (20%) — Deployments, rolling updates, Helm
3. **Application observability and maintenance** (15%) — Probes, logging, debugging
4. **Application Environment, Configuration and Security** (25%) — ConfigMaps, Secrets, SecurityContext, ServiceAccounts, Resource requirements
5. **Services & Networking** (20%) — Services, Ingress, Network Policies

### 20.4 Stratégie pour le jour J

**Avant l'examen :**

1. Pratiquer sur KillerCoda et killer.sh (simulateur officiel inclus avec l'inscription)
2. S'entraîner à créer des ressources rapidement en mode impératif
3. Maîtriser la navigation dans la documentation K8s (kubernetes.io/docs)
4. Configurer et mémoriser ses alias

**Setup initial (les 2 premières minutes)** :

```bash
# Alias essentiels
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kga='kubectl get all'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
export do='--dry-run=client -o yaml'

# Autocomplétion
source <(kubectl completion bash)
complete -o default -F __start_kubectl k

# Éditeur
export EDITOR=vim
# ou si plus à l'aise avec nano :
export EDITOR=nano
```

**Pendant l'examen :**

1. Lire TOUTES les questions d'abord et identifier les faciles (~2 min)
2. Commencer par les questions qui rapportent le plus de points
3. Si bloqué > 5 min → passer et revenir
4. TOUJOURS vérifier le contexte (chaque question peut cibler un cluster différent)
5. Utiliser `kubectl explain <resource>` quand on hésite sur un champ YAML

### 20.5 Commandes indispensables à connaître par cœur

```bash
# Génération rapide de YAML (examen = vitesse !)
k run pod1 --image=nginx $do > pod.yaml
k create deploy dep1 --image=nginx --replicas=3 $do > dep.yaml
k create svc clusterip svc1 --tcp=80:80 $do > svc.yaml
k create cm cm1 --from-literal=k=v $do > cm.yaml
k create secret generic s1 --from-literal=pass=123 $do > secret.yaml
k create job j1 --image=busybox $do > job.yaml
k create cronjob cj1 --image=busybox --schedule="*/5 * * * *" $do > cj.yaml
k create sa sa1 $do > sa.yaml
k create role r1 --verb=get,list --resource=pods $do > role.yaml
k create rolebinding rb1 --role=r1 --serviceaccount=default:sa1 $do > rb.yaml
k create ingress ing1 --rule="host/path=svc:80" $do > ingress.yaml

# Vérifications rapides
k get pods -A -o wide                  # Vue globale
k get events --sort-by=.metadata.creationTimestamp  # Chronologie
k explain pod.spec.containers.livenessProbe  # Documentation inline
```

### 20.6 Exercices d'entraînement type CKA

**Exercice 1** — Créer un Deployment "webapp" avec 3 répliques de l'image `nginx:1.25`, exposé via un Service NodePort sur le port 30080, avec des resource requests de 100m CPU et 128Mi mémoire, et une livenessProbe HTTP sur `/`.

**Exercice 2** — Créer un NetworkPolicy dans le namespace "secure" qui autorise uniquement le trafic entrant des Pods avec le label `role=frontend` sur le port 8080.

**Exercice 3** — Sauvegarder etcd, simuler une perte de données, et restaurer depuis le backup.

**Exercice 4** — Mettre à niveau un node worker de la version 1.30 à 1.31 en utilisant kubeadm.

**Exercice 5** — Un Pod en CrashLoopBackOff : diagnostiquer le problème et le corriger.

---

## Annexes

### A. Ressources d'apprentissage

| Ressource | Type | Niveau | URL |
|-----------|------|--------|-----|
| Documentation officielle K8s | Documentation | Tous | kubernetes.io/docs |
| KillerCoda | Labs interactifs gratuits | Tous | killercoda.com |
| killer.sh | Simulateur CKA/CKAD | Avancé | killer.sh |
| KodeKloud | Cours + labs | Débutant-Intermédiaire | kodekloud.com |
| Kubernetes the Hard Way (Kelsey Hightower) | Guide GitHub | Avancé | github.com/kelseyhightower/kubernetes-the-hard-way |
| CNCF Landscape | Écosystème | Référence | landscape.cncf.io |

### B. Ports importants à connaître

| Composant | Port | Protocole |
|-----------|------|-----------|
| kube-apiserver | 6443 | HTTPS |
| etcd | 2379-2380 | HTTPS |
| kubelet | 10250 | HTTPS |
| kube-scheduler | 10259 | HTTPS |
| kube-controller-manager | 10257 | HTTPS |
| NodePort range | 30000-32767 | TCP/UDP |
| CoreDNS | 53 | TCP/UDP |

### C. Cheat Sheet des objets API

```bash
# Voir toutes les ressources API disponibles
kubectl api-resources

# Ressources les plus courantes
# NAMESPACED : pods, deployments, services, configmaps, secrets, ingresses, pvc, roles, rolebindings, networkpolicies, jobs, cronjobs, statefulsets, daemonsets, hpa, serviceaccounts
# CLUSTER-WIDE : nodes, namespaces, pv, storageclasses, clusterroles, clusterrolebindings, ingressclasses, priorityclasses
```

### D. Conversions d'unités

```
# CPU
1 CPU = 1000m (millicores)
100m = 0.1 CPU = 10% d'un cœur

# Mémoire
1 Ki = 1024 bytes (kibibyte)
1 Mi = 1024 Ki = 1,048,576 bytes (mebibyte)
1 Gi = 1024 Mi (gibibyte)
# Attention : 1M (megabyte) ≠ 1Mi (mebibyte)
# K8s utilise le binaire (Ki, Mi, Gi)

# Stockage DXA (pour PV)
Gi = gibibytes
```

### E. Glossaire

| Terme | Définition |
|-------|-----------|
| **CNI** | Container Network Interface — plugin réseau pour K8s |
| **CRI** | Container Runtime Interface — interface pour les runtimes de conteneurs |
| **CSI** | Container Storage Interface — interface pour les plugins de stockage |
| **FQDN** | Fully Qualified Domain Name — nom DNS complet |
| **HPA** | Horizontal Pod Autoscaler |
| **VPA** | Vertical Pod Autoscaler |
| **PDB** | Pod Disruption Budget |
| **PSA** | Pod Security Admission |
| **RBAC** | Role-Based Access Control |
| **SLA/SLO/SLI** | Service Level Agreement/Objective/Indicator |

---

> **Félicitations !** Si tu as travaillé tous ces modules avec les exercices pratiques, tu as les connaissances nécessaires pour passer la CKA et la CKAD. La clé maintenant est la **pratique** : monte ton propre cluster, déploie des applications réelles, casse des choses et répare-les. C'est comme ça qu'on devient vraiment bon.
>
> *Dernière mise à jour : Mars 2026*
