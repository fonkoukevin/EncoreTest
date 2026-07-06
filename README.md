Le projet me semble solide — et l'AGENTS.md que tu as posé montre que tu as déjà bien réfléchi à l'architecture (domaines métier, couches propres, scoping MVP clair). C'est exactement le niveau d'exigence attendu pour un titre 7. Je vais répondre point par point, en commençant par ta question la plus concrète : quelle IA choisir.

**Quelle(s) IA intégrer**

Pour Mukeme, tu as en réalité deux besoins différents, qui n'ont pas les mêmes contraintes :
- **Mukeme Recommandation** : tâche simple (comprendre une envie, classer des livres candidats) → un modèle léger et bon marché suffit largement
- **Mukeme Assistant d'écriture** : tâche plus fine (reformuler un texte littéraire sans trahir le style de l'auteur) → mieux vaut un modèle plus qualitatif

Voici où en sont les principaux fournisseurs (tarifs par million de tokens, à vérifier au moment de l'implémentation car ça bouge vite) :

| Fournisseur | Modèle économique (recos) | Modèle qualité (rédaction) | Point fort pour Plumora | Point de vigilance |
|---|---|---|---|---|
| **Mistral AI** 🇫🇷 | Small — environ $0,15 / $0,60 | Medium ou Large — $0,5 à $1,5 / $1,5 à $7,5 | Hébergement UE natif, conçu pour le RGPD, très bon en français, propose aussi un modèle d'embeddings dédié | Écosystème/tutoriels un peu moins fournis qu'OpenAI |
| **OpenAI** | GPT-4o mini / 4.1 nano — ~$0,10-0,15 / $0,40-0,60 | Gamme GPT-5.x — $0,75 à $2,50 / $4,50 à $15 | Écosystème énorme, SDK et docs abondants | Hébergement US par défaut ; gamme qui change de nom tous les 2-3 mois |
| **Google Gemini** | Flash-Lite — ~$0,25 / $1,50 (palier gratuit) | Flash / Pro — $0,5 à $2 / $3 à $12 | Le moins cher pour prototyper, fenêtre de contexte énorme | Sur le palier gratuit, les prompts peuvent être utilisés par Google pour améliorer ses modèles — à éviter pour des manuscrits non publiés |
| **Anthropic Claude** | Haiku 4.5 — $1 / $5 | Sonnet 5, devenu le modèle par défaut depuis le 30 juin 2026 — tarif d'intro $2/$10 jusqu'à fin août 2026 | Réputation très solide sur la reformulation nuancée / écriture créative | Pas d'ancrage UE natif ; légèrement plus cher à qualité égale |

**Ma recommandation** : pars sur **Mistral** en priorité — Small pour les recommandations, Medium ou Large pour l'assistant d'écriture. Trois raisons concrètes pour *ton* projet précisément :
1. Un manuscrit non publié est une donnée sensible (propriété intellectuelle de l'auteur) — Mistral offre un hébergement UE et une conformité RGPD nativement, ce qui simplifie ton argumentaire sécurité/réglementaire (et ça sert directement ta veille réglementaire du Bloc 1, C1.3.1)
2. Mistral propose aussi un modèle d'embeddings — utile pour la recommandation (j'y reviens plus bas)
3. Un seul fournisseur pour les deux features = une seule intégration, une seule facture, plus simple pour un projet solo

Claude Haiku/Sonnet reste un très bon second choix si la qualité de reformulation prime sur tout le reste — n'hésite pas si tu veux comparer les deux en conditions réelles avant de trancher.

**Comment l'architecturer (sans te lier les mains)**

Le schéma au-dessus montre le flux que je recommande pour la recommandation : ton Flutter n'appelle jamais l'IA directement (bonne pratique déjà dans ton AGENTS.md). Le service Spring Boot interroge d'abord ton catalogue PostgreSQL pour trouver des candidats plausibles (filtre par genre/tags dans un premier temps — largement suffisant pour un MVP ; si tu as le temps, l'évolution naturelle est d'ajouter des embeddings via `pgvector` pour une recherche sémantique par "ambiance"), puis envoie cette shortlist + l'intention du lecteur au modèle IA, qui la classe et justifie chaque `match_score`. Cette étape de filtrage **avant** l'appel IA est importante : sans elle, le modèle peut halluciner des titres qui n'existent pas dans ton catalogue.

Point d'architecture important : définis une interface dans ta couche `domain` (genre `AiTextGenerationPort`), implémentée dans `infrastructure` par un `MistralAiAdapter`. Comme ça, le choix du fournisseur reste un détail d'infrastructure interchangeable — cohérent avec ta Clean Architecture existante, et ça te permet de changer d'avis (ou de comparer deux fournisseurs) sans toucher au reste du code. C'est aussi un excellent point à mettre en avant à l'oral du Bloc 1 (C1.5, architecture extensible).

Une chose à corriger dans ton AGENTS.md : tu notes bien que l'auteur doit accepter, modifier ou ignorer une suggestion, mais tes conventions d'API ne listent qu'un endpoint `accept`. Il te manque `reject` et `modify` (ou un `PATCH .../suggestions/{id}` générique avec un champ `status`) pour respecter ta propre règle métier.

**Sécurité — un point d'actualité utile pour ton dossier**

L'OWASP a publié une nouvelle édition de son Top 10 fin 2025 (celle que cite le référentiel n'a pas de millésime précisé, donc autant utiliser la plus récente). Deux nouveautés te concernent directement :
- La catégorie "Insecure Design" couvre désormais explicitement les attaques sur les applications qui appellent des API de LLM : injection de prompt, manipulation du modèle, failles dans la frontière de confiance. Concrètement : un commentaire de bêta-lecteur ou un extrait de chapitre pourrait contenir une instruction cachée destinée à manipuler ton assistant IA. Pense à bien délimiter le texte utilisateur dans ton prompt système et à ne jamais laisser une sortie IA déclencher une action automatiquement.
- La gestion des dépendances a évolué vers "Software Supply Chain Failures", qui inclut maintenant le pipeline CI/CD lui-même — pertinent vu que tu as déjà de l'intégration continue prévue.

