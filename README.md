1- Bonjour, je vais vous présenter le cadrage du projet Plumora, une plateforme destinée aux auteurs, lecteurs et bêta-lecteurs

2- Pour structurer cette présentation, je vais d’abord présenter le contexte et les objectifs du projet, puis la cartographie des acteurs impliqués. Ensuite, je détaillerai l’audit, les risques, la veille et les choix techniques. Je terminerai par l’architecture proposée, le chiffrage et les préconisations.

3- Le besoin part d’un constat simple : les auteurs indépendants utilisent souvent plusieurs outils séparés pour écrire, faire relire, corriger et publier leurs livres. Plumora vise à centraliser ce parcours dans une plateforme unique.

La valeur ajoutée de Plumora est de réunir création littéraire, bêta-lecture, publication et intelligence artificielle dans une seule expérience mobile et desktop.

L’objectif n’est donc pas d’ajouter de l’IA pour suivre une tendance, mais de l’utiliser pour répondre à deux besoins concrets : améliorer l’écriture et améliorer la découverte de livres.

C’est dans ce contexte que se pose la problématique suivante :

4- Cette slide distingue les objectifs fonctionnels et techniques de Plumora.
Côté fonctionnel, l’objectif est de permettre aux auteurs de créer et organiser leurs livres, de proposer une bêta-lecture avant publication, d’offrir aux lecteurs un catalogue varié et d’intégrer une IA utile mais contrôlée.
Côté technique, le projet doit reposer sur une API REST sécurisée, une architecture maintenable et testable, une dockerisation pour faciliter le déploiement et une application Flutter mobile et desktop.
Ces objectifs permettent de répondre au besoin tout en préparant une solution évolutive.


Les objectifs fonctionnels correspondent à ce que la plateforme doit apporter aux utilisateurs.


Les objectifs techniques concernent la manière dont la solution sera construite.


5- Dans le périmètre initial, on retrouve les fonctionnalités principales tels que .......


Le paiement et la monétisation sont exclus du périmètre initial afin de maîtriser la charge, les risques et les délais.

6- Cette slide présente la cartographie des acteurs impliqués dans le projet Plumora.
L’objectif est d’identifier les parties prenantes, leurs rôles et leur niveau d’implication dans le projet.

Au centre, on retrouve Plumora, qui représente la plateforme multi-supports.
Autour de la plateforme, j’ai distingué trois niveaux d’implication.

Le premier niveau correspond à l’implication forte, représentée en rouge.
Il s’agit de la MOA, c’est-à-dire la maîtrise d’ouvrage, qui formalise le besoin, définit la vision du projet, valide le périmètre et suit l’avancement.
On retrouve aussi la MOE, la maîtrise d’œuvre, qui conçoit l’architecture, développe l’application, teste et déploie la solution.
Ces acteurs sont essentiels à la réussite du projet.

Le deuxième niveau correspond à l’implication moyenne, représentée en vert.
Il s’agit des utilisateurs finaux : les auteurs, les lecteurs, les bêta-lecteurs et les administrateurs.
Ils sont importants car leurs besoins orientent directement les fonctionnalités de Plumora. Par exemple, l’auteur écrit et publie ses œuvres, le lecteur lit les livres et donne son avis, le bêta-lecteur teste les manuscrits, et l’administrateur gère les signalements et modère les contenus.

Le troisième niveau correspond à l’implication indirecte, représentée en bleu.
Il regroupe les acteurs externes qui apportent des services ou des ressources au projet : Ollama pour l’IA Plumo, Gutendex pour enrichir le catalogue avec des livres du domaine public, Open Library pour les métadonnées et les couvertures, et l’hébergeur pour l’infrastructure, la sécurité et la disponibilité.

Cette cartographie permet donc de clarifier l’environnement du projet, de visualiser les responsabilités de chaque acteur et de montrer comment chacun contribue à la réussite de Plumora.


Version courte à mémoriser :

