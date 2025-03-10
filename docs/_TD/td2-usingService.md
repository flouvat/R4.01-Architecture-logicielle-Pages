---
layout: post
title: "TD 2 -- Interrogation de services Web REST"
categories: jekyll update

mathjax: true
---

# TD 2 -- Interrogation de services Web REST


L'objectif de ce TD est d'intégrer l'utilisation de services web REST, publiés souvent sous forme d'API, dans l'architecture de l'application "Annonces" développée en PHP dans les TD/TP précédents.

Dans un premier temps, nous allons  intégrer dans l'application l'affichage d'entreprises proposant de l'alternance en informatique. Ces informations sont publiées sous forme d'API par la Direction Interministérielle du Numérique (DINUM). Dans un second temps, nous allons ajouter dans notre application l'affichage des annonces d'emploi en informatique disponibles à Pôle Emploi.


Le diagramme de classes suivant représente l'architecture de l'application à la fin du TD.

<img src="td2-img/diagClasses3.jpg" width="800px"/>


Les captures d'écran suivantes représentent l'application obtenue à la fin (avec une interface utilisateur plus que basique, mais ce n'est pas l'objectif du cours d'aller plus loin).

<img src="td2-img/3-finalScreenshotAnnonces.png" width="400px" border="1px solid black"/>

<img src="td2-img/3-finalScreenshotNew.png" width="400px" border="1px solid black"/>

<img src="td2-img/3-finalScreenshotCompanies.png" width="400px" border="1px solid black"/>

<img src="td2-img/3-finalScreenshotJobs.png" width="400px" border="1px solid black"/>


## Partie 0 - Améliorer l'architecture existante pour pouvoir intégrer les API

L'architecture actuelle a regroupé toutes les méthodes d'accès dans l'interface `DataAccessInterface`. Une source de données implémentant cette interface doit donc contenir à la fois des méthodes d'accès aux informations des utilisateurs, et des méthodes d'accès aux informations des annonces. Une source de données "compatible" avec cette interface doit donc fournir ces deux types de données. Or, les API que nous allons consommer ne fournissent que des informations de types "annonces". Cette interface `DataAccessInterface` englobe donc trop de méthodes. Elle  ne respecte pas le principe "single use" des principes SOLID.

Cette limite mais en avant un autre problème lié aux données utilisateurs.  L'authentification des utilisateurs est liée au cas d'utilisation "afficher les annonces" alors qu'elle devrait être indépendante. En effet, l'authentification est l'étape préalable à l'utilisation de toutes les fonctionnalités de l'outil et pas seulement du cas d'utilisation "afficher les annonces". 

Nous allons donc séparer authentification des utilisateurs et affichage des annonces, puis spécialiser les interfaces d'accès aux données.

### 0.1 - Isoler le cas d'utilisation "authentifier un utilisateur"

Si ce n'est pas déjà fait, vous créez dans `/service` une classe `UserChecking`. On **déplace** ensuite dans cette classe la méthode `authenticate` qui était initialement dans la classe `AnnoncesChecking`. 

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

<img src="td2-img/0-annoncesChecking.png" width="800px"/>


### 0.2 - Spécialiser les interfaces d'accès aux données

Nous allons ensuite ajouter une interface `UserAccessInterface` dans le répertoire `/service` pour séparer également l'accès aux données des utilisateurs. On **déplace** dans cette classe la méthode `getUser`, qui était initialement dans l'interface  `DataAccessInterface`. On en profitera pour renommer (Refactor > Rename) l'interface `DataAccessInterface` en `AnnonceAccessInterface`, afin que son nom reflète mieux son usage actuel.

```php
<?php

namespace service;

interface UserAccessInterface
{
    public function getUser($login, $password);
}
```
<img src="td2-img/0-annonceAccessInterface.png" width="800px"/>

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

<img src="td2-img/0-annonceSqlAccess.png" width="800px"/>

### 0.3 - Mettre à jour le contrôleur frontal et généraliser l'authentification


Il faut maintenant modifier `index.php` pour exploiter ces nouvelles classes et les passer en paramètres des bons objets. Nous allons en profiter pour généraliser l'authentification et mettre en place des sessions (si cela n'a pas été fait pendant le TP1). 

