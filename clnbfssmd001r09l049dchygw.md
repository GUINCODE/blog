---
title: "KubeArmor : Inline remediation security"
datePublished: Wed Oct 04 2023 07:38:25 GMT+0000 (Coordinated Universal Time)
cuid: clnbfssmd001r09l049dchygw
slug: kubearmor-inline-remediation-security
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696405018358/6436c8fe-3916-4206-a012-886e5380e13c.jpeg
tags: security, kubernetes, cloud-native

---

KubeArmor est une solution de sécurité open source qui permet de renforcer la sécurité des conteneurs sur Kubernetes en temps réel. Il s'agit d'un système de protection des conteneurs qui utilise des techniques de contrôle d'accès et de remédiation en ligne ( inline remediation ) pour empêcher les attaques.

L'approche choisi est de bloquer directement les comportements arnormaux avant qu'ils ne se produisent sans impacter l'application.

KubeArmor est également utilisable en dehors de Kubernetes et s'installe sur des serveurs linux , Bare metal etc...

**Points forts de KubeArmor**

* **Protection en temps réel** : KubeArmor protège les conteneurs en temps réel, ce qui permet de bloquer les attaques dès qu'elles se produisent.
    
* **Rémédiation en ligne** : KubeArmor peut corriger les violations de sécurité en ligne, ce qui permet de réduire les dommages.
    
* **Flexibilité** : KubeArmor permet aux administrateurs de définir des politiques de sécurité personnalisées pour répondre aux besoins spécifiques de leur environnement.
    
* **Simplicité** : KubeArmor est facile à configurer et à utiliser.
    
* **Performance** : KubeArmor a un impact minimal sur les performances des conteneurs.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696404761438/8311704f-4df4-4d98-a125-d7c63b319528.png align="center")

**Différenciateurs**

* **Prise en charge de plusieurs LSM** : KubeArmor prend en charge plusieurs modules de sécurité du noyau Linux (LSM), tels que AppArmor et Seccomp. Cela permet aux administrateurs de choisir l'LSM qui convient le mieux à leurs besoins.
    
* **Audit et reporting** : KubeArmor fournit des journaux d'audit et des rapports qui peuvent être utilisés pour surveiller l'activité des conteneurs.
    
* **Intégration avec d'autres outils** : KubeArmor peut être intégré à d'autres outils de sécurité, tels que SIEM et SOAR.
    

## Kubearmor installation

Installons Kubearmor

```bash
helm repo add kubearmor https://kubearmor.github.io/charts
helm repo update kubearmor
helm upgrade --install kubearmor-operator kubearmor/kubearmor-operator -n kubearmor --create-namespace
```

