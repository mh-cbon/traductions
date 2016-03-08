# Préambule

Url original : https://gist.github.com/staltz/868e7e9bc2a7b8c1f754

Auteur : [André Staltz](https://github.com/staltz)

Licence : [Creative Commons Attribution-NonCommercial 4.0 International License](http://creativecommons.org/licenses/by-nc/4.0/)

Ntdt : J'emploierais le tutoiement, j'essaierais de ne pas faire trop de mot à mot, mais bon je ne suis pas un traducteur professionnel. Il y aura probablement pleins de fautes, des tournues mal venues, ou bien encore des traductions maladroite. La bonne nouvelle c'est que vous pouvez éditer le document par vous même et proposer des corrections.

# L'introduction à la programmation réactive que tu as manqué

_____
Ce tutoriel est aussi disponible en vidéos

Si tu préfères regarder des tutoriels vidéos avec des sessions de programmation en directe, jettes  un oeil à cette série que j'ai enregistré avec le même contenu que cet article : [Egghead.io - Introduction to Reactive Programming](https://egghead.io/series/introduction-to-reactive-programming).
_____

Donc tu es curieux au sujet de ce nouveau truc appelé la Programmation Réactive, plus particulièrement ses adaptations tels que Rx, Bacon.js et autres.

Apprendre est une tâche difficile, plus encore lorsque les bonnes ressources sont manquante.
Lorsque j'ai débuté, j'ai recherché des tutoriels. Je n'ai pu trouver qu'une poignée de ressources utiles, mais elles ne faisaient qu'effleurer la surface du problème et ne se sont jamais aller en profondeur. Les documentation d'API ne sont souvent
que de peu d'aide pour comprendre un problème. Honnêtement, regardes un peu cela :

    Rx.Observable.prototype.flatMapLatest(selector, [thisArg])

    Projette chaque élément d'une séquence de sujets dans une nouvelle séquence de sujet
    en ajoutant l'index de cet élément, puis transforment cette séquence de sujets
    de séquence de sujets en produisant les valeurs les plus récente de la séquence de sujet.

Ndt : En anglais c'est déjà n'importe quoi, en français c'est vraiment du charabia.

    Projects each element of an observable sequence into a new sequence of
    observable sequences by incorporating the element's index and then transforms
    an observable sequence of observable sequences into an observable sequence
    producing values only from the most recent observable sequence.

Evidemment !

J'ai lu deux livres, l'un d'entre eux n'était qu'une vue générale, tandis que l'autre
expliquait comment utiliser la Programmation Réactive. J'ai fini par apprendre la
Programmation Réactive par la pratique. Lorsque je travaillais pour Futurice j'ai
eu à utiliser ce paradigme dans de vrai projets, et j'ai reçu le support de mes collègues
lorsque j'avais des difficultées.

Le plus difficile dans ce parcours est d'apprendre à penser et réfléchir Réactif.
Le gros de cet effort consiste à oublier les vieux paradigmes impératifs hérités
de nos habitudes de programmation, pour forcer notre cerveau à travailler dans
ce nouveau mode de pensée. Je n'ai pas trouvé de guide pour cela sur l'Internet,
je crois cependant que cela nécessite un tutoriel concret et pratique sur
cette approche et la manière de penser Réactif afin que chacun puisse débuter.
La documentation de la librairie vous fournira de plus amples détails dès lors
que vous en aurez fini avec cet article. J'espère que cela vous aidera.

# "Qu'est ce que la Programmation Réactive ?"

Il y a pleins de mauvaises explications et définitions sur l'Internet. Comme d'habitude Wikipedia est
trop générique et théorique. Les réponses lapidaire de Stackoverflow ne
sont évidemment pas utile aux débutants.
Le [Manifeste Reactif](http://www.reactivemanifesto.org/fr) ressemble aux genres
de documents que l'on montre à sa hiérarchie. La terminologie de microsoft
"Rx = Sujets + LINQ + Ordonnanceurs" est bien trop lourde et tellement
particulière à l'environnement microsoft que la plupart d'entre nous restons coi.
Les termes tels que "reactif" et "propagation des changements" n'apporte
pas grand chose de plus à notre modèle MV* traditionnel et son implémentation
dans nos langages favoris. Evidemment que nos vues réagissent à nos modèles.
Si ce n'était pas le cas, rien ne s'afficherait à l'écran.

Allons droit au but.

#### La Programmation Réactive consiste à programmer des flux de données asynchrone

D'une certaine manière il n'y a rien de nouveau la dedans. Les bus d'évènements ou
même le plus simple évènement de clique sont des flux de données asynchrone,
que l'on peut écouter et ainsi modifier l'état du programme.
La Programmation Réactive n'est qu'une formalisation de cela.
On peut créer des flux de données depuis n'importe quoi,
pas uniquement des cliques de souris. Les flux sont disponible partout
et ne coûtent rien à l'instanciation, il n'y a rien qui ne puisse être un
flux : variables, évènements utilisateur, propriétés, caches, structure de donées, ect.
Imaginer par exemple que votre Twitter soit un flux de données
de la même manière que les cliques le sont. Vous pourriez vous y abonner, et réagir de manière adéquat.

En plus des flux, des outils sont mis à votre dispositions pour combiner,
créer et filtrer ces flux. C'est là que le mot "Fonctionnel" entre en jeu.
Un flux peut être consommer en entré d'un autre flux.
 L'on peut même utiliser plusieurs flux pour former un nouveau flux.
 Vous pouvez fusionner deux flux. Ou bien encore filtrer un flux pour obtenir
 un nouveau flux des seuls évènements qui vous intéressent.
 Il est aussi possible de créer un flux qui est le résultat de la transformation des valeurs d'un autre flux.

Puisque les flux sont tellement important à la Programmation Réactive,
prennons le temps de les regarder de plus près en commençant avec l'exemple du flux d'évenements de clique.

![Flux d'evenement de clique](http://i.imgur.com/cL4MOsS.png)

Un flux est une séquence d'évènements à venir ordonnés dans le temps.
Un flux peut émettre trois types de choses : Une valeur (avec un type),
une erreur, ou un signal de terminaison.
Pour le moment, considérez que le flux est terminé lorsque
la fenêtre de la vue contenant ce bouton est fermée.

La capture de ces évènements est complètement asynchrone en définissant,
une fonction qui s'executera lorsqu'une valeur est écrite sur le flux,
une fonction qui sera déclenchée si une erreure survient,
et une dernière fonction terminale lorsque le flux aura complété son processus.
Certaines fois, les deux dernières fonctions ne sont pas déclarées et l'on pourra
se concentrer sur la fonction de transformation des valeurs.
L'action d'"écouter" un flux est appelé "abonnement".
Les fonctions que nous délcarons sont des "observateurs".
Le flux est le "sujet" que l'on observe. C'est précisément le patron de conception [Observateur](https://fr.wikipedia.org/wiki/Observateur_%28patron_de_conception%29).

Une autre manière d'écrire ce diagrame est d'utilsier ASCII, que nous utliserons durant ce tutoriel :
```
--a---b-c---d---X---|->

a, b, c, d les signaux émis
X est une erreur
| le sginal de 'terminaison'
---> la ligne du temps
```
Puisque cela nous est déjà tellement familier,
et que je ne veux pas vous vous ennuyiez,
faisons quelque chose de nouveau : nous allons créer un flux d'évènements
de clique depuis un flux d'évenèments de clique existant.

Tout d'abord, créons un flux de comptage `counterStream` qui indiquera
le nombre de fois qu'un bouton a été cliqué. Dans la plupart des librairires Réactive,
chaque flux possède des fonctions qui lui sont attachés tels que `map`, `filter`, `scan` ect.
Lorsque vous appelez une de ces fonctions, comme ceci `clickStream.map(f)`,
celle ci crée et retourne un nouveau flux qui consomme le flux précédent.
Ces fonctions ne modifient jamais le flux original.
C'est une propriété que l'on appelle "**immutabilité**", et cela va de pair avec
la Programmation Réactive, comme 2 et 2 font 4.
Cela nous permet de chaîner les appels de fonctions ainsi : `clickStream.map(f).scan(g)` :

```
  clickStream: ---c----c--c----c------c-->
               vvvvv map(c devient 1) vvvv
               ---1----1--1----1------1-->
               vvvvvvvvv scan(+) vvvvvvvvv
counterStream: ---1----2--3----4------5-->
```

La fonction `map(f)` remplace (dans le nouveau flux) chaque valeur écrite
par le résultat de l'appel de la fonction `f` que vous aurez fournit.
Dans notre exemple, nous transformons les cliques en nombre de valeur `1`.
`scan(g)` est une fonction qui aggrège les valeurs reçues,
tels que `x = g(accumulated, current)`, où `g` est une simple fonction d'addition.
Finalement, `counterStream` émettra le nombre total de clique à chaque fois
qu'un nouveau clique sera émis.

Pour vous montrer le vrai pouvoir de la Programmation Réactive, disons maintenant
que nous souhaitont avoir un flux des `double-clicks` uniquement.
Pour rendre cela encore plus intéressant, disons que notre nouveau flux considerera
 les `triple-clicks` comme des `double-clicks`, et même plus généralement
 de multiple cliques consécutifs. Maintenant, prenez une grande inspiration
 et imaginer deux minutes ce à quoi cela ressemblerait d'une manière traditionnelle.
 Je parie que cela vous semble avoir mauvaise allure et que vous devriez sûrement
 avoir de multiple variables pour garder une trace de l'état du programme
 en plus de jouer avec des intervalles de temps.

Figurez-vous qu'en Programmation Réactive cela est assez simple.
En fait, [4 lignes de code](http://jsfiddle.net/staltz/4gGgs/27/).
Réfléchir avec des diagrammes est le meilleur moyen de réfléchir
aux flux que vous soyez un expert ou un débutant.

![Multiple clicks stream](http://i.imgur.com/HMGWNO5.png)

Les boîtes grises sont les fonctions qui transforment en flux vers un nouveau flux.
Pour aller à l'essentiel, au début nous accummulons les cliques à intervalles de `250ms`,
c'est le but de `buffer(stream.throttle(250ms)`. Le résultat est un flux de listes,
sur lequel nous appliquons la fonction `map()` afin de transformer chaque liste
en la valeur de sa longueur (nombre d'éléments dans la liste).
Finalement, nous ignorons les valeurs inférieures à 2 avec `filter(x >= 2)`.
C'est tout : 3 opérations pour produire le flux désiré.
Il est désormais possible de s'abonner aux flux résultant et de réagir adéquatement.

# "Pourquoi devrais je considérer d'adopter la Programmation Réactive ?"

La Programmation Réactive améliore l'abstraction du code afin que nous puissions
nous concentrer sur les interdépendances d'évènements définis par la logique métier
plutôt que d'avoir à constamment gérer les détails d'implémentation.
Un Programme Réactif est souvent plus conçi.

Le bénéfice est plus évident encore dans les applications webs moderne qui sont
hautement intéractive avec une mutlitude d'évènements utilisateurs à manipuler.
Il y a 10 ans, les intéractions d'une page web consistaient à soumettre un formulaire
et réaliser de simples opérations de rendu.
Nos applications ont évoluées pour s'executer en temps réél :
la modification d'un champ de formulaire peut générer l'envoi automatique
d'une validation sur le serveur, cliquer sur un bouton "j'aimes" peut être
partagé en temps réél avec les autres visiteurs, etc.

Les applications modernes abondent d'évènements en temps réél de toutes sortes
afin de créer une meilleure expérience utilisateur. Nous avons besoins d'outils
pour faire face à cela, La Programmation Réactive est la réponse.

# Réfléchir en Programmation Réactive par l'exemple

Commençons les choses sérieuses désormais. Un exemple concret avec un guide
sur la manière de penser la Programmation Réactive. Aucun exemple de synthèse,
pas de concept à moité expliqué. A la fin de ce tutorial nous aurons implémenté
un code fonctionnel, en comprenenant chaque composant.

J'ai choisit Javascript et RxJS comme outils pour cette démonstration,
la raison : Javascript est le langage le populaire en ce moment, par ailleurs
la famille des librairies Rx* est disponible pour plusieurs langages
(.NET, Java, Scala, Clojure, JavaScript, Ruby, Python, C++, Objective-C/Cocoa, Groovy, etc).
Donc quelque soit votre outil, vous pourrez y apppliquer ces concepts.

# Implémenter une boîte de suggestions "Qui suivre"

Twitter possède cet élément d'UI qui vous suggère
d'autres comptes que vous pourriez vouloir suivre :

![Twitter Who to follow suggestions box](http://i.imgur.com/eAlNb0j.png)

Nous nous appliquerons à reproduire ces fonctions principale, qui sont :

  Au démarrage, charger la donné des comptes depuis l'API et afficher 3 résultats
  Au clique du bouton "Rafraîchir", charger 3 autres suggestions dans 3 lignes
  Au clique du bouton "x" d'une ligne de suggestion, effacer cette suggestion et y afficher une nouvelle
  Chaque ligne affichera l'avatar et un lien vers le compte suggérer

Nous pouvons laisser de côté les autres fonctionnalités et boutons
car ils ne sont pas primordiaux à notre découverte. Et plutôt que d'utiliser Twitter,
qui à récemment restreint ces API, utilisons Github.
Il y a une API Github pour obtenir la liste d'utilisateurs.

Le code complet est déjà disponible à l'addresse
http://jsfiddle.net/staltz/8jFJH/48/ au cas où vous voudriez d'ores et déjà y jeter un oeil.

# Requête et Réponse

Comment résoudre ce prolbème avec Rx?
Et bien, pour commencer, (presque) tout est un flux. C'est le mantra Rx.
Commençons par le plus simple: "Au démarrage, charger 3 comptes depuis l'API".
Rien de spécial ici, il s'agit tout juste de (1) faire une requête,
(2) obtenir la réponse, (3) afficher le résultat.
C'est partit pour commencer à représenter nos requêtes comme un flux.
Cela peut sembler excessif au début, mais il faut bien commencer quelque part, n'est ce pas ?

Au démarrage nous devons exécuter une requête, donc si nous devions
modéliser cela comme un flux, ce serait un flux qui n'émet qu'une valeur.
Plus tard, nous le savons déjà, nous aurons de mulitples requêtes à réaliser,
mais pour l'instant, concentrons nous sur 1 requête.

```
--a------|->

Où a est une URL 'https://api.github.com/users'
```

Il s'agit bien de vouloir exécuter les requetes d'un flux d'URL.
A chaque fois qu'un évenement requête survient, il indique deux choses "quand" et "quoi".
"quand" la requête doit être executée lorsqu'un évenement requête survient.
"quoi" ce qui doit être obtenu : La chaîne de caractère de l'URL

Créer ce genre de flux avec une seule valeur est très facile avec Rx*.
```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');
```

Cependant, ce n'est plus qu'un flux d'URL, qui ne fait rien de spécial,
nous devons donc faire en sorte que quelque chose se produise
lorsqu'une valeur est émise. Cela est possible en s'abonnant à ce flux.

```
requestStream.subscribe(function(requestUrl) {
  // execute the request
  jQuery.getJSON(requestUrl, function(responseData) {
    // ...
  });
}
```

Notez que nous utilisons jQuery Ajax (que vous devriez déjà connaitre) pour
implémenter la requête asynchrone. Cependant, Rx est justement là pour gérer
l'asynchronicité des flux de données. Ne se pourrait il pas que la réponse de
cette requête soit aussi un flux contenant les données reçues plus tard ?
A un niveau conceptuel c'est exactement cela, jetons y un oeil.

```
requestStream.subscribe(function(requestUrl) {
  // execute the request
  var responseStream = Rx.Observable.create(function (observer) {
    jQuery.getJSON(requestUrl)
    .done(function(response) { observer.onNext(response); })
    .fail(function(jqXHR, status, error) { observer.onError(error); })
    .always(function() { observer.onCompleted(); });
  });

  responseStream.subscribe(function(response) {
    // do something with the response
  });
}
```

`Rx.Observable.create()` vous permet de créer votre propre flux personnalisé.
Il implémente la mécanique nécessaire à la communication inter-flux (onNext(), onError()).
Nous avons simplement encapsulé notre promesse jQuery Ajax. **Cela signifie t'il qu'une promesse est un Sujet `Observable` ?**

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Amazed](http://www.reactiongifs.com/r/ib.gif)

Oui.

Un `Observable` est une promesse compatible `Promise++`.
Avec Rx vous pouvez facilement convertir une Promesse vers un Sujet `Observable`
de cette façon `var stream = Rx.Observable.fromPromise(promise)`.
La seule différence est que les Sujets `Observable` ne sont pas compatible `Promise/A+`,
mais conceptuellement il n'y à pas de conflits. Une promesse est un Sujet `Observable`
avec une seule valeur en sortie.
Les flux Rx vont au délà des Promesses en permettant de multiple valeurs de sortie.

C'est pltôt cool, et cela montre que les `Observable` sont au moins aussi puissant
que les Promesses. Donc si vous croyez en la mode des Promesses,
gardez les yeux ouverts sur ce que les `Observable` de Rx sont capable.

Retournons à notre exemple, si vous être malin, vous avez déjà remarqué que
nous avions un appel récursif à `subscribe()`,
ce qui est à peu près un `callback hell`. Aussi, notez comme la création du flux
`responseStream` est dépendant du flux `requestStream`. Comme vous l'avez lu plus tôt,
Rx fournit des méthodes simple pour transformer et créer de nouveaux flux
depuis ceux déjà existant, nous devrions donc faire cela.

La plus basique des fonctions que vous devez connaitre est `map(f)`,
qui prend chaque valeur d'un flux A, execute `f()` dessus, et produit un flux de valeurs B.
Si nous appliquions cela à notre flux de requête/réponse,
nous pouvons transformer chaque URL en une Promesse de réponse (déguisée en flux).


```javascript
var responseMetastream = requestStream
  .map(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

Ainsi nous créons une nouvelle bête appelé un `meta-flux`: un flux de flux.
Ne paniquez pas déjà. Un meta-flux est un flux dont chaque valeur est un flux.
Vous pouvez comprendre cela comme des pointeurs : Chaque valeur est un pointeur vers un autre flux.
 Dans notre exemple, chaque URL est transformée vers un pointeur de flux de
 Promesse vers la réponse correspondante.

![Response metastream](http://i.imgur.com/HHnmlac.png)

Un meta-flux de réponse semble confus, et ne nous aide pas du tout.
Nous voudrions plus simplement un flux de réponses, où chaque valeur émises
est un objet JSON, non pas une Promesse d'objet JSON.
Dites bonjour a Mr. `Flatmap`: une version de `map()` qui réduit un meta-flux en
emettant sur le flux principal tout ce que le flux secondaire emettra.
Flatmap n'est pas un "fix" ou un bug, c'est un outils vraiment utile pour
gérer des réponses asynchrones avec Rx.


```javascript
var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

![Response stream](http://i.imgur.com/Hi3zNzJ.png)

Cool. Et puisque le flux de reponse se comporte en fonction de notre flux de requete,
si plus tard d'autres évènements devaient survenir dans ce flux,
nous recevrons les réponses correspondante à chacun de ces évènements,
comme on peut s'y attendre :


```
requestStream:  --a-----b--c------------|->
responseStream: -----A--------B-----C---|->

(en minuscule les requêtes, en majuscule les responses)
```

Désormais nous avons un flux de réponses, nous pouvons donc afficher les résultats :


```javascript
responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

Assemblons le code produit jusqu'à présent, nous avons :

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });

responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

## Le bouton rafraîchir

Je n'ai pas encore mentionné que la réponse JSON est une liste de 100 utilisateurs.
L'API nous autorise à ne spécifier que l'offset de la page,
et non le nombre de résultat, en conséquence nous n'unitilisons que 3 lignes de la réponse
et gâchons le reste des 97 lignes.
Nous pouvons ignorer cela pour le moment, car plus tard nous utiliserons
une technique afin de mettre en cache nos résultats.

A chaque fois que le bouton rafraîchir est cliqué, le flux de requête
doit émettre une nouvelle URL, afin que nous puissions obtenir la réponse correspondante.
Nous avons besoins de deux choses : un flux de clique sur le bouton rafraîchir (mantra: tout est flux),
et aussi de changer le flux de requête afin de dépendre du flux de clique.
Heureusement, RxJS fournit des outils pour créer des flux `Observable` depuis un évènement.


```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');
```

Puisque le flux de clique n'émet pas de lui même une valeur d'URL pour l'API,
nous devons tranformer la valeur du clique en URL.
Changeons le flux de requête afin que le flux de rafraichissement transforme
chaque clic en une URL d'API avec un paramètre aléatoire additionnel.


```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
```

Comme je suis stupide et que je n'ai pas de tests automatique,
je vient juste casser une fonctionnalité implémentée autparavant.
Aucune requête n'est plus émise au démarrage de l'application, cela ne se produit
plus qu'au clique du bouton rafraîchir. Argh.
J'ai besoin de deux comportements : Une requête doit être émise _soit_
lorsque le bouton rafraîchir est cliqué _ou_ lorsque la page vient d'être ouverte.


Nous savons déjà comment créer des flux pour chacun de ces cas :

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var startupRequestStream = Rx.Observable.just('https://api.github.com/users');
```

Mais comment les "fusionner" en un seul flux ? Et bien il existe la méthode [`merge()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemergemaxconcurrent--other).
Expliqué avec un diagramme, voilà ce qu'il se passe :

```
flux A:   ---a--------e-----o----->
flux B:   -----B---C-----D-------->
          vvvvvvvvv merge vvvvvvvvv
          ---a-B---C--e--D--o----->
```

Cela devrait être simple désormais:

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var startupRequestStream = Rx.Observable.just('https://api.github.com/users');

var requestStream = Rx.Observable.merge(
  requestOnRefreshStream, startupRequestStream
);
```

Une alternative plus propre d'écrire cela existe, sans flux intermédiaire.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .merge(Rx.Observable.just('https://api.github.com/users'));
```

Voici même une version plus courte encore plus simple à lire:
```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .startWith('https://api.github.com/users');
```

La méthode [`startWith()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypestartwithscheduler-args)
fait exactement ce que vous pensez que celle ci fait.
Quel que soit la nature du flux, la valeur de sortie d'un flux créer avec `startWith(x)` sera `x` au début.
Mais ce code n'est pas suffisament [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself),
j'ai dupliqué l'URL de l'API. Une manière de corriger cela est de déplacer `startWith()`
à côté du flux `refreshClickStream`, pour simplement simuler un clique de rafraîchissement
au démarrage de l'application.


```javascript
var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

Cool. Maintenant si vous comparez avec le moment où
j'avais cassé les tests automatique, vous devriez constater que
la seule différence avec cette dernière approche se résume à l'ajout de `startWith()`.

## Modélisons les 3 flux de suggestions

Jusqu'à maintenant, nous n'avons implémenté que les étapes nécessaire à
l'affichage des réponses consécutive à l'abonnement de notre flux de réponses `responseStream`.
Depuis que nous avons le bouton rafraîchir, un nouveau probleme est apparu:
Dès que vous cliquez sur le bouton 'rafraichir', les 3 suggestions existante
ne sont pas effacées. Aussi, les nouvelles suggestions n'apparaissent qu'après
avoir reçut les réponses des requêtes correspondante.
Pour rendre l'interface cool, nous devons effacer les suggestions
existantes lorsqu'un clique du bouton rafraichir survient.


```javascript
refreshClickStream.subscribe(function() {
  // clear the 3 suggestion DOM elements
});
```

Pas si vite l'ami. C'est une mauvaise pratique car désormais nous avons
**deux** flux abonnés qui modifient les suggestions dans le DOM
(l'autre étant abonné à `responseStream.subscribe()`), et cela ne semble
pas très solide au regard de la [Separation des responsabilités](https://en.wikipedia.org/wiki/Separation_of_concerns).
Vous souvenez vous du mantra Rx ?

&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Mantra](http://i.imgur.com/AIimQ8C.jpg)

Nous allons donc modeler nos suggestions en tant que flux, où chaque valeur
émise est un objet JSON contenant les données de la suggestion.
Nous allons créer un flux pour chacune des suggestions à afficher.
Voici comment le flux de suggestion #1 pourrait être :

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  });
```

Les autres, `suggestion2Stream` et `suggestion3Stream`  seront simplement dupliquer depuis `suggestion1Stream`. Ce n'est pas DRY, mais nous le garderons ainsi durant ce tutoriel, laissant cela en exercice pour le lecteur.

Au lieu de générer l'affiche des résultats dans le flux `responseStream`, nous faisons ainsi :

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  // render the 1st suggestion to the DOM
});
```

Retournons en à "lors du rafraichissement, effacées les suggestions", nous pouvons désormais simplement trasnformé les cliques de rafraichissement en objet de suggestion `null`, et include cela dans notre flux `suggestion1Stream`

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  );
```

Ainsi lors du rendu, nous pouvons interpreter `null` comme "pas de données", donc masqué l'élément correspondant.

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

Maintenant la vue d'ensemble est :

```
refreshClickStream: ----------o--------o---->
     requestStream: -r--------r--------r---->
    responseStream: ----R---------R------R-->   
 suggestion1Stream: ----s-----N---s----N-s-->
 suggestion2Stream: ----q-----N---q----N-q-->
 suggestion3Stream: ----t-----N---t----N-t-->
```
Où `N` correspond à `null`.

Pour aller plus loin, nous pouvons maintenant affiché des suggestions vide au démarrage de l'application.
Nous ferons cela en ajoutant `startWith(null)` aux flux de suggestions:

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

Ce qui produire ce résultat:

```
refreshClickStream: ----------o---------o---->
     requestStream: -r--------r---------r---->
    responseStream: ----R----------R------R-->   
 suggestion1Stream: -N--s-----N----s----N-s-->
 suggestion2Stream: -N--q-----N----q----N-q-->
 suggestion3Stream: -N--t-----N----t----N-t-->
```

## Fermer une suggestion et utiliser le cache

Il y à une fonctionnalité manquante. Chaque suggestion devrait avoir un bouton `x`, pour fermer celle ci, et charger une nouvelle suggestion à sa place.
A première vue, vous pourriez pensez que c'est suffisant de faire une nouvelle requete lorsque ce bouton est cliqué.

```javascript
var close1Button = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(close1Button, 'click');
// and the same for close2Button and close3Button

var requestStream = refreshClickStream.startWith('startup click')
  .merge(close1ClickStream) // we added this
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

Ceci ne fonctionne pas. Ceci fermera est rechargera toutes les suggestions, au lieu de s'appliquer à l'élément désiré. Il y à différentes manières de résoudre cela, et pour garder cela intéressant, nous allons résoudre cela en ré utilisant les réponses des requetes dja effectuées. L'API retourne 100 utilisateurs alors que nous n'en consommons que 3, il reste donc beaucoup de lignes disponible.

Encore une fois, réfléchissons avec des flux. Lorsque un évènement de clique `close1` survient, nous voulons utilier la réponse _la plus récemment émise_ sur `responseStream` pour obtenir un utilisateur au hasard depuis la liste disponible. Ainsi:

```
    requestStream: --r--------------->
   responseStream: ------R----------->
close1ClickStream: ------------c----->
suggestion1Stream: ------s-----s----->
```

Avec Rx* il y à une fonction de combinaison appelé  [`combineLatest`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecombinelatestargs-resultselector) qui semble faire ce dont nous avons besoin. Celle ci prend deux flux A et B en paramètre, et à chaque fois qu'un des flux émets une valeur, `combineLatest` rassemble les deux valeurs plus récentes `a` et `b` de chaque flux et écrit une valeur tels que `c = f(x,y)`, où `f` est une fonction à définir. C'est plus simple avec un diagramme :


```
stream A: --a-----------e--------i-------->
stream B: -----b----c--------d-------q---->
          vvvvvvvv combineLatest(f) vvvvvvv
          ----AB---AC--EC---ED--ID--IQ---->

où f est une fonction de transformation de casse en majsucule
```

Nosu pouvons appliquer `combineLatest()` à `close1ClickStream()` et `responseStream()`, ainsi à chaque fois que le bouton `close` est cliqué, nous obtiendrons la dernière réponse émise et nous pourrons écrire une nouvelle valeur sur `suggestion1Stream`. N'oublison pas, `combineLatest()` est symétrique : A chaque fois qu'une réponse est émise depuis `responseStream()`, cela déclenchera la combinaison avec le dernier clique du bouton `close` pour produire une nouvelle suggestion. eci est intéresant ar cela nous permet de simpliier notre code précédent pour `suggestion1Stream`


```javascript
var suggestion1Stream = close1ClickStream
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

Une pièce du puzzle est toujours manquant. `combineLatest()` utilise les deux sources les plus récentes, mais si l'une de ces sources n'à pas émit de valeur, alors `combineLatest()` ne produira aucun évènement sur le flux de sortie. Si vous vérifier le diagramme au dessus, vous verrez qu'il n'y à pas de valeur de sortie lorsque le premier flux emet la valeur `a`. C'est seulement lorque la valeur `b` est émise que `combineLatest()` émet sa première valeur de sortie.

Différentes possiblités s'offrent à nous pour résoudre cela, et nous allons prendre la plus simple qui consistera à simuler un clique sur le bouton 'fermer' au démarrage de l'application :

```javascript
var suggestion1Stream = close1ClickStream.startWith('startup click') // we added this
  .combineLatest(responseStream,             
    function(click, listUsers) {l
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

## Emballer le tout

Nous y voilà ! Le code complet est donc :

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');

var closeButton1 = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(closeButton1, 'click');
// and the same logic for close2 and close3

var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var responseStream = requestStream
  .flatMap(function (requestUrl) {
    return Rx.Observable.fromPromise($.ajax({url: requestUrl}));
  });

var suggestion1Stream = close1ClickStream.startWith('startup click')
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
// and the same logic for suggestion2Stream and suggestion3Stream

suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

**vous pouvez voir l'exemple fonctionner ici http://jsfiddle.net/staltz/8jFJH/48/**

Ce bout de code est petit mais dense : Il peut gérer de multiples évènement en respectant la séparation des responsabilités, et même utiliser des réponses du cache. Le style fonctionnel rend le code plus concis et délcaratif que ele paradigme impératif : nous ne donnons pas des séquences d'instructions à éxecuter, nous décrivons **ce que chaque chose est** en définissant les rélations entre les flux. Actuellement, avec Rx nous avons décrit que _`suggestion1Stream` **est** le flux combiné de `close 1` avec un utilisateur de la dernière réponse, par ailleurs que celle ci est `null` lorsque le `rafraichissement` ou le démarrage de l'application survient_

Remarquez par ailleurs à quel point les instructions usuelles tels que `if`, `else`, `for`, `while` et les fonctions de rappels pour le control de flot sont absente. Vous pourriez même supprimer les `if`, `else` présent dans la fonction `subscribe()` en faisant appel à la méthode `filter()` si vous le souhaitiez (encore je laisse cela en exercice pour le lecteur). Avec Rx, nous des fonctions de flux tels que `map()`, `filter()`, `scan()`, `merge()`, `combineLatest()`, `startsWith()`, et bien plus encore pour controler le flot d'evenement du programme. Ces outils vous donne plus de pouvoir avec moins de code.

## Continuer votre découverte

Si vous pensez que Rx sera votre framework pour la Programmation Réactive, prennez le temps de mieux connaître [l'anorme liste de fonctions](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md) poru transformer, combiner, créer des `Observable`.
Si vous désirez comprendre ces fonctions avec des diagrammes de flux, jeter un oeil à [la très utile documentation RxJava avec ces diagrammes](https://github.com/Netflix/RxJava/wiki/Creating-Observables). Si jamais vous veniez à être bloqué, dessinez un diagramme, réfléchissez y, regarder la longue liste de fonctions, et réfléchissez encore. Cette méthode c'est révélée efficace pour mon apprentissage.

Dès que vous commencerez à comprendre la programmation avec Rx*, il est absolument vital de comprendre le concept de [Chaud Vs Froid `Obersvable`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables). Si vous ignorez cela, il vous reviendra en pleine figure. Vous avez était prévenu. Améliorez vos compétences en apprennant la vrai programmation fonctionnelle, et en comprenant les comportements spécifique qui affectent Rx*.

Cependant la programmation Réactive ce n'est pas seuelemnt Rx*. Il y à [Bacon.js](http://baconjs.github.io/) qui est très intuitif, sans les bizarreries de Rx*.  Le [langage Elm](http://elm-lang.org/) dans sa propre catégorie : Un **langage** de Programmation Réactive Fonctionnel qui est compilé en JavaScript + HTML + CSS, et apporte une fonctionnalité de [debuggage dans le temps](http://debug.elm-lang.org/). Impressionnant.

Rx fonctionne bien avec les application clientes qui utilisent beaucoup d'évènement. Mais ce n'est pas uniquement à réserver pour la partie client, il fonctionne très bien pour les backends et les bases de données. En fait, [RxJava est un composant clef pour permettre la programmation concurrente dans les API de NetFlix](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html). Rx n'est pas un framework pour un seul type d'application ou de langage. C'est un paradigme à part entiere que vous pouvez utilser des que des évènements sont en jeux.

Si ce tutoriel vous aide, [tweetez le](https://twitter.com/intent/tweet?original_referer=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754%2F&amp;text=The%20introduction%20to%20Reactive%20Programming%20you%27ve%20been%20missing&amp;tw_p=tweetbutton&amp;url=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754&amp;via=andrestaltz).

- - -

### Legal

© Andre Cesar de Souza Medeiros (alias "Andre Staltz"), 2014. Unauthorized use and/or duplication of this material without express and written permission from this site’s author and/or owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Andre Medeiros and http://andre.staltz.com with appropriate and specific direction to the original content.

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">"Introduction to Reactive Programming you've been missing"</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://andre.staltz.com" property="cc:attributionName" rel="cc:attributionURL">Andre Staltz</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.<br />Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://gist.github.com/staltz/868e7e9bc2a7b8c1f754" rel="dct:source">https://gist.github.com/staltz/868e7e9bc2a7b8c1f754</a>.
