TP 3 : Associations, Cascades et Gestion des Orphelins avec JPA et Hibernate

Ce travail pratique porte sur la conception et l'expérimentation d'un modèle relationnel complet sous JPA et Hibernate, en utilisant une base de données H2 exécutée en mémoire. L'objectif était de maîtriser le cycle de vie des entités, la synchronisation bidirectionnelle des associations, la propagation des opérations via les cascades et la suppression automatique des entités orphelines.

Le projet a d'abord été structuré avec Maven autour d'un fichier pom.xml configuré pour Java 8, intégrant l'API JPA 2.2, Hibernate Core 5.6 comme moteur de persistance, Hibernate Validator pour les contraintes de validation de données, ainsi que le pilote H2. La configuration technique a été centralisée dans le fichier persistence.xml au sein du dossier META-INF, avec une unité de persistance nommée gestion-reservations. Nous avons configuré la base en mode mémoire avec la propriété de génération de schéma create-drop et l'activation des logs SQL formatés pour pouvoir observer en détail chaque requête exécutée.

Le modèle métier est composé de quatre entités : Utilisateur, Salle, Reservation et Equipement.

L'entité Utilisateur contient des attributs validés par Bean Validation comme le nom, le prénom et un format d'adresse email valide. Elle possède une relation OneToMany vers Reservation avec les options cascade ALL et orphanRemoval à true. Afin d'éviter les incohérences en mémoire avant la transaction en base, nous avons créé des méthodes helpers comme addReservation et removeReservation pour mettre à jour les deux côtés de l'association en même temps.

L'entité Salle gère également une relation OneToMany vers Reservation, ainsi qu'une relation ManyToMany vers l'entité Equipement. Cette dernière est configurée avec une table de jointure explicite nommée salle_equipement. Sur cette association ManyToMany, nous avons fait attention à ne pas appliquer de cascade de type REMOVE, car un équipement peut être partagé entre plusieurs salles et ne doit pas disparaître si l'une d'elles est supprimée.

L'entité Reservation sert de lien central en portant les deux clés étrangères via des annotations ManyToOne vers Utilisateur et Salle, avec un chargement en mode LAZY pour optimiser les performances.

La classe principale App nous a permis d'exécuter et de valider trois scénarios essentiels :

Dans un premier temps, nous avons vérifié la persistance en cascade. En persistant uniquement l'utilisateur et les salles, Hibernate a automatiquement sauvegardé en base de données les réservations et les équipements associés grâce aux cascades configurées.

Dans un second temps, nous avons testé la suppression orpheline. Après avoir récupéré un utilisateur existant, nous avons retiré l'une de ses réservations de sa collection Java via la méthode removeReservation. Au moment du commit de la transaction, Hibernate a automatiquement détecté la rupture du lien et émis un ordre DELETE sur la table des réservations. Après avoir vidé le contexte de persistance avec em.clear(), une requête find a confirmé que la réservation avait bien disparu de la base de données.

Enfin, nous avons validé la relation ManyToMany. Lorsque nous avons retiré un équipement de la collection d'une salle, Hibernate a uniquement supprimé la ligne correspondante dans la table de jointure salle_equipement. Une requête ciblée sur l'équipement a prouvé qu'il existait toujours en base, démontrant que la séparation des données partagées est parfaitement préservée.

En conclusion, ce TP met en évidence l'importance capitale des méthodes utilitaires pour la cohérence des objets en mémoire et démontre la différence fondamentale entre un simple détachement de collection et une suppression physique orchestrée par orphanRemoval.









https://github.com/user-attachments/assets/e290c278-bbf6-446e-8cf1-68a17da7654f

