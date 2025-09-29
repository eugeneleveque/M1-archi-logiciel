# Micro services

L'acrhitecture en micro services est un style d'architecture logicielle qui consiste à diviser une application en services autonomes, chacun exécutant un processus unique et communiquant souvent via des API. Chaque micro service est responsable d'une fonctionnalité spécifique de l'application et peut être développé et déployé indépendamment des autres services.

## Architecture en micro services VS architecture monolithique
![image](https://www.redhat.com/cms/managed-files/monolithic-vs-microservices.png)

## Caractéristiques et avantages des micro services
- Comme chaque micro service est indépendant, les développeurs ont la liberté de choisir la technologie et le langage qui conviennent le mieux à chaque service.
- Ces services indépendants n'ont aucun impact les uns sur les autres. Ainsi, lorsqu'un élément tombe en panne, l'ensemble de l'application ne cesse pas de fonctionner.
- Les cycles de développement sont plus courts, l'architecture de microservices permet des déploiements et mises à jour plus agiles.
- Vu que l'application est décomposée en plusieurs éléments, les équipes de développement peuvent plus facilement comprendre, mettre à jour et améliorer chacun de ces éléments.

## Inconvénients des micro services
- La gestion d'un grand nombre de services peut être complexe et peut nécessiter un outil de gestion global.
- Le débogage et le suivi des erreurs peuvent être plus difficiles pour les micro services.
- La mise en place de la communication entre les services peut être complexe et peut introduire des latences.
- La gestion des données peut être plus complexe, car chaque service peut avoir sa propre base de données.

## L'exemple de Netflix
En 2009, Netflix faisait face à des défis significatifs à cause de sa croissance rapide. L'infrastructure de la société ne parvenait pas à supporter la demande croissante pour ses services de streaming vidéo. Pour résoudre ce problème, Netflix a choisi de transformer son architecture monolithique en une architecture de microservices. À l'époque, le concept de « microservices » n'était pas encore bien établi.
Netflix a été l'une des premières grandes entreprises à réussir cette transition. Aujourd'hui, Netflix utilise plus de mille microservices pour gérer différentes parties de sa plateforme, et ses ingénieurs déploient fréquemment des mises à jour logicielles, parfois des milliers de fois par jour.

![image](https://walkthechat.com/wp-content/uploads/2019/05/netflix-microservices-diagram-bruce-wong.jpg)


## Sources
> [RedHat](https://www.redhat.com/fr/topics/microservices/what-are-microservices)
> [DataScientest](https://datascientest.com/microservices-tout-savoir)
> [TonuxDev](https://tonux-dev.com/microservices-netflix-uber/)