et appliquon la CRD Kubearmor , pour notre exemple nous avons choisi une implémentation simple ( [Documentation](https://github.com/kubearmor/KubeArmor/tree/main/deployments/helm/KubeArmorOperator))

```yaml
apiVersion: operator.kubearmor.com/v1
kind: KubeArmorConfig
metadata:
  labels:
    app.kubernetes.io/name: kubearmorconfig
    app.kubernetes.io/instance: kubearmorconfig-sample
    app.kubernetes.io/part-of: kubearmoroperator
    app.kubernetes.io/managed-by: kustomize
    app.kubernetes.io/created-by: kubearmoroperator
  name: kubearmorconfig-default
  namespace: kubearmor
spec:
  defaultCapabilitiesPosture: audit
  defaultFilePosture: audit
  defaultNetworkPosture: audit
  defaultVisibility: process,file,network
  kubearmorImage:
    image: kubearmor/kubearmor:stable
    imagePullPolicy: Always
  kubearmorInitImage:
    image: kubearmor/kubearmor-init:stable
    imagePullPolicy: Always
  kubearmorRelayImage:
    image: kubearmor/kubearmor-relay-server
    imagePullPolicy: Always
  kubearmorControllerImage:
    image: kubearmor/kubearmor-controller
    imagePullPolicy: Always
```

### Kubearmorpolicy implementation

Kubearmor fourni la possibilité de créer de nombreuses règles bloquant les syscall , le réseau , les capabilities kernel , les accès aux différents fichiers, les process.

Prenons quelques exemples

**<mark>Process exemple : deny apt command</mark>**

```yaml
apiVersion: security.kubearmor.com/v1
kind: KubeArmorPolicy
metadata:
  name: ksp-wordpress-block-process
  namespace: default
spec:
  severity: 3
  selector:
    matchLabels:
      app: nginx
  process:
    matchPaths:
    - path: /usr/bin/apt
    - path: /usr/bin/apt-get
  action: Block
```

Test :

```bash
vincn_ledan@cloudshell:~/kubearmor (medium-article)$ kubectl exec -it nginx-77b4fdf86c-vckbc -- bash 
root@nginx-77b4fdf86c-vckbc:/# apt
apt 2.6.1 (amd64)
Usage: apt [options] command

apt is a commandline package manager and provides commands for
searching and managing as well as querying information about packages.
It provides the same functionality as the specialized APT tools,
like apt-get and apt-cache, but enables options more suitable for
interactive use by default.

Most used commands:
  list - list packages based on package names
.........................................................
```

Apres application de la policy

```yaml
incn_ledan@cloudshell:~/kubearmor (medium-article-377208)$ kubectl exec -it nginx-77b4fdf86c-vckbc -- bash 
root@nginx-77b4fdf86c-vckbc:/# apt
bash: /usr/bin/apt: Permission denied
```

**<mark>File exemple : deny cat command</mark>**

```yaml
apiVersion: security.kubearmor.com/v1
kind: KubeArmorPolicy
metadata:
  name: ksp-block-read
  namespace: default # use your namespace
spec:
  selector:
    matchLabels:
      app: nginx # use your labels here
  file:
    severity: 9
    matchPaths:
    - path: /etc/passwd
    action: Block
```

le résultat est le suivant :

```bash
I have no name!@nginx-77b4fdf86c-vckbc:/# cat /etc/hostname
nginx-77b4fdf86c-vckbc
I have no name!@nginx-77b4fdf86c-vckbc:/# cat /etc/passwd  
cat: /etc/passwd: Permission denied
```

Network Kubearmor policy

```yaml
apiVersion: security.kubearmor.com/v1
kind: KubeArmorPolicy
metadata:
  name: ksp-block-icmp
  namespace: default
spec:
  severity: 8
  selector:
    matchLabels:
      app: nginx
  network:
    matchProtocols:
    - protocol: icmp
  action:
    Block
```

Le résultat est le suivant :

```bash
I have no name!@nginx-77b4fdf86c-vckbc:/# ping 8.8.8.8
ping: 8.8.8.8: Address family for hostname not supported
```

### Automatic policy discovery with Accuknox

Kubearmor fourni une feature très intérressante qui est l'application profiling afin de créer des policies custom basé sur le comportement des applications.

Nous allons utiliser la solution Accuknox pour ce faire et enregistrer simplement notre cluster GKE dans l'interface.

```bash
 helm upgrade --install accuknox-agents oci://public.ecr.aws/k9v9d5v2/accuknox-agents \
      --version "v0.1.5" \
      --set joinToken="REDACTED" \
      --set spireHost="REDACTED" \
      --set ppsHost="REDACTED" \
      --set knoxGateway="REDACTED" \
      -n accuknox-agents --create-namespace
```

En 1 ou 2 minutes l'agent discovery engine va analyser le comportement de l'application , les calls système et créer sur mesure une KubeArmorPolicy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696349835060/fc31fbf2-df19-48e0-b91a-82d4fb2e5f4c.png align="center")

Nous avons uniquement à activer la policy dans l'interface ou l'exporter et appliquer une logique Gitops.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696350109008/f51847ec-bc74-41da-b328-4e56db29e2b0.png align="center")

Lorsque la policy est en mode innactive, le deamon va détecter les nouveaux calls système , accès aux fichiers et adapter de façon automatique la policy et demander si nous validons ou non ce nouveau comportement.

# Next step

Dans un prochain article nous verrons les points suivants:

* Création automatique des networks policy avec Cilium et Kubearmor
    
* App behavior visibility