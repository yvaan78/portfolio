# Gestion du Personnel (M2L)

BTS SIO SLAM · Épreuve E6 ·

## 1. Présentation

Application Java de gestion de **ligues sportives** et de leurs **employés**.
Elle est passée d'une version en ligne de commande mono-utilisateur à une
version **multi-utilisateurs** avec base de données MySQL et architecture en couches.

## 2. Fonctionnalités

- Créer, modifier et supprimer des ligues et des employés
- Trois rôles : root, administrateur de ligue, employé simple
- Connexion par mot de passe (haché en SHA-256)
- Dates d'arrivée et de départ avec validation
- Deux interfaces : ligne de commande (CLI) et graphique (Swing)

## 3. Règles métier

| Règle | Description |
|-------|-------------|
| Root | Gère toutes les ligues et tous les employés |
| Administrateur | Gère uniquement les employés de sa ligue |
| Employé | Aucun accès à la gestion |
| Date d'arrivée | Ne peut pas être dans le futur |
| Date de départ | Postérieure ou égale à la date d'arrivée |
| Suppression du root | Impossible |

## 4. Architecture

Architecture en couches. La logique métier est isolée de la persistance
grâce à l'interface `Passerelle` (pattern Gateway).

~~~
Présentation   commandLine.*  |  gui.PersonnelFrame
      |
Métier         GestionPersonnel, Ligue, Employe
      |        (interface Passerelle)
Persistance    jdbc.JDBC  |  serialisation.* (ancienne version)
      |
Base MySQL     base « personnel »
~~~

La constante `TYPE_PASSERELLE` de `GestionPersonnel` permet de basculer
entre JDBC et sérialisation sans toucher au code métier.

## 5. Base de données

Script de création : `DOC/script.sql`.

| Table | Colonnes principales |
|-------|----------------------|
| LIGUE | id_ligue (PK), nom_ligue (unique) |
| EMPLOYE | id_employe (PK), nom, prénom, mail (unique), mot de passe haché, rôle, date d'arrivée, date de départ, id_ligue (FK) |

Un employé appartient à une seule ligue ; une ligue a plusieurs employés.
Le root n'appartient à aucune ligue (`id_ligue` NULL).

## 6. Organisation du code

| Package | Rôle |
|---------|------|
| `personnel` | Logique métier : `GestionPersonnel` (singleton), `Ligue`, `Employe`, `PasswordHasher`, exceptions |
| `jdbc` | Accès MySQL : `JDBC` (requêtes SQL), `Credentials` (connexion) |
| `serialisation` | Sauvegarde dans un fichier (ancienne version) |
| `commandLine` | Menus en ligne de commande |
| `gui` | Interface Swing (8 écrans gérés par un `CardLayout`) |
| `testsUnitaires` | Tests JUnit |

Exceptions métier : `DateInvalide`, `SauvegardeImpossible`,
`DroitsInsuffisants`, `ImpossibleDeSupprimerRoot`.

## 7. Sécurité

- Mots de passe **jamais stockés en clair** : hachage SHA-256 (`sha256$<empreinte>`)
- Droits vérifiés avant chaque action sensible (CLI et interface graphique)
- Suppression d'une ligue dans une **transaction SQL** avec rollback en cas d'erreur
- `Credentials.java` (vrais identifiants) ignoré par Git ;
  `CredentialsExample.java` sert de modèle

## 8. Tests

Tests unitaires JUnit 5 : `testEmploye` (13 tests) et `testLigue` (9 tests),
dont les exceptions sur les dates invalides et le changement d'administrateur.

## 9. Technologies

Java 8 · Maven (JAR autonome avec maven-shade) · MySQL + JDBC
(mysql-connector-java 8.0.16) · CommandLineMenus 2.0 · JUnit 5.8.2

## 10. Lancement

1. Installer Java 8 et MySQL
2. Créer la base : `mysql -u root -p < DOC/script.sql`
3. Copier `CredentialsExample.java` en `Credentials.java` et le remplir
4. Lancer une classe `main` :

| Classe | Mode |
|--------|------|
| `gui.PersonnelApp` | Interface graphique, ou CLI si pas d'écran |
| `gui.PersonnelGuiMain` | Interface graphique uniquement |
| `commandLine.PersonnelConsole` | Ligne de commande uniquement |

