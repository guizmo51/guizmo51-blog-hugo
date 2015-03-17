+++
date = "2015-03-16T14:24:31+01:00"
draft = false
title = "boostAdopt"

+++

Tout ce projet s'articule autour du site AdopteUnMec.com . Il s'agit d'un site de rencontre Français lancé en 2008. Il a fait le buzz avec un système de charme auxquels répondent les femmes qui ont la possibilité de lancer la conversation. D'un point de vue business model le site en a semble t-il eu plusieurs. Tout le site était gratuit au départ, puis a lancé des systèmes d'abonnements ou de barrière horaires  (accès impossible entrer 18h et 1h pour les abonnés non-payants). Je ne suis pas radin mais payer minimum 30€ par mois pour ce genre de service : sans moi.

En 2012 je suis parti d'un constat : sur X profils de femmes visités si ta page est intéressante un certain pourcentage reviendra automatiquement et une partie t'autoriseras à lui écrire. J'ai donc souhaité industrialiser tout ce process. A quoi bon cliquer manuellement alors qu'un robot / script peut le faire pour vous ?

Dans un premier temps j'avais fait un vulgaire crawler qui utilisait la partie Desktop du site. Je simulais donc une recherche de 50 ou 100 profils selon des critères déterminés puis les visiter à une certaine fréquence. Tout ce procédé était long mais fonctionnait. L'application parsait le DOM HTML et avec un système de modèles resortait les données utiles.

Un jour j'étais frustré de ne pouvoir utiliser mon script entre 18h et 1h du matin (le fameux blocage pour les comptes gratuits). Toutefois je me suis rendu compte que l'application Android / iphone pouvait se connecter pendant ce créneau. 


## API mobile
J'ai donc décidé d'écouter les appels effectués par l'application Android. En quelques minutes et avec l'aide de Wireshark. En quelques minutes j'ai récupéré une série d'URL.

Exemples d'url :
"http://adopteunmec.com/api/home" "connexion"
"http://adopteunmec.com/api/users" "recherche"
"http://adopteunmec.com/api/users/XXXXX" "visite du profil XXXXX"
L'API permet aussi de charmer et d'envoyer des messages (possible si on a au préalable été autorisé).

Tout le projet initial a donc été converti en utilisant du JSON comme données en entrées. Cela rendait l'application plus rapide puisqu'au lieu de charger des pages HTML de plusieurs Mo l'appel je chargeais du json de quelques Ko.

désormais tout fonctionnait : un script php faisait tout le boulot.

Puis un jour une alert mail automatisée m'informe que les scripts ne fonctionnent plus. Après quelques minutes de recherche je me rends compte que le serveur a été blacklisté, il n'est plus autorisé à accèder aux serveurs d'AdopteUnMec. A ce stade j'avais atteint à peu près 100 000 requetes chez eux. 

Il a donc fallu réfléchir à de nouvelles solutions : 
- racheter de nouvelles IP une fois ce pallier de requetes atteint : faisable mais cela a un cout non négligeable 
- utiliser Tor mais je ne souhaitais pas exposer les données sensibles à travers de noeuds corrompus.
- trouver une solution client en javascript.

J'ai opté pour la dernière solution : réaliser un client Javascript qui se connecterait directement aux serveurs d'AdopteUnMec. Cela metterait le code métier exposé à tout le monde mais l'IP se connectant aux serveurs serait celle de l'utilisateur et non des serveurs.

