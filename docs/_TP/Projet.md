---
layout: post
title: "Projet -- Mise en pratique du patron Microservices"
categories: jekyll update

mathjax: true
---

# Projet -- Mise en pratique du patron Microservices

- TP sur machine en salle TP et à la maison
- Travail en groupe de 4 étudiants
- Compétence 1 – « développer », Compétence 2 « administrer » et Compétence 6 – « collaborer »


## Le contexte
L'objectif de ce projet est de développer une application web pour une entreprise livrant des repas à des personnes inscrites. L'entreprise propose différents plats (p.ex. salade niçoise, aïoli, gratin dauphinois, etc.) et les clients peuvent combiner ces plats pour constituer des menus pour une personne. Les abonnés peuvent ensuite commander ces menus en ligne et ils leur sont livrés à domicile à une date précise. Le paiement se fera au moment de la livraison.

## L'application

### Description
L'application web devra permettre de visualiser les plats disponibles, leur description et leur prix. Elle devra permettre de composer des menus en y ajoutant/supprimant des plats. Le nom de la personne ayant créé le menu est enregistré, tout comme la date de création (ou de mise à jours). Les menus pourront ensuite être commandés par les abonnés (tous). Un abonné peut commander plusieurs menus dans les quantités souhaitées. Lorsqu'une personne validera une commande, l'application enregistrera les menus associés, enregistrera la date de commande, calculera leur prix au moment de la commande, et enregistrera l'adresse et la date de livraison.

Pour des questions de temps, on supposera que les fonctionnalités associées à l'enregistrement et la mise à jours des plats proposés par l'entreprise ne seront pas intégrées dans l'interface graphique de l'application, tout comme celles liées à l'inscription des abonnés. Par contre, elles seront implémentées dans l'application et prêtes à être intégrer dans l'interface graphique.

### Contraintes non fonctionnelles
L'application suivra une architecture orientée services (RESTful). Elle sera constituée des 4 composants logiciels suivants: 
- le composant "IHM" gérera la partie interface graphique avec l'utilisateur; 
- le composant "Plats et Utilisateurs" gérera l'accès aux données des plats et des utilisateurs;
- le composant "Menus" gérera les opérations sur les menus 
- le composant "Commandes" gérera la partie commande

Le composant "IHM" sera codé en HTML/CSS/PHP. Les autres composants seront codés avec Jakarata EE sous forme d'API REST implémentant les opérations CRUD. Chaque composant sera codé indépendamment, par un étudiant différent, et utilisera une base de données dédiée.

Par ailleurs, le composant "Menus" devra consommer l'API mise à disposition par le composant "Plats et Utilisateurs" pour son fonctionnement (en évitant de dupliquer trop d'informations dans sa base de données). 

Le composant "Commandes" consommera quant à lui l'API du composant "Menus".

Le schéma suivant résume cette architecture:

<img src="projet-img/architecture_livraison_repas.jpg" width="600px"/>
 
## Développer son composant grâce à JSON-Server

Comme vous pouvez le constater, les API "Menus", "Commandes" et "IHM" dépendent d'autres composants. Afin de ne pas devoir attendre le développement des composants associés, vous pourrez utiliser le logiciel [JSON-Server](https://www.npmjs.com/package/json-server?activeTab=readme). Il permet  d'accélérer le prototypage de vos applications en imitant les APIs dont vous avez besoin, sans attendre que les autres membres du projet aient leurs endpoints opérationnels. Cet outil permet de créer un mock d'API REST en local en utilisant simplement un fichier JSON.

Plusieurs services ont été préparés pour ce projet. Vous n'avez plus qu'à les utiliser.

### Services disponibles

| Service              | Port | Données disponibles     |
| -------------------- | ---- | ----------------------- |
| Plats & Utilisateurs | 3003 | 8 plats, 4 utilisateurs |
| Menus                | 3004 | 4 menus                 |
| Commandes            | 3005 | 3 commandes             |

Les fichiers de données JSON, tout comme la description des endpoints (au format YAML), sont disponibles ci-dessous:

- [plats-utilisateurs.json](projet-data/plats-utilisateurs.json) et [openapi-plats-utilisateurs.yaml](projet-data/openapi-plats-utilisateurs.yaml)
- [menus.json](projet-data/menus.json) et [openapi-menus.yaml](projet-data/openapi-menus.yaml)
- [commandes.json](projet-data/commandes.json) et [openapi-commandes.yaml](projet-data/openapi-commandes.yaml)


### Faire tourner le mock en local avec `json-server`

#### 1. Installer `json-server`

```bash
npm install json-server
```

#### 2. Lancer chaque service sur le port correspondant

Ouvrez trois terminaux (ou trois onglets) et lancez une commande dans chacun :

```bash
npx json-server --watch plats-utilisateurs.json --port 3003
```

```bash
npx json-server --watch menus.json --port 3004
```

```bash
npx json-server --watch commandes.json --port 3005
```

Vos services sont maintenant disponibles sur `http://localhost:3003`, `http://localhost:3004` et `http://localhost:3005`.

> **Note :** `json-server` génère automatiquement des routes REST à partir des clés de premier niveau du fichier JSON (ex. `/plats`, `/utilisateurs`, etc.). Les requêtes `GET`, `POST`, `PUT`, `DELETE` sont toutes supportées, et les modifications sont persistées dans le fichier local.



## Les livrables du projet

Chaque composant sera à la charge d'un étudiant du groupe et sera associé à un dépôt GitHub.
L'URL du dépôt devra être renseigné sur Ametice. Vous devrez aussi déposer le diagramme de classes associé au composant.

Le code devra être commenté et documenté (Javadoc).