---
title: "Exploration de l'Auto-Provisionnement de Configurations avec Kyverno"
datePublished: Thu Oct 12 2023 14:19:52 GMT+0000 (Coordinated Universal Time)
cuid: clnn9nvbq000m0amjh5fo0xtj
slug: exploration-de-lauto-provisionnement-de-configurations-avec-kyverno
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697120357461/06d0c994-334d-4dc2-aa98-84010b074328.png
tags: kubernetes, kyverno

---

Kyverno, en tant que contrôleur d'admission spécialement élaboré pour les contextes Kubernetes, assume un rôle essentiel dans l'administration et la sauvegarde des clusters Kubernetes.

Dans cet article, nous explorerons des scénarios d'usage tels que l'auto-approvisionnement des configurations de sécurité, tout en scrutant également comment identifier des changements ainsi que les méthodes pour automatiser certaines configurations par défaut.

Un contrôleur d'admission est généralement déployé pour surveiller et, si besoin bloquer les configurations qui ne sont pas en accord avec les règles de conformité définies.

Se positionnant au centre des interactions avec le serveur kube-api, le contrôleur d'admission détient un accès complet à tous les appels API et, par conséquent, est apte à diriger des actions de mutation ou de génération, basées sur des règles précises. Cela permet une gestion exacte et automatisée des ressources et des configurations au sein d'un cluster Kubernetes. Génération automatique de politiques réseau et quotas.

# Automatisation des configurations par defaut

Cette Clusterpolicy va automatiquement créer une networkpolicy ainsi qu'un resourcequotas dès que nous créerons un nouveau namespace

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: default-policy-and-quota
spec:
  rules:
  - name: create-default-network-policy
    match:
      resources:
        kinds:
        - Namespace
    generate:
      kind: NetworkPolicy
      name: default-network-policy
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          podSelector: {}
          policyTypes:
          - Ingress
          - Egress
  - name: create-default-resource-quota
    match:
      resources:
        kinds:
        - Namespace
    generate:
      kind: ResourceQuota
      name: default-resource-quota
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          hard:
            pods: "10"
            requests.cpu: "1000m"
            requests.memory: "1Gi"
            limits.cpu: "2000m"
            limits.memory: "2Gi"
```

Créons un namespace pour tester la policy:

```bash
ledan@vincent-Inspiron-7370:~/kyverno$ kubectl create ns top
namespace/top created

ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get resourcequotas -n top

NAME                     AGE   REQUEST                                                 LIMIT
default-resource-quota   3s    pods: 0/10, requests.cpu: 0/1, requests.memory: 0/1Gi   limits.cpu: 0/2, limits.memory: 0/2Gi

ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get networkpolicy -n top

NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         11s
```

Une approche qui pourrait s'avérer intéressante consiste à appliquer cette action de manière rétroactive mais de façon maîtrisée.

Pour ce faire, nous allons activer l'option `background: true`.

Cela aura pour effet d'appliquer cette politique aux namespaces existants avec une spécificité : le "generate" ne se déclenchera qu'après une modification des namespaces (quelle qu'elle soit).

Pour faire preuve de prudence, nous allons également exclure certains namespaces de la règle en ajoutant

```yaml
    exclude:
      resources:
        namespaces:
        - kube-system
        - kube-public
        - kyverno
```

Appliquons désormais la nouvelle `clusterpolicy :`

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: default-policy-and-quota
spec:
  background: true
  rules:

  - name: create-default-network-policy
    match:
      resources:
        kinds:
        - Namespace
    exclude:
      resources:
        namespaces:
        - kube-system
        - kube-public
        - kyverno
    generate:
      kind: NetworkPolicy
      apiVersion: networking.k8s.io/v1
      name: default-network-policy
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          podSelector: {}
          policyTypes:
          - Ingress
          - Egress
  - name: create-default-resource-quota
    match:
      resources:
        kinds:
        - Namespace
      exclude:
        resources:
          namespaces:
          - kube-system
          - kube-public
          - kyverno
    generate:
      kind: ResourceQuota
      apiVersion: v1
      name: default-resource-quota
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          hard:
            pods: "10"
            requests.cpu: "1000m"
            requests.memory: "1Gi"
            limits.cpu: "2000m"
            limits.memory: "2Gi"
```

À présent, nous allons attribuer des labels aux namespaces éxistant afin d'activer la politique :

```bash
ledan@vincent-Inspiron-7370:~/kyverno$ kubectl label namespaces ns1 ns2 ns3 ns4 ns5 trigger-kyverno-policy=v1
namespace/ns1 labeled
namespace/ns2 labeled
namespace/ns3 labeled
namespace/ns4 labeled
namespace/ns5 labeled

ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get networkpolicy -n ns4
NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         3s
ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get networkpolicy -n ns3
NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         5s
```

Si nous testons une modification sur le nom "kube-public", aucune action ne devrait être déclenchée :

```yaml
ledan@vincent-Inspiron-7370:~/kyverno$ kubectl label namespaces kube-public  trigger-kyverno-policy=v1
namespace/kube-public labeled

ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get networkpolicy -n kube-public
No resources found in kube-public namespace.

ledan@vincent-Inspiron-7370:~/kyverno$ kubectl label namespaces  default  trigger-kyverno-policy=v1
namespace/default labeled
ledan@vincent-Inspiron-7370:~/kyverno$ kubectl get networkpolicy -n default
NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         5s
```

Il est bien sur possible d'aller plus loin avec des controles supplémentaires comme celui-ci. le controle ci-dessous fera en sorte de n'executer la policy uniquement si le namespace ne commence pas par `admin.`

```yaml
    preconditions:
      - key: "{{request.object.metadata.name}}"
        operator: NotEquals
        value:  "admin*"
```

Testons quelques exemples:

```bash
:~/kyverno$ kubectl create ns yy
namespace/yy created

:~/kyverno$ kubectl get networkpolicy -n yy
NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         5s

:~/kyverno$ kubectl create ns admin
namespace/admin created
:~/kyverno$ kubectl get networkpolicy -n admin
No resources found in admin namespace.

:~/kyverno$ kubectl create ns adminrrrrrrrr
namespace/adminrrrrrrrr created

:~/kyverno$ kubectl get networkpolicy -n adminrrrrrrrr
No resources found in adminrrrrrrrr namespace.

:~/kyverno$ kubectl create ns admi
namespace/admi created

:~/kyverno$ kubectl get networkpolicy -n admi
NAME                     POD-SELECTOR   AGE
default-network-policy   <none>         3s
```

# Conclusion

Les possibilités sont aussi variées que les architectures elles-mêmes. La flexibilité et la puissance que ce puissant outil nous offre peuvent transformer non seulement comment nous gérons les configurations de sécurité, mais également comment nous envisageons l’automatisation au sein de nos clusters Kubernetes.

À titre d'exemple, imaginez la création automatisée de role bindings, ou de toute autre ressource et configuration, orchestrée de manière à s'aligner précisément avec les spécificités et les exigences de votre architecture.

Ainsi, qu'il s'agisse d’instancier des politiques de réseau, de définir des quotas ou d’automatiser la gestion des rôles et permissions, Kyverno nous offre un tableau sur lequel nous pouvons peindre, avec précision et habileté, les politiques de gestion et de conformité adaptées à notre environnement.