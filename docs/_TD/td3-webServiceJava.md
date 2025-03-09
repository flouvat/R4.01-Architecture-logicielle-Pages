---
layout: post
title: "TD 3 -- Conception de services Web REST en Java"
categories: jekyll update

mathjax: true
---

# TD 3 -- Conception de services web REST en Java

Dans ce TD, nous allons créer un service web (une API REST) permettant de réserver les livres disponibles dans une bibliothèque. On supposera que la gestion des emprunts est faite par une autre application non couverte par ce TD. Nous allons faire cela en Java avec [Jakarta EE](https://jakarta.ee/), une spécification orientée applications d'entreprises en Java. L'IDE utilisé sera [IntelliJ IDEA](https://www.jetbrains.com/idea/), l'environnement de développement intégré de la compagnie JetBrains.  Cette entreprise propose un [programme "éducation"](https://www.jetbrains.com/help/idea/product-educational-tools.html), qui permet aux étudiants d'utiliser gratuitement tous les outils qu'ils développent. 
Nous utiliserons aussi un serveur d'applications [GlassFish](https://projects.eclipse.org/projects/ee4j.glassfish) pour tester notre code en local.

Afin de montrer les possibilités de cette architecture orientée services, nous allons développer indépendamment 3 petites applications ("micro-services") sous forme d'API REST. 

La première appelée `book` aura pour objectif de publier  les livres disponibles dans notre bibliothèque. Elle permettra de consulter la liste des livres de la bibliothèque, ainsi que leurs informations (référence, titre, auteurs et disponibilité). Elle permettra aussi de modifier les informations d'un livre. 

La deuxième application appelée `user` publiera les informations sur les utilisateurs enregistrés. Elle implémentera aussi le mécanisme d'authentification.

La troisième application appelée `reservation` gèrera les réservations de livres par les utilisateurs. Elle permettra de faire des réservations ou de les supprimer. Pour cela, elle consommera les données publiées par les deux autres applications.



## Partie 1 : Création de l'application permettant d'accéder aux livres


### 1.1 Création du projet

Pour cette étape, nous allons simplement suivre le [tutoriel "Your first RESTful web service"](https://www.jetbrains.com/help/idea/creating-and-running-your-first-restful-web-service.html) proposé sur le site de IntelliJ. 

**Attention**, il faut télécharger le [serveur GlassFish (**Web Profile**)](https://glassfish.org/download) avant de commencer ce tutoriel (prérequis). Une fois l'archive téléchargée, il suffit de la décompresser pour pouvoir utiliser le serveur d'applications.

Dans la fenêtre de création du projet, on adapte aussi le nom du projet, et le groupe associé, à nos besoins. Notre projet s'appelle `book` et il est associé au groupe `fr.univ-amu.iut`. IntelliJ vous affiche potentiellement des alertes en bas de la fenêtre (en rouge). Il faut les "fixer" comme indiqué dans le tutoriel. 

<img src="td3-img/1-newProject.png" width="600px"/>

Dans la liste des dépendances qui suit, on laisse les dépendances par défaut (i.e. `CDI`, `JAX-RS` et `Servlet`) et on ajoute les dépendances à `JSON-B`, `JSON-P` et `JPA`.  Puis, on créé le projet. 

A cette étape, il faut encore configurer le serveur d'applications, et définir une configuration d'exécution,  **comme indiqué dans le tutoriel**. Pour configurer le serveur d'applications, il faut aller dans la configuration du projet, i.e. `Settings` (File > Settings...). Il faut sélectionner `Build, Execution, Deployment | Application Servers`, cliquer sur `+` et choisir  `Glassfish Server`. Il faut ensuite  indiquer le répertoire où vous avez décompressé GlassFish.

<img src="td3-img/1-configServer.png" width="500px"/>

Pour définir la configuration d'exécution, il faut sélectionner `Run | Edit Configurations`, cliquer sur `+`, étendre le noeud `Glassfish Server ` et sélectionner `Local`. Dans la section `Before launch` en bas de la fenêtre, nous allons juste ajouter (+) un `Build Artifacts` > `book:war`. Cette option permet de générer à chaque construction un fichier archive WAR (Web application ARchive).  Ce fichier WAR est l'équivalent d'un fichier JAR pour les applications web. Il regroupe tous les fichiers de l'application  dans un seul et même fichier. 

<img src="td3-img/1-configSettings.png" width="400px"/>

Par défaut lors du déploiement, IntelliJ transfert l'application sous forme de répertoire "war exploded" au serveur d'applications. Comme indiqué dans la [documentation d'IntelliJ](https://www.jetbrains.com/help/idea/configure-web-app-deployment.html#:~:text=A%20Web%20application%20can%20be,contains%20all%20the%20required%20files.), cela consiste à simplement transférer le contenu du fichier WAR  de manière décompressée, et non dans l'archive   construite par Java, ce qui permet de gagner en rapidité d'exécution.

Il faut aussi noter l'URL utilisée pour tester le service : `http://localhost:8080/book-1.0-SNAPSHOT/api/hello-world`. On remarque que le nom de l'application `book` est suivi par `-1.0-SNAPSHOT`. Cette extension est rajoutée par [Maven](https://maven.apache.org/), l'outil de gestion de projets Java utilisé ici. Dans les conventions de nommage de Maven, cela indique que la version actuelle de l'application est une pré-version de la version 1 de l'application (i.e. une version en cours de développement). A la différence d'une "release", le code d'un "SNAPSHOT" peut être mis à jours, et donc Maven recherche ces mises à jours et les intègrent lors de la construction et du déploiement de l'application (cf. la[ documentation de Maven](https://maven.apache.org/guides/getting-started/index.html#What_is_a_SNAPSHOT_version)  pour plus d'infromations). 


Une fois la configuration finalisée, vous pouvez  exécuter ce service ("run") et voir son résultat suite à une requête HTTP GET. Il s'agit d'un simple texte "Hello, World!".

<img src="td3-img/1-helloWorld.png" width="400px"/>

 Veuillez vous référer au tutoriel d'IntelliJ pour avoir une description plus détaillée des principaux fichiers du projet. 

<img src="td3-img/1-filesProject.png" width="800px"/>


Pour information, vous pouvez directement accéder à l'interface d'administration de GlassFish par l'URL [http://localhost:4848](http://localhost:4848).

<img src="td3-img/1-adminGlassFish.png" width="800px"/>


### 1.2 Création de l'entité métier 

Nous allons profiter des fichiers générés par IntelliJ pour développer notre petite application/service web permettant de manipuler les livres de notre bibliothèque. 

Nous appliquerons les principes d'architecture propre vus en cours. Le service fourni se voulant très basique, nous allons simplifier certaines parties, notamment en limitant certains "intermédiaires" (p.ex. le "presenter" ou les structures d'échanges de données entre les couches). Nous implémenterons l'architecture décrite dans le diagramme de classes ci-dessous. Au niveau de la couche "entités métier", nous retrouvons l'entité métier `Book` représentant un livre. Au niveau de la couche "cas d'utilisation", nous avons la classe `BookService` représentant les traitements/services fournis par l'application sur les livres. Cette classe `BookService` utilise l'interface `BookRepositoryInterface` pour manipuler les données stockées sur les livres (toujours dans cette couche). Dans la couche "IHM et BD", nous avons la classe `BookRepositoryMariadb`, implémentant l'interface `BookRepositoryInterface`. Elle permet d'accéder aux données stockées dans Mariadb. La classe `BookRessource` défini la ressource "livre" publiée par cette API REST. Cette classe contiendra la description des différents points d'accès à l'API ("endpoints") et les modalités d'interaction. Il s'agit du point d'entrée/sortie du client de cette API. Pour finir, la classe `BookApplication` est le "main" de l'application. Elle décrit la création des principaux objets et l'injection des dépendances (i.e. le passage en paramètres des objets à utiliser dans les différentes classes). On remarque que le sens des dépendances dans cette architecture est bien orienté vers le domaine, en laissant les entrées/sorties à la "périphérie" de l'application.

<img src="td3-img/1-classDiagram.jpg" width="600px"/>


Dans la trame de projet créé par IntelliJ, nous commençons par renommer la classe `HelloApplication` en `BookApplication` (Refactor > Rename File ...). 

Nous nous focalisons ensuite sur les entités métier, i.e. la classe `Book` représentant les informations sur un livre. Il faut donc créer cette  classe Java `Book` en faisant un clic droit sur le répertoire `src/main/java/fr.univamu.iut.book`, puis `New > Java Class`. On suppose dans ce TD qu'un livre est caractérisé par les informations suivantes : la référence du livre, son titre, la liste de ses auteurs et son statut (réservé, emprunté ou disponible). Nous allons donc ajouter ses informations sous forme d'attribut de la classe `Book`. Tous ces attributs sont des `String` à l'exception du statut qui est un `char` (avec pour valeur 'r' pour réservé, 'e' pour emprunté, et 'd' pour disponible). Nous ajoutons aussi un constructeur permettant de créer un objet `Book` en initialisant ces différents attributs à partir d'informations passées en paramètres. Seul le statut est initialisé par défaut à 'd' (disponible) à la création de l'objet. Nous ajoutons aussi à cette classe des méthodes "getters" et "setters" afin de pouvoir accéder et modifier ces attributs.

```java
package fr.univamu.iut.book;

/**
 * Classe représentant un livre
 */
public class Book {

    /**
     * Référence du livre
     */
    protected String reference;

    /**
     * titre du livre
     */
    protected String title;

    /**
     * Auteurs du livre
     */
    protected String authors;

    /**
     * Statut du livre
     * ('r' pour réservé, 'e' pour emprunté, et 'd' pour disponible)
     */
    protected char status;

    /**
     * Constructeur par défaut
     */
    public Book(){
    }

    /**
     * Constructeur de livre
     * @param reference référence du livre
     * @param title titre du livre
     * @param authors auteurs du livre
     */
    public Book(String reference, String title, String authors){
        this.reference = reference;
        this.title = title;
        this.authors = authors;
        this.status = 'd';
    }

    /**
     * Méthode permettant d'accéder à la réference du livre
     * @return un chaîne de caractères avec la référence du livre
     */
    public String getReference() {
        return reference;
    }

    /**
     * Méthode permettant d'accéder au titre du livre
     * @return un chaîne de caractères avec le titre du livre
     */
    public String getTitle() {
        return title;
    }

    /**
     * Méthode permettant d'accéder aux auteurs du livre
     * @return un chaîne de caractères avec la liste des auteurs
     */
    public String getAuthors() {
        return authors;
    }

    /**
     * Méthode permettant d'accéder au statut du livre
     * @return un caractère indiquant lestatu du livre ('r' pour réservé, 'e' pour emprunté, et 'd' pour disponible)
     */
    public char getStatus() {
        return status;
    }

    /**
     * Méthode permettant de modifier la référence du livre
     * @param reference une chaîne de caractères avec la référence à utiliser
     */
    public void setReference(String reference) {
        this.reference = reference;
    }

    /**
     * Méthode permettant de modifier le titre du livre
     * @param title une chaîne de caractères avec le titre à utiliser
     */
    public void setTitle(String title) {
        this.title = title;
    }

    /**
     * Méthode permettant de modifier les autheurs du livre
     * @param authors une chaîne de caractères avec la liste des auteurs
     */
    public void setAuthors(String authors) {
        this.authors = authors;
    }

    /**
     * Méthode permettant de modifier le statut du livre
     * @param status le caractère 'r' pour réservé, 'e' pour emprunté, ou 'd' pour disponible
     */
    public void setStatus(char status) {
        this.status = status;
    }

    @Override
    public String toString() {
        return "Livre{" +
                "reference='" + reference + '\'' +
                ", titre='" + title + '\'' +
                ", auteurs='" + authors + '\'' +
                ", statut=" + status +
                '}';
    }
}
```


### 1.3 Création de l'interface d'accès aux données

Nous supposons dans la suite que les données sur les livres sont stockées dans une base de données Mariadb accessible par le web. Nous allons créer cette base de données sur Alwaysdata en suivant la même procédure que pour le TD1. Une fois connecté, nous sélectionnons dans le menu "Bases de données" le lien "MySQL". Puis, nous créons un nouvel utilisateur de base de données pour notre application. Cet utilisateur sera nommé `[compte]_library`. Puis, nous  sélectionnons l'onglet "Base de données" pour créer notre base de données `[compte]_library_db`. Dans la section "Permissions" de l'interface de création, nous attribuerons tous les droits à notre utilisateur `[compte]_library`. Une fois la base de données créée, nous allons dans l'interface de phpMyAdmin pour créer la table `Book` et y insérer quelques tuples. Cette table aura les champs `reference`, `title`, `authors`, et `status`.


<img src="td3-img/1-createDB.png" width="800"/>

La base de données créée, nous ajoutons dans notre projet IntelliJ une interface `BookRepositoryInterface` pour représenter les méthodes attendues pour accéder et manipuler les données. Cette interface permet d'éviter que le cas d'utilisation dépende du type d'accès aux données (Mariadb dans notre cas). Elle délimite donc une frontière architecturale.

```java
package fr.univamu.iut.book;

import java.util.*;

/**
 * Interface d'accès aux données des livres
 */
public interface BookRepositoryInterface {

    /**
     *  Méthode fermant le dépôt où sont stockées les informations sur les livres
     */
    public void close();

    /**
     * Méthode retournant le livre dont la référence est passée en paramètre
     * @param reference identifiant du livre recherché
     * @return un objet Book représentant le livre recherché
     */
    public Book getBook( String reference );

    /**
     * Méthode retournant la liste des livres
     * @return une liste d'objets livres
     */
    public ArrayList<Book> getAllBooks() ;

    /**
     * Méthode permettant de mettre à jours un livre enregistré
     * @param reference identifiant du livre à mettre à jours
     * @param title nouveau titre du livre
     * @param authors nouvelle liste d'auteurs
     * @param status nouveau status du livre
     * @return true si le livre existe et la mise à jours a été faite, false sinon
     */
    public boolean updateBook( String reference, String title, String authors, char status);
}

```

Nous créons maintenant une classe `BookRepositoryMariadb` implémentant l'interface précédente `BookRepositoryInterface`. Cette classe implémente les méthodes d'accès et de modification de livre spécifiques à ce stockage dans Mariadb. L'architecture mise en place permettra néanmoins de changer facilement de type de stockage sans avoir à modifier le code métier (grâce à l'interface créée juste avant). Cette classe `BookRepositoryMariadb` implémente aussi l'interface `Closeable` car la connexion à la base de données doit être libérée lorsque l'application termine. Cette classe s'appuie sur des drivers [JDBC](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html) pour interroger la base de données.

```java
package fr.univamu.iut.book;

import java.io.Closeable;
import java.sql.*;
import java.util.ArrayList;

/**
 * Classe permettant d'accèder aux livres stockés dans une base de données Mariadb
 */
public class BookRepositoryMariadb   implements BookRepositoryInterface, Closeable {

    /**
     * Accès à la base de données (session)
     */
    protected Connection dbConnection ;

    /**
     * Constructeur de la classe
     * @param infoConnection chaîne de caractères avec les informations de connexion
     *                       (p.ex. jdbc:mariadb://mysql-[compte].alwaysdata.net/[compte]_library_db
     * @param user chaîne de caractères contenant l'identifiant de connexion à la base de données
     * @param pwd chaîne de caractères contenant le mot de passe à utiliser
     */
    public BookRepositoryMariadb(String infoConnection, String user, String pwd ) throws java.sql.SQLException, java.lang.ClassNotFoundException {
        Class.forName("org.mariadb.jdbc.Driver");
        dbConnection = DriverManager.getConnection( infoConnection, user, pwd ) ;
    }

    @Override
    public void close() {
        try{
            dbConnection.close();
        }
        catch(SQLException e){
            System.err.println(e.getMessage());
        }
    }

    @Override
    public Book getBook(String reference) {

        Book selectedBook = null;

        String query = "SELECT * FROM Book WHERE reference=?";

        // construction et exécution d'une requête préparée
        try ( PreparedStatement ps = dbConnection.prepareStatement(query) ){
            ps.setString(1, reference);

            // exécution de la requête
            ResultSet result = ps.executeQuery();

            // récupération du premier (et seul) tuple résultat
            // (si la référence du livre est valide)
            if( result.next() )
            {
                String title = result.getString("title");
                String authors = result.getString("authors");
                char status = result.getString("status").charAt(0);

                // création et initialisation de l'objet Book
                selectedBook = new Book(reference, title, authors);
                selectedBook.setStatus(status);
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        return selectedBook;
    }

    @Override
    public ArrayList<Book> getAllBooks() {
        ArrayList<Book> listBooks ;

        String query = "SELECT * FROM Book";

        // construction et exécution d'une requête préparée
        try ( PreparedStatement ps = dbConnection.prepareStatement(query) ){
            // exécution de la requête
            ResultSet result = ps.executeQuery();

            listBooks = new ArrayList<>();

            // récupération du premier (et seul) tuple résultat
            while ( result.next() )
            {
                String reference = result.getString("reference");
                String title = result.getString("title");
                String authors = result.getString("authors");
                char status = result.getString("status").charAt(0);

                // création du livre courant
                Book currentBook = new Book(reference, title, authors);
                currentBook.setStatus(status);

                listBooks.add(currentBook);
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        return listBooks;
    }

    @Override
    public boolean updateBook(String reference, String title, String authors, char status) {
        String query = "UPDATE Book SET title=?, authors=?, status=?  where reference=?";
        int nbRowModified = 0;

        // construction et exécution d'une requête préparée
        try ( PreparedStatement ps = dbConnection.prepareStatement(query) ){
            ps.setString(1, title);
            ps.setString(2, authors);
            ps.setString(3, String.valueOf(status) );
            ps.setString(4, reference);

            // exécution de la requête
            nbRowModified = ps.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        return ( nbRowModified != 0 );
    }
}
```

Pour pouvoir utiliser ce code, il faut ajouter dans la section `dependencies` du fichier `pom.xml` (fichier Maven décrivant le projet) les dépendances vers les drivers JDBC pour Mariadb.

```xml
        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>LATEST</version>
        </dependency>
```

Il faut ensuite charger ces modifications dans Maven en cliquant sur le "pop-up" `Load Maven changes`.

<img src="td3-img/1-loadMavenChanges.png" width="900"/>

Puis, il faut reconstruire le projet (Build > Rebuild Project). Vous pouvez maintenant tester que le code précédent fonctionne bien.  



### 1.4 Création de la classe "service" (cas d'utilisation) 

Une fois ce code testé, nous allons créer la classe `BookService`. Pour rappel, cette classe vise à interroger le "dépôt de données" (i.e. la base de données Mariadb). Lorsque nécessaire, elle retourne l'information au format JSON pour pouvoir être directement publiée par l'API.  Pour faire cela, cette classe s'appuie sur l'[API JSON-B (JSON Binding) de Jakarta](https://javaee.github.io/jsonb-spec/).

```java
package fr.univamu.iut.book;

import jakarta.json.bind.Jsonb;
import jakarta.json.bind.JsonbBuilder;
import java.util.ArrayList;


/**
 * Classe utilisée pour récupérer les informations nécessaires à la ressource
 * (permet de dissocier ressource et mode d'éccès aux données)
 */
public class BookService {

    /**
     * Objet permettant d'accéder au dépôt où sont stockées les informations sur les livres
     */
    protected BookRepositoryInterface bookRepo ;

    /**
     * Constructeur permettant d'injecter l'accès aux données
     * @param bookRepo objet implémentant l'interface d'accès aux données
     */
    public  BookService( BookRepositoryInterface bookRepo) {
        this.bookRepo = bookRepo;
    }

    /**
     * Méthode retournant les informations sur les livres au format JSON
     * @return une chaîne de caractère contenant les informations au format JSON
     */
    public String getAllBooksJSON(){

        ArrayList<Book> allBooks = bookRepo.getAllBooks();

        // création du json et conversion de la liste de livres
        String result = null;
        try( Jsonb jsonb = JsonbBuilder.create()){
            result = jsonb.toJson(allBooks);
        }
        catch (Exception e){
            System.err.println( e.getMessage() );
        }

        return result;
    }

    /**
     * Méthode retournant au format JSON les informations sur un livre recherché
     * @param reference la référence du livre recherché
     * @return une chaîne de caractère contenant les informations au format JSON
     */
    public String getBookJSON( String reference ){
        String result = null;
        Book myBook = bookRepo.getBook(reference);

        // si le livre a été trouvé
        if( myBook != null ) {

            // création du json et conversion du livre
            try (Jsonb jsonb = JsonbBuilder.create()) {
                result = jsonb.toJson(myBook);
            } catch (Exception e) {
                System.err.println(e.getMessage());
            }
        }
        return result;
    }

    /**
     * Méthode permettant de mettre à jours les informations d'un livre
     * @param reference référence du livre à mettre à jours
     * @param book les nouvelles infromations a été utiliser
     * @return true si le livre a pu être mis à jours
     */
    public boolean updateBook(String reference, Book book) {
        return bookRepo.updateBook(reference, book.title, book.authors, book.status);
    }
}
```


### 1.5 Création de la classe "ressource" (API REST)

Cette classe `BookService` sera instanciée dans la classe `BookApplication` et injectée dans la classe `BookResource`, point d'accès de l'API publiée. Avant de compléter `BookApplication`, nous allons donc créer la classe `BookResource`. Elle définit 3 "endpoints" : `/api/books` en GET, `/api/books/{reference}` en GET, et `/api/books/{reference}` en PUT. Le premier permet de récupérer au format JSON la liste des livres de la bibliothèque. Le deuxième permet de récupérer au format JSON  les informations d'un livre dont la référence est passée en paramètre. Le troisième permet de mettre à jours à partir d'un JSON les informations d'un livre dont la référence est passée en paramètre.


```java
package fr.univamu.iut.book;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.Response;


/**
 * Ressource associée aux livres
 * (point d'accès de l'API REST)
 */
@Path("/books")
public class BookResource {

    /**
     * Service utilisé pour accéder aux données des livres et récupérer/modifier leurs informations
     */
    private BookService service;

    /**
     * Constructeur par défaut
     */
     public BookResource(){}

    /**
     * Constructeur permettant d'initialiser le service avec une interface d'accès aux données
     * @param bookRepo objet implémentant l'interface d'accès aux données
     */
    public BookResource( BookRepositoryInterface bookRepo ){
        this.service = new BookService( bookRepo) ;
    }

    /**
     * Constructeur permettant d'initialiser le service d'accès aux livres
     */
    public BookResource( BookService service ){
        this.service = service;
    }

    /**
     * Enpoint permettant de publier de tous les livres enregistrés
     * @return la liste des livres (avec leurs informations) au format JSON
     */
    @GET
    @Produces("application/json")
    public String getAllBooks() {
        return service.getAllBooksJSON();
    }

    /**
     * Endpoint permettant de publier les informations d'un livre dont la référence est passée paramètre dans le chemin
     * @param reference référence du livre recherché
     * @return les informations du livre recherché au format JSON
     */
    @GET
    @Path("{reference}")
    @Produces("application/json")
    public String getBook( @PathParam("reference") String reference){

        String result = service.getBookJSON(reference);

        // si le livre n'a pas été trouvé
        if( result == null )
            throw new NotFoundException();

        return result;
    }

    /**
     * Endpoint permettant de mettre à jours le statut d'un livre uniquement
     * (la requête patch doit fournir le nouveau statut sur livre, les autres informations sont ignorées)
     * @param reference la référence du livre dont il faut changer le statut
     * @param book le livre transmis en HTTP au format JSON et convertit en objet Book
     * @return une réponse "updated" si la mise à jour a été effectuée, une erreur NotFound sinon
     */
    @PUT
    @Path("{reference}")
    @Consumes("application/json")
    public Response updateBook(@PathParam("reference") String reference, Book book ){

        // si le livre n'a pas été trouvé
        if( ! service.updateBook(reference, book) )
            throw new NotFoundException();
        else
            return Response.ok("updated").build();
    }
}
```

### 1.6 Modification de la classe initialisant l'application afin de gérer la connexion à la base de données

Dans `BookApplication`, nous pouvons maintenant créer un objet `BookService` et lui injecter un objet `BookRepositoryMariadb`   permettant l'accès aux données. Plusieurs options sont possibles pour faire cela.

Une première solution est d'ajouter dans la classe `BookApplication` une méthode  `getSingletons()`. Elle permet de créer les différents objets et de définir la classe ressource qui sera publiée. 

```java
    @Override
    public Set<Object> getSingletons() {
        Set<Object> set = new HashSet<>();

        // Création de la connection à la base de données et initialisation du service associé
        BookService service = null ;
        try{
            BookRepositoryMariadb db = new BookRepositoryMariadb("jdbc:mariadb://mysql-[compte].alwaysdata.net/[compte]_library_db", "[compte]_library", "mdp");
            service = new BookService(db);
        }
        catch (Exception e){
            System.err.println(e.getMessage());
        }

        // Création de la ressource en lui passant paramètre les services à exécuter en fonction
        // des différents endpoints proposés (i.e. requêtes HTTP acceptées)
        set.add(new BookResource(service));
        
        return set;
    }

```

Cette solution est fonctionnelle dans notre cas, mais elle peut poser des problèmes dans d'autres. En effet, avec cette approche, la ressource n'est créée qu'une seule fois pour toute la durée de vie de l'application, vu qu'il s'agit d'un singleton. L'API pouvant être interrogée par de multiples utilisateurs en même temps, cela peut poser des problèmes d'accès concurrents. Dans notre cas, les accès concurrents aux données sont gérés par le SGBD. C'est pour cela que cette solution ne pose pas de problème.

Une autre solution consiste à laisser le comportement par défaut en terme de création des ressources, i.e. un objet est créé à chaque nouvelle requête HTTP. Il se pose alors la question de l'injection de l'objet `BookService` dans la classe `BookRessource`. Pour cela, une solution consiste à utiliser l'[API CDI (Contexts and Dependency Injection) de Jakarta](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html). Elle a pour but de simplifier le contrôle de la vie des composants, notamment via l'injection de dépendances. Son principe est d'utiliser des annotations pour indiquer où et quand injecter les objets. Pour plus d'informations, vous pouvez par exemple vous reporter à l'[article dédié à ce sujet sur InfoQ](https://www.infoq.com/fr/articles/injection-dependance-jakarta-cdi/).


```java
package fr.univamu.iut.book;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Disposes;
import jakarta.enterprise.inject.Produces;
import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;


@ApplicationPath("/api")
@ApplicationScoped
public class BookApplication extends Application {

    /**
     * Méthode appelée par l'API CDI pour injecter la connection à la base de données au moment de la création
     * de la ressource
     * @return un objet implémentant l'interface BookRepositoryInterface utilisée
     *          pour accéder aux données des livres, voire les modifier
     */
    @Produces
    private BookRepositoryInterface openDbConnection(){
        BookRepositoryMariadb db = null;

        try{
            db = new BookRepositoryMariadb("jdbc:mariadb://mysql-[compte].alwaysdata.net/[compte]_library_db", "[compte]_library", "mdp");
        }
        catch (Exception e){
            System.err.println(e.getMessage());
        }
        return db;
    }

    /**
     * Méthode permettant de fermer la connexion à la base de données lorsque l'application est arrêtée
     * @param bookRepo la connexion à la base de données instanciée dans la méthode @openDbConnection
     */
    private void closeDbConnection(@Disposes BookRepositoryInterface bookRepo ) {
        bookRepo.close();
    }
}
```

La méthode `openDbConnection()` est celle qui sera utilisée pour instancier l'objet `BookRepositoryInterface` utilisé dans `BookService` (patron de conception "factory"). Ce code est associé aux annotations `@Produces` et `@ApplicationScoped`. Elles indiquent que l'objet `BookService` sera produit une seule fois pour toute la durée de l'application. Autrement dit, il y aura une seule connexion établit à la base de données par l'application, mais bien un objet `BookService` différent créé à chaque requête (contrairement à la première solution). Nous allons implémenter cette solution.



```java
package fr.univamu.iut.book;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.*;

/**
 * Ressource associée aux livres
 * (point d'accès de l'API REST)
 */
@Path("/books")
@ApplicationScoped
public class BookResource {

    /**
     * Service utilisé pour accéder aux données des livres et récupérer/modifier leurs informations
     */
    private BookService service;

    /**
     * Constructeur par défaut
     */
     public BookResource(){}

    /**
     * Constructeur permettant d'initialiser le service avec une interface d'accès aux données
     * @param bookRepo objet implémentant l'interface d'accès aux données
     */
    public @Inject BookResource( BookRepositoryInterface bookRepo ){
        this.service = new BookService( bookRepo) ;
    }
//...
```

Notre service est maintenant opérationnel. Vous pouvez le tester complètement. Pour tester vos requêtes GET, vous avez deux options : 
- utiliser une commande `cURL` dans le terminal tel que `curl http://localhost:8080/book-1.0-SNAPSHOT/api/books` ou `curl http://localhost:8080/book-1.0-SNAPSHOT/api/books/ABC` 
- saisir ces URL dans votre navigateur web comme dans les captures d'écran ci-dessous.


<img src="td3-img/1-testGetBooks.png" width="600"/>

<img src="td3-img/1-testGetBook.png" width="600"/>

<img src="td3-img/1-testGetBookNotFound.png" width="600"/>

N.B. : La configuration d'exécution d'IntelliJ est actuellement paramétrée pour ouvrir  le navigateur avec l'URL `http://localhost:8080/book-1.0-SNAPSHOT/api/hello-world`. Vous pouvez changer cela en éditant cette configuration (`Run/Debug Configuration`) et en changeant l'URL ouverte par le navigateur.

<img src="td3-img/1-runConfig.png" width="600"/>

Pour tester les requêtes PUT (mise à jours), il est possible d'utiliser cURL dans le terminal avec la ligne de commande (Windows) ci-dessous, puis de faire une requête pour vérifier si la modification a bien été faite. 

`curl -X PUT http://localhost:8080/book-1.0-SNAPSHOT/api/books/ABC -H "Content-Type: application/json" -d "{\"authors\": \"Robert C. Martin\",\"reference\": \"ABC\",\"status\": \"d\",\"title\": \"Clean architecture\"}"`.

<img src="td3-img/1-testGetBookUpdate.png" width="600"/>




## Partie 2 : Création de l'application de gestion des utilisateurs

Dans cette partie, nous allons développer une autre application web sous forme d'API REST. Cette application sera en charge de gérer les utilisateurs de la bibliothèque. Elle permettra d'accéder à la liste des utilisateurs et d'authentifier un utilisateur à partir de son identifiant et de son mot de passe. Comme précédemment, elle s'appuiera sur la base de données `[compte]_library_db` (sur Alwaysdata) pour stocker les informations sur les utilisateurs.

Le diagramme de classes de l'application `user` est le suivant:

<img src="td3-img/2-classDiagram.jpg" width="800px"/>


## 2.1 - Accéder aux informations des utilisateurs

Pour faire cela, vous suivrez les mêmes étapes que dans la partie 1 c'est-à-dire :

1. créer le projet `user`;
2. créer l'entité métier `User`;
3. créer la table `User(id,name,pwd,mail)` dans la base de données `[compte]_library_db` (Alwaysdata), puis créer  l'interface d'accès aux données  `UserRepositoryInterface` et la classe `UserRepositoryMariadb`  permettant d'interroger Mariadb. Ces classes implémenteront pour cela  les méthodes `User getUser(String id)` et `ArrayList<User> getAllUSers()`;
4. créer la classe service `UserService` (cas d'utilisation) implémentant les méthodes `String getAllUsersJSON()` et `String getUserJSON(String id)` ;
5. créer la classe ressource `UserResource` publiant deux points d'accès ("endpoints"): `api/users/` accessible en GET pour récupérer la liste de tous les utilisateurs (sans les mails et mots de passe) et `api/users/{id}` accessible en GET pour récupérer les informations d'un utilisateur;  
6. compléter la classe `UserApplication` pour ouvrir la connexion et fermer à la base de données à la fin de l'application avec l'API CDI de Jakarata.

Le code des méthodes `String getAllUsersJSON()` et `String getUserJSON(String id)` est donné ci-dessous.

```java
     /**
     * Méthode retournant les informations (sans mail et mot de passe) sur les utilisateurs au format JSON
     * @return une chaîne de caractère contenant les informations au format JSON
     */
    public String getAllUsersJSON(){

        ArrayList<User> allUsers = userRepo.getAllUsers();

        // on supprime les informations sur les mots de passe et les mails
        for( User currentUser : allUsers ){
            currentUser.setMail("");
            currentUser.setPwd("");
        }

        // création du json et conversion de la liste de livres
        String result = null;
        try( Jsonb jsonb = JsonbBuilder.create()){
            result = jsonb.toJson(allUsers);
        }
        catch (Exception e){
            System.err.println( e.getMessage() );
        }

        return result;
    }

    /**
     * Méthode retournant au format JSON les informations sur un utilisateur recherché
     * @param id l'identifiant de l'utilisateur recherché
     * @return une chaîne de caractère contenant les informations au format JSON
     */
    public String getUserJSON( String id ){
        String result = null;
        User myUser = userRepo.getUser(id);

        // si le livre a été trouvé
        if( myUser != null ) {

            // création du json et conversion du livre
            try (Jsonb jsonb = JsonbBuilder.create()) {
                result = jsonb.toJson(myUser);
            } catch (Exception e) {
                System.err.println(e.getMessage());
            }
        }
        return result;
    }
```

## 2.2 - Ajouter le cas d'utilisation "authentifier un utilisateur"

Une fois les précédentes classes implémentées et testées, nous ajoutons une classe `UserAuthenticationService` qui implémente le cas d'utilisation "authentifier un utilisateur". Elle intègre notamment  une méthode `boolean isValidUser(String id, String pwd)` permettant de vérifier un identifiant et un mot de passe utilisateur. 

```java
package fr.univamu.iut.user;

/**
 * Classe représentant le cas d'utilisation "authentifier un utilisateur"
 */
public class UserAuthenticationService {

    /**
     * Objet permettant d'accéder au dépôt où sont stockées les informations sur les utilisateurs
     */
    protected UserRepositoryInterface userRepo ;

    /**
     * Constructeur permettant d'injecter l'accès aux données
     * @param userRepo objet implémentant l'interface d'accès aux données
     */
    public UserAuthenticationService(UserRepositoryInterface userRepo) {
        this.userRepo = userRepo;
    }

    /**
     * Méthode d'authentifier un utilisateur
     * @param id identifiant de l'utilisateur
     * @param pwd mot de passe de l'utilisateur
     * @return true si l'utilisateur a été authentifié, false sinon
     */
    public boolean isValidUser( String id, String pwd){

        User currentUser = userRepo.getUser( id );

        // si l'utilisateur n'a pas été trouvé
        if( currentUser == null )
            return false;

        // si le mot de passe n'est pas correcte
        if( ! currentUser.pwd.equals(pwd) )
            return false;

        return true;
    }
}
```

Cette classe est utilisée dans un nouveau point d'accès `api/authenticate` implémenté par une nouvelle classe ressource `UserAuthenticationResource`. Ce point d'accès permettra de traiter une requête GET avec le protocole ["basic authentication"](https://www.ibm.com/docs/en/ibm-mq/9.1?topic=security-using-http-basic-authentication-rest-api). Dans ce protocole, l'identifiant et le mot de passe sont encodés (en [base 64](https://fr.wikipedia.org/wiki/Base64)) dans l'en-tête de la requête HTTP.

```java
package fr.univamu.iut.user;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.*;
import jakarta.ws.rs.container.ContainerRequestContext;
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.Response;

import java.io.UnsupportedEncodingException;
import java.util.Base64;

/**
 * Ressource associée à l'authentification des utilisateurs
 * (point d'accès de l'API REST)
 */
@Path("/authenticate")
@ApplicationScoped
public class UserAuthenticationResource {

    /**
     * Service utilisé pour accéder aux données des utilisateurs
     */
    private UserAuthenticationService auth;

    /**
     * Constructeur par défaut
     */
    public UserAuthenticationResource(){}

    /**
     * Constructeur permettant d'initialiser le service avec une interface d'accès aux données
     * @param userRepo objet implémentant l'interface d'accès aux données
     */
    public @Inject UserAuthenticationResource( UserRepositoryInterface userRepo ){
        this.auth = new UserAuthenticationService( userRepo ) ;
    }

    /**
     * Enpoint permettant de publier de tous les utilisateurs enregistrés
     * @return la liste des utilisateurs (avec leurs informations) au format JSON
     */
    @GET
    @Produces("text/plain")
    public Response authenticate(@Context ContainerRequestContext requestContext) throws UnsupportedEncodingException {
        boolean res = false;

        // Récupération du header de la requête HTTP et
        // vérification de la présence des informations pour une authentification "basic"
        String authHeader = requestContext.getHeaderString("Authorization");
        if (authHeader == null || !authHeader.startsWith("Basic")) {
            // envoie d'un code d'erreur avec dans l'en-tête le protocol à utiliser
            return Response.status(Response.Status.UNAUTHORIZED).header("WWW-Authenticate", "Basic").build();
        }

        // Récupération et transformation de l'identifiant et du mot de passe encodé en base 64
        String[] tokens = (new String(Base64.getDecoder().decode(authHeader.split(" ")[1]), "UTF-8")).split(":");
        final String id = tokens[0];
        final String pwd = tokens[1];

        // authentification
        res = auth.isValidUser(id, pwd);

        // envoie d'une réponse avec la valeur de l'authentification
        return Response.ok(String.valueOf(res)).build();
    }
}
```

Ce service d'authentification peut être testé à partir du terminal en utilisant des commandes cURL du type : `curl -u idBob:bobpwd -X GET http://localhost:8080/user-1.0-SNAPSHOT/api/authenticate`


## Partie 3 : Création de l'application permettant de gérer les réservations

Dans cette partie, nous allons développer une dernière application web sous forme d'API REST. Cette application sera en charge de gérer la réservation des livres de la bibliothèque (mais pas les emprunts). Elle permettra d'accéder à la liste des réservations, et d'en créer ou supprimer une. Pour réserver un livre, elle devra vérifier si le livre est disponible (ni réservé, ni emprunté) et que l'utilisateur est enregistré. Pour cela, elle s'appuiera sur les services créés précédemment. Elle stockera aussi les réservations en cours dans une base de données locale à l'application. Pour des raisons de simplicité, nous utiliserons notre base de données `[compte]_library_db` sur Alwaysdata. On supposera aussi que l'historique des réservations n'est pas conservé.

Le diagramme de classes de l'application `reservation` est le suivant:

<img src="td3-img/3-classDiagram.png" width="900px"/>


## 3.1 - Accéder aux informations des réservations

Vous suivrez les mêmes étapes que dans les parties précédentes c'est-à-dire :

1. créer le projet `reservation`;
2. créer l'entité métier `Reservation` composée d'un identifiant d'utilisateur, d'une référence de livre, et d'une date de réservation;
3. créer la table `Reservation(id, reference, date)` dans la base de données `[compte]_library_db` (Alwaysdata), puis créer  l'interface d'accès aux données  `ReservationRepositoryInterface` et la classe `ReservationRepositoryMariadb`  permettant d'interroger Mariadb. Ces classes implémenteront  les méthodes `ArrayList<Reservation> getAllReservations()` (récupération de toutes les réservations) et `Reservation getReservation(String reference)` (récupération d'une réservation);
4. créer la classe  `ReservationService` (cas d'utilisation) implémentant les méthodes `String getAllReservationsJSON()`, et `String getReservationJSON(String reference)`
5. créer la classe ressource `ReservationResource` publiant 2 points d'accès `api/reservations/` en GET et `api/reservations/{reference}` en GET, correspondant aux  "services" actuellement offerts par `ReservationService`. 
6. compléter la classe `ReservationApplication` pour gérer la connexion à la base de données pour les réservations (utiliser comme précédemment l'API CDI de Jakarata).


Le code de la classe `ReservationResource` est donné ci-dessous.

```java
package fr.univamu.iut.reservation;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.Response;


/**
 * Ressource associée aux réservations de livres
 * (point d'accès de l'API REST)
 */
@Path("/reservations")
@ApplicationScoped
public class ReservationResource {

    /**
     * Service utilisé pour accéder aux données de réservations et récupérer/modifier leurs informations
     */
    private ReservationService service;

    /**
     * Constructeur par défaut
     */
    public ReservationResource(){}

    /**
     * Constructeur permettant d'initialiser le service avec les interfaces d'accès aux données
     * @param reservationRepo objet implémentant l'interface d'accès aux données des réservations
     */
    public @Inject ReservationResource(ReservationRepositoryInterface reservationRepo){
        this.service = new ReservationService( reservationRepo ) ;
    }

    /**
     * Enpoint permettant de publier de toutes les réservations enregistrées
     * @return la liste des livres (avec leurs informations) au format JSON
     */
    @GET
    @Produces("application/json")
    public String getAllReservations() {
        return service.getAllReservationsJSON();
    }

    /**
     * Endpoint permettant de publier les informations d'une réservation
     * dont la référence est passée paramètre dans le chemin
     * @param reference référence du livre recherché
     * @return les informations de réservation pour le livre recherché au format JSON
     */
    @GET
    @Path("{reference}")
    @Produces("application/json")
    public String getReservation( @PathParam("reference") String reference){

        String result = service.getReservationJSON( reference );

        // si aucune réservation n'a été trouvée
        // on retourne simplement un JSON vide
        if( result == null )
           return "{}";

        return result;
    }
}

```

## 3.2 - Réserver un livre

Après avoir testé le code précédent, nous allons maintenant intégrer dans l'API un point d'accès permettant de réserver un livre. D'un point de vue règle métier, le livre doit être disponible pour pouvoir être réservé.  De plus, l'utilisateur doit être enregistré. Une fois ces deux conditions vérifiées, il est possible d'enregistrer la réservation. A ce moment, il faut aussi changer le statut du livre (le passer de "d" à "r").

L'application doit donc manipuler les informations sur les utilisateurs et les livres. Nous utiliserons les API REST `book` et `user` développées dans les  parties précédentes, et non les tables correspondantes dans la base de données. L'objectif est de découpler totalement les différentes applications, et de permettre un stockage des informations sur des serveurs (et des bases de données) différents.


### Déployer les services web `book` et `user` en parallèle

Pour faire cela, il faut déployer les deux applications `book` et `user`, en même temps que l'application `reservation`. Dans IntelliJ, nous modifions donc la configuration d'exécution du projet (`Run/Debug Configuration`) afin de permettre cela. Il faut sélectionner la configuration `GlassFish ` et l'éditer. Dans la fenêtre de configuration, il faut sélectionner l'onglet `Deployment`, puis ajouter (+ suivi de `Add`) des sources externes (`External source ...`). Vous allez dans les répertoires `/target` de vos projets IntelliJ `book` et `user`, et sélectionnez les fichiers WAR des deux applications. 

<img src="td3-img/3-editConfig.png" width="200px"/>

<img src="td3-img/3-deployment.png" width="600px"/>


### Intégrer les interfaces d'accès aux données des livres et des utilisateurs

Pour pouvoir consommer ces API REST dans notre application, nous allons reprendre dans ce projet les classes `Book` et `User`, ainsi que les interfaces  `BookRepositoryInterface` et `UserRepositoryInterface`. Ces classes sont des copies de celles développées dans les deux autres applications. 

Une fois ces classes copiées, nous ajoutons deux classes implémentant ces interfaces et décrivant comment interroger ("consommer") ces API. Nous appelons ces classes `BookRepositoryAPI` et `UserRepositoryAPI`. Leur code est donné ci-dessous.

```java
package fr.univamu.iut.reservation;

import jakarta.ws.rs.client.Client;
import jakarta.ws.rs.client.ClientBuilder;
import jakarta.ws.rs.client.Entity;
import jakarta.ws.rs.client.WebTarget;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

import java.util.ArrayList;

public class BookRepositoryAPI implements BookRepositoryInterface {

    /**
     * URL de l'API des livres
     */
    String url;

    /**
     * Constructeur initialisant l'url de l'API
     * @param url chaîne de caractères avec l'url de l'API
     */
    public BookRepositoryAPI(String url){
        this.url = url ;
    }

    @Override
    public void close() {}

    @Override
    public Book getBook(String reference) {
        Book myBook = null;

        // création d'un client
        Client client = ClientBuilder.newClient();
        // définition de l'adresse de la ressource
        WebTarget bookResource  = client.target(url);
        // définition du point d'accès
        WebTarget bookEndpoint = bookResource.path("books/"+reference);
        // envoi de la requête et récupération de la réponse
        Response response = bookEndpoint.request(MediaType.APPLICATION_JSON).get();

        // si le livre a bien été trouvé, conversion du JSON en Book
        if( response.getStatus() == 200)
            myBook = response.readEntity(Book.class);

        // fermeture de la connexion
        client.close();
        return myBook;
    }

    @Override
    public boolean updateBook(String reference, String title, String authors, char status) {
        boolean result = false ;

        Book updatedBook = new Book(reference, title, authors);
        updatedBook.setStatus( status ) ;

        // création d'un client
        Client client = ClientBuilder.newClient();
        // définition de l'adresse de la ressource
        WebTarget bookResource  = client.target(url);
        // définition du point d'accès
        WebTarget bookEndpoint = bookResource.path("books/"+reference);
        // envoi de la requête avec le livre en JSON et récupération de la réponse
        Response response = bookEndpoint.request(MediaType.APPLICATION_JSON)
                .put( Entity.entity(updatedBook, MediaType.APPLICATION_JSON) );

        // si la mise à jour a été faite
        if( response.getStatus() == 200)
            result = true;

        // fermeture de la connexion
        client.close();

        return result;
    }
}
```

```java
package fr.univamu.iut.reservation;

import jakarta.ws.rs.client.Client;
import jakarta.ws.rs.client.ClientBuilder;
import jakarta.ws.rs.client.WebTarget;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

public class UserRepositoryAPI implements UserRepositoryInterface {

    /**
     * URL de l'API des utilisateurs
     */
    String url;

    /**
     * Constructeur initialisant l'url de l'API
     * @param url chaîne de caractères avec l'url de l'API
     */
    public UserRepositoryAPI(String url){
        this.url = url ;
    }

    @Override
    public void close() {

    }

    @Override
    public User getUser(String id) {
        User myUser = null;

        // création d'un client
        Client client = ClientBuilder.newClient();
        // définition de l'adresse de la ressource
        WebTarget bookResource  = client.target(url);
        // définition du point d'accès
        WebTarget bookEndpoint = bookResource.path("users/"+id);
        // envoi de la requête et récupération de la réponse
        Response response = bookEndpoint.request(MediaType.APPLICATION_JSON).get();

        // si le livre a bien été trouvé, conversion du JSON en User
        if( response.getStatus() == 200)
            myUser = response.readEntity(User.class);

        // fermeture de la connexion
        client.close();
        return myUser;
    }
}
```


### Ajouter la méthode de réservation dans le service


Nous utilisons ensuite ces classes dans notre service pour vérifier si un livre peut être réservé par un utilisateur, et enregistrer cette réservation. Dans `ReservationService`, nous ajoutons  deux attributs  `BookRepositoryInterface bookRepo` et `UserRepositoryInterface userRepo` pour pouvoir facilement manipuler ces API. Nous ajoutons aussi deux paramètres pour les initialiser dans le constructeur de la classe (cf. code ci-dessous).

```java
   /**
     * Constructeur permettant d'injecter l'accès aux données
     * @param reservationRepo objet implémentant l'interface d'accès aux données des réservations
     * @param bookRepo  objet implémentant l'interface d'accès aux données des livres
     * @param userRepo  objet implémentant l'interface d'accès aux données des utilisateurs
     */
    public ReservationService(ReservationRepositoryInterface reservationRepo, BookRepositoryInterface bookRepo,
                              UserRepositoryInterface userRepo) {
        this.reservationRepo = reservationRepo;
        this.bookRepo = bookRepo ;
        this.userRepo = userRepo ;
    }
```

 Nous ajoutons ensuite une méthode `boolean registerReservation(String id, String reference)` dans la classe `ReservationService`. Elle sera chargée d'implémenter les règles métier liées aux réservations et d'enregistrer la réservation si elle les satisfait.

```java
  /**
     * Méthode permettant d'enregistrer une réservation (la date est définie automatiquement)
     * @param id identifiant de l'utilisateur
     * @param reference référence du livre à réserver
     * @return true si le livre a pu être réservé, false sinon
     */
    public boolean registerReservation(String id, String reference) {
        boolean result = false;

        // récupération des informations du livre
        Book myBook = bookRepo.getBook( reference );

        //si le livre n'est pas trouvé
        if( myBook == null )
            throw  new NotFoundException("book not exists");

        // si le livre est disponible
        if( myBook.status == 'd')
        {
            // recherche l'utilisateur
            User myUser = userRepo.getUser(id);

            // si l'utilisateur n'existe pas
            if( myUser == null)
                throw  new NotFoundException("user not exists");

            // modifier le statut du livre à "r" (réservé) dans le dépôt
            result = bookRepo.updateBook(myBook.reference, myBook.title, myBook.authors, 'r');

            // faire la réservation
            if( result )
                result = reservationRepo.registerReservation(id, reference);
        }
        return result;
    }
```


### Publier le point d'accès "créer une réservation"

Une fois le service implémenté, nous pouvons ajouter le point d'accès ("endpoint") dans notre classe ressource `ReservationResource`. Ce point d'accès sera associé au chemin `api/reservations/` et traitera des requêtes HTTP POST. Le données sont supposées être transmises sous forme de couples clé-valeur séparés par '&' , avec un '=' entre la clé et la valeur (comme les données transmises par les formulaires HTML).

```java
    /**
     * Enpoint permettant de soumettre une nouvelle réservation de livre demandée par un utilisateur enregistré
     * @param id identifiant de l'utilisateur souhaitant faire la réservation
     * @param reference référence du livre à réserver
     * @return un objet Response indiquant "registred" si la réservation a été faite ou une erreur "conflict" sinon
     */
    @POST
    @Consumes("application/x-www-form-urlencoded")
    public Response registerReservation(@FormParam("id") String id, @FormParam("reference") String reference){

        if( service.registerReservation(id, reference) )
            return Response.ok("registred").build();
        else
            return Response.status( Response.Status.CONFLICT ).build();
    }
```


### Ajouter la connexion/déconnexion aux API au lancement de l'application

Il reste un dernier point à aborder pour finaliser cette fonctionnalité: la création des objets `BookRepositoryAPI` et `UserRepositoryAPI`, et leur injection dans le service. Cette création sera faite dans la classe `ReservationApplication` afin de contrôler les dépendances entre le "service" (logique métier) et l'accès aux données. Par ailleurs, nous allons faire en sorte de créer une seule fois ces objets (et non pas à chaque requête) pour éviter de multiples clients (connexions) vers ces API. Comme pour la connexion à la base de données, nous allons donc créer des méthodes ("factory") dans `ReservationApplication` et utiliser l'API CDI de Jakarata pour injecter (via des annotations) ces objets dans le constructeur de `ReservationResource` lorsque nécessaire.

Pour cela, il nous suffit d'ajouter les méthodes suivantes dans le code de `ReservationApplication`.

```java
    /**
     * Méthode appelée par l'API CDI pour injecter l'API Book au moment de la création de la ressource
     * @return une instance de l'API avec l'url à utiliser
     */
    @Produces
    private BookRepositoryInterface connectBookApi(){
        return new BookRepositoryAPI("http://localhost:8080/book-1.0-SNAPSHOT/api/");
    }

    /**
     * Méthode appelée par l'API CDI pour injecter l'API User au moment de la création de la ressource
     * @return une instance de l'API avec l'url à utiliser
     */
    @Produces
    private UserRepositoryInterface connectUserApi(){
        return new UserRepositoryAPI("http://localhost:8080/user-1.0-SNAPSHOT/api/");
    }
```

Vous pouvez maintenant tester ce nouveau point d'accès en faisant une requête cURL tel que `curl -X POST -d "id=idBob&reference=ABC" http://localhost:8080/reservation-1.0-SNAPSHOT/api/reservations/` 



## 3.3 - Supprimer une réservation

Pour finir, nous allons ajouter dans l'API la possibilité de supprimer une réservation. Le statut du livre doit être changé (de "r" à "d") lors de la suppression pour le rendre à nouveau disponible.


### Ajouter la méthode de suppression de réservation dans le service

L'accès aux API `user` et `livre` étant déjà intégré, il nous suffit d'ajouter dans la classe `ReservationService` une méthode implémentant le processus de suppression. Cette méthode aura pour signature `boolean removeReservation(String reference)`, et son code est donné ci-dessous.

```java
  /**
     * Méthode supprimant une réservation
     * @param reference référence du livre à libérer
     * @return true si la réservation a été supprimée, false sinon
     */
    boolean removeReservation(String reference){
        boolean result = false;

        // récupération des informations du livre
        Book myBook = bookRepo.getBook( reference );

        //si le livre n'est pas trouvé
        if( myBook == null )
            throw  new NotFoundException("book not exists");
        else
        {
            // modifier le statut du livre à "d" (disponible) dans le dépôt des livres
            result = bookRepo.updateBook(myBook.reference, myBook.title, myBook.authors, 'd');

            // supprimer la réservation
            if(result ) result = reservationRepo.releaseReservation( reference );
        }
        return result;
    }
```


### Publier le point d'accès "supprimer une réservation"

Une fois le service implémenté, nous pouvons ajouter le point d'accès ("endpoint") dans notre classe ressource `ReservationResource`. Ce point d'accès sera associé au chemin `api/reservations/{reference}` et traitera des requêtes HTTP DELETE. Le code de cette méthode est décrit ci-dessous.


```java
/**
     * Endpoint permettant de supprimer une réservation
     * @param reference référence du livre à "libérer"
     * @return un objet Response indiquant "removed" si la réservation a été annulée ou une erreur "not found" sinon
     */
    @DELETE
    @Path("{reference}")
    public Response removeReservation(@PathParam("reference") String reference){

        if( service.removeReservation(reference) )
            return Response.ok("removed").build();
        else
            return Response.status( Response.Status.NOT_FOUND ).build();
    }
```
Vous pouvez maintenant tester ce nouveau point d'accès en faisant une requête cURL tel que `curl -X DELETE  http://localhost:8080/reservation-1.0-SNAPSHOT/api/reservations/ABC` 




