Citer explicitement l'OWASP Top 10:2025 (plutôt que la version 2021 que la plupart des étudiants citent par réflexe) est un excellent point de veille technique à valoriser (C1.3.1).

**Le budget IA, pour ton chiffrage (C1.4.2)**

Avec Mistral Small à ~$0,15/M tokens en entrée, une suggestion de reformulation (disons 500 tokens de contexte + 300 de réponse) coûte une fraction de centime. Même à plusieurs milliers de requêtes/mois en test, tu restes sous quelques dollars. C'est un argument budgétaire simple et concret à présenter au jury — et pense à ajouter un quota par utilisateur (ex. avec Bucket4j côté Spring) pour te protéger d'un abus qui ferait exploser la facture.

**Points de vigilance pour la certification, bloc par bloc**

- **Bloc 1** : n'oublie pas l'**impact environnemental** — il revient à plusieurs endroits du référentiel (SWOT, architecture) et c'est facile à zapper. Une mention simple (choix d'hébergement, sobriété des appels IA via le cache de prompts) suffit.
- **Bloc 2** : ton accessibilité doit citer un référentiel nommé (RGAA/OPQUAST sont pensés pour le web — pour du Flutter mobile/desktop, tu peux t'appuyer sur les mêmes principes WCAG via les widgets `Semantics` de Flutter, en le justifiant explicitement). Vérifie aussi que tes mesures de sécurité couvrent bien les failles OWASP citées ci-dessus.
- **Bloc 3** : si tu es seul sur ce projet, le jury attend quand même une présentation de style managérial, RACI, gestion des compétences d'équipe. Le référentiel autorise une mise en situation "fictive" — prépare un scénario d'équipe (même imaginaire) plutôt que d'arriver les mains vides sur cette partie.
- **Bloc 4** : le monitoring (sondes, seuils d'alerte) et le journal de version sont éliminatoires — prévois-les dès la conception, pas en dernière minute.
- **Transversal** : le scope de Plumora est ambitieux (auth, livres/chapitres/versions, bêta-lecture complète, 2 features IA, notifications...). Pour un candidat qui travaille seul, je te conseille de sécuriser d'abord un noyau démontrable de bout en bout (écrire → publier → lire → recommander) avant d'étoffer le reste — mieux vaut un périmètre plus restreint mais entièrement fonctionnel, testé et sécurisé qu'une couverture large mais fragile sur les compétences éliminatoires.

Qu'est-ce qui t'aiderait le plus maintenant ?















Pendant mes deux années d’alternance à La Banque Postale, j’ai travaillé comme apprenti concepteur développeur mainframe. 

Au début, l’environnement mainframe était nouveau pour moi, mais j’ai progressivement appris à analyser les traitements, comprendre les règles métiers et corriger des programmes

Cette alternance m’a permis de gagner en autonomie, en rigueur et en méthode. J’ai appris à analyser un sujet, vérifier les impacts, préparer des tests, contrôler les résultats et communiquer avec les différentes équipes.



Points d’amélioration

Un point qui pourrait être amélioré concerne le suivi des tâches réalisées pendant l’alternance. Même si j’ai pu travailler sur plusieurs sujets techniques et gagner en autonomie, il serait intéressant de formaliser davantage les missions confiées, les tâches réalisées et les compétences mobilisées.

Cela pourrait passer par un suivi plus régulier dans le PDC, afin de mieux tracer les travaux effectués, les objectifs atteints et les compétences développées au fil de l’alternance. Cette formalisation permettrait aussi d’avoir une meilleure visibilité sur la progression de l’alternant et de faciliter les échanges entre l’école, l’entreprise et le tuteur.















1 

Fond principal : #11100E
Fond secondaire : #1A1714
Carte / surface : #221E1A
Texte princ

ipal : #F5F1E8
Texte secondaire : #B8AEA2
Bordure légère : #332C26

Couleur primaire : #C89B63
Couleur accent Plumora : #8F5C7A
Accent IA Plumo : #A67CFF
Erreur : #E57373
Succès : #81C784Usage
Couleur
Code
Fond principal
Bleu nuit presque noir
#0E1117
Fond secondaire / sections
Bleu ardoise foncé
#161B22
Cartes de livres
Gris nuit
#1F2633
Texte principal
Blanc cassé
#F4F1EA
Texte secondaire
Gris doux
#A8A8B3
Couleur de marque
Violet plume
#7C5CFF
Accent premium
Doré doux
#D6B25E
Bordures fines
Gris bleuté
#2A3142
Succès / validation
Vert doux
#3FBF7F
Alerte / erreur
Rouge doux


2 

Usage
Couleur
Code
Fond principal
Bleu nuit presque noir
#0E1117
Fond secondaire / sections
Bleu ardoise foncé
#161B22
Cartes de livres
Gris nuit
#1F2633
Texte principal
Blanc cassé
#F4F1EA
Texte secondaire
Gris doux
#A8A8B3
Couleur de marque
Violet plume
#7C5CFF
Accent premium
Doré doux
#D6B25E
Bordures fines
Gris bleuté
#2A3142
Succès / validation
Vert doux
#3FBF7F
Alerte / erreur
Rouge doux






Fond principal
Blanc cassé
#F8F7F3
Texte principal
Noir doux
#1F1F1F
Couleur de marque
Violet encre
#4B2E83
Couleur secondaire
Bleu nuit
#16213E
Accent léger
Doré doux
#C9A227
Fond de cartes
Blanc
#FFFFFF
Bordures / séparateurs
Gris clair
#E5E2DA





Je te conseille cette combinaison :
Fond clair + texte noir + violet encre + bleu nuit + petite touche dorée.
Exemple :
arrière-plan de l’application : blanc cassé ;
cartes de livres : blanc ;
titres : bleu nuit ou noir ;
boutons principaux : violet encre ;
éléments importants : doré doux ;
textes secondaires : gris foncé ;
bordures : gris clair.


2 



Une interface claire, élégante et littéraire, avec un fond beige papier, du texte noir doux, un marron chaleureux pour les actions principales et une touche prune pour l’identité Plumo / IA.


Fond principal : #F8F5EF
Blanc cassé / papier : #FFFDF8
Texte principal : #1F1F1F
Marron doux : #8B5E3C
Marron clair : #C9A27E
Prune / accent : #6D3A5D
Gris texte secondaire : #6B6B6B
Bordures légères : #E6DED3











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

17- Oui, ta slide est cohérente maintenant. Elle montre une estimation de charge en jours-homme, répartie par rôle et par phase projet.

L’idée à faire comprendre au jury est simple :

Cette estimation montre l’effort nécessaire pour réaliser un MVP de Plumora en contexte équipe. Elle permet d’identifier les phases les plus consommatrices, les rôles les plus sollicités et les points de vigilance en matière de charge.

Explication détaillée de ta slide

Ton estimation est organisée par rôle :

PO / Chef de projet : cadrage, validation du besoin, pilotage.
Architecte : conception technique, choix d’architecture, appui au déploiement.
UI/UX Designer : conception des parcours et maquettes.
Dev Backend : API REST, logique métier, sécurité, base de données, intégration.
Dev Front Flutter : application mobile/desktop, écrans, navigation, consommation API.
Dev IA : intégration d’Ollama, logique de recommandation ou assistance à l’écriture.
QA / Testeur : recette fonctionnelle globale.
DevOps : environnement, Docker, déploiement, supervision technique.

Ensuite, les colonnes représentent les grandes phases du projet :

Cadrage : clarification du besoin, périmètre, acteurs, risques.
Conception : architecture, maquettes, modèle de données, choix techniques.
Développement : réalisation des fonctionnalités du MVP.
Intégration : connexion front/back, API, IA, sources externes.
Tests / Recette : validation technique et fonctionnelle.
Déploiement : mise en environnement, Docker, publication technique.
Pilotage : suivi projet, coordination, ajustements.
Points importants à faire ressortir

Le total est de 101 jours-homme.
Cela ne veut pas forcément dire 101 jours calendaires. Cela veut dire une charge totale équivalente à 101 jours de travail répartis entre plusieurs rôles.

La phase la plus chargée est le développement, avec 44 JH. C’est logique, car Plumora demande plusieurs fonctionnalités : authentification, gestion des livres, publication, bêta-lecture, catalogue, administration et intégration de l’IA.

Le rôle le plus sollicité est le Dev Backend, avec 33 JH. C’est cohérent, car le backend porte les éléments les plus critiques : sécurité, API REST, logique métier, base de données, intégration des sources externes et connexion à l’IA.

Les tests / recette représentent 17 JH. Ce n’est pas seulement le QA qui teste : les développeurs participent aussi aux tests techniques. Le backend a plus de charge de test que le frontend, car il doit vérifier les API, la sécurité, les règles métier, les accès aux données et les erreurs.

Texte complet à dire à l’oral

Tu peux dire :

Cette slide présente l’estimation de la charge de travail pour le MVP de Plumora.
L’estimation est exprimée en jours-homme, c’est-à-dire en effort de travail, et non en durée calendaire exacte.
Elle est répartie par rôle et par phase projet afin de visualiser les zones les plus consommatrices.

Le total estimé est de 101 jours-homme. Cette charge correspond à un MVP en contexte équipe, avec une estimation macro.
La phase la plus chargée est le développement, avec 44 jours-homme. C’est cohérent, car cette phase concentre la réalisation des principales fonctionnalités : gestion des livres, authentification, publication, bêta-lecture, catalogue, administration et intégration de l’IA.

Le rôle le plus sollicité est le développeur backend, avec 33 jours-homme. Cela s’explique par le fait que le backend porte la logique métier, la sécurité, les API REST, l’accès à la base de données PostgreSQL, les imports contrôlés depuis les sources externes et l’appel à l’IA Plumo.

Le frontend Flutter représente aussi une charge importante, avec 23 jours-homme, car l’application doit couvrir les parcours mobile et desktop, les écrans utilisateurs, la navigation et la communication avec le backend.

Les tests et la recette représentent 17 jours-homme. J’ai réparti cette charge entre le QA, le backend, le frontend, l’IA et le DevOps, car la qualité ne repose pas uniquement sur le testeur. Les développeurs participent aux tests techniques, tandis que le QA se concentre sur la validation fonctionnelle globale.

Enfin, cette estimation permet de montrer que le projet est réalisable, mais qu’il nécessite une attention particulière sur la phase de développement, l’intégration front/back et la validation des fonctionnalités critiques.

Version plus courte si tu veux aller vite

Cette estimation présente la charge du MVP Plumora en jours-homme. Le total est de 101 JH.
La phase la plus lourde est le développement avec 44 JH, car elle concentre la réalisation des fonctionnalités principales.
Le rôle le plus sollicité est le développeur backend avec 33 JH, car il porte la sécurité, les API, la logique métier, la base de données, les imports externes et l’intégration IA.
Les tests et la recette représentent 17 JH et sont répartis entre QA, backend, frontend, IA et DevOps, car la qualité doit être vérifiée à plusieurs niveaux.
Cette estimation reste une charge macro pour un MVP en contexte équipe.

Phrase importante si le jury te demande “pourquoi ce n’est pas un planning ?”

Tu réponds :

Ici, je ne présente pas un planning détaillé, mais une estimation de charge. Le but est d’évaluer l’effort nécessaire par rôle et par phase. Le planning viendrait ensuite pour organiser ces charges dans le temps selon les disponibilités de l’équipe.

Phrase si le jury te demande “pourquoi le backend est plus chargé ?”

Tu réponds :

Le backend est plus chargé parce qu’il centralise les traitements critiques : authentification, droits d’accès, règles métier, API REST, base de données, IA et services externes. C’est donc la partie qui porte le plus de complexité technique et fonctionnelle.


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















22- budget 

Oui. Pour ton budget prévisionnel, l’idée est d’expliquer que tu pars de ton estimation de charge en jours-homme, puis que tu valorises cette charge avec un TJM différent selon les profils.

Le budget n’est donc pas inventé : il est directement lié à ta charge de travail.

1. Logique générale du budget

Tu peux expliquer comme ça :

Le budget prévisionnel est calculé à partir de l’estimation de charge. Chaque rôle a une charge exprimée en jours-homme, puis cette charge est multipliée par un TJM, c’est-à-dire un taux journalier moyen.
Cela permet d’obtenir un coût estimé par profil, puis un coût global du projet.

La formule est simple :

Coût estimé = Charge JH × TJM

Exemple :

Dev Backend = 33 JH × 500 € = 16 500 €

Donc ton tableau est cohérent parce qu’il reprend les rôles de ton estimation de charge.

2. Explication par rôle
MOA / Chef de projet — 7 JH — 3 850 €

Le chef de projet intervient surtout sur le cadrage et le pilotage.

Il sert à :

formaliser le besoin
suivre l’avancement
coordonner les acteurs
valider les choix

Tu peux dire :

La charge du chef de projet est de 7 jours-homme, principalement sur le cadrage et le pilotage. Son rôle est de structurer le besoin, suivre l’avancement et garantir que le projet reste aligné avec les objectifs du MVP.

Architecte — 8 JH — 5 200 €

L’architecte a un TJM plus élevé parce que son rôle est stratégique.

Il intervient sur :

choix d’architecture
sécurité
cohérence technique
structure backend
déploiement

Tu peux dire :

L’architecte représente 8 jours-homme. Son TJM est plus élevé car il intervient sur des décisions structurantes : architecture, sécurité, choix techniques et cohérence globale de la solution.

UI/UX Designer — 7 JH — 3 150 €

Le designer intervient principalement en conception.

Il travaille sur :

parcours utilisateurs
maquettes
ergonomie
expérience mobile et desktop

Tu peux dire :

Le profil UI/UX est estimé à 7 jours-homme. Il intervient surtout en phase de conception pour définir les parcours des auteurs, lecteurs, bêta-lecteurs et administrateurs.

Dev Backend — 33 JH — 16 500 €

C’est le poste le plus important du budget.

C’est logique parce que le backend porte :

API REST
sécurité
rôles et droits
logique métier
base de données
imports externes
connexion IA

Tu peux dire :

Le développeur backend est le poste le plus coûteux, avec 33 jours-homme, soit 16 500 €. C’est cohérent car le backend porte la majorité des traitements critiques : API REST, sécurité, logique métier, accès PostgreSQL, intégration des sources externes et appel à l’IA Plumo.

Dev Front Flutter — 23 JH — 10 350 €

Le front est aussi important, mais moins lourd que le backend.

Il couvre :

écrans mobile / desktop
navigation
formulaires
connexion aux APIs
affichage catalogue
parcours utilisateur

Tu peux dire :

Le développement Flutter représente 23 jours-homme, soit 10 350 €. Cette charge couvre les écrans, la navigation, les formulaires, l’expérience mobile et desktop, ainsi que la communication avec les APIs backend.

Dev IA — 8 JH — 4 800 €

Le dev IA est ciblé sur un MVP, donc la charge reste limitée.

Il sert à :

intégration Ollama
requêtes IA
réponses Plumo
aide à l’écriture
recommandation
tests des réponses IA

Tu peux dire :

Le développement IA est estimé à 8 jours-homme. L’objectif n’est pas de créer un modèle d’IA, mais d’intégrer Ollama pour fournir une assistance à l’écriture et à la recommandation dans le cadre du MVP.

QA / Testeur — 8 JH — 2 800 €

Le QA valide le bon fonctionnement global.

Il vérifie :

parcours utilisateur
recette fonctionnelle
non-régression
cas d’erreur
conformité au besoin

Tu peux dire :

Le testeur représente 8 jours-homme. Il intervient principalement sur la recette fonctionnelle globale afin de vérifier que les parcours principaux du MVP fonctionnent correctement.

DevOps — 7 JH — 3 850 €

Le DevOps intervient surtout sur l’environnement et le déploiement.

Il couvre :

Docker
environnement de déploiement
configuration
mise en service
suivi technique

Tu peux dire :

Le DevOps est estimé à 7 jours-homme. Sa charge concerne principalement la dockerisation, la préparation de l’environnement et le déploiement reproductible de l’application.

3. Pourquoi les TJM sont différents ?

C’est important à expliquer, car tu avais raison : mettre 400 € pour tout le monde n’était pas très réaliste.

Tu peux dire :

Les TJM ne sont pas identiques car les profils n’ont pas le même niveau d’expertise, ni le même niveau de responsabilité. Par exemple, l’architecte et le développeur IA ont un TJM plus élevé car ils interviennent sur des sujets plus spécialisés. Le QA a un TJM plus faible car son intervention est davantage centrée sur la validation fonctionnelle.

Tes TJM sont cohérents :

Architecte : 650 €
Dev IA : 600 €
Chef de projet / DevOps : 550 €
Dev Backend : 500 €
Dev Front Flutter : 450 €
UI/UX : 450 €
QA : 350 €
4. Coût total du développement

Ton total est :

101 JH
Coût total estimé : 50 500 €
TJM moyen : 500 €

Tu peux dire :

Le coût total du développement est estimé à 50 500 €. Ce montant correspond uniquement à la valorisation des charges humaines du MVP. Il ne comprend pas encore les coûts techniques récurrents comme l’hébergement, le nom de domaine ou les comptes développeur.

5. Explication des coûts techniques prévisionnels

Si tu as aussi une partie “coûts techniques”, tu peux l’expliquer séparément.

Exemples cohérents :

Hébergement backend + base de données : 600 € / an
Nom de domaine : 15 € / an
Compte Google Play Developer : 25 € une fois
Outils / monitoring / stockage : 300 € / an
Marge d’imprévus : 1 000 €

Tu peux dire :

En complément du coût humain, j’ai identifié des coûts techniques prévisionnels. Ils couvrent l’hébergement du backend et de PostgreSQL, le nom de domaine, le compte développeur Google Play, les outils de suivi technique et une marge d’imprévus.
Ces coûts restent limités car le projet est présenté comme un MVP et non comme une infrastructure de production à grande échelle.

6. Texte complet à dire à l’oral

Tu peux dire exactement ceci :

Cette slide présente le budget prévisionnel du projet Plumora.
Le calcul est directement basé sur l’estimation de charge précédente. Pour chaque rôle, j’ai repris la charge en jours-homme, puis je l’ai multipliée par un TJM, c’est-à-dire un taux journalier moyen.

Le coût total du développement est estimé à 50 500 € pour 101 jours-homme. Ce budget correspond à un MVP en contexte équipe, avec une estimation macro.

Le poste le plus important est le développement backend, avec 33 jours-homme et un coût estimé à 16 500 €. C’est cohérent car le backend porte les éléments les plus critiques : sécurité, API REST, logique métier, base de données PostgreSQL, imports depuis les sources externes et intégration de l’IA Plumo.

Le développement Flutter représente 23 jours-homme, soit 10 350 €. Cette charge couvre les écrans mobile et desktop, les parcours utilisateurs, la navigation et la communication avec le backend.

Les autres profils interviennent de manière plus ciblée : le chef de projet sur le cadrage et le pilotage, l’architecte sur la conception technique, le designer sur l’expérience utilisateur, le Dev IA sur l’intégration d’Ollama, le QA sur la recette fonctionnelle, et le DevOps sur l’environnement et le déploiement.

Les TJM sont volontairement différenciés selon les profils, car les niveaux de responsabilité et d’expertise ne sont pas les mêmes. L’architecte et le développeur IA ont un TJM plus élevé, tandis que le QA a un TJM plus faible.

Enfin, ce budget de développement doit être complété par des coûts techniques prévisionnels, comme l’hébergement, le nom de domaine, le compte Google Play Developer, les outils de suivi et une marge d’imprévus.

Version courte à retenir

Le budget prévisionnel est calculé à partir de la charge estimée.
Chaque rôle a une charge en jours-homme, multipliée par un TJM adapté au profil.
Le total développement est de 101 JH pour un coût estimé à 50 500 €.
Le poste principal est le backend, car il concentre la sécurité, les APIs, la logique métier, la base de données, les imports externes et l’intégration IA.
Ce budget reste une estimation macro pour un MVP Plumora en contexte équipe.












preco: 


Oui, pour cette slide, tu dois expliquer que tu arrives à ta recommandation finale après tout ce que tu as présenté avant : besoins, risques, faisabilité, architecture, charge et budget.

L’objectif est de montrer au jury que tu ne présentes pas juste des fonctionnalités, mais une préconisation argumentée.

Ce que tu peux dire à l’oral

Cette slide présente les axes de solution que je préconise pour le projet Plumora.
Après avoir analysé le besoin, les risques, la faisabilité technique, la charge et le budget, ma recommandation est de lancer Plumora sous forme de MVP.

L’objectif est de partir sur une solution moderne, sécurisée et évolutive, capable de supporter les principaux usages du projet : l’écriture, la lecture, la publication, la bêta-lecture et l’assistance IA avec Plumo.

Ensuite, tu présentes les 4 axes.

Axe 1 — Architecture évolutive

Le premier axe est l’architecture évolutive.
Le choix proposé repose sur un frontend Flutter, un backend Spring Boot, une base PostgreSQL, Docker et des services externes isolés.
L’idée est d’avoir une architecture claire, maintenable et capable d’évoluer plus tard sans tout reconstruire.

Axe 2 — Plateforme multi-rôles

Le deuxième axe est la plateforme multi-rôles.
Plumora ne s’adresse pas à un seul utilisateur, mais à plusieurs profils : auteur, lecteur, bêta-lecteur et administrateur.
Chaque rôle dispose d’un parcours adapté à ses besoins : écrire, lire, commenter, publier, modérer ou gérer les contenus.

Axe 3 — IA maîtrisée

Le troisième axe est l’IA maîtrisée.
L’IA Plumo doit assister l’utilisateur, notamment pour l’aide à l’écriture et la recommandation de livres.
Mais elle ne modifie pas automatiquement les contenus : l’utilisateur garde toujours le contrôle.
Cela permet d’intégrer l’IA de manière utile, mais encadrée.

Axe 4 — Catalogue enrichi

Le quatrième axe est le catalogue enrichi.
Plumora peut proposer à la fois des livres créés directement sur la plateforme et des livres du domaine public provenant de sources externes comme Gutendex ou Open Library.
Cela permet d’enrichir le catalogue dès le lancement du MVP.

Conclusion orale de la slide

Ces quatre axes permettent de répondre à la problématique du projet : concevoir une plateforme multi-supports, sécurisée, évolutive et enrichie par l’IA.
Ma préconisation est donc de valider ce cadrage et de lancer la phase de conception et de développement du MVP.

Version courte à retenir

Cette slide résume ma préconisation finale. Je recommande de lancer Plumora sous forme de MVP autour de quatre axes : une architecture évolutive, une plateforme multi-rôles, une IA maîtrisée et un catalogue enrichi.
Ces axes permettent de répondre au besoin identifié tout en gardant une solution réaliste, sécurisée et évolutive.

