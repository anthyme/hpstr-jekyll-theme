---
layout: post
title:  "Pourquoi choisir Xamarin pour votre stratégie mobile"
excerpt: "Voici une présentation de Xamarin et de ce qui en fait un choix pertinent pour vos développements multiplateformes."
date:   2015-06-28 00:00:00
categories: post
tags : [Xamarin, C#]
lang : [fr]
image:
  feature: abstract-11.jpg
---

#Pourquoi choisir Xamarin pour votre stratégie mobile

##Introduction

Le marché des applications mobiles se développe énormément ces dernières années jusqu'au point où l'utilisation d'internet sur mobile est devenue supérieure à l'usage d'internet sur ordinateur.

![Internet mobile desktop market](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/internetmobiledesktopmarket.jpg)

Aujourd'hui les entreprises présentes sur le Web ont tout intérêt à se positionner sur le mobile si ce n'est pas déjà fait, sous peine de perdre une exposition importante à leurs utilisateurs.

Le problème est que si les technologies web ont fini, après bien des années,  par être standardisées pour avoir une approche de développement unique pour les différents navigateurs, le monde de la mobilité est encore extrêmement fragmenté par ses principaux acteurs. Décider d'une stratégie multiplateforme est un véritable défi.

##Approche WebView

Une des solutions pour arriver à réaliser une application avec un seul code source ou presque se fait grâce à une astuce : créer une application qui ne contient qu'un conteneur WebView affiché en plein écran et développer toute l'application en technologie Web (type Webapp).
Il existe pas mal de technologies de ce type, la plus connu étant surement Apache Cordova.
Si cette approche peut avoir du sens quand on veut réaliser des applications simples on reste loin d'une intégration native à la plateforme.

Ces approches offrent une expérience du plus petit dénominateur commun des différentes plateformes ciblées, le design n'est en général adapté qu'à une seule plateforme si ce n'est aucune, l'application essayant de paraitre native mais n'y arrivant que très rarement. 

Et plus on tente de paraitre natif, plus l'effort de développement est important et le gain sur les coûts par rapport à une application native est donc amoindri.
Il ne faut pas oublier qu'il y a encore de la fragmentation entre les différents navigateurs mobiles et que donc la promesse d'écriture unique du code n'est pas encore réelle.

![WebView blackbox](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/webviewblackbox.png)

##Approche native spécifique ou "silo"

L’approche native de chaque plateforme est le meilleur moyen d’avoir l’expérience utilisateur la plus cohérente, performante et efficace possible.
Néanmoins vous vous retrouvez avec des technologies différentes et donc des API différentes, des environnements de développement différents et des langages différents.
Il en résulte un projet multiplateformes avec des équipes différentes (une par silo), des bases de code différentes avec un décalage de cohérence d'implémentation et potentiellement fonctionnelle.
L'utilisation d'outils de développements différents rend difficile les ponts que l'on souhaiterait créer entre les plateformes.
Cette approche est bien plus qualitative que l'approche Webview mais est pleine de challenges.

![Native silo](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/nativesilo.png)

##Approche Xamarin

La meilleure approche pour cibler la mobilité étant de faire une application native Xamarin base tous ses fondamentaux sur le natif.
Xamarin permet d'avoir un support des API natives de chaque plateforme avec un Framework de développement unique (.Net) ce qui permet d'avoir toutes les implémentations réalisées dans un même langage et en utilisant les mêmes outils de développement.
La part de votre code qui n'est pas relative aux vues ou accès aux API natives est donc commune à toutes vos plateformes et garantit une cohérence entre les plateformes.
Au-delà de l'économie sur l'écriture du code partagé, avec ces fondamentaux il est donc beaucoup plus simple d'avoir une seule équipe de développement et donc d'économiser les coûts des ressources.
Bien souvent les API proposées par Xamarin sont adaptées à .Net et proposent une utilisation bien plus efficace que l’API officielle en utilisant les avantages de la plateforme .Net.
Il faut aussi garder à l'esprit que TOUT ce que vous pouvez faire en ObjectiveC/Swift ou Java peut être fait avec C# en Xamarin dans Visual Studio.

![Xamarin](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/xamarinform.png)

##Performances natives

L'intérêt de l'approche native est bien sûr d'avoir accès à tout ce qui est réalisable sur la plateforme, et d'avoir une cohérence ergonomique et graphique avec les autres applications de la plateforme.
Mais n'oublions pas un point, et pas des moindres : avoir des performances natives.

Avec Xamarin IOS le code source est compilé entièrement en AOT (Ahead Of Time) pour produire un binaire ARM pour l'Apple Store identique à un binaire créé avec de ObjectiveC dans Xcode.

Avec Xamarin Android on a une compilation en code intermédiaire puis une compilation JIT (Just In Time) à l'exécution avec la plateforme .Net telle que le fait la JVM sur les applications Android développées en Java.

![Performances](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/performances.jpg)

##C♯

C# est un langage moderne, simple, typé statiquement, multi usages, orienté objet et compatible programmation fonctionnelle. Bien que s'inspirant d'une syntaxe classique, il a reçu tout au long de sa vie des ajouts réguliers de fonctionnalités avancées comme par exemple Linq comme mécanisme de requête au niveau syntaxique ou encore les async/await permettant de créer du code asynchrone avec la simplicité d'écriture du code séquentiel, ce qui est vraiment un gros plus pour le développement d'applications mobiles. Autour de ce langage s'est construit une grande communauté, des outils avancés et des bibliothèques éprouvées.
En quelques mots C# est vraiment un excellent langage pour le développement multiplateforme.

![C#](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/csharp.png)

##Plus de partages de code avec Xamarin Forms

L'approche Xamarin permet un partage de code important estimé à 70% avec son approche classique ce qui est déjà un très bon point mais on peut aller encore plus loin avec Xamarin Forms.
Xamarin Forms est une approche qui permet d'écrire sa couche UI et applicative en une seule technologie : XAML. Ce qui permet de monter le ratio de partage de code à un niveau compris entre 90 et 99%.
Seuls les accès aux fonctionnalités spécifiques à la plateforme sont à implémenter dans chaque plateforme.
Cette approche est une abstraction de l'approche native et encapsule des contrôles natifs. On garde donc de la cohérence UI par rapport à la plateforme contrairement à l’approche WebView.
Aujourd'hui plus de 40 Pages, Layouts, et Contrôles sont disponibles. Le framework est très moderne, très productif avec une approche de DataBinding et MVVM similaire à ce que l'on trouve dans WinRT. Il incorpore aussi des patterns de navigation, une API d'animation, une IoC et un messenger.

![Xamarin Forms](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/xamarinforms.jpg)

##Support Day 0

Un autre point important sur ce type de technologie, qui s'appuie sur des technologies et plateformes en perpétuelle mutation, est le support associé.
Sur ce sujet, Xamarin est extrêmement réactif sur la livraison de mise à jour et supporte les SDK le jour de leur sortie.
Il s'agit aussi d'un support plus large. Au-delà des plateformes mobiles IOS et Android, Xamarin supporte aussi Google Glass, Android Wear, Apple Watch, Amazon Phone/TV et bien d'autres.

![2,5 milliard de dispositifs](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/3milliarsdispositifs.jpg)

##Des outils All stars

Xamarin intègre aussi des outils de haut niveau pour la productivité et l'industrialisation de vos développements.
On a tout d'abord Xamarin Studio sous Mac et Windows qui est un IDE de bonne qualité et qui est simplement le meilleur éditeur C# sous Mac.
Xamarin nous fournit un émulateur Android de très bonne qualité qui, à sa sortie, était l'émulateur le plus rapide du marché (largement devant les émulateurs fournis par Google).
On trouve Xamarin Insights qui vous permet de traquer les crashs de votre application mais aussi de tracer l'utilisation que les utilisateurs font de votre application.
Dernier outil en date, et non des moindres, Xamarin Test Cloud. Une plateforme cloud permettant d'exécuter des tests applicatifs automatisés (unitaires ou UI) directement sur un millier de modèles de téléphones différents.

![Xamarin Insights](https://dl.dropboxusercontent.com/u/44389563/blog/code/why-xamarin-strategy-mobile/xamarininsights.png)

##Conclusion

Par cet article, j'ai voulu vous transmettre les principaux éléments qui ont construit ma conviction que Xamarin est l'approche la plus qualitative du développement multi-plateformes.
Si on peut se questionner sur la pérennité de l'approche Xamarin il faut savoir que l'entreprise a aujourd'hui plus de 4 ans, dispose de 50 employés et est largement autosuffisante. On a aussi une communauté de plus d'un million de développeurs qui croît continuellement.
Si vous envisagez un projet d'applications cross plateformes, je ne saurais trop vous recommander d'étudier avec soin l'approche Xamarin.