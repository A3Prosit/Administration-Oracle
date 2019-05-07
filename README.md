
Prosit 2: Administration Oracle

Team :
Animateur : Emilien
Secrétaire : JOHHHHHHHHHNNNNN
Scribe : Flo
Gestionnaire : Mehdi

## Mot clés (rien à définir) :
- Vue
- Commande du LCD :
- PUMP :
- DUMP :
- Mode ArchiVelog :
- RMAN :
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

### Sécuriser la bdd
il faut protéger les fichiers de contrôle et les redo log
Le fichier de contrôle est le premier fichier ouvert et permet de localiser les autres fichiers. Si on le perds la bdd reste en état NOMOUNT et on ne peut pas l'ouvrir. Il est automatiqueement mis à jour par Oracle à chaque modif de la structue de la bdd. Il contient :
- le nom et l'id de la bdd
- le nom et l'emplacement des fichiers de données et les fichiers de Redo Log
-  le nom des tablespaces
- la date et l'heure de créa de la bdd
- le numéro de séquence du journal courant
- des infos relatives au point de synchronisation
- l'historique du journal
- les infos de sauvegarde du Recovery Manager

Lors de saves faites avec le RMAN certaines vues du du dico et commandes du RMAN permettent d'interroger le fichier de contrôle pour connaître l'état des saves

Quand RMAN save une bdd, des infos sont toujours consignées dans le fichier de contrôle de la base cible. Pour limiter sa taille les anciennes entrées sont écrasées après un certain nombre de jours défini par CONTROL_FILE_RECORD_KEEP_TIME et est 7 par défaut et est monté à 15 jours quand RMAN utilise le fichier
Les fichiers de contrôle doivent être saved lors de chaque save complète ou partielle de la bdd et il est conseillé de les save après chaque restructuration importantes pendant les saves normales

pour save le fichier on fait 
```
ALTER DATABASE BACKUP CONTROLFILE TO to [ nouveau_nom | trace ] [ reuse ];
```
trace génère un script dans les fichiers de traces desusers (diagnostic_dest)

**Protection du fichier de contrôle**

la première action est le multiplexage du fichier. On fait une cpie qui sera maintenue sur un miroir par Oracle. Si la copie n'est pas jugée cohérente par Oracle il y aura une erreur au redémarrage

On peut faire le multiplexage à la création de la base en spécifiant la liste des fichiers de contrôle à copier dans CONTROL_FILES avant de faire SQL CREATE DATABASE
Pour le faire après coup il faut:
- arrêter la base proprement (pas ABORT)
- dupliquer le fichier vers sa destination
- mentionner le nouveau fichier das CONTROL_FILES
- rédmarrer la base

autre formulation
- Exporter le fichier de paramètres serveur 
CREATE PFILE FROM SPFILE
- Modifier le fichier SPFILE 
ALTER SYSTEM ... SCOPE=SPFILE 
- Arrêter la base 
SHUTDOWN IMMEDIATE 
- Recopier le fichier de contrôle vers le nouvel emplacement 
- Ouvrir la base 
STARTUP

on peut interroger les vues du dico de donnée V$CONTROLFILE et V\$PARAMETER 
ex:
```
select value from v$parameter where name = ‘control_file’;
```
on peut obtenir les infos sur les sections des fichiers de contrôle avec la vue dynamique V$CONTROLFILE_RECORD_SECTION
```
Select type, record_size, records_total, record used From v$controlfile_record_section Where type = ‘DATAFILE’ ;
```