```php
<?php

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
    $bd = new PDO('mysql:host=mysql-[compte].alwaysdata.net;dbname=[compte]_annonces_db', '[compte]_annonces', 'mdp');
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
<img src="td2-img/0-viewError.png" width="800px"/>


## Partie 1 - Afficher les entreprises proposant de d'alternance

Une fois ces transformations effectuées, nous pouvons intégrer le nouveau cas d'usage lié à l'affichage des entreprises susceptibles de proposer de l'alternances en informatique.

La documentation de l'[API Alternance](https://www.data.gouv.fr/fr/dataservices/api-alternance/) à utiliser est disponible sur le site [data.gouv.fr](https://www.data.gouv.fr/fr/dataservices/). Ce site vise à promouvoir l'utilisation et la diffusion de données open source en mettant à disposition les données de toutes les administrations. La [documentation technique de cette API](https://labonnealternance-recette.apprentissage.beta.gouv.fr/api/docs/static/index.html) présente plusieurs ressources et permet de tester des requêtes. Nous allons plus particulièrement utiliser le point d'accès permettant de récupérer la liste des entreprises proposant des contrats en alternance en informatique à proximité (dans un rayon de 100km). Cette liste sera affichée dans l'application Annonces dans une interface graphique dédiée. Cette interface sera accessible par un menu commun à toute l'application.


### 1.1 - Récupérer les informations sur les entreprises à partir de l'API


Nous allons créer maintenant la  classe `ApiAlternance` dans le répertoire `/data`. Cette classe implémente  l'interface `AnnonceAccessInterface`. Cette classe doit donc implémenter les deux méthodes de l'interface: `getAllAnnonces` et `getPost`. 

La méthode `getAllAnnonces` doit consommer les données de l'API REST de l'API Alternance. Comme l'indique la [documentation de l'API](https://labonnealternance-recette.apprentissage.beta.gouv.fr/api/docs/static/index.html), il faut interroger la ressource `API/V1/jobsEtFormations` en GET en lui envoyant un certain nombre de paramètres. L'URL interrogée sera de la forme `https://labonnealternance-recette.apprentissage.beta.gouv.fr/api/v1/jobsEtFormations?romes=M1801%2CM1810&caller=contact%40domaine%20nom_de_societe&latitude=43.5283&longitude=5.44973&radius=100&insee=13100&diploma=6%20%28Licence%2C%20BUT...%29`. Elle indique notamment les métiers ciblés (par leur code ROME), limite la recherche à une certaine zone géographique et à une formation. L'objectif de la méthode `getAllAnnonces` est de générer cette requête en PHP en ciblant les métiers de l'informatique (M1801 "Administrateur / Administratrice réseau informatique" et M1810 "Technicien / Technicienne informatique"), une zone de 100km autour d'Aix-en-Provence et un niveau de formation BUT/Licence. Le résultat de cette requête est au format JSON. La méthode parcourt ensuite ce résultat pour construire un tableau d'entreprises susceptibles de prendre des alternants. Ce tableau est ensuite enregistré dans un fichier sur le serveur afin de pouvoir être réutilisé pour afficher le détail d'une entreprise. Cette étape est indispensable car l'API ne permet pas de récupérer les informations d'une entreprise en particulier. En effet, les variables initialisées par la méthode `getAllAnnonces` sont supprimées au rechargement de `index.php`. Il est donc nécessaire d'enregistrer les informations produites par l'API pour qu'elles soient persistantes, et ainsi éviter de devoir tout recharger à chaque fois. Nous créons pour cela un fichier appelé `cache_alternance` dans le répertoire `/data`. Cet encodage d'un objet sous une forme plus petite pour le rendre persistant sous forme de fichier s'appelle de la sérialisation.


```php
    public function getAllAnnonces(){

        $romes = urlencode('M1801,M1802,M1803,M1804,M1805,M1806,M1810');

        $latitudeAix = '43.529742';
        $longitudeAix = '5.447427';
        $radius = '100';
        $inseeAix = '13100';
        $diploma = urlencode('6 (Licence, BUT...)');

        // URL de l'API
        $apiUrl = "https://labonnealternance-recette.apprentissage.beta.gouv.fr/api/v1/jobsEtFormations";

        // paramètres de la requête HTTP
        $query ='?romes='.$romes.'&latitude='.$latitudeAix.'&longitude='.$longitudeAix.'&radius='.$radius.'&insee='.$inseeAix.'&diploma='.$diploma.'&caller=contact%40domaine%20nom_de_societe';

        // initialisation de la connexion à l'API avec CURL
        $curlConnection  = curl_init();
        
        // définition des paramètres de la requête CURL
        $params = array(
            CURLOPT_URL =>  $apiUrl.$query,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => array('accept: application/json')
        );
        curl_setopt_array($curlConnection, $params);
        
        // exécution de la requête HTTP avec CURL
        $response = curl_exec($curlConnection);
        curl_close($curlConnection);

        if( !$response )
            echo curl_error($curlConnection);

        // transformation du JSON récupéré en tableau associatif
        $response = json_decode( $response, true );
        
        // parcours du tableau associatif pour extraire les
        // entreprises à fort potentiel de recrutement en alternance dans la région d'Aix
        $annonces = array();
        foreach ( $response['jobs']['lbaCompanies']['results'] as $entreprise){

            $id = $entreprise['company']['siret'];
            $title = $entreprise['title'];
            $body = $entreprise['nafs'][0]['label'].'; '.$entreprise['contact']['email'].'; '.$entreprise['place']['fullAddress'];

            $currentPost = new Post($id, $title, $body, date("Y-m-d H:i:s") );
            $annonces[$id] = $currentPost;
        }

        // enregistrement des annonces dans un fichier sur le serveur (serialisation)
        $annoncesSerialized = serialize($annonces);
        file_put_contents('data/cache_alternance', $annoncesSerialized);

        return $annonces;
    }
```