Cette cartographie permet d’identifier les acteurs du projet, leurs rôles et leur niveau d’implication.
Les acteurs en rouge, MOA et MOE, ont une implication forte car ils portent la vision, la conception, le développement et la validation du projet.
Les acteurs en vert sont les utilisateurs finaux : auteurs, lecteurs, bêta-lecteurs et administrateurs. Ils orientent les fonctionnalités de Plumora.
Les acteurs en bleu sont les partenaires externes : Ollama, Gutendex, Open Library et l’hébergeur. Ils apportent des services ou des ressources nécessaires au fonctionnement de la plateforme.


7- Cette slide montre les besoins principaux des utilisateurs de Plumora.
L’auteur a besoin d’écrire, structurer et publier. Le lecteur veut découvrir et lire facilement des livres adaptés à ses envies. Le bêta-lecteur doit pouvoir commenter précisément un manuscrit. L’administrateur doit gérer les comptes, les signalements et l’historique.
Ces besoins servent ensuite de base pour définir le périmètre fonctionnel du MVP.


8- Cette slide présente l’analyse SWOT du projet Plumora.
L’objectif est d’identifier les forces, les faiblesses, les opportunités et les menaces afin de mieux cadrer le projet et d’anticiper les points de vigilance.

Dans les forces, Plumora propose une plateforme complète autour du livre, avec une application mobile et desktop. L’intégration de l’IA apporte aussi une vraie valeur ajoutée, car elle peut aider à l’écriture et à la recommandation, tout en restant encadrée.

Concernant les faiblesses, le périmètre fonctionnel est assez large. Le projet demande donc une bonne priorisation. Il existe aussi une complexité de synchronisation entre le frontend Flutter et le backend Spring Boot. Enfin, l’IA crée une dépendance à un fournisseur ou à un service IA.

Du côté des opportunités, le projet s’inscrit dans le développement de l’autoédition. Il répond aussi à un besoin de bêta-lecture structurée et de recommandations personnalisées pour les lecteurs.

Pour les menaces, il faut surveiller la concurrence, les risques juridiques liés aux contenus externes, ainsi que l’indisponibilité possible des API externes comme Gutendex ou Open Library.

En bas de la slide, j’ai ajouté les points de vigilance à suivre pendant le projet : la sécurité des données, les droits d’auteur, la performance sur les textes longs, la maîtrise du périmètre et la qualité de l’expérience utilisateur.

Cette analyse me permet donc de transformer les risques et opportunités en actions concrètes à surveiller pendant le développement.


9- Cette slide présente la démarche d’audit et le diagnostic de l’existant réalisés pour le projet Plumora.
L’objectif est de comprendre le contexte de départ, les contraintes du projet et les éléments à prendre en compte avant de valider la solution.

J’ai séparé l’audit en deux parties : une partie fonctionnelle et une partie technique.


Cette démarche d’audit permet donc de justifier les choix techniques et fonctionnels retenus pour le projet.










Version courte :

Cette slide présente l’audit réalisé avant de définir la solution.
Sur le plan fonctionnel, j’ai identifié les rôles, les parcours utilisateurs et les fonctionnalités prioritaires du MVP.
Sur le plan technique, j’ai analysé les plateformes cibles, l’architecture nécessaire et les contraintes de sécurité.
Le diagnostic montre que Plumora part d’un besoin nouveau, sans application existante. Il faut donc prévoir une base PostgreSQL solide, une IA contrôlée et un déploiement reproductible avec Docker.
Cette analyse permet de valider la faisabilité du projet et de justifier les choix techniques.


10- Cette slide présente la cartographie des risques techniques et fonctionnels du projet Plumora.
L’objectif est de ne pas seulement lister les risques, mais de les prioriser selon leur probabilité, leur impact et leur criticité.

Pour chaque risque, j’ai indiqué son type, sa probabilité de réalisation, son impact, un score de criticité, une mesure de maîtrise et un indicateur de contrôle.
Par exemple, la fuite de données utilisateur et la visibilité publique d’un livre privé sont des risques élevés, car ils touchent directement à la sécurité, à la confidentialité et à la confiance des utilisateurs.

Le risque le plus important est le périmètre trop large, avec un score de 16. C’est pour cela que j’ai choisi de cadrer Plumora sous forme de MVP, afin de limiter les fonctionnalités au départ et éviter une dérive du projet.