**Protéger les Redo Logs**
Les redo logs enregistrent toutes les modifs faites à la base, ils sont orga en groupes d'1 ou plusieurs membres. Oracle les utilise de manière circulaire il en faut donc au moins 2 et on écrase régulièrement les anciennes données 
On se sert des redo logs pour restaurer la base après un arret anormal (really? je me demande si y a un rapport avec le nom...)
LGWR écrit en parallèle dans chaque membre d'un même groupe. Si un groupe contient plusieurs membres et qu'un est indisponible la base peut continuer à fonctionner. Il est donc conseillé de mettre minimum 2 à 3 membres par groupe.
Depuis la 9i on peut:
- forcer l'archivage périodiquement avec ARCIVE_LAG_TARGET
- gartantir une durée de max de restauratin d'instance en cas d'arret normal avec FAST_START_MTTR_TARGET
- utiliser plusieurs destinations d'archivage avec LOG_ARCHIVE_DEST_n avec n entre 1 et 10

On peut dimensionner les fichiers des redolog pour avoir:
- un basculement de redo log tt les 20 à 30 min
- ne pas avoir d'attente lors d'un basculement de fichié à cause d'un checkpoint ou d'un archivage non terminé 

On recommande aussi:
- si les basculement sont trop rapide d'augmenter sa taillle
- si le temps d'attente d'un basculement est trop long d'ajouter un groupe
- si la base est en Archivelog prévoir min 3 groupes
- si la base posède un DATA Guard en prévoir un 4eme voir un 5eme
- si la base est montée en RAC prévoir 2 groupes par noeud

Les opers d'aministration possibles sur les redologs sont:
- ajouter ajouter un groupe
- multiplexer les membres d'un groupe
- déplacer les fichiers
- supprimer un groupe
- supprimer un membre d'un groupe
- forcer le basculement d'un groupe courant au suivant


Pour multiplexer les redo logs on peut les lister les membres de chaque groupe dans la close LOGFILE de SQL CREATE DATABASE ou avec 

```
ALTER DATABASE ADD LOGFILE MEMBER ‘nom_fichier’ [,…] TO GROUP numéro ;
```
ex
```
ALTER DATABASE ADD LOGFILE MEMBER ‘f:\oracle\oradata\HERMES\redo01.log’ TO GROUP 1;
```

on peut ajouter un groupe avec
```
ALTER DATABASE ADD LOGFILE [GROUP numéro] spécification_fichier_redo [,…] ;
```
où les specs du fichier sont de forme 
```
(‘nom_fichier’ [,…]) [ SIZE valeur [K|M] ] [REUSE]
```

exemple 

```
ALTER DATABASE ADD LOGFILE GROUP 4 (‘d:\oracle\oradata\HERMES\redo04.log’, ‘e:\oracle\oradata\HERMES\redo04.log’) SIZE 10240K;
```
pour déplacer un redo log

```
Connect / as sysdba
Shutdown immediate 
Startup mount C:\> host copy ‘e:\oracle\oradata\tahiti\redo04.log’
‘g:\oracle\oradata\tahiti\redo04.log’ 
Alter database 
Rename file ‘e:\oracle\oradata\tahiti\redo04.log’ to ‘g:\oracle\oradata\tahiti\redo04.log’ ; 
Alter database open ; 
Suppression de l’ancien fichier à l’aide d’une commande du système d’exploitation.
```

```
- Arrêter la base de données 
SHUTDOWN IMMEDIATE 
- Déplacer les fichier de Redo Log vers le nouvel emplacement 
COPIER .. +.. COLLER 
- Monter la base 
STARTUP MOUNT 
- Indiquer à Oracle le nouvel emplacement 
ALTER DATABASE RENAME FILE 
- Ouvrir la base 
ALTER DATABASE OPEN 
```

**suppr un groupe de redo log**
- pour suppr un groupe il faut en avoir au moins 3
- on ne peut pas suppr le groupe courrant
- en mode archivelog un groupe qui n'est pas encore archivé ne peut pas être suppr
- les fichiers ne sont pas suppr par Oracle après, il faut donc le faire soit même

```
ALTER DATABASE DROP LOGFILE GROUP [numéro] ;
```