Vous pouvez tester rapidement cette méthode en créant un objet `ApiAlternance` au début de `index.php` et en faisant un `var_dump` du résultat retourné par la méthode `getAllAnnonces`.


Une fois la méthode `getAllAnnonces` définie, il faut aussi implémenter dans la classe `ApiAlternance` la méthode `getPost`. Cette méthode se contente de recharger les données des entreprises enregistrées dans le fichier `data/cache_alternance`, puis d'accéder à l'entreprise ciblée via son identifiant (siret).

```php
    public function getPost($id){

        $annoncesSerialized = file_get_contents('data/cache_alternance');
        $annonces = unserialize( $annoncesSerialized );

        return $annonces[$id];
    }
```

### 1.2 - Créer l'interface de visualisation des entreprises

Tout comme pour les annonces enregistrées dans la base de données interne de l'application, il faut maintenant créer deux vues pour visualiser ces entreprises. Nous allons appeler ces vues: `ViewCompanyAlternance` et `ViewAnnoncesAlternance`.  La première est très similaire à `ViewPost`. La principale différence est la route associée au lien de retour. En effet, cette page doit rediriger vers la route `/annonces/index.php/annoncesAlternance` et non vers `/annonces/index.php/annonces`.  La classe `ViewAnnoncesAlternance` est similaire à `ViewAnnonces`. Elle utilise néanmoins une autre méthode de `Presenter` appelée `getAllAlternanceHTML`. Cette méthode n'existe pas encore et doit être définie dans la classe `Presenter`. Elle génère la liste des entreprises sous forme de liens hypertextes redirigeant vers l'interface d'affichage des détails d'une entreprise (i.e. la vue `ViewCompanyAlternance`).

```php
<?php
namespace gui;

include_once "View.php";

class ViewCompanyAlternance extends View
{
    public function __construct($layout, $login, $presenter)
    {
        parent::__construct($layout, $login);

        $this->title= 'Exemple Annonces Basic PHP: Entreprise';

        $this->content = $presenter->getCurrentPostHTML();

        $this->content .= '<a href="/annonces/index.php/annoncesAlternance">retour</a>';
    }
}
```

```php
<?php
namespace gui;

include_once "View.php";

class ViewAnnoncesAlternance extends View
{
    public function __construct($layout, $login, $presenter)
    {
        parent::__construct($layout, $login);

        $this->title= 'Exemple Annonces Basic PHP: Alternance';

        $this->content = $presenter->getAllAlternanceHTML();
    }
}
```
<img src="td2-img/1-presenter.png" width="800px"/>


### 1.3 - Mettre à jour le contrôleur frontal et ajouter un menu de navigation

Il faut maintenant utiliser ces classes dans `index.php`. Il faut commencer par importer les fichiers des différentes classes,  puis créer une instance de `ApiAlternance`. Ensuite, il faut définir et traiter deux nouvelles routes pour traiter les deux nouvelles fonctionnalités (afficher la liste des entreprises et afficher le détail d'une entreprise). On remarque que le code est très similaire à celui du cas d'utilisation "Afficher les annonces". Il utilise d'ailleurs les mêmes classes et méthodes, dont la classe `AnnoncesChecking`. Seule la source de données est différente (un objet `ApiAlternance` au lieu d'un objet `AnnonceSqlAccess`), tout comme les vues appelées.

```php
elseif ( '/annonces/index.php/annoncesAlternance' == $uri ){
      // Affichage de toutes les entreprises offrant de l'alternance

    $controller->annoncesAction($apiAlternance, $annoncesCheck);

    $layout = new Layout("gui/layout.html" );
    $vueAnnoncesAlternance= new ViewAnnoncesAlternance( $layout,  $_SESSION['login'], $presenter);

    $vueAnnoncesAlternance->display();
}
elseif ( '/annonces/index.php/companyAlternance' == $uri
            && isset($_GET['id'])) {
    // Affichage d'une entreprise offrant de l'alternance

    $controller->postAction($_GET['id'], $apiAlternance, $annoncesCheck);

    $layout = new Layout("gui/layout.html" );
    $vuePostAlternance = new ViewCompanyAlternance( $layout,  $_SESSION['login'], $presenter );

    $vuePostAlternance->display();
}

```

L'affichage des informations sur les entreprises susceptibles d'offrir des contrats d'alternance est fonctionnel, mais reste difficilement accessible dans l'interface graphique actuelle (les routes doivent être saisies manuellement). Face à cette limite, nous allons très rapidement ajouter un menu à l'interface graphique pour pouvoir naviguer dans les différentes parties de l'application ("accueil", "annonces", et "alternance"). Nous allons en profiter aussi pour intégrer un lien de déconnexion de session. Pour cela, il suffit de créer un nouveau fichier "layout", appelé `layoutLogged.html` (dans `/gui`), qui intègre le menu évoqué. 

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <title> %title% </title>
  <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