Les graphiques en bas permettent de visualiser rapidement la répartition des risques par criticité et les cinq risques les plus importants.
On voit que les risques élevés sont prioritaires et doivent être suivis avec des indicateurs comme les alertes de sécurité, les tests d’accès, le suivi du MVP, ou encore la disponibilité des API externes.

Cette matrice permet donc de piloter les risques pendant le projet et de prévoir des actions de maîtrise avant qu’ils n’impactent le développement.


Version courte :

Cette slide présente les risques techniques et fonctionnels de Plumora. Chaque risque est évalué selon sa probabilité, son impact et sa criticité. Les risques les plus importants concernent le périmètre trop large, la sécurité des données, la visibilité des livres privés et la conformité juridique. Les mesures de maîtrise et les indicateurs permettent de suivre ces risques pendant le développement.


11- Cette slide présente le référentiel de suivi des risques et incidents prévu pour Plumora.
Après avoir identifié les risques dans la matrice précédente, l’objectif ici est d’expliquer comment ces risques seront suivis et traités pendant le projet.

L’outil choisi est GitHub Issues ou GitHub Projects, car il permet de centraliser les bugs, les incidents, les risques et les actions correctives dans le même environnement que le code source.

Le cycle de traitement est organisé en cinq étapes.
D’abord, le risque ou l’incident est identifié. Ensuite, il passe en analyse pour comprendre sa cause, son impact et la solution possible. Puis il est mis en cours lorsqu’une action corrective est lancée. Une fois la correction réalisée, il passe au statut corrigé. Enfin, il est validé après test ou vérification.

En bas de la slide, les indicateurs de contrôle permettent de suivre l’état du projet.
Par exemple, le nombre de bugs critiques ouverts permet de savoir s’il reste des anomalies bloquantes. Le taux d’erreurs API et le temps de réponse moyen permettent de surveiller la stabilité technique. La couverture des tests backend permet de contrôler la qualité du code. Les incidents de sécurité et les livres importés avec source vérifiée permettent de suivre les risques liés à la sécurité et aux droits.

Cette démarche permet donc de ne pas simplement lister les risques, mais de les suivre jusqu’à leur résolution.




Version courte :

Cette slide montre comment les risques et incidents seront suivis.
J’utilise GitHub Issues ou GitHub Projects pour centraliser les anomalies, les bugs et les actions correctives.
Chaque incident suit un cycle : identifié, en analyse, en cours, corrigé puis validé.
Les indicateurs comme les bugs critiques ouverts, le taux d’erreurs API, le temps de réponse moyen, la couverture des tests ou les incidents sécurité permettent de contrôler l’impact des risques pendant le projet.


12- Cette slide présente les ressources techniques nécessaires à Plumora. Spring Boot sert à développer l’API REST sécurisée, PostgreSQL stocke les données, Flyway gère les migrations de base, et Docker facilite le déploiement.
Flutter permet de créer l’application mobile et desktop, tandis que Figma sert à concevoir les maquettes.
Ollama est utilisé pour l’IA Plumo, et Gutendex avec Open Library permettent d’enrichir le catalogue avec des livres du domaine public et leurs couvertures.


13- Voici un texte prêt à dire :

Cette slide présente l’étude comparative des solutions techniques envisagées pour Plumora.
L’objectif est de comparer plusieurs options pour le backend, le frontend et l’IA, puis de justifier les choix retenus.

Pour le backend, j’ai comparé Spring Boot, NestJS et Django.
Spring Boot a été retenu parce qu’il est robuste, mature, sécurisé et bien adapté à la création d’API REST. Il est aussi cohérent avec mon expertise Java et avec les besoins de sécurité du projet.
NestJS et Django sont de bonnes solutions, mais elles sont moins alignées avec l’écosystème Java choisi pour Plumora.

Pour le frontend, j’ai comparé Flutter, React Native et Angular Web.
Flutter a été retenu parce qu’il permet de développer une application mobile et desktop avec une base de code commune. C’est important pour Plumora, car l’auteur peut avoir besoin d’écrire sur desktop, tandis que le lecteur peut utiliser principalement le mobile.
React Native est très bon pour le mobile, mais moins naturel pour le desktop. Angular est très adapté au web, mais moins pertinent pour une application mobile native.

