
Prosit 2: Administration Oracle

Team :
Animateur : Emilien
Secrétaire : JOHHHHHHHHHNNNNN
Scribe : Flo
Gestionnaire : Mehdi

## Mot clés (rien à définir) :
- Vue
- Commande du LCD :  Langage Contrôle de Données
- **PUMP** : 11gR2 - Fournit une infrastructure serveur pour déplacer les données de manière optimisée (performances, restart...)
- **DUMP** : Fonctions qui retourne des informations sur une expression (Type, longueur, représentation interne)
- **Mode ArchiVelog** : Mode pour la base de données pour créer des backups de transaction, permettant de revenir en arrière à n'importe quel moment. Nécessite plus de place sur le disque pour placer les archived log files.
- **RMAN** : Client qui effectuée des tâches de backups et restaurations, et automatise l'administration sur une BDD oracle. Simplifie les mécanismes de restauration.
- Privilèges / rôle / profil
- DBA
- Sécurité et optimisation
- Dictionnaire de données
- Compte particulier
- Système de sauvegarde et de restauration

## Contexte :

Quoi ?
- On souhaite mettre en place différent profils avec des privilèges différents
- Sécuriser et optimiser la base de données
Comment ?
- Mettre en place des différents profils utilisateurs avec différents privilèges
- En mettant en place un système de sauvegarde et de restauration
Pourquoi ?
- Pour répondre aux attentes du client

## Contraintes :
- Oracle

## Problématique :
- Comment gérer la sécurité et la sauvegarde d’une base de données et ses utilisateurs sous Oracle ?
### Généralisation :
- Sécurité / Sauvegarde
- MCO
- Gestion des données	

## Hypothèses :
- Le LCD est un outil en ligne de commande
- RMAN est le processus Recovery Manager qui restaure la base
- DUMP permet de remplir les fichiers de logs
- Différence mineure entre dump et pump

## Plan d’action
### Études
###	Gestion des profils
- Droits
- Privilèges
- Rôles
- LCD
	
### Sauvegarde et restauration
- DUMP
- PUMP
- RMAN
- Archivelog
	
### Réalisation
- Corbeille


## I - Gestion des profils 

Afin d'augmenter la sécurité, il est intéressant de mettre des gestion pwd, tentatives de co, temps vérouillage, ressource systèmes...