</head>
<body>

<nav>
  <ul>
    <li><a href="/annonces/index.php">Acceuil</a></li>
    <li><a href="/annonces/index.php/annonces">Annonces IUT</a></li>
    <li><a href="/annonces/index.php/annoncesAlternance">Alternance - Pôle Emploi</a></li>
    <li><a href="/annonces/index.php/logout">Déconnexion</a></li>*
  </ul>
</nav>

%content%

</body>
</html>
```

Dans `index.php`, le nom de ce fichier est passé en paramètre des instances de `Layout` correspondant à des fenêtres affichées lorsque l'on est connecté. Il s'agit plus particulièrement du code associé aux routes: `/annonces/index.php/annoncesAlternance`, `/annonces/index.php/companyAlternance`, `/annonces/index.php/annonces` et `/annonces/index.php/post`.  A noter qu'une route `/annonces/index.php/logout` a été créée au passage. Elle est associée à la suppression de la session (`session_destroy`) et à l'affichage du formulaire de connexion.  

<img src="td2-img/1-index.png" width="800px"/>




## Partie 2 - Afficher les annonces d'emploi de Pôle Emploi

Dans cette seconde partie, nous allons ajouter dans notre application l'affichage des annonces disponibles à Pôle Emploi. Pour cela, nous allons consommer l'[API Offres d'emploi](https://pole-emploi.io/data/api/offres-emploi?tabgroup-api=presentation&doc-section=api-doc-section-caracteristiques). Contrairement à la première API, il faut être au préalable enregistré sur le site Pôle Emploi pour utiliser ces données. Il faut ensuite identifier son application, et  souscrire à une des API publiées pour obtenir un identifiant ainsi qu'une clé secrète. Par ailleurs, il est nécessaire de générer un jeton ("access token", encore appelé "bearer token") avant de pouvoir effectuer une requête. L'ensemble de la procédure est décrit dans la [documentation technique de cette API](https://pole-emploi.io/data/documentation).


### 2.1 - Créer un compte sur l'API Pôle Emploi et enregistrer son application

La première étape consiste donc à [créer son espace sur le site pole-emploi.io](https://pole-emploi.io/inscription).

<img src="td2-img/2-creerCompte.png" width="800px"/>

Une fois le compte créé, un mail d'activation vous est envoyé pour valider ce dernier. Vous pouvez ensuite vous connecter et aller dans votre espace pour créer une application. Il faut saisir le nom de son application, une petite description, et l'URL à partir de laquelle l'application va accéder à l'API (`https://[mon_compte].alwaysdata.net/annonces/index.php`).

<img src="td2-img/2-creationAppli.png" width="800px"/>

Lorsque l'application est enregistrée, le site vous affiche l'identifiant et la clé à utiliser pour générer un jeton d'accès par la suite.

<img src="td2-img/2-monAppli.png" width="800px"/>

Avant d'intégrer cela dans notre code, il faut encore ajouter des API dans la liste des API autorisées pour notre application. Nous allons ajouter l'API "Offres d'emploi", qui permet d'accéder à tout moment et en temps réel à l’ensemble des offres d’emploi disponibles sur le site de Pôle emploi. A ce niveau, la plateforme permet de visualiser la [documentation technique de cette API](https://pole-emploi.io/data/api/offres-emploi?tabgroup-api=documentation&doc-section=api-doc-section-rechercher-par-crit%C3%A8res) en particulier, et donc de voir comment accéder et utiliser cette ressource (URI, paramètre des requêtes, etc.). Gardez cette fenêtre ouverte car elle vous sera utile par la suite.

<img src="td2-img/2-docAPI.png" width="800px"/>

### 2.2 - Récupérer les informations sur les emplois à partir de l'API 