Pour l’IA, j’ai comparé Ollama local, OpenAI API et un Fake Provider.
Ollama est retenu pour le MVP parce qu’il est local, gratuit, démontrable et permet de garder la maîtrise des données.
L’API OpenAI reste une évolution possible pour une future mise en production plus scalable.
Le Fake Provider peut être utilisé en test pour simuler des réponses IA sans dépendre d’un vrai modèle.

Cette comparaison permet donc de justifier une stack cohérente pour Plumora : Spring Boot pour le backend, Flutter pour le frontend, et Ollama pour l’IA du MVP.



Version courte :

Cette slide justifie les choix techniques de Plumora.
Spring Boot est retenu pour le backend, car il est robuste, sécurisé et adapté aux API REST.
Flutter est retenu pour le frontend, car il permet de viser mobile et desktop avec un code partagé.
Ollama est retenu pour l’IA du MVP, car il fonctionne localement, sans coût API, et permet une démonstration maîtrisée.
OpenAI API reste une évolution possible pour une future version en production.


14- Voici un texte prêt à dire :

Cette slide présente la veille technique réalisée pour le projet Plumora.
L’objectif de cette veille était d’identifier les technologies les plus adaptées, de limiter les risques techniques et d’anticiper les contraintes de sécurité.

Pour cela, je me suis appuyé principalement sur des sources fiables, notamment les documentations officielles de Spring Boot, Spring Security, Flutter, Ollama, Gutendex et Open Library Covers API.
J’ai aussi consulté des références importantes comme l’OWASP Top 10 pour la sécurité, ainsi que le RGAA et Opquast pour les bonnes pratiques d’accessibilité.

Les outils utilisés pour cette veille sont la documentation officielle, GitHub, Stack Overflow avec vérification, la veille réglementaire Google Play et les retours d’expérience de développeurs.
L’idée était de ne pas prendre une décision uniquement sur une seule source, mais de croiser les informations.

Cette veille a permis de confirmer plusieurs choix techniques : Spring Boot pour la robustesse du backend, Flutter pour le développement mobile et desktop, Ollama pour une IA locale au démarrage, et Gutendex avec Open Library pour enrichir le catalogue de livres.

Cette démarche montre que les choix techniques de Plumora sont argumentés, documentés et cohérents avec les besoins du projet.

Version courte :

Cette slide présente ma démarche de veille technique.
J’ai utilisé des sources fiables comme les documentations officielles Spring Boot, Flutter, Ollama, Gutendex et Open Library, ainsi que l’OWASP Top 10 et les référentiels RGAA / Opquast.
Cette veille m’a permis de sécuriser mes choix techniques : Spring Boot pour le backend, Flutter pour le multi-support, Ollama pour l’IA locale, et Gutendex / Open Library pour enrichir le catalogue.

Petite correction visuelle à faire dans ta slide : remplace “Identifier les thcnologies adaptées” par “Identifier les technologies adaptées”.


15- Réflexion durant 18s

Tu peux présenter cette slide comme l’architecture cible globale de Plumora, c’est-à-dire la manière dont les différents composants techniques communiquent entre eux.

Voici un texte prêt à dire :

Cette slide présente l’architecture préconisée pour Plumora.
L’objectif est de montrer comment le frontend, le backend, la base de données, l’IA et les sources externes interagissent dans le système.

À gauche, on retrouve l’application Flutter. Elle représente l’interface utilisateur mobile et desktop. Elle communique avec le backend via des requêtes HTTP sécurisées, au format JSON.

Au centre, le backend est développé avec Spring Boot sous forme d’API REST. Cette API reçoit les requêtes du frontend, expose les endpoints et transmet les traitements aux services applicatifs.

Les services applicatifs contiennent la logique métier de Plumora : gestion des utilisateurs, livres, chapitres, lecture, bêta-lecture, recommandations et intégration de l’IA.

Les données sont stockées dans une base PostgreSQL. Les échanges avec la base passent par le backend, ce qui permet de centraliser les règles de sécurité et d’éviter que le frontend accède directement aux données.

En bas, on retrouve les sources externes : Gutendex et Open Library. Elles permettent d’enrichir le catalogue avec des livres du domaine public, des métadonnées et des couvertures. Ces imports sont contrôlés côté backend.

