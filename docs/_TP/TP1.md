---
layout: post
title: "TP1 -- Adaptation d'une application Web en PHP suivant le patron MVC"
categories: jekyll update

mathjax: true
---

# TP1 -- Adaptation d'une application Web en PHP suivant le patron MVC

L'objectif de ce TP est de poursuivre le travail sur l'application Web "Annonces" commencé dans le TD1. Il s'agira plus particulièrement de manipuler l'architecture mise en place en intégrant de nouvelles fonctionnalités (cas d'utilisation). Il faudra tirer un maximum partie de cette architecture MVC "propre" pour factoriser du code et faciliter les futures maintenances.


## L'inscription des utilisateurs

On ajoutera tout d'abord la procédure d'inscription des nouveaux utilisateurs. Le lien vers cette page sera affiché dans la fenêtre de connexion. L'utilisateur devra saisir les informations de son compte: login (identifiant unique), mot de passe fort (au moins 12 caractères, avec lettres minuscules et majuscules, chiffres et caractères spéciaux), nom et prénom. L'application inclura donc une vérification de l'identifiant et du mot de passe.  Par ailleurs, elle enregistrera automatiquement la date de création du compte. Si l'utilisateur saisi de mauvaises informations, il sera à nouveau redirigé vers l'interface d'inscription. 

Pour cela, vous commencerez par faire le diagramme de cas d'utilisation en UML. Vous  ajouterez ensuite toutes les classes, tables, attributs et méthodes nécessaires, tout en respectant le patron MVC et l'architecture "propre" mise en place (attention au sens des dépendances entre les classes). 

A la fin, vous générerez  le diagramme de classes à partir de [PhpStorm](https://www.jetbrains.com/help/phpstorm/class-diagram.html).


## L'ajout d'annonces

On intègrera ensuite la possibilité de poster une annonce. Le "propriétaire" de l'annonce sera conservé par l'application.  L'utilisateur devra renseigner le titre de l'annonce (maximum 20 caractères) et son contenu (maximum 200 caractères). La date de publications sera automatiquement enregistrée. L'application devra aussi vérifier que le titre et le contenu de l'annonce ne contiennent pas autre chose que du texte, et ainsi éviter l'injection de code malicieux dans l'application. Une fois validée, l'annonce sera ensuite affichée avec toutes les autres par ordre décroissant de publication (la date sera d'ailleurs affichée dans l'interface).

Cette nouvelle fonctionnalité sera intégrée à un menu affiché sur toutes les pages lorsque l'utilisateur est connecté. Ce menu intègrera aussi un élément permettant de retourner à l'accueil (le formulaire de connexion) et un élément redirigeant vers l'interface de visualisation des annonces.

Tout comme précédemment, vous commencerez par faire le diagramme de cas d'utilisation en UML, respecterez l'architecture mise en place, et générerez le diagramme de classes final.