---
layout: post
title: "TD 1 -- Structuration d'une application Web en PHP"
categories: jekyll update

mathjax: true
---

# TD 1 -- Structuration d'une application Web en PHP

L'objectif est de partir d'une application web en PHP classique et de transformer étape après étape son architecture pour qu'elle respecte le patron MVC et les principes des architectures propres.

Nous partirons d'une application très basique permettant d'afficher une liste d'annonces. Cette application est codée en PHP sans réflexion particulière sur l'architecture, mais plus sur l'enchaînement des traitements. Son fonctionnement est illustrée dans la figure suivante.

<img src="td1-img/archi_flat_php.jpg" width="600px"/>


## Partie 1 - Mise en place de l'environnement de développement

Nous allons mettre en place notre environnement de développement. Nous allons travailler dans cette partie du cours avec PhpStorm. Cet environnement de développement intégré (EDI) est installé sur le VDI d'AMU, notamment dans la VM "Lunix (Linux pour tous)". Pour information, la société JetBrains commercialisant ce logiciel propose des licences gratuites pour les étudiants. Vous pouvez donc aussi l'installer sur vos machines personnelles. Pour cela, il suffit de remplir [ce formulaire](https://www.jetbrains.com/shop/eform/students).

Pour pouvoir fonctionner, notre application Web a besoin d'un serveur Web et d'un serveur de bases de données (MySQL/MariaDB dans notre cas). Pour le serveur web, nous allons simplement utiliser celui interne à PHP ("built-in web server"). A noter qu'il faut pour cela qu'un interpréteur PHP soit préalablement installé, ce qui est le cas sur les machines de l'IUT. Pour le serveur de base de données, nous serons obligés d'utiliser un service externe car il n'y en a pas d'installé en local sur les machines de l'IUT. Nous allons opter à ce niveau pour une base de données hébergée chez [AlwaysData](https://www.alwaysdata.com/fr/). 

Si vous travaillez sur votre PC, vous pouvez à la place utiliser des outils comme [Xampp](https://www.apachefriends.org/fr/index.html), une solution permettant d'installer et de gérer simplement un serveur web  Apache, en local, intégrant MariaDB et PHP.

Dans tous les cas, nos développements seront effectués à partir d'un dépôt GitHub préalablement créé. Pour cloner ce dépôt dans votre compte GitHub, vous devez aller sur le cours **Ametice** et cliquer sur le lien GitHub dans la section "Dépôt GitHub pour la partie architecture propre (TD1 et TP1)".


### 1 - Création et paramétrage du projet PhpStorm

Nous allons maintenant créer le projet dans PhpStorm et configurer l'environnement de développement local. 

Pour cela, vous lancez PhpStorm et sélectionnez `Get from VCS`, puis copiez l'url du dépôt GitHub créé précédemment et cloner le dépôt. 

<img src="td1-img/phpstorm-open.png" width="600px">

Vous devez ensuite autoriser PhpStorm à se connecter à votre dépôt GitHub. Pour cela, vous utiliserez un token et cliquerez ensuite sur `Generate` pour générer un token pour l'application. Avant de valider la génération du token en bas de la page, vous modifierez la date d'expiration du token à 90 jours et sélectionnerez tous les droits ("scopes").  Vous copierez ensuite le token généré par GitHub dans votre fenêtre PhpStorm et vous connecterez.



<img src="td1-img/phpstorm-newProjectFromGitHub.png" width="600px">

Nous allons maintenant configurer le serveur web local. Vous devez aller dans les paramètres de PhpStorm en passant par le sous-menu `File` et en sélectionnant `Settings`. Il faut ensuite aller dans `Build, Execution, Deployment` et sélectionner `Deployment`. Nous allons ajouter ici le server web intégré à l'interpréteur  PHP en cliquant sur `+` et en sélectionnant un serveur de type `in place` (car le code sera directement exécuté  à partir du répertoire du projet PhpStorm). Une fenêtre s'ouvre ensuite pour nous demander un nom pour ce serveur. Nous pouvons par exemple mettre "built-in webserver". 

<img src="td1-img/phpstorm-localServer1.png" width="600px">

Il faut ensuite configurer ce serveur en remplissant simplement son URL (`Web server URL`). Nous allons travailler avec un serveur local sur le port 8080. Nous allons donc simplement mettre `http://localhost:8000` (le port peut être changé si ce dernier est déjà pris) . 

<img src="td1-img/phpstorm-localServer2.png" width="600px">

Le serveur web intégré à l’interpréteur PHP est maintenant enregistré dans PhpStorm. Il faut maintenant paramétrer PhpStorm pour pouvoir l'utiliser facilement dans le projet. Pour cela, nous allons créer une configuration d'exécution en sélectionnant dans le bandeau principal à droite `Current File` et `Edit Configurations`. 

<img src="td1-img/phpstorm-runtimeConfig1.png" width="600px">

Nous allons ensuite ajouter une nouvelle configuration en cliquant sur `+` et sélectionner `PHP Built-in WebServer`. Nous allons appeler cette configuration "localhost", mettre `localhost` comme adresse `Host` et `8000` comme port.

<img src="td1-img/phpstorm-runtimeConfig2.png" width="600px">



Pour tester ce code, il suffit de lancer le serveur web en cliquant sur la configuration d'exécution précédemment créée, via le bouton triangle vert dans le bandeau supérieur droit. 

<img src="td1-img/phpstorm-projetBasique.png" width="600px">

Le serveur étant démarré, il est possible par exemple d'ouvrir la page "index.html" et de l'afficher. A ce niveau, deux options vous sont proposées par PhpStorm sous forme de petits icônes affichés à droite du code : ouvrir la page dans une fenêtre à l'intérieur de PhpStorm (`Built-in Preview`) ou ouvrir la page dans un navigateur externe (p.ex. Firefox). On va choisir la première option dans l'exemple mais, si vous êtes sur les machines de l'IUT, vous devrez passer par la deuxième solution à cause des droits limités de votre compte. 



<img src="td1-img/phpstorm-execution1.png" width="600px">

<img src="td1-img/phpstorm-execution2.png" width="600px">



Attention, le code n'est pas totalement opérationnel car ce dernier utilise une base de données. Or, celle-ci n'a pas été créée, et son accès n'ont pas paramétrés dans le code PHP.


### 2 - Paramétrage de la base de données

Si ce n'est pas déjà fait, il faut tout d'abord créer un compte (gratuit) chez AlwaysData en remplissant [ce formulaire](https://www.alwaysdata.com/fr/inscription/?d).


Avant de créer la base de données, nous allons créer un utilisateur de base de données spécifique. Cet utilisateur sera utilisé pour connecter l'application à la base de données, et ainsi mieux contrôler la sécurité. 

<img src="td1-img/database-panel.png" width="800px"/>

Allez dans l'onglet `Bases de données > MySQL` de votre interface d’administration, et sélectionnez l'onglet `utilisateurs` du bandeau principal. Vous devez ensuite sélectionner `Ajouter un utilisateur` pour saisir les informations du nouvel utilisateur à créer. Il faudra le nommer `[compte]_annonces`, et lui affecter un mot de passe. Ensuite, il faut ajouter la base de données de l'application via l'onglet `Bases de données` du bandeau principal. Dans le formulaire qui s'affiche, vous allez saisir le nom de la base de données `[compte]_annonces_db`, donner `tous les droits` à l'utilisateur que vous venez de créer et valider. 

La base de données est créée mais vide. Il faut donc maintenant se connecter à l'interface d'administration de la base de données (phpMyAdmin) pour créer les tables et saisir quelques données de test. Comme le montre la figure précédente, `phpMyAdmin` est accessible par le lien au dessus du bandeau principal. Une fois connecté, l'interface peut-être mise en français si besoin.

<img src="td1-img/phpmyadmin.png" width="800px"/>

Pour créer de nouvelles tables dans cette base de données, il suffit de sélectionner votre base de données `[compte]_annonces_db` dans la liste à gauche. Ensuite, un formulaire vous permettra de saisir le nom d'une nouvelle table ainsi que le nombre de colonnes. Nous allons commencer par créer une table `Users` avec deux colonnes : `login` et `password` (attention au choix des types de données). Une fois la structure de la table saisie, il suffit de `Enregistrer` pour finaliser la création de la table. A noter que la clé primaire peut être définie par l'intermédiaire de la liste déroulante `Index` au moment de la définition de la structure de la table.

<img src="td1-img/phpmyadmin-create.png" width="800px"/>

L'ajout de données exemples dans cette table peut se faire par l'intermédiaire de l'onglet `Insérer` du bandeau principal. Saisissez deux tuples exemples et `Exécuter` l'insertion.

<img src="td1-img/phpmyadmin-insert.png" width="800px"/>

Il faut ensuite reproduire ces différentes étapes pour créer une table `Post` avec les colonnes : `id`, `date`, `title` et `body`.

Une fois la base de données créées et quelques données enregistrées, le code de l'application devrait fonctionner dans PhpStorm. Attention, il ne faut pas oublier de modifier le code PHP afin d'intégrer les informations de connexion à Alwaysdata.


### 3 - Mise à jours de l'application basique avec les informations de connexion de la base de données sur Alwaysdata

Avant cela, il faut tout de même corriger un certain nombre de problèmes dans l'application basique. Les informations de la base de données doivent notamment être mises à jours pour correspondre à la base de données déployée sur Alwaysdata. Pour rappel, les informations sont les suivantes:
- hôte MySQL : `mysql-[compte].alwaysdata.net`
- nom de la base de données : `[compte]_annonces_db`
- nom de l'utilisateur pour la connexion : `[compte]_annonces`
- mot de passe : celui défini dans l'interface d'administration d'Alwaysdata

Dans les fichiers `annonces.php` et `post.php`, vous devez plus particulièrement modifier les paramètres de la fonction [`mysqli_connect`](https://www.php.net/manual/fr/function.mysqli-connect.php).

Afin d'améliorer la sécurité du code, vous isolerez ensuite les paramètres de connexion à la base de données dans un fichier `config.php` qui suivra le modèle suivant.

```php
<?php
/**
 * Fichier de configuration
 * Le fichier config.php ne doit JAMAIS être versionné
 */

// Configuration de la base de données
define('DB_HOST', 'localhost');
define('DB_USER', 'votre_utilisateur');
define('DB_PASS', 'votre_mot_de_passe');
define('DB_NAME', 'blog_db');

?>
```

Ce fichier ne devra pas être versionné par Git pour éviter de diffuser ces informations. Pour cela, il vous faut créer un fichier `.gitignore` et l'intégrer à l'intérieur.

Il vous suffit ensuite d'importer le fichier `config.php` dans le code PHP faisant des connexions à la base de données et de remplacer toutes les informations de connexion par les constantes définies dans `config.php`.


## Partie 2 -  Mise en place du patron MVC

L'objectif de cette partie est d'améliorer l'application précédente en la transformant étape après étape pour qu'elle suive le patron MVC. Nous allons suivre pour cela l'approche présentée sur le site de Symfony pour expliquer leur framework ([ici](https://symfony.com/doc/current/introduction/from_flat_php_to_symfony.html)). Symfony est un framework MVC libre en PHP, qui est à la base d'un autre framework PHP très populaire: [Laravel](https://laravel.com/).



### 1 - Isolation de l'interface utilisateur

Dans cette étape, nous allons déjà isoler la partie interface utilisateur du reste de l'application. Autrement dit, nous allons isoler dans des fichiers séparés la partie du code chargée de générer le code HTML affiché par le navigateur. A la fin de cette étape, notre application aura la structure suivante:

<img src="td1-img/MVC-1-isolationIHM.jpg" width="600px"/>

Dans PhpStorm, nous allons ajouter un nouveau répertoire `view` au projet et y créer un nouveau fichier `login.php`. Pour faire cela, il suffit de faire un clic droit sur le nom du projet dans la fenêtre de gauche et de sélectionner `New > Directory`, puis sur le nouveau répertoire `New > PHP File`.

<img src="td1-img/phpstorm-login.png" width="400px">

Dans ce fichier, il faut ensuite copier le code HTML qui est lié au formulaire de connexion et qui se situe dans le fichier `index.html`. 

<img src="td1-img/phpstorm-logincode.png" width="800px"/>

Ce code sera supprimé de `index.html` et remplacé par le code suivant, qui inclut le code du formulaire au chargement de la page index.

```php
<?php
    require 'view/login.php';
?>
```

Etant donné qu'il s'agit de code PHP, il faut changer l'extension du fichier `index.html` en `index.php` pour que ce dernier soit interprété par le serveur web. Vous pouvez directement faire cela dans PhpStorm via un clic droit sur le fichier et `Refactor > Rename` (ou plus simplement en utilisant le raccourci  `Maj + F6`).

Vous pouvez tester à nouveau l'application. Son fonctionnement est en apparence identique, mais l'organisation du code n'est plus la même. 

Nous allons faire de même pour la page `annonces.php` qui affiche la liste des annonces enregistrées dans la base de données. Comme précédemment, nous allons commencer par créer un fichier `annonces.php` dans le répertoire `view`. Ensuite, nous allons couper/coller la partie HTML de `\annonces.php` dans `\view\annonces.php`.

<img src="td1-img/phpstorm-annonces.png" width="800px"/>

Comme le montre la figure, PhpStorm indique maintenant qu'il y une erreur dans `\view\annonces.php` (code souligné en rouge). Cette erreur est due au fait que la variable `$resultall` n'est plus reconnue dans cette nouvelle page car elle est créée dans la page  `\annonces.php` sur la racine du projet. Plus généralement, cette erreur met en avant des dépendances entre ces deux fichiers. Nous allons donc essayer de les limiter en passant par des variables intermédiaires. Pour cela, nous allons déjà remplacer la boucle `while` par le code suivant :

```php
    <?php foreach( $annonces as $post ) : ?>
        <li>
            <a href="post.php?id=<?php echo $post['id']; ?>">
              <?php echo $post['title']; ?>
            </a>
        </li>
    <?php endforeach ?>

```

Nous allons également modifier le paragraphe qui précède  avec le `Hello` par le code suivant:

```php
    <p> Hello <?php echo $login; ?> </p>
```

Dans ces deux mises à jours, nous avons utilisé les variables `$login` et `$annonces` comme variables intermédiaires pour transférer les données de la page `\annonces.php` où sont faites les requêtes vers la vue `\view\annonces.php` où ces données sont affichées. 

<img src="td1-img/phpstorm-annonces2.png" width="800px"/>

Toutefois, le code n'est pas encore fonctionnel car il faut encore initialiser ces variables avec les résultats des requêtes dans `\annonces.php`. A ligne 8 de   `\annonces.php`, nous allons donc ajouter l'instruction `$login = $_POST['login'];`. Puis, nous allons initialiser la variable `$annonces`, qui est un tableau associatif constitué des tuples résultats de la requête, avec le code suivant:

```php
        $annonces = array();
        while ($row = mysqli_fetch_assoc($resultall)) {
            $annonces[] = $row;
        }
```

Pour finir, il faut également inclure la vue `\view\annonces.php` dans `\annonces.php` avec un `require` à la fin du fichier. 

<img src="td1-img/phpstorm-annonces3.png" width="800px"/>

Le fichier `\annonces.php` contient encore du code lié à l'interface utilisateur. Il s'agit du message affiché, et de la redirection effectuée lorsque le login ou le mot de passe ne sont pas corrects (lignes 19 à 23). Nous allons déplacer ce code dans la vue et légèrement l'adapter de la manière suivante :

```php
<?php
    if( !isset( $login) or $login == '' ){
        header( "refresh:5;url=index.php" );
        echo 'Erreur de login et/ou de mot de passe (redirection automatique dans 5 sec.)';
        exit;
    }
?>
```

Ce code est au début de `\view\annonces.php`. Il commence par vérifier si la variable `$login` est définie. Autrement dit, il vérifie si le résultat de la requête à la ligne 4 de `\annonces.php` a retourné quelque chose. Si ce n'est pas le cas, cela signifie que les identifiants de connexion ne sont pas dans la base de données. L'utilisateur est donc redirigé vers la page de connexion (i.e. `index.php`).

Si ce n'est pas déjà fait, vous pouvez tester à nouveau l'application. Son fonctionnement est toujours en apparence identique, mais l'organisation du code a encore changé. 

Après avoir isolé l'interface utilisateur des pages `\index.php` et `\annonces.php`, il faut maintenant faire de même avec la page `\post.php`. Tout comme précédemment, il faut créer un fichier `\view\post.php`, y transférer les éléments de l'interface utilisateur et inclure cette vue dans  `\post.php`. 

<img src="td1-img/phpstorm-postview.png" width="800px"/> 

<img src="td1-img/phpstorm-post.png" width="800px"/>


### 2 - Isoler l'accès aux données

Après avoir isolé l'interface utilisateur, nous allons isoler l'accès aux données. A la fin de cette étape, notre application aura la structure suivante:

<img src="td1-img/MVC-2-isolationBd.jpg" width="600px" />

Nous allons créer un fichier `model.php`  à la racine du projet. Dans ce fichier, nous allons constituer une bibliothèque de fonctions permettant d'interagir indirectement avec la base de données. Nous aurons notamment les fonctions `openConnection` et `closeConnection` qui seront utilisées dans les autres fonctions de la bibliothèque pour ouvrir/fermer la connexion à la base de données. Leur code est le suivant:

```php 
    function openConnection()
    {
        $link = mysqli_connect('mysql-[compte].alwaysdata.net', '[compte]_annonces', 'pwd', '[compte]_annonces_db');
        return $link;
    }

    function closeConnection($link) 
    {
        mysqli_close($link);
    }
```

Ce fichier intègre aussi la fonction `isUser` qui vérifie si l'utilisateur est enregistré dans la base de données.

```php
    function isUser( $login, $password )
    {
        $isuser = False ;
        $link = openConnection();
    
        $query= 'SELECT login FROM Users WHERE login="'.$login.'" and password="'.$password.'"';
        $result = mysqli_query($link, $query );
    
        if( mysqli_num_rows( $result) )
            $isuser = True;
    
        mysqli_free_result( $result );
        closeConnection($link);
    
        return $isuser;
    }
```

Il y a aussi la fonction `getAllAnnonces` qui retournera un tableau associatif avec toutes les annonces enregistrées dans la base de données. 


```php
   function getAllAnnonces()
    {
        $link = openConnection();
    
        $result = mysqli_query($link,'SELECT id, title FROM Post');
        $annonces = array();
    
        while ($row = mysqli_fetch_assoc($result)) {
            $annonces[] = $row;
        }
    
        mysqli_free_result( $result);
        closeConnection($link);
    
        return $annonces;
    }
```

Pour finir, le fichier `/model.php` contient aussi la fonction `getPost` qui retourne l'annonce ayant l'identifiant passé en paramètre. 

```php
    function getPost( $id )
    {
        $link = openConnection();
    
        $id = intval($id);
        $result = mysqli_query($link, 'SELECT * FROM Post WHERE id='.$id );
        $post = mysqli_fetch_assoc($result);
    
        mysqli_free_result( $result);
        closeConnection($link);
        return $post;
    }
```

Grâce à cette bibliothèque de fonctions, il est maintenant possible de  simplifier le code de `\annonces.php` et `\post.php`, et de le rendre indépendant des détails d'accès et d'interrogation de la base de données. 

Le code de `\annonces.php` devient simplement:

```php
<?php
       require_once 'model.php';

       if( isUser( $_POST['login'], $_POST['password'] ) ) {
	$login = $_POST['login'];
      	$annonces = getAllAnnonces();
       }

       // inclut le code de la présentation HTML
       require 'view/annonces.php';
?>
```

Le code de `\post.php` devient:

```php
<?php
require_once 'model.php';

$post = getPost( $_GET['id'] );

// inclut le code de la présentation HTML
require 'view/post.php';
?>
```

L'application devrait être à nouveau totalement fonctionnelle et identique en apparence, même si encore une fois sa structure a évolué.


### 3 - Isoler la mise en page de l'interface utilisateur

L'interface utilisateur a été isolée du reste de l'application mais il est encore possible de factoriser du code entre les différentes parties de l'IHM. Nous pouvons notamment isoler la mise en page ("layout") des pages de l'application car celle-ci est commune. Nous allons donc sensiblement restructurer notre code de la manière suivante:

<img src="td1-img/MVC-3-isolationLayout.jpg" width="600px" />

Nous allons commencer par créer un fichier `layout.php` dans le répertoire `view` de l'application. Dans ce fichier, nous allons mettre le squelette de page suivant:

```php
<!DOCTYPE html>
<html lang="fr">
 <head>
  <title><?php echo $title; ?></title>
  <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
 </head>
 <body>
    <?php echo $content; ?>
 </body>
</html>
```

Il met en avant que toutes les pages de l'application seront composées d'un titre et d'un contenu. Les variables `$title` et `$content` seront utilisées pour adapter ce titre et ce contenu dans chaque vue. En l'état, ce squelette est très basique, mais il permet d'ajouter très facilement un menu, un bandeau ou un bas de page communs à toutes les pages de l'application. 

Ce squelette est défini mais il n'est pas encore utilisé à ce stade. Nous allons maintenant changer cela en l'utilisant dans les vues (avec un `require`). Avant d'appeler le squelette de mise en page commun, chaque vue devra initialiser les variables `$title` et `$content`. Pour la vue `/view/login.php`, il faut modifier le code de la manière suivante:

```php
<?php $title= 'Exemple Annonces Basic PHP: Connexion'; ?>

<?php ob_start(); ?>
    <form method="post" action="annonces.php">
        <label for="login"> Votre identifiant </label> :
        <input type="text" name="login" id="login" placeholder="defaut" maxlength="12" required />
        <br />
        <label for="password"> Votre mot de passe </label> :
        <input type="password" name="password" id="password" maxlength="12" required />
    
        <input type="submit" value="Envoyer">
    </form>
<?php $content = ob_get_clean(); ?>

<?php require 'layout.php'; ?>
```

On remarque trois blocs PHP dans ce code. Le premier sert à initialiser le titre de la page, le deuxième son contenu (le formulaire de login ici) et le troisième de transférer ces données au squelette ("layout"). Pour le deuxième bloc, on note l'utilisation des méthodes [`ob_start()` et `ob_get_clean()`](https://www.php.net/manual/fr/ref.outcontrol.php). Ces méthodes permettent de mettre dans un "buffer" (i.e. mémoire tampon) le code HTML situé entre les deux appels au lieu de l'envoyer au navigateur. Ainsi, il est possible de le récupérer simplement dans une variable. Cela permet aussi d'avoir  un affichage des pages sensiblement plus rapide. 

Pour la vue `/view/annonces.php`, il faut par exemple modifier le code de la manière suivante:

```php
<?php
    if( !isset( $login) or $login=='' ){
        header( "refresh:5;url=index.php" );
        echo 'Erreur de login et/ou de mot de passe (redirection automatique dans 5 sec.)';
        exit;
    }
?>

<?php $title= 'Exemple Annonces Basic PHP: Annonces'; ?>

<?php ob_start(); ?>
    <p> Hello <?php echo $login; ?> </p>
    <h1>List of Posts</h1>
    <ul>
        <?php foreach( $annonces as $post ) : ?>
          <li>
            <a href="post.php?id=<?php echo $post['id']; ?>">
              <?php echo $post['title']; ?>
            </a>
         </li>
        <?php endforeach ?>
    </ul>
<?php $content = ob_get_clean(); ?>

<?php require 'layout.php'; ?>
```

Il faut maintenant reproduire le même principe pour la vue `/view/post.php`.

L'application devrait être à nouveau totalement fonctionnelle à ce stade, et identique en apparence, même si encore une fois sa structure a évolué.


### 4 - Regrouper les contrôleurs et mettre en place un contrôleur frontal

A ce stade, les fichiers `\index.php`, `\annonces.php` et `\post.php` récupèrent les données de l'utilisateur ou de la base de données, et les transfèrent à l'interface graphique. Ils "contrôlent" donc l'exécution de l'application. Nous allons maintenant regrouper ce code sous forme de méthodes et les stocker dans le fichier `/controllers.php`. Ce fichier constituera une sorte de bibliothèque de fonctions de contrôle, i.e un "contrôleur" de l'application au sens MVC (Modèle-Vue-Contrôleur). 
Suite à ce changement, le code sera structuré de la manière suivante:

<img src="td1-img/MVC-4-controller.jpg" width="600px" />

Le code de  `/controllers.php` est relativement simple (cf. code ci-dessous).

```php
<?php

function loginAction()
{
    require 'view/login.php';
}

function annoncesAction( $login, $password)
{
    if( isUser( $login, $password ) )
        $annonces = getAllAnnonces();
    else
        $login='';

    require 'view/annonces.php';
}

function postAction($id)
{
    $post = getPost($id);
    require 'view/post.php';
}

?>
```

Une fois ces contrôleurs créés, il faut supprimer les fichiers `\annonces.php` et `\post.php` du projet car ils n'ont plus d'utilité. 

Il faut maintenant appeler ces fonctions dans `index.php`. L'index va aussi être utilisé pour "router" l'utilisateur vers la bonne partie de l'application. Ce routage sera basé sur l'URL, et permettra de déclencher les bonnes fonctions du contrôleur (et de leur passer les données nécessaire à leur fonctionnement). Ce routage par `index.php` permet de complètement masquer de l'extérieur l'organisation interne du code. En apparence, le seul fichier exécuté est `index.php`. Cela apporte beaucoup de flexibilité pour intégrer de futures modifications ou évolutions. Le fichier `index.php` devient ce que l'on appelle le "contrôleur frontal" de l'application. Son code est le suivant:

```php
<?php

// charge et initialise les bibliothèques globales
require_once 'model.php';
require_once 'controllers.php';

// chemin de l'URL demandée au navigateur
// (p.ex. /annonces/index.php)
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// route la requête en interne
// i.e. lance le bon contrôleur en focntion de la requête effectuée
if ( '/annonces/' == $uri || '/annonces/index.php' == $uri) {
    loginAction();
}
elseif ( '/annonces/index.php/annonces' == $uri
            && isset($_POST['login']) && isset($_POST['password']) ){

    annoncesAction($_POST['login'], $_POST['password']);
}
elseif ( '/annonces/index.php/post' == $uri
            && isset($_GET['id'])) {

    postAction($_GET['id']);
}
else {
    header('Status: 404 Not Found');
    echo '<html><body><h1>My Page NotFound</h1></body></html>';
}

?>
```

Il commence par extraire le chemin demandé au navigateur, et il le compare avec les différentes routes possibles pour l'application. Dans notre cas, il y a les routes suivantes (à partir de la racine du serveur web):
- `/annonces/` ou `/annonces/index.php` :  authentification de l'utilisateur
- `/annonces/index.php/annonces` :  affichage de toutes les annonces 
- `/annonces/index.php/post` :  affichage d'une annonce en particulier (un post)

En plus du routage, ce code vérifie que les données nécessaires aux contrôleurs pour faire leur action sont bien disponibles (avec des `isset`). Par exemple, l'application a besoin du login et du mot de passe (transmis en mode POST) pour afficher toutes les annonces. Elle a besoin d'un identifiant d'annonce (id, transmis en mode GET) pour afficher une annonce.

Avec ce système de routage, il est nécessaire de modifier les vues `\view\login.php` et `\view\annonces.php`. En effet, le formulaire contenu dans `\view\login.php`  a actuellement pour destination `annonces.php`. Or ce fichier n'existe plus. Il faut à la place appeler le contrôleur de la partie affichant les annonces (i.e la méthode `annoncesAction`). Comme indiqué dans le fichier `index.php`, cet appel est associé à la route `/annonces/index.php/annonces`. Il faut donc modifier la destination du formulaire et mettre cette "route" à la place. Il faut faire de même pour les liens hypertextes générés dans la vue `\view\annonces.php`. 

<img src="td1-img/phpstorm-loginRoute.png" width="800px"/>

<img src="td1-img/phpstorm-annoncesRoute.png" width="800px"/>



## Partie 3 -  Vers une architecture plus "propre"

L'objectif de cette partie est d'améliorer encore l'application en la passant en orienté objets et en réorganisant les classes pour avoir une architecture plus "propre". 

### 1 - Utilisation des PDO (PHP Data Objects)

Actuellement, l'application se connecte et interroge  la base de données par l'intermédiaire de l'extension MySQLi ("MySQL improved"). Cette extension fournit une interface vers la base de données. Elle est composée de méthodes permettant d'envoyer des requêtes (préparées ou non), de gérer des transactions et de débugger. L'extension propose une interface orientée objets ainsi qu'une interface procédurale. Cette interface est totalement fonctionnelle mais elle a l'inconvénient d'être dépendante de MySQL. Autrement dit, si on veut changer de SGBD (Système de Gestion de Bases de Données), il faut modifier tous les appels aux méthodes de  MySQLi. Pour éviter ce genre de dépendance et permettre de changer plus facilement de base de données ultérieurement, nous allons modifier le code pour qu'il utilise PDO  (PHP Data Objects)  à la place.

PDO est une couche d'abstraction de base de données. Elle permet de faire en sorte que votre application soit indépendante du type de base de données choisie. Pour changer de SGBD, il suffit de faire des changements mineurs dans le code (seul l'appel au constructeur de PDO diffère).  PDO propose une API simple et portable, mais aussi un pilote ("driver") pour communiquer avec MySQL ou d'autres SGBD. Comme évoqué dans la [documentation de PHP](https://www.php.net/manual/fr/mysqli.overview.php), PDO ne permet toutefois pas d'utiliser toutes les fonctionnalités avancées des dernières versions de MySQL (p.ex. les requêtes multiples).

Dans cette partie, vous devez donc remplacer les appels à MySQLi par des appels à PDO. Grâce à l'architecture MVC mise en place, ces appels sont normalement tous localisés dans le fichier `model.php`.

Tout d'abord, il faut remplacer les instructions de connexion (`mysqli_connect`) par la création d'objets PDO (le constructeur initialisant la connexion avec le SGBD). Pour cela, vous pouvez vous aider des indications de la [documentation technique](https://www.php.net/manual/fr/pdo.connections.php). Vous intégrerez aussi une gestion des erreurs de connexion comme dans l'exemple 2 de la documentation.

<img src="td1-img/phpstorm-pdo1.png" width="800px"/>

Ensuite, il faut changer les appels à la fonction `mysqli_query` faisant les requêtes,  par des appels à la méthode [`query`](https://www.php.net/manual/fr/pdo.query.php) de la classe PDO. Cette fonction retourne un objet de la classe `PDOStatement` qui stocke le résultat de la requête retourné par le SGBD. Il faut donc utiliser cet objet pour remplacer les appels à la fonction MySQLi donnant le nombre de tuples résultats ( `mysqli_num_rows`) par des appels à la méthode [`rowCount`](https://www.php.net/manual/fr/pdostatement.rowcount.php)  de `PDOStatement`. Il faut également remplacer  les appels à la fonction MySQLi retournant le tuple courant dans les résultats ( `mysqli_fetch_assoc`), par des appels à la méthode [`fetch`](https://www.php.net/manual/fr/pdostatement.fetch.php)  de `PDOStatement`.
Une fois le résultat de la requête traité, il faut libérer la mémoire utilisé. Avec MySQLi, on fait cela avec la fonction `mysqli_free_result`. Avec PDO, il faut remplacer ces appels par des appels à la méthode [`closeCursor`](https://www.php.net/manual/fr/pdostatement.closecursor.php) de `PDOStatement`. Il faut également fermer la connexion à la base de données à la fin des traitements, i.e. remplacer l'appel à `mysqli_close` par l'instruction `$dbh = null;`. 

<img src="td1-img/phpstorm-pdo2.png" width="800px"/>


### 2 - Passage de l'application en orienté objets

Nous allons maintenant restructurer notre code pour qu'il suive un paradigme orienté objets. Dans un premier temps, nous allons faire une traduction assez basique du code en orienté objets, et allons améliorer cette architecture progressivement ensuite.  

Nous allons commencer par transformer le code de `model.php` en classe `Model`. Nous allons en profiter pour renommer le fichier en `Model.php` (clique droit sur le nom du fichier, puis `Refactor > Rename`). Cette classe aura pour propriété  protégée `$dataAccess` (initialisée à `null`). Le constructeur de cette classe (la méthode `__construct`)  prendra en paramètre un `$dataAccess` (qui sera en fait un objet `PDO`) et l'utilisera pour initialiser la propriété `$this->dataAcess`.  L'objet `PDO` n'est pas créé dans le constructeur afin d'éviter l'injection d'une dépendance entre ces deux classes. Tout sera fait dans le contrôleur frontal `index.php`. 
 
 Le destructeur (`__destruc`) de cette classe mettra à `null` la propriété `$this->dataAcess`, ce qui aura pour conséquence de fermer la connexion à la base de données si celle-ci a été initiée par un objet `PDO`. A noter que cette organisation implique que la connexion à la base de données est laissée "ouverte" toute la durée de vie de l'objet.

<img src="td1-img/phpstorm-model-construct.png" width="800px"/>

Il faut aussi modifier le code des méthodes de `Model.php` pour les rendre `public`, changer les appels à `$dbh` par  `$this->dataAcess`, et supprimer les fonctions  `openConnection` et `closeConnection` (définitions et appels).

<img src="td1-img/phpstorm-model.png" width="800px"/>

Nous allons ensuite transformer `controllers.php` en classe `Controllers` (et renommer le fichiers `Controllers.php`). Cette classe aura une propriété protégée `data` (initialisée à `null`). Le constructeur de cette classe initialisera cette propriété  `$this->data` avec un objet `$data` (de type `Model`) passé en paramètre.

<img src="td1-img/phpstorm-controllers-construct.png" width="800px"/>

Il faut ensuite modifier le code des méthodes  de `Controllers` pour les rendre `public` et remplacer les appels des anciennes fonctions de `model.php` par les méthodes de l'objet `$this->data` (i.e. `$this->data->isUser`, `$this->data->getAllAnnonces`, et `$this->data->getPost`).

<img src="td1-img/phpstorm-controllers.png" width="800px"/>

Nous pouvons maintenant modifier le code de `index.php` afin qu'il utilise ces nouvelles classes. Les principaux changements se situent au début du code. Nous allons créer un objet `Model` en lui passant en paramètre un objet `PDO` avec les informations de connexion de notre base de données. Puis, nous construisons un objet `Controllers` avec en paramètre l'objet `Model` nouvellement créé. On obtient le code suivant:

```php
try {
    // construction du modèle
    $data = new Model( new PDO('mysql:host=mysql-[compte].alwaysdata.net;dbname=[compte]_annonces_db', '[compte]_annonces', '[compte]_annonces_mdp') );

    // initialisation du controller
    $controller = new Controllers( $data );

} catch (PDOException $e) {
    print "Erreur de connexion !: " . $e->getMessage() . "<br/>";
    die();
}
```

L'objet `$controller` est ensuite utilisé pour invoquer les méthodes `loginAction`,  `annoncesAction` et `postAction`.

<img src="td1-img/phpstorm-index-oo.png" width="800px"/>

Comme on peut le constater, l'application est toujours fonctionnelle et identique en apparence, même si elle est maintenant en partie développée en orienté objets. 

Continuons le passage du code de l'application en orienté objets. Nous avons différentes vues et un squelette de mise en page (`layout`). Nous commençons par traiter le squelette de mise en page (celui-ci étant utilisé dans les vues). Nous renommons le fichier  `layout.php` en `layout.html`, puis  créons un fichier `Layout.php`. Le fichier HTML servira de patron ("template") pour toutes les pages. Il sera passé en paramètre du constructeur de la classe `Layout` dans le fichier `Layout.php`, et stocké comme propriété de cette classe.  Le code de la page `layout.html` sera le suivant:

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <title> %title% </title>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
</head>
<body>
%content%
</body>
</html>
```

Le code de `Layout.php` sera:

```php
<?php

class Layout
{
    protected $templateFile;

    public function __construct( $templateFile )
    {
        $this->templateFile = $templateFile;
    }

    public function display( $title, $content )
    {
        $page = file_get_contents( $this->templateFile );
        $page = str_replace( ['%title%','%content%'], [$title,$content], $page);
        echo $page;
    }

}

```

La méthode `display` charge en mémoire la trame stockée dans le fichier `$templateFile`. Puis, elle remplace dans cette trame les chaînes de caractères `%title%` et `%content%` par le contenu des paramètres `$title` et `$content` (cf. [str-replace](https://www.php.net/manual/en/function.str-replace.php)), qui seront fournis par la vue.   Ainsi, il sera facile de changer la mise en page sans modifier le code de la classe `Layout`. Il suffira de changer le code HTML de `layout.html`.


 Nous allons maintenant créer une classe mère abstraite `View` dont hériteront les classes filles `ViewAnnonces`, `ViewLogin`, et `ViewPost`. Cet héritage va nous permettre de factoriser  du code entre les vues. Dans le répertoire `\view`, nous allons créer le fichier `View.php`. Dans ce fichier, nous allons mettre le code suivant:

```php
<?php
include_once "Layout.php";

abstract class View
{
    protected $title = '';
    protected $content = '';
    protected $layout;

    public function __construct($layout)
    {
        $this->layout = $layout;
    }

    public function display()
    {
        $this->layout->display( $this->title, $this->content );
    }
}
```

Cette classe est constituée des propriétés `$title` (titre de la page), `$content` (contenu de la page) et `$layout` (structure de la page). Elle contient aussi une méthode `display` qui envoie le titre et le contenu au `layout`, puis déclenche son affichage. 

Ensuite, nous allons créer les classes filles. Nous commençons par renommer le fichier `login.php` en `ViewLogin.php`, puis nous définissons la classe `ViewLogin` à l'intérieur. Son code est:

```php
<?php
include_once "View.php";

class ViewLogin extends View
{
    public function __construct($layout)
    {
        parent::__construct($layout);

        $this->title = 'Exemple Annonces Basic PHP: Connexion';

        $this->content = '
            <form method="post" action="/annonces/index.php/annonces">
                <label for="login"> Votre identifiant </label> :
                <input type="text" name="login" id="login" placeholder="defaut" maxlength="12" required />
                <br />
                <label for="password"> Votre mot de passe </label> :
                <input type="password" name="password" id="password" maxlength="12" required />
        
                <input type="submit" value="Envoyer">
            </form>';
    }
}
```

Comme le montre ce code, cette classe se contente d'initialiser le titre de la page et son contenu avec le formulaire de login.

Les changements effectués à `login.php` doivent ensuite être appliqués à `annonces.php` et `post.php`, ce qui permettra d'aboutir aux classes `ViewAnnonces` et `ViewPost` dans les fichiers `ViewAnnonces.php` et `ViewPost.php`.  Ces deux vues sont légèrement différentes de la vue "login" car elles ont besoin de données supplémentaires transmises par le contrôleur. Il s'agit plus précisément du login (`$login`) et d'un tableau d'annonces (`$annonces`) pour `ViewAnnonces`, et d'un tableau stockant les information d'une annonces (`$post`) pour `ViewPost`. Il convient donc d'ajouter ces paramètres à leurs constructeurs respectifs. Leur code devient donc :

```php
<?php
<?php
include_once "View.php";

class ViewAnnonces extends View
{
    public function __construct($layout, $login, $annonces)
    {
        parent::__construct($layout);

        if( $login =='' ){
            header( "refresh:5;url=/annonces/index.php" );
            echo 'Erreur de login et/ou de mot de passe (redirection automatique dans 5 sec.)';
            exit;
        }

        $this->title= 'Exemple Annonces Basic PHP: Annonces';

        $this->content = "<p> Hello $login </p>";
        $this->content.= '<h1>List of Posts</h1>  <ul>';

        foreach( $annonces as $post ) {
            $this->content .= ' <li>';
            $this->content .= '<a href="/annonces/index.php/post?id=' . $post['id'] . '">' . $post['title'] . '</a>';
            $this->content .= ' </li>';

        }

        $this->content.= '</ul>';
    }
}
```

```php
<?php
include_once "View.php";

class ViewPost extends View
{
    public function __construct($layout, $post)
    {
        parent::__construct($layout);

        $this->title= 'Exemple Annonces Basic PHP: Post';

        $this->content = '<h1>'.$post['title']. '</h1>';
        $this->content .= '<div class="date">'.$post['date'].'</div>';
        $this->content .= '<div class="body">'.$post['body'].'</div>';
    }
}
```

Toutes les vues sont redéfinies sous forme de classes. Il faut donc modifier la classe `Controllers` pour créer ces différentes vues, leur passer les bonnes données en paramètre et exécuter leur méthode `display`. Pour cela, il faut avant inclure les différentes vues au début du contrôleur.  

<img src="td1-img/phpstorm-controllers-oo1.png" width="800px"/>

<img src="td1-img/phpstorm-controllers-oo2.png" width="800px"/>

Vous pouvez tester l'application. Elle fonctionne toujours de la même manière pour l'utilisateur.

### 3 - Faire apparaître le cas d'utilisation et la logique métier

L'architecture actuelle reflète bien le patron ("design pattern") implémenté, mais elle ne respecte pas les principes d'une architecture logicielle "propre". Les règles métiers et le cas d'utilisation implémenté n'apparaissent pas au niveau de l'architecture. Nous pouvons déjà identifier deux entités métiers: `User` et `Post`. Ces entités représentent respectivement les utilisateurs enregistrés de l'application et les annonces. Ces entités sont liées au cas d'utilisation `AnnoncesChecking` qui contrôle la consultation des annonces par les utilisateurs enregistrés. Ce cas comprend l'affichage de toutes les annonces avec une authentification obligatoire, et la consultation éventuelle du détail d'une annonce par l'utilisateur. Les diagrammes ci-dessous modélisent cela.

<img src="td1-img/useCase.jpg" width="400px" />

<img src="td1-img/diagClassesMVC1.jpg" width="600px" />

Nous allons modifier le code pour implémenter ce diagramme de classes. Nous définissons une classe `User` par l'intermédiaire de la création du fichier `User.php`. Cette classe a les propriétés protégées `login` et `password`. Son constructeur permet de les initialiser. Elle contient par ailleurs deux méthodes permettant d'accéder à ces propriétés (des "getter"). Son code est le suivant:

```php
<?php

class User
{
    protected $login;
    protected $password;

    public function __construct( $login, $password )
    {
        $this->login = $login;
        $this->password = $password;
    }
    
    public function getLogin()
    {
        return $this->login;
    }
}
```

Nous faisons de même pour la classe `Post` qui contient les propriétés protégées `$id` (identifiant de l'annonce), `$title` (titre de l'annonce), `$body` (corps de l'annonce), et `$date` (date de publication). Elle contient aussi un constructeur et les méthodes "getter" associées.

```php
<?php

class Post
{
    protected $id;
    protected $title;
    protected $body;
    protected $date;

    public function __construct($id, $title, $body, $date)
    {
        $this->id = $id;
        $this->title = $title;
        $this->date = $date;
        $this->body = $body;
    }

    public function getId()
    {
        return $this->id;
    }

    public function getTitle()
    {
        return $this->title;
    }

    public function getBody()
    {
        return $this->body;
    }

    public function getDate()
    {
        return $this->date;
    }
}
```

Une fois les entités métiers implémentées, nous les utilisons dans `Model` et retournons des objets métiers suite aux requêtes dans la base de données. Nous en profitons pour changer le nom de la méthode `isUser` en `getUser`  afin que cela corresponde mieux à son fonctionnement.

```php
    public function getUser($login, $password)
    {
        $user = null;

        $query = 'SELECT login FROM Users WHERE login="' . $login . '" and password="' . $password . '"';
        $result = $this->dataAccess->query($query);

        if ($result->rowCount())
            $user = new User($login, $password);

        $result->closeCursor();

        return $user;
    }
```

La méthode `getAllAnonces` et `getPost` de `Model` sont également modifiées afin de retourner un tableau de `Post` et un `Post` (respectivement).

```php
    public function getAllAnnonces()
    {
        $result = $this->dataAccess->query('SELECT * FROM Post');
        $annonces = array();

        while ($row = $result->fetch()) {
            $currentPost = new Post($row['id'], $row['title'], $row['body'], $row['date']);
            $annonces[] = $currentPost;
        }

        $result->closeCursor();

        return $annonces;
    }

    public function getPost($id)
    {
        $id = intval($id);
        $result = $this->dataAccess->query('SELECT * FROM Post WHERE id=' . $id);
        $row = $result->fetch();
        
        $post = new Post($row['id'], $row['title'], $row['body'], $row['date']);
        
        $result->closeCursor();

        return $post;
    }
```


Pour finir, nous définissons la classe `AnnoncesChecking` correspondant au cas d'utilisation traité, avec ses différentes méthodes correspondant aux différents usages. Cette classe est composée de méthodes statiques car elle ne va pas être instanciée à ce stade. Les méthodes `getAllAnnonces` et `getPost` transforment les objets métiers en données textes, plus brutes, pouvant directement être utilisées dans les vues (sans pour autant créer de dépendances entre les vues et les entités métiers).

```php
<?php

class AnnoncesChecking
{

    public static function authenticate($login, $password, $data)
    {
        return ( $data->getUser($login, $password) != null );
    }

    public static function getAllAnnonces($data)
    {
        $annonces = $data->getAllAnnonces();

        $annoncesTxt = array();
        foreach ($annonces as $post){
            $annoncesTxt[] = [ 'id' => $post->getId(), 'title' => $post->getTitle(), 'body' => $post->getBody(), 'date' => $post->getDate() ];
        }

        return $annoncesTxt;
    }

    public static function getPost($id, $data)
    {
        $post = $data->getPost($id);

        return array( 'id' => $post->getId(), 'title' => $post->getTitle(), 'body' => $post->getBody(), 'date' => $post->getDate() );
    }
}
```

Il ne reste plus qu'à modifier le contrôleur pour utiliser cette classe `AnnoncesChecking` au lieu d'utiliser directement `Model`. D'ailleurs, le modèle n'est plus une propriété de `Controllers`. Il est simplement passé en paramètres à partir du contrôleur frontal.


```php
<?php
include_once "view/ViewLogin.php";
include_once "view/ViewAnnonces.php";
include_once "view/ViewPost.php";
include_once "AnnoncesChecking.php";

class Controllers
{
    public function loginAction()
    {
        $layout = new Layout("view/layout.html" );
        $vueLogin = new ViewLogin( $layout );

        $vueLogin->display();
    }

    public function annoncesAction($login, $password, $data)
    {
        $annoncesTxt = null;
        if ( AnnoncesChecking::authenticate($login, $password, $data) )
            $annoncesTxt = AnnoncesChecking::getAllAnnonces($data);
        else
            $login = '';

        $layout = new Layout("view/layout.html" );
        $vueAnnonces= new ViewAnnonces( $layout, $login, $annoncesTxt );

        $vueAnnonces->display();
    }

    public function postAction($id, $data)
    {
        $postTxt = AnnoncesChecking::getPost($id, $data);

        $layout = new Layout("view/layout.html" );
        $vuePost= new ViewPost( $layout, $postTxt );

        $vuePost->display();
    }
}
```

Il faut également modifier le contrôleur frontal `index.php`, et  passer en paramètre des méthodes de `Controllers` l'objet `Model` créé au début du code.

Testez à nouveau l'application.

### 4 - Ajouter des frontières

Si on dessine les différentes couches de l'architecture, on obtient le diagramme suivant:

<img src="td1-img/diagClassesMVC2.jpg" width="600px" />

On constate alors que certaines dépendances ne satisfont font pas encore les recommandations d'une architecture propre. Il s'agit des dépendances entre `Controllers` et les vues, ainsi que les dépendances entre `Model` et `AnnoncesChecking`. En effet, elles ne sont pas orientées vers des couches ayant un niveau abstraction plus élevé (en direction du coeur "métier" de l'architecture), comme le rappel la figure suivante.  

<img src="td1-img/cleanArchitecture.png" width="200px"/>

Pour cela, il faut inverser certaines dépendances en utilisant des classes/interfaces intermédiaires, ce qui aura pour effet de retracer les dépendances entre les couches de la manière suivante.

<img src="td1-img/diagClassesMVCclean.jpg" width="600px" />

Nous allons déjà créer une interface `DataAccessInterface` pour inverser la dépendance entre la couche "cas d'utilisation" et celle liée aux données. Nous en profitons pour renommer la classe `Model` en `DataAccess` pour que l'architecture soit plus "hurlante" (en faisant "Refactor > Rename" dans PhpStorm).

```php
<?php

interface DataAccessInterface
{
    public function getUser($login, $password);
    public function getAllAnnonces();
    public function getPost($id);
}
```

Il faut indiquer que la classe `DataAccess` implémente maintenant cette interface (`class DataAccess implements DataAccessInterface`). Si on faisait un typage strict (déclaration des types des paramètres et variables), il faudrait aussi modifier `AnnoncesChecking` pour qu'elle référence l'interface `DataAccessInterface` et non la classe `DataAccess`.


Nous allons maintenant inverser la dépendance entre les vues et le contrôleur. Pour cela, nous allons nous appuyer sur une classe `Presenter` chargée de lire les données générées par `AnnoncesChecking`. En effet, dans la version actuelle, le contrôleur récupère les données retournées par le cas d'utilisation, et les transfère aux vues. C'est ce mécanisme qui créé une dépendance entre le contrôleur et les vues. Pour éviter cela, nous allons créer un objet `AnnoncesChecking` dans le contrôleur frontal, puis  créer deux objets `Controllers` et `Presenter` à la suite, et leur  passer cet objet `AnnoncesChecking` en paramètre. Ainsi, les objets `Controllers` et `Presenter` auront directement accès aux méthodes et propriétés de `AnnoncesChecking`. `Presenter` pourra donc à tout moment récupérer les données à afficher. D'ailleurs, l'objet `Presenter` sera lui aussi passé en paramètre des vues afin qu'elles puissent récupérer ces informations au moment de leur affichage. Les vues ne seront plus créées par `Controllers` mais par le contrôleur frontal (`index.php`) en même temps que les autres objets.  

Dans la classe `AnnoncesChecking`, nous allons ajouter une propriété `annoncesTxt` qui sera un tableau de tableaux associatifs avec le texte des annonces. Il faut modifier le code des méthodes de cette classe pour utiliser cette propriété plutôt qu'une variable locale. Elles n'auront donc plus besoin de retourner une valeur résultat. Cette valeur sera stockée dans l'objet comme propriété, puis retourner via une méthode "getter" lorsque nécessaire.  Il faut aussi supprimer le `static` devant les méthodes. La classe n'est plus statique. Elle a vocation à être instanciée maintenant. 

```php
<?php

class AnnoncesChecking
{
    protected $annoncesTxt;

    public function getAnnoncesTxt()
    {
        return $this->annoncesTxt;
    }

    public function authenticate($login, $password, $data)
    {
        return ( $data->getUser($login, $password) != null );
    }

    public function getAllAnnonces($data)
    {
        $annonces = $data->getAllAnnonces();

        $this->annoncesTxt = array();
        foreach ($annonces as $post){
            $this->annoncesTxt[] = [ 'id' => $post->getId(), 'title' => $post->getTitle(), 'body' => $post->getBody(), 'date' => $post->getDate() ];
        }
    }

    public function getPost($id, $data)
    {
        $post = $data->getPost($id);

        $this->annoncesTxt[] = array( 'id' => $post->getId(), 'title' => $post->getTitle(), 'body' => $post->getBody(), 'date' => $post->getDate() );
    }
}
```

On notera que l'annonce spécifique ciblée par `getPost` est maintenant stockée également dans un tableau de tableaux. De manière générale, on remarque que cette classe va accéder aux données sur les annonces, récupérer les objets associés et enregistrer leurs informations sous forme textuelle. Le `Presenter` stocke une référence vers cet objet `AnnoncesChecking` et transforme ces données au format HTML pour qu'elles soient directement exploitables par les vues. Si `AnnoncesChecking` n'a pas retourné d'information (cas où l'utilisateur n'est pas authentifier), le `Presenter` retourne `null`.

```php
<?php

class Presenter
{
    protected $annoncesCheck;

    public function __construct($annoncesCheck)
    {
        $this->annoncesCheck = $annoncesCheck;
    }

    public function getAllAnnoncesHTML()
    {
        $content = null;
        if( $this->annoncesCheck->getAnnoncesTxt() != null )
        {
            $content = '<h1>List of Posts</h1>  <ul>';
            foreach( $this->annoncesCheck->getAnnoncesTxt()as $post ) {
                $content .= ' <li>';
                $content .= '<a href="/annonces/index.php/post?id=' . $post['id'] . '">' . $post['title'] . '</a>';
                $content .= ' </li>';
            }
            $content.= '</ul>';
        }
        return $content;
    }

    public function getCurrentPostHTML()
    {
        $content = null;
        if( $this->annoncesCheck->getAnnoncesTxt() != null )
        {
            $post = $this->annoncesCheck->getAnnoncesTxt()[0];

            $content = '<h1>' . $post['title'] . '</h1>';
            $content .= '<div class="date">' . $post['date'] . '</div>';
            $content .= '<div class="body">' . $post['body'] . '</div>';
        }
        return $content;
    }
}
```

Les vues `ViewAnnonces` et `ViewPost` n'ont donc plus besoin de faire cette transformation en HTML. Elles se contentent d'intégrer et d'affiche les données fournies par le `Presenter`. Il faut donc passer en paramètre de leur constructeur un `Presenter`, et utiliser  la méthode permettant de récupérer les informations propres à chacune.
 
 <img src="td1-img/phpstorm-viewAnnoncesPresenter.png" width="600px"/>
 
 <img src="td1-img/phpstorm-viewPostPresenter.png" width="600px"/>


Il faut maintenant supprimer cette partie affichage des vues de `Controllers` (ainsi que les `include` associés) et la transférer dans le contrôleur frontal. La méthode `loginAction` sera donc supprimée de cette classe. Dans `Controllers`, il faut aussi passer en paramètre des méthodes le cas d'utilisation (un objet `AnnoncesCheking`) et l'utiliser ensuite pour les appels de méthodes.

 <img src="td1-img/phpstorm-controllersUseCase.png" width="600px"/>

Au final, le contrôleur frontal `index.php` sera le suivant.

```php
<?php

// charge et initialise les bibliothèques globales
include_once 'DataAccess.php';
include_once 'Controllers.php';
include_once 'Presenter.php';
include_once 'AnnoncesChecking.php';
include_once 'view/ViewLogin.php';
include_once 'view/ViewAnnonces.php';
include_once 'view/ViewPost.php';


$data = null;
try {
    // construction du modèle
    $data = new DataAccess( new PDO('mysql:host=mysql-[compte].alwaysdata.net;dbname=[compte]_annonces_db', '[compte]_annonces', 'mdp') );

} catch (PDOException $e) {
    print "Erreur de connexion !: " . $e->getMessage() . "<br/>";
    die();
}

// initialisation du controller
$controller = new Controllers();

// intialisation du cas d'utilisation AnnoncesChecking
$annoncesCheck = new AnnoncesChecking() ;

// intialisation du presenter avec accès aux données de AnnoncesCheking
$presenter = new Presenter($annoncesCheck);

// chemin de l'URL demandée au navigateur
// (p.ex. /annonces/index.php)
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// route la requête en interne
// i.e. lance le bon contrôleur en focntion de la requête effectuée
if ( '/annonces/' == $uri || '/annonces/index.php' == $uri) {

    $layout = new Layout("view/layout.html" );
    $vueLogin = new ViewLogin( $layout );

    $vueLogin->display();
}
elseif ( '/annonces/index.php/annonces' == $uri
            && isset($_POST['login']) && isset($_POST['password']) ){

    $controller->annoncesAction($_POST['login'], $_POST['password'], $data, $annoncesCheck);

    $layout = new Layout("view/layout.html" );
    $vueAnnonces= new ViewAnnonces( $layout, $_POST['login'], $presenter);

    $vueAnnonces->display();
}
elseif ( '/annonces/index.php/post' == $uri
            && isset($_GET['id'])) {

    $controller->postAction($_GET['id'], $data, $annoncesCheck);

    $layout = new Layout("view/layout.html" );
    $vuePost= new ViewPost( $layout, $presenter );

    $vuePost->display();
}
else {
    header('Status: 404 Not Found');
    echo '<html><body><h1>My Page NotFound</h1></body></html>';
}

?>

```


### 5 - Regrouper les éléments en "packages"

L'étape suivante consiste à définir des packages ou des composants afin d'organiser le code et de donner une vision globale de l'application plus claire. Ils permettent par ailleurs de définir de nouvelles frontières entre les différentes parties de l'application, de masquer les fonctionnements internes, ce qui facilitera la modification du code lors de futures évolutions. Malheureusement, il n'y a pas de notion de packages en PHP comme on peut le trouver dans d'autres langages (p.ex. Java). Il est toutefois possible d'utiliser le concept d'espace de noms ou [`namespace`](https://www.php.net/manual/fr/language.namespaces.php).

Nous allons créer cinq espaces de nommages: `gui` pour les classes de l'interface utilisateur,  `data` pour l'accès aux données, `control` pour les classes intermédiaires faisant du contrôle et de la transformation, `service` pour les cas d'utilisation et `domain` pour les entités métiers. Chacun de ces espaces sera dans un répertoire séparé.

<img src="td1-img/diagClassesCleanPackages.jpg" width="800px"/>

Nous commençons donc par renommer le répertoire `view` en `gui` (avec "Refactor > Rename" de PhpStorm). Nous déclarons au début de chaque fichier PHP de ce répertoire l'espace de nommage avec l'instruction `namespace gui;`. Dans le contrôleur frontal `index.php`, on ajoutera l'instruction `use gui\{ViewLogin, ViewAnnonces, ViewPost, Layout};` (après les `include`) pour cibler les classes utilisées dans ce namespace.


 <img src="td1-img/phpstorm-namespaceIndex.png" width="600px"/>

 On remarque que `index.php` contient à la fois les inclusions des fichiers et l'inclusion des classes de l'espace de nommage. Ce problème propre à PHP est la raison pour laquelle les développeurs passent souvent par un gestionnaire de dépendances tel que Composer. Nous ne le ferons pas ici car cela sort du cadre de ce cours centré sur l'architecture.

Il faut maintenant faire de même pour les autres classes et créer leur espace de nommage associé comme illustré dans le diagramme de packages précédent. Le fait d'avoir un typage faible en PHP permet de supprimer une grande partie des `include` dans les fichiers sources. En effet, les types des variables sont déterminés à l'exécution et il n'y a pas de références aux classes associées dans le code, mis à part dans `index.php`, `DataAccess.php` (à cause de implémentation de l'interface et de la création d'objets) et les vues (à cause de l'héritage) .

 <img src="td1-img/phpstorm-indexFinal.png" width="800px"/>

 <img src="td1-img/phpstorm-dataAccessFinal.png" width="800px"/>

 Nous obtenons ainsi une première version de application implémentant le patron MVC et les principes des architectures propres.  Elle reste très basique fonctionnellement mais cette architecture la rend notamment  plus facile à faire évoluer.


### 6 - Isoler le cas d'utilisation "authentifier un utilisateur"

A ce sujet, une analyse de l'architecture actuelle  mais en avant une limite liée à l'authentification des utilisateurs. Cette dernière est intégrée au cas d'utilisation "afficher les annonces" alors qu'elle devrait être indépendante. En effet, l'authentification est l'étape préalable à l'utilisation de toutes les  fonctionnalités (actuelles ou futures) de l'outil et pas seulement du cas d'utilisation "afficher les annonces". 

Nous allons donc séparer authentification des utilisateurs et affichage des annonces.

Pour cela, vous commencez par créer dans `/service` une classe `UserChecking`. On **déplace** ensuite dans cette classe la méthode `authenticate` qui était initialement dans la classe `AnnoncesChecking`. 

```php
<?php

namespace service;

class UserChecking
{
    public function authenticate($login, $password, $data)
    {
        return ($data->getUser($login, $password) != null);
    }

}
```

<img src="td1-img/11-annoncesChecking.png" width="800px"/>


Nous allons ensuite ajouter une interface `UserAccessInterface` dans le répertoire `/service` pour séparer également l'accès aux données des utilisateurs. On **déplace** dans cette classe la méthode `getUser`, qui était initialement dans l'interface  `DataAccessInterface`. On en profitera pour renommer (Refactor > Rename) l'interface `DataAccessInterface` en `AnnonceAccessInterface`, afin que son nom reflète mieux son usage actuel.

```php
<?php

namespace service;

interface UserAccessInterface
{
    public function getUser($login, $password);
}
```
<img src="td1-img/11-annonceAccessInterface.png" width="800px"/>

Nous allons faire de même pour la classe `DataAccess` dans `/data` et la renommer `AnnonceSqlAccess`. Nous allons par ailleurs créer dans ce répertoire `/data` la classe `UserSqlAccess`  dans laquelle nous déplacerons le code associé aux utilisateurs (p.ex. `getUser`).


```php
<?php

namespace data;

use service\UserAccessInterface;
include_once "service/UserAccessInterface.php";

use domain\User;
include_once "domain/User.php";

class UserSqlAccess implements UserAccessInterface
{
    protected $dataAccess = null;

    public function __construct($dataAccess)
    {
        $this->dataAccess = $dataAccess;
    }

    public function __destruct()
    {
        $this->dataAccess = null;
    }

    public function getUser($login, $password)
    {
        $user = null;

        $query = 'SELECT * FROM Users WHERE login="' . $login . '" and password="' . $password . '"';
        $result = $this->dataAccess->query($query);

        if ( $row = $result->fetch() )
            $user = new User( $row['login'] , $row['password'], $row['name'], $row['firstName'], $row['date'] );

        $result->closeCursor();

        return $user;
    }
}
```

<img src="td1-img/11-annonceSqlAccess.png" width="800px"/>


Il faut maintenant modifier le contrôleur frontal `index.php` pour exploiter ces nouvelles classes et les passer en paramètres des bons objets. Nous allons en profiter pour généraliser l'authentification et mettre en place des sessions (si cela n'a pas été fait pendant le TP1). 

```php
<?php

require_once 'config.php';

// charge et initialise les bibliothèques globales
include_once 'data/AnnonceSqlAccess.php';
include_once 'data/UserSqlAccess.php';

include_once 'control/Controllers.php';
include_once 'control/Presenter.php';

include_once 'service/AnnoncesChecking.php';
include_once 'service/UserChecking.php';

include_once 'gui/Layout.php';
include_once 'gui/ViewLogin.php';
include_once 'gui/ViewAnnonces.php';
include_once 'gui/ViewPost.php';
include_once 'gui/ViewError.php';
include_once 'gui/ViewCreate.php';

use gui\{ViewLogin, ViewAnnonces, ViewPost, ViewError, Layout};
use control\{Controllers, Presenter};
use data\{AnnonceSqlAccess, UserSqlAccess};
use service\{AnnoncesChecking, UserChecking};

$data = null;
try {
    // initialisation de la connexion à la bd
    $dsn = 'mysql:host=' . DB_HOST . ';dbname=' . DB_NAME;
    $bd = new PDO($dsn, DB_USER, DB_PASS) );  
    
    // construction du modèle
    $dataAnnonces = new AnnonceSqlAccess($bd);
    $dataUsers = new UserSqlAccess($bd);

} catch (PDOException $e) {
    print "Erreur de connexion !: " . $e->getMessage() . "<br/>";
    die();
}

// initialisation du controller
$controller = new Controllers();

// intialisation du cas d'utilisation service\AnnoncesChecking
$annoncesCheck = new AnnoncesChecking() ;

// intialisation du cas d'utilisation service\UserChecking
$userCheck = new UserChecking() ;

// intialisation du presenter avec accès aux données de AnnoncesCheking
$presenter = new Presenter($annoncesCheck);

// chemin de l'URL demandée au navigateur
// (p.ex. /annonces/index.php)
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// définition d'une session d'une heure
ini_set('session.gc_maxlifetime', 3600);
session_set_cookie_params(3600);
session_start();

// Authentification et création du compte (sauf pour le formulaire de connexion et la route de déconnexion)
if ( '/annonces/' != $uri and '/annonces/index.php' != $uri and '/annonces/index.php/logout' != $uri ){

    $error = $controller->authenticateAction($userCheck, $dataUsers);

    if( $error != null )
    {
        $uri='/annonces/index.php/error' ;
        if( $error == 'bad login or pwd' or $error == 'not connected')
            $redirect = '/annonces/index.php';
    }
}

// route la requête en interne
// i.e. lance le bon contrôleur en fonction de la requête effectuée
if ( '/annonces/' == $uri || '/annonces/index.php' == $uri || '/annonces/index.php/logout' == $uri) {
    // affichage de la page de connexion

    session_destroy();
    $layout = new Layout("gui/layout.html" );
    $vueLogin = new ViewLogin( $layout );

    $vueLogin->display();
}
elseif ( '/annonces/index.php/annonces' == $uri ){
    // affichage de toutes les annonces

    $controller->annoncesAction($dataAnnonces, $annoncesCheck);

    $layout = new Layout("gui/layout.html" );
    $vueAnnonces= new ViewAnnonces( $layout,  $_SESSION['login'], $presenter);

    $vueAnnonces->display();
}
elseif ( '/annonces/index.php/post' == $uri
            && isset($_GET['id'])) {
    // Affichage d'une annonce

    $controller->postAction($_GET['id'], $dataAnnonces, $annoncesCheck);

    $layout = new Layout("gui/layout.html" );
    $vuePost= new ViewPost( $layout,  $_SESSION['login'], $presenter );

    $vuePost->display();
}
elseif ( '/annonces/index.php/error' == $uri ){
    // Affichage d'un message d'erreur

    $layout = new Layout("gui/layout.html" );
    $vueError = new ViewError( $layout, $error, $redirect );

    $vueError->display();
}
else {
    header('Status: 404 Not Found');
    echo '<html><body><h1>My Page NotFound</h1></body></html>';
}

?>
```

Le code de `index.php` est centré sur la création des objets (lignes 25 à 47) et le routage (lignes 49 à 120). Dans la partie routage, on trouve au début (lignes 50 à 70) le code lié à l'authentification (appliqué à toutes les pages à l'exception de la page de connexion). La méthode `authenticateAction` de la classe `Controllers` s'occupe de gérer cela. Elle retourne un message d'erreur si l'authentification a échoué (variable `$error`). Dans ce cas, une nouvelle route `/annonces/index.php/error` est définie (ligne 65), et une adresse de redirection est initialisée (`/annonces/index.php`). Cette route "erreur" est traitée à la ligne 103.  Elle consiste à afficher une fenêtre d'erreur et à rediriger l'utilisateur automatiquement vers une autre page ensuite. Une nouvelle classe `ViewError` est définie pour cela (cf. capture d'écran ci-dessous).

```php
<?php
namespace control;

class Controllers
{

    public function  authenticateAction($userCheck, $dataUsers){

        // Si l'utilisateur n'a pas de session ouverte
        if( !isset($_SESSION['login']) ) {

            // Si la page d'origine est le formulaire de connexion ou de création de compte
            if( isset($_POST['login']) && isset($_POST['password']) )
            {
                if( !$userCheck->authenticate($_POST['login'], $_POST['password'], $dataUsers) )
                {
                    // retourne une erreur si le compte n'est pas enregistré
                    $error = 'bad login or pwd';
                    return $error;

                }
                // Enregistrement des informations de session après une authentification réussie
                else {
                    $_SESSION['login'] = $_POST['login'] ;                    
                }
            }
            else{
                // retourne une erreur si la personne ne passe pas par le forumlaire de création ou de connexion
                $error = 'not connected';
                return $error;
            }

        }
    }

    public function annoncesAction( $data, $annoncesCheck)
    {
            $annoncesCheck->getAllAnnonces($data);
    }

    public function postAction($id, $data, $annoncesCheck)
    {
        $annoncesCheck->getPost($id, $data);
    }
}
```
<img src="td1-img/11-viewError.png" width="800px"/>

Le diagramme de classes suivant représente l'architecture de l'application à la fin du TD.

<img src="td1-img/diagClassesFinal.jpg" width="800px"/>

## Partie 4 -  Utilisation d'une IA générative pour construire l'architecture

Dans cette partie, vous allez comparer l'architecture construite avec celle générée par une IA générative (le choix du modèle est libre).

Dans un premier temps, vous allez écrire (ou générer) un prompt demandant à l'IA de vous générer le code de l'application précédente (à partir de rien, sans donner en entrée le sujet du TD). Vous spécifierez que l'application devra suivre une architecture MVC et appliquer les principes d'architecture propre.

Dans un second temps, vous analyserez et discuterez l'architecture produite par l'IA générative et celle produite à la fin du TD.

Les diagrammes de classes des deux architectures et l'analyse de leurs différences seront mis dans un document PDF et uploadé sur Ametice.