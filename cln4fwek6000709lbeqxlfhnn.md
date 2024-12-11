---
title: "Falco: Surveillez et Neutralisez les Menaces dans Votre Cluster Kubernetes"
datePublished: Fri Sep 29 2023 10:06:50 GMT+0000 (Coordinated Universal Time)
cuid: cln4fwek6000709lbeqxlfhnn
slug: falco-surveillez-et-neutralisez-les-menaces-dans-votre-cluster-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695982370803/e866b4ce-b243-4d04-a670-ce7ce31969a6.png
tags: kubernetes, kubernetes-container, kubernetes-security

---

Falco est un outil de détection des comportements anormaux et non désirés au sein des applications en conteneurs. Il offre une surveillance en temps réel des activités des conteneurs et des machines hôtes, aidant ainsi les équipes de sécurité à maintenir la sûreté et la conformité des systèmes. Récemment, Falco a introduit une nouvelle interface graphique, nommée Falco Sidekick UI (falcosidekick), qui vise à rendre l’utilisation et la gestion de Falco encore plus pratique et efficace.

Dans cet article, nous examinerons de plus près ce nouvel outil en mettant l’accent sur son déploiement ainsi que sur la création d’une règle personnalisée pour Falco. De plus, nous explorerons en détail l’interface utilisateur de Falco Sidekick UI, une addition bienvenue qui augmente la facilité d’utilisation de Falco.

Prérequis : Avoir un cluster Kubernetes en fonctionnement, avoir kubectl installé et installer helm.

### Déploiement de Falco

Falco offre par défaut de nombreuses règles prédéfinies se basant sur les meilleures pratiques de sécurité les plus courantes. Dans le cadre de cet article, nous allons surcharger ces règles par défaut avec nos propres règles personnalisées. Pour ce faire, nous allons créer notre fichier custom-rules.yaml.

```bash
customRules:
  custom-rules.yaml: |-
    - rule: detect_ls_command
      desc: Detect execution de la commande 'ls' dans un conteneur
      condition: evt.type = execve and evt.dir = < and container.id != host and proc.name = ls
      output: "Execution de la commande 'ls' dans un conteneur (utilisateur=%user.name commande=%proc.cmdline)"
      priority: WARNING
 
    - rule: detect_date_command
      desc: Detect execution de la commande 'date' dans un conteneur
      condition: evt.type = execve and evt.dir = < and container.id != host and proc.name = date
      output: "Execution de la commande 'date' dans un conteneur (utilisateur=%user.name commande=%proc.cmdline)"
      priority: CRITICAL
    - rule: detect_whoami_command
      desc: Detecte execution de la commande 'whoami' dans un conteneur
      condition: evt.type = execve and evt.dir = < and container.id != host and proc.name = whoami
      output: "Execution de la commande 'whoami' dans un conteneur (utilisateur=%user.name commande=%proc.cmdline)"
      priority: ERROR
```

Dans ce fichier custom-rules.yaml, nous avons écrit 3 règles. La première a pour objectif de détecter l’exécution de la commande ls dans le shell d’un conteneur et d’envoyer une notification de priorité WARNING. Maintenant, passons à l’installation de Falco.

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
kubectl create ns falco
```

Dans la commande qui suit, nous procédons à l’installation de Falco et à l’activation de Falcosidekick. Cela nous offre une interface graphique et la possibilité d’envoyer des notifications sur les canaux spécifiés. Dans notre contexte, nous avons opté pour l’utilisation de Slack.

Note : Dans notre configuration, toutes les notifications d’un niveau supérieur ou égal à DEBUG seront transmises sur notre canal Slack.

```bash
helm install falco -n falco - namespace falco -f custom-rules.yaml - set driver.kind=ebpf - set tty=true falcosecurity/falco \
 - set falcosidekick.enabled=true \
 - set falcosidekick.webui.enabled=true \ 
 - set falcosidekick.config.slack.webhookurl=https://hooks.slack.com/services/REDACTED \
 - set falcosidekick.config.slack.minimumpriority=DEBUG \
 - set falcosidekick.config.customfields="user:barrybagata96"
