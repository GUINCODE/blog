---
title: "New Kubernetes security workshop chez Point Base"
datePublished: Fri Sep 29 2023 10:09:36 GMT+0000 (Coordinated Universal Time)
cuid: cln4fzyjx000z09mnc51239z0
slug: new-kubernetes-security-workshop-chez-point-base
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695982276208/21b6e9c5-fb23-4aab-bbd6-e413fdc8e9b0.png
tags: kubernetes, cloud-native-security

---

Nous avons le plaisir de lancer notre nouveau “ Kubernetes security workshop “ chez Point Base , mais de quoi parlons nous exactement.

Tout d’abord pourquoi proposons nous cela, la réponse est plutot simple, faire prendre conscience des problématiques ou mauvaises pratiques quant à la gestion et la sécurité dans ce types d’environnements.

Il est parfois facile de se perdre dans l’écosystème cloud natif sécurity , la sortie de nouveaux outils , des évolutions extrêmement rapides et complexes.

Notre Objectif est simple passer une journée avec vos équipes touchant de près ou de loin ce types écosystème ou nous repasserons en revue l’ensemble des problématiques de sécurité tant techniques mais également organisationnelles , les vecteurs d’attaques courants , les outils les plus pertinent , les dernières innovations , les bests practices dans le domaine et ce afin qu’a la fin de cette journée vous ayez une vision claire de la priorisation des efforts à mener.

Parfois cela pourra vous rassurer sur vos actions en cours , mais parfois cela vous fera probablement réfléchir de façon différente quant à la priorisation ou aux choix des outils.

Nous échangerons autour des problématiques réelles

#### Default cloud configuration :

Il est très courant que nos clients utilisent les configurations par défaut des providers, ou par exemple des modules terraform. Bien souvent, ces configurations ne sont pas suffisantes en termes de sécurité. Nous verrons pourquoi.

#### Networking :

Comme j’aime le rappeler, dans un cluster Kubernetes, tous les flux sont ouverts, non chiffrés et non contrôlés. Cela concerne les interactions entre les différents pods ou namespaces, mais aussi comment les pods peuvent communiquer avec l’extérieur ou scanner votre réseau VPC, entre autres. Un autre aspect concerne le contrôle des flux entrants et sa consolidation via des logiques d’ingress ou d’API gateway.

#### Admission controller et Gouvernance :

Dans un cluster Kubernetes, il est très facile de devenir admin, de corrompre les nœuds des clusters, car aucune restriction pour devenir root ou utiliser des capabilities avancées comme CAP\_SYSADMIN n’est en place. La gestion des labels, la gouvernance, l’uniformité et la sécurité des déploiements, ainsi que leur protection au niveau du cluster doivent être implémentées.

#### Runtime protection / Sandboxing

Cette partie est souvent oubliée mais très importante. Par défaut, les hosts et l’activité des containers ne sont pas protégés contre des activités suspectes dans les containers (ex : cat /etc/…./credentials). Pour détecter ce genre d’activités, vous devez premièrement comprendre les risques et les possibilités des attaquants, mais également mettre en place un ensemble de règles pour détecter ou bloquer. Nous verrons plusieurs approches ensemble, par exemple “detection and response” ou “Block syscall”. Ce sont des décisions de sécurité.

#### 3rd Party risks (argocd — gitlab…)

Bien que vous contrôliez la majeure partie de vos workloads tout au long de la chaîne, certaines composantes ne peuvent être contrôlées. Alors, quelle stratégie avez-vous adoptée ? Sandboxing container, cluster isolation, whitelist container registry ?

#### Surveillance et monitoring

Si nous faisons un focus sur les sujets de monitoring, plusieurs facteurs sont à prendre en compte. Que souhaitez-vous détecter ? Analyse comportementale des containers, activités suspectes sur le cluster, création de rôles RBAC suspects, activités suspectes sur vos buckets S3 ? Il y a beaucoup à dire sur le sujet.

#### Quotas et gestion des ressources

À la croisée des chemins entre les préoccupations des SRE et celles de vos équipes sécurité, il est crucial de contrôler à quel point et dans quelles conditions votre infrastructure peut évoluer (scale). Imaginons qu’un container soit compromis et souhaite perturber certains services. Êtes-vous en mesure, par exemple, de gérer et d’imposer des règles sur la consommation réseau de vos pods ?

#### Gestion des identités utilisateur et des workloads

Avoir une bonne gestion des droits et des groupes IAM n’est pas une nouveauté. Toutefois, comprendre les implications ou la puissance de certains “cluster roles”, par exemple (VERBS=GET, Kind=secret = Admin), est crucial. Trouver le juste milieu entre sécurité et flexibilité n’est pas toujours aisé.

Ce que l’on constate souvent, c’est une gestion inadéquate de l’authentification entre vos différents pods et des services cloud tels que DynamoDB, S3 buckets ou Cloud SQL chez GCP. Historiquement, l’utilisation des “access keys” intégrées secrètement était couramment utilisée. Toutefois, depuis un certain temps, les fournisseurs cloud ont développé des mécanismes permettant une approche plus granulaire et sans clé (ex : “workload identity” chez GCP).

#### Sécurité des workloads et des containers

Avez-vous mis en place des templates de déploiement ou des helm charts avec les configurations de sécurité les plus appropriées ? Comment vos images de containers sont-elles construites : multi-step, distroless, scan de vulnérabilités, scan du registre ? Comment priorisez-vous les dizaines de CVEs remontées par vos outils ? Autant de questions auxquelles nous [réfléchirons.Cloud](http://xn--rflchirons-b7ac.Cloud) security

Votre flotte de clusters Kubernetes s’intégre logiquement dans une landing zone mais est-elle vraiment conçu pour être assez isoler , backuper vous vos clusters ? avez vous des procédures ou de l’automatisation pour déplacer vos workloads d’un cloud à un autre ?

#### Day2

Quand on parle de sécurité dans ce domaine, de nombreux outils open source sont efficaces. Cependant, assurez-vous d’avoir des équipes bien formées et suffisamment staffées pour garantir une continuité d’exploitation. Recourir sans cesse à de nouveaux outils pour pallier les manques d’autres n’est pas toujours la meilleure stratégie. Avez-vous mis en place des outils pour détecter les nouvelles CVEs exploitables dans vos workloads et des processus pour y remédier rapidement et à grande échelle ?

#### Sensibilisation à la sécurité offensive

Il est toujours intéressant d’adopter une perspective critique et offensive lors de la sécurisation d’un cluster. Comment pourrais-je contourner cette règle ? Comment pourrais-je compromettre notre CI/CD ? La gamification est un outil puissant pour changer les mentalités. Trois heures pour devenir administrateur du cluster : en discutons-nous ?

Nous discuterons plus en détail sur une série d’attque éffectuée sur des environnements Kubernetes : SCARLETEEL par exemple.

#### Gestion multi-cluster

Une fois que vous avez assimilé tous les points mentionnés précédemment, posez-vous la question : mes processus et mon architecture sont-ils adaptés à une montée en puissance ? Pensez à une échelle 100 fois supérieure. Mettez en œuvre davantage d’automatisation, une approche GitOps, et choisissez une source d’information unique.

Contactez nous dès maintenant pour avoir plus de détails.