Pendant de long mois j'ai cherché les solutions permettant d'effectuer des appels Ajax vers les serveurs d'AdopteUnMec. 
Bien sur la première étape est le cross-origin : c'est une sécurité implémentée dans chaque navigateur. Elle s'assure qu'un navigateur ne se connecte pas à un serveur que ce dernier n'a pas autorisé. Cette sécurité est contournable en modifiant les options de chaque navigateur mais pas très confortable pour l'utilisateur. 
Une autre solution envisagée était de faire une requete en [JSONP pour JSON with Padding](http://en.wikipedia.org/wiki/JSONP). Cela est faisable ainsi grace au fait que le navigateur ne controle pas le "same-origin" pour les scripts. J'ai renoncé à utiliser cette technique car elle nécessite de rajouter le padding sur les données coté serveur ce que je ne peux bien sur pas faire.
j'ai trouvé la oslution finale par hasard : je me suis aperçu que des extensions Chrome pouvaient faire des appels à des serveurs sans déclencher ce problème de "cross-origin". Il faut toutefois que les urls des serveurs soient déclarées dans le manifest de l'extension. J'ai donc decidé de faire une extension Chrome pour effectuer tous les appels aux serveurs d'AdopteUnmec.

Bien que cette solution ne soit pas idéale car elle oblige à utiliser Chrome j'ai décidé de l'expérimenter pour lancer mon projet boostAdopt. 

## Extension Chrome.
l'objectif de l'extension Chrome est de faire les appels Cross-Origin sur les serveurs d'AdopteUnMec. Il a donc fallu faire communiquer le Javascript de mon application avec cette extension Chrome. A l'origine je pensais que des méthodes simples existaient pour la communication entre extension et le javascript de l'application. Ce ne fut pas le cas. Lorsque j'envoyais un message depuis le Javascript vers l'extension ou inversement ce message était re-capturé par l'émetteur ainsi que le destinataire. Suite à ca j'ai donc mis en place un système de flag mentionnant l'emetteur du message. Ensuite si l'émetteur n'était pas le destinataire celui-ci était traité. Afin de rendre l'extension disponible plus facilement je l'ai publié sur le Google Store en ayant un compte développeur. La création de ce dernier a un couut de quelques dollars.

## Le front
Le front est la partie visible de l'iceberg. C'est l'applicatiojn en elle-meme et elle doit ^etre fonctionnelle et intuitive. Ce n'est pas un effet de mode mais j'ai souhaité utiliser AngularJS que je ne maitrisais pas à l'époque pour réaliser ce front. J'ai été agréablement surprise par ce framework JS. N'ayant que très peu de qualité dans tout ce qui est UI / Design je suis parti sur un Bootstrap d'autant plus qu'une bliblothèque fait cela très bien (http://angular-ui.github.io/bootstrap/). 


## Le back-end 
Tout le back-end de l'application est développé en utilisant CakePHP 2 qu est un petit framework tout à fait adapté à ce genre de projets. Cette partie du projet était assez light mais suffisament sécurisée ayant à gérer des comptes et paypal. En effet la monétisation du projet passait par l'utilisation d'un compte Paypal 




## Business model
Je souhaitais monétiser cette application : j'ai donc mis en place un système de crédit. Un profil visité = un crédit en moins, reste en suite à définir et adapter la valeur du crédit. 

## Communication
Etant fan de développement de robots en tous genres pour la partie communication j'avais décidé de faire un robot crée sous un compte féminin et envoyant des messages invitant essayer le projet boostAdopt.
Plusieurs freins ont fait diminuer l'impact de la communication sur ce projet :
- la messagerie AdopteUnMec ne permet pas de poster des liens, l'utilisateur était donc obligé de copier-coller l'URL.
- la personnalisation du message, bien que j'utilisais un message du style "Bonjour Benoit," le message pouvait apparaitre comme du spam.
J'ai aussi fait quelques tests de robots permettant d'imiter le comportement humain. Ce robot engageait une conversation avant de suggérer un lien vers boostAdopt.

D'autres prospects ont aussi fait remonter leurs craintes concernant la sécurité. Ils pensaient que le site avait pour unique objectif de leurs voler leurs identifiants.

Une vidéo avait été réalisée permettant de visualiser toutes les fonctionnalités de l'application.

Enfin : un système de parrainage avait été mis en place. En partageant le lien le parrain recevait des crédits à chaque inscription d'un filleul.

## Faille XSS 
J'avais découvert une faille XSS sur le mini-chat utilisé par AdopteUnMec. 
En effet lors de tests qui m'auraient permis de diffuser l'URL de boostAdopt de façon cliquable je me suis aperçu que le Javascript de la chatbox n'était pas filtré. Lors de mes premiers tests j'ai pu de manière très simple faire afficher une alert box à chaque ouverte du chat. Ce javascript était bel et bien exécuté que ça soit coté auteur mais aussi destinataire du message. Un beau jour cette faille avait été corrigée. Une exploitation possible aurait pu etre la réalisation d'un message se dupliquant de contacts en contacts.

## Securité du site
Lors que le compte auteur d'un message était classé "spam" il n'était plus possible d'utiliser ce compte. Autre conséquence assez facheuse : chaque compte s'inscrivant ou se connectant avec l'IP utilisée par ce compte "span" était automatiquement blacklisté. Cela a rendu compliqué la communication à travers le chat. Un des solutions a été d'utiliser Tor et de programmer un refresh d'IP fréquent et l'utilisation multiple de comptes emails pour s'inscrire au site AdopteUnMec et pouvoir continuer à diffuser des messages.

## Fin du projet
Le projet a souffert de plusieurs choses :
AdopteUnMec était devenu totalement payant, un utilisateur aurait du payer quelques euros supplémentaires par mois en plus d'un minimum de 30€. 
J'ai reçu une mise en demeure (bien que le site n'était pes en France)  et d'autres éléments m'ont poussé à stopper ce projet boostAdopt.
