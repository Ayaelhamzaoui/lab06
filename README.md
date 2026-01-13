# Lab 6 – Déploiement Kubernetes d’un système MLOps Churn
## Objectif du laboratoire

Ce Lab a pour objectif de mettre en place et de déployer un système MLOps complet de prédiction du churn client sur Kubernetes, en s’appuyant sur un cluster local Minikube.

L’approche suivie couvre l’ensemble du cycle de vie MLOps :

-entraînement initial d’un modèle de machine learning,

-déploiement d’une API d’inférence,

-gestion de la configuration et des secrets,

-persistance des modèles et des logs,

-supervision de la santé de l’application,

-sécurité réseau interne,

-monitoring et détection de dérive.

## Contexte MLOps

Dans un environnement industriel, un système de machine learning ne se limite pas à l’entraînement d’un modèle. Il doit être :

-reproductible,

-déployable,

-monitorable,

-robuste aux pannes,

-scalable.

Kubernetes est utilisé ici comme orchestrateur standard pour répondre à ces exigences, en remplaçant une approche locale ou mono-conteneur par une architecture cloud-native.

## Architecture générale du système

Le système repose sur les composants suivants :

1- API Churn (FastAPI)
  Expose des endpoints de prédiction et de santé.

2- Job Kubernetes d’entraînement
Lance l’entraînement initial du modèle et initialise les artefacts MLOps.

3-Persistent Volume (PVC)
Permet de conserver :

les modèles entraînés,

le registre de versions,

les fichiers de logs,
même en cas de redémarrage des Pods.

4- ConfigMap
Contient la configuration applicative (nom du modèle, niveau de logs).

5- Secret Kubernetes
Stocke les informations sensibles utilisées pour le monitoring.

6- Service NodePort
Expose l’API à l’extérieur du cluster.

7- NetworkPolicy
Restreint les communications réseau aux seuls services autorisés.

## Organisation du projet

Le projet est structuré de manière modulaire afin de respecter les bonnes pratiques MLOps :

un dossier dédié au code applicatif (API, entraînement, monitoring),

un dossier pour les données,

un registre de modèles versionnés,

un dossier pour les logs,

un dossier k8s regroupant tous les manifests Kubernetes.

Cette organisation facilite :

la maintenance,

la traçabilité,

l’industrialisation du projet.

## Conteneurisation de l’API

L’API Churn est encapsulée dans une image Docker versionnée.
L’utilisation d’un tag explicite garantit la reproductibilité des déploiements et évite les erreurs liées à l’usage du tag latest.

L’image contient :

l’API FastAPI,

les dépendances Python verrouillées,

les scripts MLOps (entraînement, monitoring).

## Déploiement Kubernetes

Le déploiement Kubernetes utilise :

plusieurs réplicas pour la haute disponibilité,

un Service pour l’exposition réseau,

des probes pour surveiller l’état de l’application.

Kubernetes est ainsi capable de :

redémarrer automatiquement un Pod défaillant,

ne diriger le trafic que vers les Pods prêts,

détecter les problèmes au démarrage.

## Supervision de la santé de l’application

Trois endpoints de santé sont exposés par l’API :

1-Health : vérifie l’état global de l’application.

2-Startup : confirme que le modèle courant est bien disponible.

3-Ready : indique si l’API est prête à recevoir du trafic.

Ces endpoints sont utilisés par Kubernetes pour garantir la résilience du service.

## Gestion de la configuration et des secrets

Les paramètres applicatifs non sensibles sont gérés via un ConfigMap.

Les informations sensibles sont stockées dans un Secret Kubernetes, encodé et injecté sous forme de variables d’environnement.

Cette séparation respecte les bonnes pratiques de sécurité MLOps.

## Persistance des modèles et des logs

Un volume persistant est utilisé pour :

conserver les modèles entraînés,

partager les artefacts entre le Job d’entraînement et l’API,

stocker les logs applicatifs.

Cela garantit la continuité du système même en cas de redéploiement.

## Entraînement via Job Kubernetes

L’entraînement du modèle est réalisé à l’aide d’un Job Kubernetes indépendant de l’API.

Ce Job :

entraîne un premier modèle,

met à jour le registre MLOps,

définit le modèle courant utilisé par l’API.

Cette approche sépare clairement :

la phase d’entraînement,

la phase d’inférence.

## Sécurité réseau

Une NetworkPolicy est mise en place afin de :

limiter les flux entrants vers l’API,

autoriser uniquement les communications internes nécessaires.

Cela renforce la sécurité du cluster.

## Monitoring et détection de dérive

Un script de détection de dérive des données est intégré au système.
Il peut être exécuté directement dans un Pod afin d’analyser :

les nouvelles données,

l’écart avec les distributions d’entraînement.

Cette étape est essentielle dans une démarche MLOps avancée.

✅ Résultats obtenus

À l’issue de ce laboratoire, le système présente :

une API de churn fonctionnelle sur Kubernetes,

un pipeline MLOps structuré et persistant,

une gestion robuste de la configuration et des secrets,

une supervision complète de la santé applicative,

une base solide pour une mise en production réelle.

## Conclusion

Ce lab illustre une implémentation réaliste d’un système MLOps cloud-native, en combinant :

Docker,

Kubernetes,

FastAPI,

bonnes pratiques DevOps et MLOps.

Il constitue une base fiable pour des projets industriels de machine learning déployés à grande échelle.