Nous allons maintenant pouvoir intégrer ce nouveau cas d'utilisation dans notre code. Tout comme précédemment, nous allons commencer par ajouter une nouvelle classe `ApiEmploi` dans le répertoire `/data`. Cette classe implémente l'interface `AnnonceAccessInterface`, et doit donc en implémenter les deux méthodes `getAllAnnonces` et `getPost`. Avant d'implémenter ces méthodes, nous allons créer dans cette classe une méthode `getToken()` afin de récupérer un jeton d'accès de l'API. Comme indiqué dans la documentation de [auth0](https://auth0.com/docs/secure/tokens/access-tokens), ces jetons sont utilisés pour permettre à une application d'accéder à une API. On parle d'authentification basée sur les jetons. L'application reçoit un jeton d'accès une fois qu'un utilisateur s'est authentifié avec succès, puis transmet le jeton d'accès en tant qu'information d'identification lorsqu'il appelle l'API cible. Ce jeton informe l'API que l'application a été autorisée à accéder à celle-ci et à effectuer les actions  spécifiées. Pour récupérer ce jeton, il faut faire une requête HTTP dont les paramètres sont décrits dans [la documentation de l'API (partie "Générer un access token")](https://pole-emploi.io/data/documentation/utilisation-api-pole-emploi/generer-access-token).

<img src="td2-img/2-accessToken.png" width="800px"/>

```php
function getToken(){
        $curl = curl_init();

        $url = "https://entreprise.pole-emploi.fr/connexion/oauth2/access_token";

        $auth_data = array(
            "grant_type" => "client_credentials",
            "client_id" => CLIENT_ID,
            "client_secret" => CLIENT_SECRET,
            "scope" => "api_offresdemploiv2 o2dsoffre"
        );

        $params = array(
            CURLOPT_URL =>  $url."?realm=%2Fpartenaire",
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => http_build_query($auth_data),
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => array(
                "content-type: application/x-www-form-urlencoded"
            )
        );

        curl_setopt_array($curl, $params);

        $response = curl_exec($curl);

        if(!$response)
            die("Connection Failure");

        curl_close($curl);

        return json_decode($response, true);
    }
```

Comme indiqué dans la documentation, le point d'accès pour avoir ce jeton est `https://entreprise.pole-emploi.fr/connexion/oauth2/access_token` (accès en POST). La requête HTTP envoyée doit avoir comme paramètre suite à cette URL la valeur `realm=/partenaire`. Dans le code ci-dessus, cette information est mise au niveau du paramètre `CURL_OPT`. A noter que le caractère `/` a été remplacé par `%2F` (son code d'encodage) car il s'agit d'un caractère spécial. L'en-tête  `content-type: application/x-www-form-urlencoded` a été associée à l'option `CURLOPT_HTTPHEADER` de cURL. Le corps de la requête POST, correspondant aux informations d'authentification demandées par cette API, a été stocké sous forme de tableau associatif `$auth_data`, puis encodé pour être compatible avec les conventions des URL, et associé au paramètre `CURLOPT_POSTFIELDS` de cURL. Le paramètre `CURLOPT_RETURNTRANSFER` initialisé à `true`, permet quant à lui de demander une réponse sous forme de chaîne de caractères. Comme indiqué dans la documentation, cette réponse sera plus particulièrement au format JSON (`application/json`). Ces paramètres sont ensuite associés à la requête cURL (`curl_setopt_array`). Puis, cette dernière est exécutée (`curl_exec`) et la connexion est fermée (`curl_close`). La fonction retourne un tableau associatif avec les informations de la chaîne de caractères JSON renvoyée par l'API (`json_decode`). 

Vous pouvez tester cette fonction, et afficher le jeton généré, en copiant  le code suivant dans `index.php`. Une fois ce test effectué, vous mettrez cette fonction `getToken` en privé dans la classe  `ApiEmploi`.

```php
// initialiser la source de données "API Emploi"
$apiEmploi = new ApiEmploi();
$token = $apiEmploi->getToken() ;
echo $token['access_token'];
```

