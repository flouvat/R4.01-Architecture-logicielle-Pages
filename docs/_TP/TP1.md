---
layout: post
title: "TP1 -- Mise en pratique de l'architecture propre"
categories: jekyll update

mathjax: true
---

# TP1 -- Mise en pratique de l'architecture propre


## Partie 1 -- Extension d'une application suivant une architecture propre

L'objectif de cette première partie du TP est de poursuivre le travail sur l'application Web "Annonces" commencé dans le TD1. Il s'agira plus particulièrement de manipuler l'architecture mise en place en intégrant de nouvelles fonctionnalités (cas d'utilisation). Il faudra tirer un maximum partie de cette architecture MVC "propre" pour factoriser du code et faciliter les futures maintenances.

Les deux cas d'utilisation suivants devront être plus particulièrement ajoutés dans le logiciel :
- Inscrire un utilisateur
- Ajouter une annonce

Pour cela, vous pourrez vous aider du diagramme de classes ci-dessous représentant l'architecture de l'application après avoir ajouté ces deux fonctionnalités.

<img src="tp1-img/diagClasses.jpg" width="800px"/>


### L'inscription des utilisateurs

On ajoutera tout d'abord la procédure d'inscription des nouveaux utilisateurs. Le lien vers cette page sera affiché dans la fenêtre de connexion. L'utilisateur devra saisir les informations de son compte: login (identifiant unique), mot de passe fort (au moins 12 caractères, avec lettres minuscules et majuscules, chiffres et caractères spéciaux), nom et prénom. L'application inclura donc une vérification de l'identifiant et du mot de passe.  Par ailleurs, elle enregistrera automatiquement la date de création du compte. Si l'utilisateur saisi de mauvaises informations, il sera à nouveau redirigé vers l'interface d'inscription. 


### L'ajout d'annonces

On intégrera ensuite la possibilité de poster une annonce. Le "propriétaire" de l'annonce sera conservé par l'application.  L'utilisateur devra renseigner le titre de l'annonce (maximum 20 caractères) et son contenu (maximum 200 caractères). La date de publications sera automatiquement enregistrée. L'application devra aussi vérifier que le titre et le contenu de l'annonce ne contiennent pas autre chose que du texte, et ainsi éviter l'injection de code malicieux dans l'application. Une fois validée, l'annonce sera ensuite affichée avec toutes les autres par ordre décroissant de publication (la date sera d'ailleurs affichée dans l'interface).

Cette nouvelle fonctionnalité sera intégrée à un menu affiché sur toutes les pages lorsque l'utilisateur est connecté. 

A la fin, vous générerez  le diagramme de classes à partir de [PhpStorm](https://www.jetbrains.com/help/phpstorm/class-diagram.html) afin de vérifier si celui-ci correspond bien au diagramme de classes présenté au début.


## Partie 2 -- Analyse et modification de l'architecture d'une application existante

Dans cette partie, vous appliquerez les principes d'architecture propre à l'application à développer pour votre SAé. Si vous avez déjà du code, vous  analyserez le diagramme de classes de votre application et vous le modifierez afin qu'il respecte les principes d'architecture propre. Dans le cas contraire, vous partirez des différents cas d'utilisation et proposerez un diagramme de classes.