Enfin, l’IA Plumo repose sur Ollama en local pour le MVP. Flutter n’appelle jamais directement Ollama : l’appel passe par Spring Boot, ce qui permet de maîtriser les échanges, de sécuriser l’intégration et de pouvoir remplacer Ollama plus tard par un autre fournisseur IA si nécessaire.

Cette architecture est donc cohérente avec les objectifs du projet : elle est sécurisée, maintenable, évolutive et adaptée à une application multi-supports.



Version courte :

Cette architecture montre le fonctionnement global de Plumora.
L’application Flutter communique avec une API REST Spring Boot en JSON. Le backend orchestre les services applicatifs, applique les règles métier et échange avec PostgreSQL. Les imports Gutendex et Open Library sont contrôlés côté backend. L’IA Plumo utilise Ollama localement pour le MVP.
L’intérêt de cette architecture est de séparer les responsabilités, sécuriser les données et préparer les évolutions futures du projet.

16- Texte à dire à l’oral

Cette slide présente l’architecture backend de Plumora.
J’ai choisi une architecture en couches, inspirée de la Clean Architecture, afin de séparer clairement les responsabilités et de rendre le projet plus maintenable, testable et évolutif.

Le frontend Flutter communique avec le backend via HTTPS en JSON. Le frontend ne communique pas directement avec la base de données, ni avec Ollama, ni avec les API externes. Tout passe par le backend.

La première couche est la couche presentation. Elle contient les contrôleurs REST et les DTO. Son rôle est de recevoir les requêtes HTTP, valider les données d’entrée et retourner les réponses JSON.

La deuxième couche est la couche application. Elle contient les services et les cas d’usage. C’est ici que sont orchestrées les actions comme créer un livre, publier un livre, importer des livres ou demander une aide IA.

La troisième couche est la couche domain. Elle représente le cœur métier de Plumora avec les entités, les enums et les règles métier. Par exemple, un livre archivé ne doit plus être modifiable, ou un livre doit avoir au moins un chapitre pour être publié.

Enfin, la couche infrastructure regroupe les éléments techniques : les repositories PostgreSQL, les appels aux API externes Gutendex et Open Library, ainsi que le provider IA Ollama.

L’intérêt de cette architecture est d’isoler les dépendances techniques. Par exemple, si plus tard je remplace Ollama par OpenAI API, je peux le faire dans la couche infrastructure sans modifier toute la logique métier.

Cette organisation permet donc d’avoir un backend plus sécurisé, plus maintenable et plus facile à faire évoluer.

Version courte

Cette slide présente l’architecture backend de Plumora.
Le backend est organisé en quatre couches : presentation, application, domain et infrastructure.
La couche presentation expose les contrôleurs REST et les DTO. La couche application orchestre les cas d’usage. La couche domain contient les règles métier. La couche infrastructure gère PostgreSQL, les API externes et l’IA Ollama.
Cette séparation permet de sécuriser les échanges, de faciliter les tests et de rendre le projet évolutif.
17- L’estimation totale est de 61 jours-homme.
Cela signifie 61 jours de travail cumulés, répartis entre plusieurs profils : chef de projet, architecte, UI/UX designer, développeur backend, développeur front Flutter, développeur IA, QA/testeur et DevOps.

Les phases prises en compte sont le cadrage, la conception, le développement, l’intégration, les tests et recette, le déploiement et le pilotage.

On voit que la phase la plus chargée est le développement, avec 25 jours-homme. C’est logique, car c’est la phase où sont développées les fonctionnalités principales : authentification, gestion des livres, chapitres, catalogue, bêta-lecture, IA Plumo et frontend Flutter.

Le rôle le plus sollicité est le développeur backend, avec 21 jours-homme. Cela s’explique par le fait que le backend porte la logique métier, la sécurité, les API REST, la gestion des données et les intégrations avec les services externes.

Cette estimation reste une estimation macro, adaptée à un MVP en contexte équipe. Elle permet de cadrer l’effort nécessaire, d’identifier les phases les plus importantes et de préparer ensuite le budget prévisionnel.

Version courte à mémoriser :