Nous pouvons maintenant utiliser cette fonction pour générer un jeton d'authentification dans la méthode `getAllAnnonces` et pouvoir faire ainsi une requête GET à l'API pour récupérer les offres d'emploi en informatique. Le point d'entrée de l'[API Offres d'emploi](https://pole-emploi.io/data/api/offres-emploi?tabgroup-api=documentation&doc-section=api-doc-section-caracteristiques) est `https://api.pole-emploi.io/partenaire/offresdemploi/v2/offres/search`. Cette API offre un grand nombre de paramètres d'entrée pour filtrer les offres. Nous allons utiliser le paramètre `sort=1` pour trier par date de création décroissante,  et le paramètre `domaine=M18` représentant les offres portant sur le domaine des systèmes d'information et de télécommunication.

<img src="td2-img/2-getAllAnnonces1.png" width="800px"/>
<img src="td2-img/2-getAllAnnonces2.png" width="800px"/>

La principale différence avec l'API précédente est qu'il faut générer le jeton (ligne 54) et le passer en paramètre de la requête HTTP (ligne 62), avant d'exécuter la requête (ligne 65). Bien entendu, la structure des données reçues est aussi différente (même si elle reste en JSON). Il faut donc adapter la création des objets `Post` en fonction (lignes 76 à 91).

### 2.3 - Créer l'interface de visualisation des emplois

Une fois cette classe définie, il faut procéder comme dans la partie précédente. Il faut créer une vue `ViewAnnoncesEmploi` dans `/gui`, puis ajouter dans la classe `Presenter` une méthode `getAllEmploiHTML` (qui sera utilisée dans la vue pour récupérer les annonces à afficher au format HTML), et instancier ces classes dans `index.php` pour définir une nouvelle route `/annonces/index.php/annoncesEmploi`.

<img src="td2-img/2-viewAnnoncesEmploi.png" width="800px"/>
<img src="td2-img/2-presenter.png" width="800px"/>
<img src="td2-img/2-index.png" width="800px"/>

Nous allons également ajouter un lien vers cette liste d'annonces dans le menu de l'application situé dans `layoutLogged.html`.

Tout comme avec l'API de la partie 1, nous devons aussi définir la méthode  `getPost` dans la classe `ApiEmploi`. Cette méthode sera utilisée pour afficher le détail d'une offre d'emploi. On remarque dans la capture d'écran précédente que la route vers cette partie de l'application a déjà été préparée dans la vue `ViewAnnoncesEmploi`. En effet, on voit la génération de liens hypertextes pour chaque offre redirigeant vers `/annonces/index.php/offreEmploi?id=`. 

```php
  public function getPost($id)
    {
        $token = $this->getToken() ;

        $api_url = "https://api.pole-emploi.io/partenaire/offresdemploi/v2/offres/";

        $curlConnection  = curl_init();
        $params = array(
            CURLOPT_URL =>  $api_url.$id,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => array("Authorization: Bearer " . $token['access_token'] )
        );
        curl_setopt_array($curlConnection, $params);
        $response = curl_exec($curlConnection);
        curl_close($curlConnection);

        if( !$response )
            echo curl_error($curlConnection);

        $response = json_decode( $response, true );

        // récupération des informations et création du Post
        $id = $response['id'];
        $title = $response['intitule'];
        $body = $response['description'];

        if( isset($response['salaire']['libelle']) )
            $body.='; '.$response['salaire']['libelle'];
        if( isset($response['entreprise']['nom']) )
            $body.='; '.$response['entreprise']['nom'];
        if ( isset($response['contact']['coordonnees1']) )
            $body.='; '.$response['contact']['coordonnees1'];

        return  new Post($id, $title, $body, date("Y-m-d H:i:s") );
    }
```

Par contre, contrairement à la précédente API, l'API Offres Emploi permet de récupérer les informations d'une seule offre. Pour cela, il suffit d'interroger la ressource [https://api.pole-emploi.io/partenaire/offresdemploi/v2/offres/{offerId}](https://pole-emploi.io/data/api/offres-emploi?tabgroup-api=documentation&doc-section=api-doc-section-consulter-une-offre). Il n'est donc plus nécessaire de sérialiser les objets pour les stocker sur le disque. Nous pouvons directement interroger cette ressource, comme cela a été fait dans la méthode `getAllAnnonces` (i.e. en générant un jeton d'authentification et en envoyant la requête GET).

Il faut  ensuite créer une vue `ViewOffreEmploi` pour afficher les informations détaillées de chaque offre d'emploi.

<img src="td2-img/2-viewOffreEmploi.png" width="800px"/>

### 2.4 - Mettre à jour le contrôleur frontal 

Pour finir, il faut instancier ces classes dans `index.php` et associée cette partie à la nouvelle route `/annonces/index.php/offreEmploi`.


<img src="td2-img/2-index2.png" width="800px"/>



## Partie 3 - Ajouter des annonces avec détection de la ville

Dans cette partie, nous allons rapidement ajouter la possibilité d'ajouter une annonce (stage, alternance ou emploi). Afin de faciliter la saisie de la localisation de l'offre, nous intègrerons dans le formulaire un bouton permettant à l'utilisateur de détecter automatiquement  la ville dans laquelle il se trouve. Cette fonctionnalité s'appuiera sur une API pour récupérer le nom de la ville à partir de coordonnées GPS.

### 3.1 - Compléter les informations sur les annonces dans la base de données

Nous profitons de ce nouveau cas d'utilisation pour compléter les informations stockées dans la base de données. Pour chaque annonce, nous allons plus particulièrement ajouter l'identifiant de l'utilisateur  l'ayant relayé (son "login"), la ville associée à l'offre (p.ex. la ville où doit être faite l'alternance), le mail de la personne à contacter, et le type de contrat (stage, alternance, CDD ou CDI). Pour cela, nous passons par l'interface administration de PhpMyAdmin accessible via l'interface d'Alwaysdata.

Pour ajouter ces colonnes par l'interface graphique de PhpMyAdmin, il faut sélectionner la base de données dans l'arborescence à gauche, aller dans l'onglet `Structure`, indiquer le nombre de colonnes à ajouter (champ de saisie `Add`) et cliquer sur `Go`;

<img src="td2-img/3-structurePhpMyAdmin.png" width="800px"/>

PhpMyAdmin demande ensuite de saisir le nom des nouvelles colonnes (`login`, `location`, `contactMail`, et `contractType`), et de préciser leurs types. Ces colonnes sont de type `VARCHAR` avec une longueur de 50, à l'exception de `contractType` qui est de type `ENUM` avec pour valeur `'stage','alternance','CDD','CDI'` . Une fois les informations saisies, il faut les enregistrer en cliquant sur `Save`.

Si ce n'est pas déjà fait, nous mettons la colonne `id` de la table `Post` (i.e. la clé primaire) en mode `AUTO_INCREMENT`. Cette commande permet de spécifier que les valeurs de cette colonne sont remplies automatiquement et auto-incrémentées. Il n'est pas nécessaire ainsi de donner l'identifiant d'une annonce lors de sa saisie. Il est automatiquement défini par le SGBD. Cette option s'active à partir de l'onglet `Structure` de PhpMyAdmin. Il faut sélectionner l'option `Change` du champs `id`, cocher la case `A_I` à la fin et sauvegarder.

Nous en profitons aussi pour définir `Post.login` comme clé étrangère de `Users.login`. Pour ajouter cette contrainte, nous restons dans l'onglet  `Structure` de PhpMyAdmin, et sélectionnons `Relation view`. Dans la fenêtre qui s'affiche (section "Foreign key constraints"),  nous ajoutons notre contrainte `fk_post-users` liant la colonne `login`de `Post` et la colonne `login` de `Users`, et l'enregistrons (bouton `Save`). A ce moment, si une erreur `a foreign key constraint fails` s'affiche, il faut modifier les données enregistrées dans `Post` afin qu'elle vérifie cette contrainte (i.e. que chaque annonce enregistrée soit associée à un identifiant d'utilisateur existant), et ajouter à nouveau cette contrainte.

