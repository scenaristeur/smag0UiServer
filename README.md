# smag0UiServer
Un serveur bi-directionnel en temps réel pour échanger des informations avec le projet Smag0,
sur lequel on greffe les modules qui nous intéressent, Thérèse.

Utilise node.js, express, socket.io

[Quelques explications complémentaires](http://cra-s.blogspot.fr/2016/09/smag0uiserver-un-serveur-de-base-pour.html)


# smag0UiServer
Un serveur bi-directionnel en temps réel pour échanger des informations avec le projet Smag0, sur lequel on greffe les modules qui nous intéressent, Thérèse.

Contact & questions via twitter : @dfaveris #smag0


  

Utilise node.js (v6.5.0), express, socket.io
Testé avec Chrome (Version 53.0.2785.101 m (64-bit)), Microsoft Edge 38.14393.0.0) , Firefox (v48.0.2)
# code source

Après les nombreux tests effectués pour implémenter une interface utilisateur simple mais complète pour le projet Smag0, il apparaît clairement que de nombreux éléments sont redondants entre les projets.

On va donc ici tenter de les synthétiser, afin de fournir une base de travail réutilisable, et éviter de se reposer les mêmes questions pour tous les projets.

L'interface utilisateur de Smag0 doit permettre :

d'enregistrer des informations, et de les consulter de manière la plus simple qui soit
de partager ces informations
en les enregistrant sous forme de fichier
en les mettant à disposition sur un serveur
en proposant la collaboration
Je ne reviendrais pas sur les raisons qui me poussent à utiliser le format RDF, je me suis déjà largement étendu sur le sujet sur le blog du projet.

Le choix a été fait d'utiliser Node.js mais la discussion est ouverte, et la solution proposée ici se veut adaptable à d'autres environnements.
--> prérequis : installation de node.js et npm.


Les fichiers de base : 

créons un dossier "smag0UiServer".
à intérieur, on aura besoin d'un fichier "index.js" qui gérera notre serveur.
Ajoutons un dossier "public" dans lequel on mettra un fichier "index.html" qui sera la page visible de notre serveur.
Dans notre répertoire, lançons la commande " npm init ". Ceci créé un fichier package.json, bien utile aux dépendances de node.js . Répondez aux questions selon les caractéristiques de votre projet. Seule chose à veiller : gardez bien "index.js" comme point d'entrée (entry point).  Mon package.js à cette étape ressemble donc à ça.
commit  (les différents "commit" que vous trouverez dans cet article vous fournissent le code de l'étape que l'on vient de réaliser).

Pour construire une application à partir de rien, vous pouvez toujours vous référer aux nombreux cours existants, comme celui-ci, mais ce n'est pas le but ici.
On va plutôt tenter de factoriser l'expérience acquise au cours de nos expérimentations, notamment en s'inspirant de DreamCatcher Autonome pour cette partie serveur (socket.io -> temps-réel, multi-user).

Pour l'architecture, le routage, nous avions utilisé le module Express, et pour la communication entre notre page "public/index.html" et le serveur, nous avions utilisé socket.io.
Installons maintenant ces deux modules node, par l'invite de commande dans notre répertoire "smag0UiServer" :
npm install express socket.io --save
n'oubliez pas le "--save", car c'est ça qui met à jour votre fichier " package.json " de ces deux dépendances.

Ceci vous créé par la même occasion un dossier "node_modules" dans lequel sont stockés les modules de node.

commit ( un fichier .gitignore a été utilisé ici pour le pas envoyer ce dossier inutilement sur github )


Entrons maintenant dans le vif du sujet.


Un peu de code, côté serveur :


On va s'attaquer au fichier "index.js" pour lui demander de nous afficher la page "public/index.html"
Ouvrez le fichier "index.js" et collez lui le code suivant :


// Setup basic express server
var express = require('express');
var app = express();
var server = require('http').createServer(app);
var io = require('socket.io')(server);
var port = process.env.PORT || 3000;
server.listen(port, function () {
console.log('Server listening at port %d', port);
});
// Routing
app.use(express.static(__dirname + '/public'));

Pour s'assurer que ça fonctionne écrivez également quelque chose dans le fichier "public/html", un truc du style "si cette phrase s'affiche, c'est OK !!! " . Peu importe ce que vous y écrivez, de toute façon, il y a peu de chances que ça reste bien longtemps...

Pour tester, il ne vous reste plus qu'à exécuter le serveur, grâce à la commande
 node . 

N'oubliez pas le point : il designe le répertoire courant, et par défaut le fichier index.js. En cas de doute, vous pouvez également taper
node index.js
Le terminal devrait maintenant vous indiquer "Server listening at port 3000", ce qui signifie que vous pouvez consulter la page publique (public.html) en vous rendant à l'adresse "http://127.0.0.1:3000/" ou équivalent à "http://localhost:3000/".

Si la phrase "si cette phrase s'affiche, c'est OK !!!" s'affiche dans votre navigateur, vous avez remporté cette étape.

commit

Établir la communication entre la page publique et le serveur : 


Selon les préconisations de socket.io,
ajoutez à la fin de notre fichier "index.js", les lignes suivantes :

//Socket.IO
  io.on('connect', function(){});
  io.on('event', function(data){});
  io.on('disconnect', function(){});

!! ATTENTION : A CHAQUE MODIFICATION DU FICHIER " INDEX.JS" , il faut couper le serveur (Ctrl+C) et le relancer (node . ), pour qu'il prenne en compte les modifications.

et dans le fichier "public/index.html", supprimez la ligne "si cette phrase s'affiche, c'est OK !!!" et remplacez-là par le code suivant :

<!DOCTYPE html>
     <head>
     <title>Dreamcatcher / Smag0 Socket.IO</title>
      <!-- GENERAL LIBRAIRIES -->       
      <script src="/socket.io/socket.io.js"></script>
      <script>
          var host = window.document.location.host;
          var socket = io(host);
           socket.on('connect', function(){});
           socket.on('event', function(data){});
           socket.on('disconnect', function(){});
           console.log(socket);
      </script>
     </head>
     <body>
           Version avec Socket 
     </body>
</html>

Si tout se passe bien, (après avoir arrêté le serveur (Ctrl+C) et l'avoir redémarré (node .), la consultation de la page "http://127.0.0.1:3000" devrait maintenant afficher "Version avec Socket", et dans la console Javascript de votre navigateur, la définition de la socket devrait s'afficher,...
 et seulement ça. Si d'autres lignes s'affichent, y'a un problème.

commit

Éventuellement, une petite erreur au sujet d'un fichier "service-worker.js" introuvable peut apparaître. C'est une nouvelle fonctionnalité des navigateurs.
Vous pouvez ignorer cette erreur, ou créer un fichier "service-worker.js" vide dans le répertoire racine ou dans le répertoire "public".

A ce stade, nous avons notre serveur (index.js) qui fournit une page publique (public.index.html) et la communication est maintenant établie entre ces deux éléments...

La base est là...

Passons maintenant aux choses sérieuses, et développons les différentes fonctionnalités évoquées précédemment...

Contact :
twitter : @dfaveris #smag0









