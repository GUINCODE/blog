---
title: "Kubernetes Validating Admission Policy : Introduction"
datePublished: Fri Sep 29 2023 10:29:09 GMT+0000 (Coordinated Universal Time)
cuid: cln4gp3ew000c08jl9sll16i4
slug: kubernetes-validating-admission-policy-introduction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695983311097/baea3d36-e2e3-4ffb-ac15-28bb4589a6ec.webp
tags: kubernetes

---

Depuis longtemps, au sein de l’écosystème Kubernetes, OPA (Open Policy Agent) et Kyverno sont instinctivement choisis pour mettre en place un admission controller. Plusieurs facteurs expliquent ce choix, notamment les lacunes de l’API Kubernetes à cet égard ou la flexibilité d’utiliser OPA dans d’autres scénarios comme l’infrastructure as code. Cependant, à l’heure actuelle, Kubernetes propose-t-il nativement des solutions matures et pertinentes sur cette thématique? 

Le concept a été lancé avec la version 1.26 de Kubernetes et a atteint le stade beta dans la version 1.28. 

L’un des points forts de cette fonctionnalité est son intégration directe à l’API de Kubernetes, éliminant le besoin de recourir à des tiers supplémentaires. Cela diminue ainsi l’effort requis pour la maintenance.

### Validating Admission components  

La logique de fond reste la même , si on prends l’exemple d’OPA par exemple : 

Nous avons ces deux composants : 

* **ConstraintTemplate** qui défini la logique de la policy et qui contient le controle en language **Rego** à éffectuer par exemple : *“maximum de replicas d’un déploiement ne peux pas éxéder 5”*
    
* **Template** utilise cette policy pour l’appliquer à tels ou tels composant POD , filtre la cible avec des parameters …. 
    

Si on prends l’exemple de cette même policy mais pour la version native de Kubernetes , les composants principaux sont: 

* **ValidatingAdmissionPolicy** qui défini la policy de controle mais cette fois ci en language **CEL** 
    
* **ValidatingAdmissionPolicyBinding** est le même principe qu’un template il filtre le scope et parametre selon le contexte. 
    

### Validation admission controller in action

Afin de pouvoir tester cette feature, il est important d’avoir un environnement qui le permet. Minikube est vraiment intérressant pour tester de nouvelles Features comme celle-ci. 

```bash
minikube start  --feature-gates='ValidatingAdmissionPolicy=true' \  
--kubernetes-version=1.28.0
```

Nous aimerions restreindre la possibilité de créer des services de type “NodePort” mais plutôt de centraliser l’exposition des applications avec un ingress ou un Kubernetes API gateway.:

Prenons maintenant un exemple avec OPA pour ce faire : 

```bash
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sblocknodeport
  annotations:
    metadata.gatekeeper.sh/title: "Block NodePort"
    metadata.gatekeeper.sh/version: 1.0.0
    description: >-
      Disallows all Services with type NodePort.

      https://kubernetes.io/docs/concepts/services-networking/service/#nodeport
spec:
  crd:
    spec:
      names:
        kind: K8sBlockNodePort
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblocknodeport

        violation[{"msg": msg}] {
          input.review.kind.kind == "Service"
          input.review.object.spec.type == "NodePort"
          msg := "User is not allowed to create service of type NodePort"
        }
```

```bash
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sBlockNodePort
metadata:
  name: block-node-port
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Service"]
```

Maintenant voyons comment faire cette équivalent via l’admission controller natif de kubernetes : 

**Validatingadmissionpolicy**: 

```bash
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingAdmissionPolicy
metadata:
  name: deny-nodeport
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups:   [""]
      apiVersions: ["v1"]
      operations:  ["CREATE", "UPDATE"]
      resources:   ["services"]
  validations:
    - expression: object.spec.type != "NodePort" 
```

**Validatingadmissionpolicybinding**: 

```bash
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "vincent-binding-test.example.com"
spec:
  policyName: "deny-nodeport"
  validationActions: [Deny]
  matchResources:
    namespaceSelector:
      matchLabels:
        env: prod
```

Le Kind ValidatingAdmissionPolicyBinding va dans notre cas appliquer cette policy sur l’ensemble des namespaces ayant un label **env=prod**

Maintenant faisons un petit test en déployant un simple service N**odePort** dans un namespace ayant un label **env=prod**

```bash
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: toto
  name: toto
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: toto
  type: NodePort
```

```bash
kubectl apply -f service.yaml -n titi 

The services "toto" is invalid: 
: ValidatingAdmissionPolicy 'deny-nodeport' with binding 
'deny-service-nodeport' denied request: failed expression:
 object.spec.type != "NodePort"
```

Nous pouvons observer que la Requête à été bloquée par Kubernetes. 

### Conclusion 

Cette feature n’est pas encore ready for production mais va dans le bon sens. si on compare à Kyverno par exemple , Kyvernoo offre plus de flexibilité ou d’automatisation , par exemple en fournissant la capacité a provisionner de façon automatique des kinds dans certains namespace. 

Si on parle d’OPA , lui est utilisable dans bien d’autres contextes que Kubernetes , Policy as code , authentication on demand et beaucoup d’autres contextes.