<img src="td2-img/3-fkPhpMyAdmin.png" width="800px"/>


### 3.2 - Créer le formulaire de saisie d'une nouvelle annonce

Nous pouvons maintenant revenir au code de notre application dans PhpStorm. Nous commençons par rapidement modifier le menu de notre interface graphique pour que le nom des différentes sections soit plus parlant. Pour faire cela, nous allons dans le fichier `layoutLogged.html` et renommons "Annonces" en "Annonces IUT", "Alternance" en "Alternance- Pôle Emploi", et "Emploi" en "Jobs - Pôle Emploi". Grâce à notre architecture, cette modification est appliquée sur toutes les fenêtres ("connectées") de notre application. 

Nous allons maintenant ajouter les classes nécessaires à notre nouveau cas utilisation "ajouter une annonce".  Nous commençons par créer le formulaire de création d'annonce. Nous créons donc une classe `ViewCreateAnnonce` héritant de `View`, et ajoutons la construction de notre formulaire de saisie d'annonce dans le constructeur. L'affichage sera effectué par la méthode `display` héritée de la classe `View` et utilisant les champs `title` et `body` initialisés dans ce constructeur de `ViewCreateAnnonce`. 

```php
<?php
namespace gui;

include_once "View.php";


class ViewCreateAnnonce extends View
{
    public function __construct($layout)
    {
        parent::__construct($layout);

        $this->title = 'Exemple Annonces Basic PHP: Création d\'une annonce';

        $this->content = ' 
            <form method="post" action="/annonces/index.php/annonces">
                <label for="title"> Titre </label> :
                <input type="text" name="title" id="title" placeholder="defaut" required />
                <br />           
                <label for="contractType"> Type de contrat </label> :
                <select name="contractType" id="contractType">
                    <option value="stage">stage</option>
                    <option value="alternance">alternance</option>
                    <option value="CDD">CDD</option>
                    <option value="CDI">CDI</option>
                </select>
                <br />
                <label for="body"> Description </label> :        
                <textarea name="body" id="body" rows="3" cols="50"> </textarea>
                <br />
                <label for="location"> Localisation </label> :
                <input type="text" name="location" id="location" required />
                <button type="button" id="localizeBtn">ici</button>
                <br />                
                <label for="contactMail"> Mail de contact </label> :
                <input type="text" name="contactMail" id="contactMail" />
                <br />
                <input type="submit" value="Envoyer">
                <input type="reset"> 
            </form>
            <a href="/annonces/index.php/annonces">retour</a>
            ';
    }
}
```

L'affichage de ce formulaire est associé à la route  `/annonces/index.php/createAnnonce` dans `index.php.`

<img src="td2-img/3-indexCreateAnnonce.png" width="800px"/>

Par ailleurs, pour le rendre plus facilement accessible, nous ajoutons un lien hypertexte vers cette nouvelle route à la fin de la vue `ViewAnnonces`.

<img src="td2-img/3-viewAnnonce.png" width="800px"/>


### 3.3 - Ajouter l'annonce dans la base de données