**suppr un membre d'un groupe**
- le groupe doit avoir au mons 2 membres
- un membre du groupe courrant ne peut pas être suppr
- en ARCHIVELOG il doit avoir été archivé
- fichier on suppr par Oracle
- pour suppr tous les memres il faut suppr le groupe
- Si un membre est corrompu il a le statut "invalide" dans le dico mais il n'y a pas de message, pour détecter qu'un membre est invalide il faut consulter le fichier des alertes

```
ALTER DATABASE DROP LOGFILE MEMBER ‘nom_fichier’ [,…] ;
```

**forcer le basculement du groupe courrant**

```
ALTER SYSTEM SWITCH LOGFILE ;
```
la vue V$LOG_HISTORY colone FIRST_TIME peut être utilisée pour analyser la fréquence des switch

FAST_START_MTTR_TARGET peut être utilisé pour obliger la base à faire des points de reprise régulièrement

**infos sur les redo logs**

plusieurs vues ont des infos sur les redo logs

- v$log: info sur les groupes
- v$logfile: info sur les membres
- v$log_history: info sur l'historique des fichiers redo log
- v$instance_recovery: infos sur les temps estimés de restauration d'instance

![](/pics/infoVues.jpg)
![](/pics/infoVues2.jpg)

### Sauvegarde et restauration
- DUMP

DUMP( expression [, return_format] [, start_position] [, length] )

ex:
 DUMP('Tech', 1016)
 Result:_ 'Typ=96 Len=4 CharacterSet=US7ASCII: 54,65,63,68'

retourne un varchar avec le datatype code, taille en bytes, et représentation interne de expr on peut spécifier le format 8,10,16 ou 17(1 seul char)

par defaut y a pas de chars et il faut faire 1000+base pour avoir des strings

- PUMP
- RMAN
- Archivelog
	
### Réalisation
- Corbeille

A: gestion des users
1 créer un user avec le tablespace tempo TEMP

CREATE USER Vincent IDENTIFIED BY vinc TEMPORARY TABLESPACE TEMP

3 attribuer à vincent le privilège system CREATE SESSION
grant create session to Vincent identified by vinc;

4 comme 1 mais en plus tablespace par défaut USERS limité à 10Mo
create user Theo identified by theo temporary tablespace temp default tablespace users quota 10M on users;

5 ajouter le privilèges de co et de créa de tables
grant create table to Theo;
grant connect to Theo;

6 profil autorisant 1 seule co et 2min d'inactivité et l'appliquer à théo
create profile une_deux limit sessions_per_user 1;
alter profile une_deux limit connect_time 2;
alter user Theo profile une_deux;

8 attribuer à théo les privilèges de créer une table département à partir de la table dept de Scoot
faut créer scoot, grant ses privilèges, créer la table dept
grant select on Scoot.dept to Theo;

9 voir la vue user_object

ptdr t'as crû v$user_objet éh ben non:
select * from user_objects;

12 autoriser vincent à modif la colonne ville de departement
 grant update(ville) on departement to Vincent;

15 créer un role et ajouter lesdroits de créer des tables, vues, séquences et synonymes 
create role developper identified globally
grant create sequence, create synonym to developper;

partie B 
1. a 
 CREATE TABLESPACE TS_TAB_AIRBASE DATAFILE '/app/Nico/oradata/orcl/ts_tab_airbasex1.dbf' SIZE 5M EXTENT MANAGEMENT LOCAL ;

CREATE TABLESPACE TS_IND_AIRBASE DATAFILE '/app/Nico/oradata/orcl/ts_ind_airbasex1.dbf' SIZE 2M EXTENT MANAGEMENT LOCAL ;


2 inserer ds données dans une tabl pilote créée
create table pilote (name varchar(100));
sur admin : alter user nico quota unlimited on USERS
insert into pilote values ('michel')


CREATE INDEX emp_ename ON emp(ename)
      TABLESPACE users
      STORAGE (INITIAL 20K
      NEXT 20k
      PCTINCREASE 75);
      

12 
blockrecover datafile [nombre] bloc [numéro du bloc a recup]

restore datafile [numéro du tablespace]
recover datafile [same num]

restore controlfile