```

On vérifie que tout s’est bien passé

```bash
$ kubectl get pods -n falco
NAME                                      READY   STATUS    RESTARTS   AGE
falco-7dvcj                               2/2     Running   0          3h44m
falco-cpd44                               2/2     Running   0          3h44m
falco-falcosidekick-54fffbc85b-f2xh2      1/1     Running   0          3h44m
falco-falcosidekick-54fffbc85b-nsfn5      1/1     Running   0          3h44m
falco-falcosidekick-ui-5ffc494bf8-b8wqk   1/1     Running   0          77m
falco-falcosidekick-ui-5ffc494bf8-pqzv6   1/1     Running   0          77m
falco-falcosidekick-ui-redis-0            1/1     Running   0          3h44m
falco-rsbqg                               2/2     Running   0          3h44m
falco-sl9j8                               2/2     Running   0          3h44m
barrybagata96@cloudshell:~/exo-falco (kubernetese-project-399608)$
```

Maintenant, testons les règles Falco que nous avons écrites dans custom-rules.yaml.

```bash
kubectl create deploy nginx --image=nginx
 
kubectl exec -it nginx-app-5c64488cdf-dnjm7 -- bash
```

```bash
root@nginx-app-5c64488cdf-dnjm7:/# ls
bin   dev                  docker-entrypoint.sh  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   lib64  media   opt  root  sbin  sys  usr
```

```bash
kubectl logs -l app.kubernetes.io/name=falco -n falco -c falco
```

```bash
..............................................................
..............................................................
                 logs filtrées
..............................................................
..............................................................
```

```bash
{"hostname":"falco-rsbqg","output":"12:11:59.619762538: 
Warning Ex\u00e9cution de la commande 'ls' dans un conteneur 
(utilisateur=root commande=ls) container_id=fca790abe8c7 
container_image=docker.io/library/nginx container_image_tag=latest 
container_name=nginx k8s_ns=default k8s_pod_name=nginx-app-5c64488cdf-dnjm7",
"priority":"Warning","rule":"detect_ls_command","source":"syscall","tags":[],
"time":"2023-09-28T12:11:59.619762538Z", "output_fields": 
{"container.id":"fca790abe8c7","container.image.repository":"docker.io/library/nginx",
"container.image.tag":"latest","container.name":"nginx","evt.time":1695903119619762538,
"k8s.ns.name":"default","k8s.pod.name":"nginx-app-5c64488cdf-dnjm7",
"proc.cmdline":"ls","user.name":"root"}}
```

On peut retrouver notre alerte sur slack:

![](https://cdn-images-1.medium.com/max/800/0*hT9_dFhqAhNyeLe3.png align="left")

### Présentation de Falco Sidekick UI

Falco fournit une interface graphique facilitant la lecture des événements et des notifications. Pour ce test, nous avons simplement exposé le service falco-falcosidekick-ui en tant que service de type LoadBalancer pour pouvoir y accéder depuis notre navigateur.

![](https://cdn-images-1.medium.com/max/800/0*cgWyEI2pmF9BM3eE.png align="left")

Nous pouvons constater que la règle detect\_ls\_command a bien été affichée avec la priorité WARNING que nous avons spécifiée dans le fichier custom-rule.yaml.

![](https://cdn-images-1.medium.com/max/800/0*eGBuQrs3Ei54c4K5.png align="left")

### Conclusion

En conclusion, notre exploration actuelle n’a fait qu’effleurer la surface des immenses possibilités et intégrations offertes par Falco. Cet outil offre une panoplie de fonctionnalités pour la surveillance et la sécurisation de vos environnements Kubernetes, et son potentiel va bien au-delà de ce que nous avons vu jusqu’à présent. Dans les prochains articles, nous plongerons plus profondément dans les capacités de Falco, explorant davantage ses fonctionnalités avancées, ses options de configuration et d’intégration, ainsi que les meilleures pratiques pour l’utiliser efficacement dans divers scénarios. Restez à l’écoute pour en apprendre davantage sur la puissance et la flexibilité que Falco peut apporter à votre infrastructure de conteneurs !