La destination de ce formulaire est `/annonces/index.php/annonces`. Il faut donc ajouter son traitement (la création de l'annonce) dans `index.php` dans la partie du code correspondant à cette route. Pour cela, nous allons créer une nouvelle classe `AnnonceCreation` dans `/service`  correspondant à ce cas d'utilisation.  Cette classe utilisera la classe `AnnonceSqlAccess` pour enregistrer les  informations dans la base de données. Il faut donc d'abord ajouter une méthode `createAnnonce`  dans `AnnonceSqlAccess` pour faire l'insertion dans la base de données. Cette fonction prend en paramètre l'identifiant de l'utilisateur et  un tableau associatif avec toutes les informations à enregistrer.

```php
public function createAnnonce($login, $info)
    {
        $query = 'INSERT INTO Post(date, title, body, login, location, contactMail, contractType)
            VALUES("' . date('Y-m-d H:i:s') . '","'
            . $info['title'] . '","'
            . $info['body'] . '","'
            . $login . '","'
            . $info['location'] . '","'
            . $info['contactMail'] . '","'
            . $info['contractType'] . '")';

        try {
            $this->dataAccess->query($query);
        }
        catch ( \PDOException $e){
            return false;
        }
        return true;
    }
```

### 3.4 - Créer le cas d'utilisation "créer une annonce"

Une fois cette fonction ajoutée, nous pouvons l'utiliser dans la classe `AnnonceCreation` correspondant au cas d'utilisation, et ajouter une nouvelle méthode `annonceCreationAction` dans `Controllers`.

<img src="td2-img/3-annonceCreation.png" width="800px"/>

```php
    public function annonceCreationAction($id, $info, $data, $annonceCreation)
    {
        $annonceCreation->createAnnonce($id, $info, $data);
    }
```

Nous ajoutons ensuite dans `index.php` l'appel à ce cas d'utilisation. Il est effectué lorsque la route `/annonces/index.php/annonces` est appelée (car il s'agit de la destination du formulaire de création). Il faut tout de même détecter si l'appel à cette route a été effectué par le formulaire de création (et non par le menu ou suite à la connexion). Pour cela, nous pouvons utiliser `isset($_GET['contractType'])` car cette variable est seulement définie quand le formulaire de création a été envoyé.

<img src="td2-img/3-index.png" width="800px"/>

### 3.5 - Ajouter au formulaire la détection de la ville  grâce à une API

A cette étape, notre formulaire d'ajout d'une annonce est opérationnel. Il reste toutefois un point à traiter : la détection  automatique de la localisation lorsque l'utilisateur appuie sur le bouton "ici". Pour cela, nous allons devoir utiliser du code JavaScript.

Nous allons créer un fichier `viewCreateAnnonceForm.js`dans le répertoire `/gui/js/`. Ce fichier contiendra le code Javascript qui réagira au clic sur le bouton. Il faut ensuite inclure une référence à ce fichier à la fin du code HTML généré par `ViewCreateAnnonce`.

<img src="td2-img/3-viewCreateAnnonce.png" width="800px"/>

Le code de `viewCreateAnnonceForm.js` est relativement simple. Lorsque l'utilisateur clique sur le bouton, il récupère la latitude et la longitude, puis interroge à une API REST pour récupérer la ville associée. Cette API est l'[API Adresse (Base Adresse Nationale - BAN)](https://api.gouv.fr/les-api/base-adresse-nationale) produite par la DIRNUM du gouvernement. Elle permet d'interroger la base adresse nationale, qui référence l’intégralité des adresses du territoire et vise à les rendre utilisables par tous. Cette API libre a été conçue pour faire de l'autocomplétion et de vérification d'adresse, pour géolocaliser une adresse sur une carte, ou faire une recherche géographique inversée (c'est cette dernière ressource que nous utiliserons).

Comme le montre le code ci-dessous, nous utilisons le système de géolocalisation du navigateur pour récupérer les coordonnées GPS de l'utilisateur. Cela n'est possible que si vous **autorisez votre navigateur à accéder à votre emplacement**.

Pour interroger cette API REST, nous utilisons l'[API `fetch` de JavaScript](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). Elle permet d'interroger des ressources de manière asynchrone à travers le réseau. L'appel à `fetch` retourne un [objet `Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response), qui est une représentation complète de la réponse HTTP. Dans notre code, cette réponse est transformée en [objet JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON) par la méthode [`json()` de `Reponse`](https://developer.mozilla.org/en-US/docs/Web/API/Response/json).


```javascript
// récupération du boutton "localiser"
const localizeBtton = document.getElementById('localizeBtn');

localizeBtton.onclick = (e) => {

    // récupération du champ de saisie de localisation
    locationField = document.getElementById('location');

    navigator.geolocation.getCurrentPosition(async (position) => {
        // récupération de la latitude et de la longitude
        let lat = position.coords.latitude.toFixed(6);
        let long = position.coords.longitude.toFixed(6);

        // envoie d'une requête GET à l'API adresse du gouvernement
        let url = 'https://api-adresse.data.gouv.fr/reverse/?lon='+long+'&lat='+lat+'&type=locality';
        let response = await fetch(url);

        // recupération du résultat de la requête (un objet JSON)
        let result = await response.json();

        // ajout du nom de la ville dans le champ de saisie
        locationField.value = result.features[0].properties.city;
    });
};
```