Cette slide présente l’estimation de charge du MVP Plumora.
La charge totale est estimée à 61 jours-homme, répartis par rôle et par phase projet.
La phase la plus chargée est le développement avec 25 jours-homme, et le rôle le plus sollicité est le développeur backend avec 21 jours-homme.
Cette estimation permet de justifier l’effort nécessaire et sert de base au budget prévisionnel.

18- Tu peux présenter cette slide comme le passage entre charge de travail et budget prévisionnel.

Voici un texte prêt à dire :

Cette slide présente le budget prévisionnel du MVP Plumora.
Le calcul est basé sur l’estimation de charge précédente, soit 61 jours-homme.

Pour rendre le budget plus réaliste, j’ai réparti la charge par rôle : chef de projet, architecte, UI/UX designer, développeur backend, développeur Flutter, développeur IA, QA/testeur et DevOps.

Chaque rôle possède un TJM différent, car tous les profils n’ont pas le même niveau d’expertise ni le même coût journalier.
Par exemple, l’architecte et le développeur IA ont un TJM plus élevé, car leurs missions demandent une expertise plus spécialisée. Le QA/testeur a un TJM plus bas, car son coût journalier est généralement moins élevé.

Le coût de développement est calculé simplement avec la formule :
charge en jours-homme × TJM du rôle.

Le coût total de développement du MVP est donc estimé à 29 600 €.

En dessous, j’ai ajouté les coûts techniques prévisionnels. Ils comprennent l’hébergement backend avec la base PostgreSQL, le nom de domaine, le compte Google Play Developer, les outils de monitoring et de stockage, ainsi qu’une marge d’imprévus.

Les coûts techniques sont estimés à 1 940 €.
Le budget total prévisionnel du projet est donc de 31 540 €, en additionnant le développement et les coûts techniques.

Version courte :

Cette slide présente le budget prévisionnel de Plumora.
Il est calculé à partir de la charge estimée de 61 jours-homme, répartie par rôle avec des TJM différenciés.
Le coût de développement est estimé à 29 600 €.
À cela s’ajoutent les coûts techniques : hébergement, nom de domaine, compte Google Play, monitoring, stockage et marge d’imprévus, pour 1 940 €.
Le budget total prévisionnel du MVP est donc de 31 540 €.

Petite correction à faire sur ta slide : dans le tableau du haut, écris “MOA / Chef de projet” au lieu de “MOA / C hef de projet”.

19- Pour conclure ce cadrage, je présente maintenant les axes de solution préconisés et la décision proposée au client.


Voici un texte prêt à dire :

Cette slide présente les axes de solution préconisés pour Plumora.
Après l’analyse du besoin et des contraintes du projet, je recommande de lancer Plumora sous forme de MVP, avec une architecture moderne, sécurisée et évolutive.

Le premier axe est l’architecture évolutive.
L’objectif est de séparer clairement le frontend Flutter, le backend Spring Boot, la base PostgreSQL, Docker et les services externes. Cette séparation permet de faciliter la maintenance, les tests et les évolutions futures.

Le deuxième axe est la plateforme multi-rôles.
Plumora doit répondre aux besoins de plusieurs profils : auteurs, lecteurs, bêta-lecteurs et administrateurs. Chaque rôle dispose d’un parcours adapté à son usage.

Le troisième axe est l’IA maîtrisée.
Plumo accompagne l’auteur dans l’écriture et aide à la recommandation de livres, mais l’IA ne modifie pas automatiquement les contenus. L’utilisateur garde toujours le contrôle.

Le quatrième axe est le catalogue enrichi.
Plumora ne proposera pas seulement les livres créés sur la plateforme. Le catalogue sera aussi enrichi avec des livres du domaine public, ce qui permet d’offrir plus de contenu aux lecteurs dès le démarrage.

Ces axes répondent directement à la problématique du projet : concevoir une plateforme multi-supports intégrant l’IA pour accompagner l’écriture, la publication et la découverte personnalisée de livres.





Version courte :

Cette slide présente la solution préconisée pour Plumora.
Je recommande de lancer le projet sous forme de MVP, avec quatre axes principaux : une architecture évolutive, une plateforme multi-rôles, une IA maîtrisée et un catalogue enrichi.
L’objectif est de proposer une première version réaliste, sécurisée et capable d’évoluer progressivement