Deux types de **privilèges**/**droits** : 
- Système (exécution) avec GRANT
- Niveau objet : Exécuter une action sur un objet spécifique

- Si WITH ADMIN OPTION, le bénéficiaire peut à nouveau assigner ce rôle
- DBA peut révoquer les privilèges (+rôles), ça ne supprime pas le compte
- Les privilèges/droits sont les suivants :
	- Table/Vues/snapshot : INSERT, UPDATE, DELETE, SELECT
	- Table propriétaire : ALTER/REFERENCES/INDEX/ALL
	- Proc, fonctions... : EXECUTE
	- Séquence : SELECT, ALTER


**Les rôles**
- CONNECT : Création tables / vues / séquences / Cluster / synonymes / 
- RESSOURCE : Utilisateurs réguliers : Droits supplémentaires sur ceux du connect
- DBA : Tous les privilèges de niveau sysstème avec quotas d'espace illimités, et accorder les droits aux autres users. Compte SYSTEM




## II - Sauvegarde et restauration

⇒ Est-il acceptable de perdre des données :
- Si oui, sur quelle période ?
⇒ Est-il possible d'arrêter périodiquement la base ? 
- Si oui combien de temps ?
- Est-il possible de faire une save complète pendant l'arrêt de celle-ci ?

Permettent de répondre à :
- Archivelog : Archives des fichiers Redo Log


### Pump


### Dump


### RMAN

- Effectuer du "Grid Control"
- Définit une stratégie de save recommandée par oracle
- protégé les données et backups pour les 24h
- SAuvegarde incrémental,, de copie complète pour la première save

Le RMAN est un outil qui accomplit une bonne partie du travail à la place du DBA dans le but de protéger la BDD :
- Copie de fichiers de BDD : RMAN peut créer des copies images (sauvegarde à chaud)
- Création BDD dupliquée 
- Création d'une BDD de secours : Permet de bacjup jusquà un certain seuil dans le temps (TSPITR - Tablespace Point In Time Recovery)

### Archivelog


### Fichiers de controles / Redo Log 

DBA : Tâches principales de l'admin
- **Fichier de contrôles**: 
	- Premier fichier de contrôle à l'ouverture de la BDD
	- Localiser et ouvrir les autres fichiers de la BDD
	- Permet de MOUNT (sinon NOMOUNT et ne peut pas s'ouvrir)
	- Automatiquement MAJ par Oracle à chaque modif
	- Contient :
		- Nom et Identifiants
		- Nom et emplacement fichiers Redo Log
		- Nom tablespace
		- Date Heure et création BDD
		- Numéro de séqence journal courant
		- Informations relatives au point de synchro 
		- Historique du journal
		- Informations de sauvegarde du Recovery Manager
	- Peut-être interrogé par le RMAN (vues pour savoir si save bien réussis)
	- Le RMAN enregistre des infos dedans (puis écrase précèdente au bout de x temps)
	- Il faut les sauvegarder à chaque sauvegarde (partielle / complète)
	- ``ALTER DATABASE BACKUP CONTROLFILE TO to [ nouveau_nom | trace ] [ reuse ]`` pour en save un
	- Un fichier de contrôle doit-être multiplexé (copie qui sera maintenue en miroir)
		- Sans fichier de contrôlé la BDD ne démarre pas (MOUNT)
		- Au moins deux fichiers de contrôlé, si possible sur des disques différent, un par disque
		- peut-être mis en place lors de la création de la base, spécifier les fichiers OU le déclarer dans le CONTROL_FILES

 **Fichiers de Redo Log**
- Enregistrent les transactions (modifications apportées à la base)
- Organisés en groupe
- Utilisé de manière circulaire, anciennes données écrasé, 2 groupes au mini
- Utilisé pour la restauration de la BDD après arrêt anormal
- Pas indépendant des autres fichiers du même type, au minimum 2 ou 3 membres par groupe

![](https://image.noelshack.com/fichiers/2019/18/5/1556866898-redolog.png)
- On peut multiplexer les fichiers (dupliquer un groupe) ⇒ Ajouter / supprimer / déplacer (commandes...)
- Des vues permettent d'accéder aux infos des fichiers Redo Log, comme : 
	- V$LOG : Infos sur les groupes
	- V$LOGFILE : Info sur les membres
	- V$LOG_HISTORY : Historique des fichiers
	- V$INSTANCE_RECOVERY : Temps estimé de restauration d'instance




### Fichier de trace

Pour connaître l'emplacement et le nom des fichiers de base ==> ASTUCE
- Générer la trace du fichier de contrôle :
``ALTER DATABASE BACKUP CONTROLFILE TO TRACE ;``
- On a une fichier .TRC qui est généré 
- Ne précise pas de fichier de "paramètres". Il faut les rajouter à la fin "STARTUP + nom et chemin"
- Il devient un fichier de contrôle, récupéré à la prochaine instance


### Les différentes sauvegardes : 


**Sauvegarde base arrêtée NORMALEMENT**

Les fichiers suivants doivent-être sauvegardés :
- Fichier de contrôle
- Fichier de données
- Fichiers de Redo Log ==> NOARCHIVELOG
- Fichier init.ora et spfile.ora (optionnel)


 **Sauvegarde base en ligne**

- Nécessite ARCHIVELOG
- Génération d'archive de fichiers REDO LOG sous forme de journal, des transactions
	- Fichiers de contrôle
	- Fichiers de données (sauvegardé à chaud)
	- Fichier init.ora et spfile.ora

 **Sauvegarde du fichier de contrôle**

- Obligatoire à sauvegarder à chaque save complète OU partielle
- Save via une commande (pas de C/C)

**Sauvegarde partielle d'un tablespace ONLINE**

- Lorsqu'on est en backup, on effectue une sauvegarde partielle
- on a plus de checkpoints (logique) ⇒ Il faut attendre le END BACKUP. (Super important)
- On a une sauvegarde auto qui sont les CHECKPOINT qui sont enregistré dans les fichiers REDO LOG archivé pour retrouver l'état après

 **Sauvegarde de tous les tablespaces de la base ONLINE**

``ALTER DATABASE BEGIN BACKUP ; ----- ------ ----- ----- ALTER DATABASE END BACKUP ;``

La bse doit être OPEN et en mode ARCHIVELOG


### **Flash Back**
- Permet de récupérer un ensemble de données du passé et de les réinjecter dans la BDD, en intérogeant des versions anciennes du schéma
- Permet de réparer facilement les données corrompus d'une table




## Tablespace [Optionnel]


### Monitoring de l'utilisation d'un tablespace 
- On a des seuls en %, une alerte est déclenchée si dépassé (85% taux remplissage, 97% alerte critique)
- Le processuce MMON vérifie toutes les dix minutes ces seuils
- On peut changer ces valeurs
- 
### Tablespace Temporaire :

- Oracle effectue de nombreux tris (utilisateurs / noyau) suite à des requêtes Order By / Group B {...}
- Effectués dans PGA, mais si celle ci insufisante ⇒ Tablepsace Temporaire
- Si il n'y en a pas : tablespace SYSTEM
==> Amélioration des temps de réponse
- Possibilité de faire des groupes de tablespace temporaires
- On retrouve des vues pour voir les tablespace temporaires

### Tablespace permanant 
Utiliser pour stocker des objets permanants comme des tables / index
⇒ Création / delete / modifier / modifier les fichiers {...} / vues {...}

### Tablespace UNDO 
- Tablespace UNDO pour la gestion auto des segments d'annulation.  Fonctionnalité appellée AUM (Automatic Undo Management)
- Permet de ROLLBACK / RECOVER
- Si COMMIT ==> Espace libéré. Si ROLLBACK 
- Lorsqu'un utilisateur interroge une table en cours de modification, Oracle utilise l'image AVANT des données stockées dans les segments d'annulation pour lui répondre
⇒ Création / delete / Modifier / vues


## Corbeille Exo (super importante)

### Gestion des users 


1. Créer un utilisateur Vincent ayant pour mot de passe vinc. Le tablespace temporaire de Vincent sera
TEMP (Attention, sous Database Control, de ne pas lui attribuer le rôle CONNECT) :

 ``CREATE USER Vincent IDENTIFIED BY vinc TEMPORARY TABLESPACE TEMP;``

2. Tenter de se connecter à Vincent. Que se passe-t-il ? Pourquoi ?

``CONNECT Vincent //Impossibilité de se connecter``

3. Attribuer à Vincent le privilège system CREATE SESSION. Refaire la question 2.

``GRANT CREATE SESSION TO Vincent;``

5. Créer un utilisateur Theo ayant pour mot de passe theo. Le tablespace temporaire de Theo sera TEMP et
son tablespace par défaut sera USERS limité à 10Mo.
idem q1

 ```
 CREATE USER Theo IDENTIFIED BY theo
 TEMPORARY TABLESPACE TEMP
 DEFAULT TABLESPACE USERS 
 QUOTA 10M ON USERS;
 ```

6. Attribuer à Theo les seuls privilèges nécessaires pour se connecter à la base et créer des tables. Tester.

``GRANT CREATE SESSION GRANT UNLITIMED TABLESPACE TO Theo``

6. Créer un profil une_deux qui autorise une seule connexion et deux minutes d’inactivité. Attribuer ce profil à
Theo. Tester.

```
CREATE PROFILE une_deux LIMIT
SESSIONS_PER_USER          1
CONNECT_TIME               2

ALTER USER Theo
    PROFILE une_deux;
```


7. Appliquer les actions correctives afin que le profil une_deux soit mis en oeuvre.

``??``


8. Attribuer à Theo les privilèges nécessaires pour créer la table département à partir de la table dept
appartenant à Scoot.

``GRANT CREATE Scoot.département TABLE TO Theo``

9. Se connecter à Theo, créer la table département et consulter les informations issues de la vue USER_
OBJECTS.

```
CONNECT THEO

CREATE TABLE département FROM (Select * FROM Scoot.département)

SELECT * FROM USER_OBJECTS
```

10. Theo autorise Vincent à lire sa table département

`` GRANT Select on département to Vincent``

11. Sous le compte Theo, vérifier la création de ces droits à travers la vue USER_TAB_PRIVS.

``SELECT * FROM USER_TAB_PRIVS``

12. Theo autorise Vincent à modifier la colonne ville de sa table département.

`` GRANT UPDATE (ville) on departement to Vincent``

13. Se connecter à Vincent et interroger les vues USER_TAB_PRIVS et USER_COL_PRIVS.

```
DISCONNECT
CONNECT Theo

SELECT * FROM USER_TAB_PRIVS

SELECT * FROM USER_COL_PRIVS
```

14. Vincent transfère toutes les activités réalisées à Dallas vers Grenoble et change le nom du département Sales en Ventes. Exécuter les requêtes correspondant à ces modifications. Que se passe-t-il ? Visualiser le contenu de la table. Faire un rollback. Visualiser de nouveau le contenu de la table.

```
insert into département(nomDepartement);
 VALUES  (select * from nomDepartement where nomDepartement = Dallas)
 
UPDATE département
SET nomDepartement= 'Grenoble'
WHERE nomDepartement = 'Dallas'

ROLLBACK; //Oh mon dieu ça s'annule

SELECT * FROM Grenoble

```
15. Créer un rôle Developper qui autorise les utilisateurs à créer des tables, des vues, des séquences et des synonymes.

```
CREATE ROLE Developper
   GRANT CREATE TABLE GRANT CREATE VIEW, CREATE SEQUENCE, CREATE SYNONYM to developper
```

16. Attribuer ce rôle à Theo. Tester.

``SQL> GRANT Developper TO Theo;``


### Tablespace

Création tableSpace TS_TAB_AIRBASE dont le fichier est ts_tab_airbasex1 et dont la volumétrie est de 5 Mo. Managé en Local
```
CREATE TABLESPACE TS_TAB_AIRBASE DATAFILE %ORACLE\%oradata\%DBCOURS\TSTAB\
ts_tab_airbasex1’ SIZE 5 MO EXTENT MANAGEMENT LOCAL ;
```

Création index TS_IND_AIRBASE dont le fichier est ts_int_airbasex1 et dont la volumétrie est de 2Mo. Managé en Local
```
CREATE INDEX TS_INDEX_AIRBASE DATAFILE %ORACLE\%oradata\%DBCOURS\TSTAB\
ts_int_airbasex1’ SIZE 2 MO EXTENT MANAGEMENT LOCAL ;
```

Utilisateur Orcalce avec votre nom XXX avec les caractéristiques tablespace par défaut : users / temporaire : temp / pwd : exia

 ```
 CREATE USER Emilien IDENTIFIED BY exia
 TEMPORARY TABLESPACE TEMP
 DEFAULT TABLESPACE USERS 
 ```


Etant System, affectez le privilège CREATE TABLE à l’utilisateur et reconnectez-vous comme étant XXX.
Si la connexion a réussi, tentez de créer la table PILOTE.
```
GRANT Create table to Emilien
Disconnect
Connect Emilien

Create table PILOTE
(
nom varchar(50),
age number(10),
);

Insert into PILOTE(nom, age)
Values (Frank, 40)

```

### PARTIE B - Sauvegarde et Restauration manuelle

//Sauvegarder à froid ⇒ Script
```
-- Variables d'environnement de SQL*Plus de formatage de l'affichage
Set feedback off
Set Linesize 200
Set Heading off
Set Pagesize 0
Set Trimspool off
Set Verify off
define repertoire ='c:\archive' -- répertoire de destination des fichiers
sauvegardés
define fichier_control=c:\control_backup.sql -- définition du fichier de sortie
spool &fichier_control
select 'host copy ' || name || ' &repertoire ' from v$datafile order by 1 ;
select 'host copy ' || member || ' &repertoire ' from v$logfile order by 1 ;
select 'host copy ' || name || ' &repertoire ' from v$controlfile order by 1 ;
select 'host copy ' || name || ' &repertoire ' from v$tempfile order by 1 ;
spool off
-- Fermeture de la base de données pour avoir des fichiers synchronisés
shutdown immediate
@&fichier_control
startup
```

//Afficher le mode archive :

ARCHIVE LOG LIST //Consulter le mod actuel
ou
SELECT LOG_MODE FROM SYS.V$DATABASE;

//Désactiver le mode archive

```
SHUTDOWN IMMEDIATE
STARTUP MOUNT //Démarre la base sans l'ouvrir
ALTER DATABASE NOARCHIVELOG; //Change le mode
ALTER DATABASE OPEN; //Ouvre la base
```

Provoquer de l'activié dans la BDD en ajoutant un fichier de 100K dans votre tablespace TS_IND_AIRBASE.

```
ALTER TABLESPACE TS_IND_AIRBASE ADD DATAFILE ‘E:\oracle\database\TEST\TEST.ora’ SIZE 100K;
```

Insérer des enregistrmeents dans la table PILOTE
```
Insert into PILOTE(nom, age)
Values (Mikael, 25)
```

Faire une sauvegarde complète
```
sql> SHUTDOWN IMMEDIATE
LOG_ARCHIVE_START = TRUE //Effectue une sauvegarde complète à froid
```

Mettre le bordel dans les fichiers ==> ça marche plus !
Puis restaurer via la dernière sauvegarde :

 ⇒ Coincé question 7


## Partie B : Sauvegarde et restauration automatique RMAN



