.. _ref_save:

****************************
Sauvegardes
****************************

On va sauvegarder les fichiers important de Centreon, ainsi que toutes les BDD tous les soirs grâce à 2 scripts de sauvegarde qui seront lancés par une tâche cron. 
Les scripts se trouvent dans le dossier "/home/<user>/script_sauvegarde/".
Le premier, celui qui sauvegarde la configuration ainsi que les fichiers/scripts/plugins …, se nomme "sauvegarde.sh", le second qui sauvegardera nos BDD Centreon se nomme "sauv_mysql.sh".

.. _save_conf:

Sauvegarde des fichiers de configuration Centreon
---------------------------------------------------

Voici le script ``sauvegarde.sh``.

.. code-block:: bash

	#!/bin/bash

	DATE=`date +%y%m%d`
	rep=/var/spool/centreon/dump/
	dest=/home/backup/sauvegardes/dump_centreon/dump_centreon.${DATE}.tar.gz

	# Création du répertoire à compresser
	mkdir ${rep}
	# Suppression de l'ancienne sauvegarde
	#rm -f ${dest}

	mkdir ${rep}httpdconf/
	mkdir ${rep}nagioskeys/
	mkdir ${rep}rootkeys/

	NbArchive=$(ls -A /home/backup/sauvegardes/dump-centreon/ | wc -l)
	if [${NbArchive} -gt 7 ]
	then
		# On recupere l'archive la plus ancienne 
		Old_ackup=$(ls -lrt /home/backup/sauvegardes/dump_centreon/ |grep ".tar.g" |head -n 1 |cut -d":" -f 2 |cut -d " " -f 2)
		rm -f /home/backup/sauvegardes/dump_centreon/${Old_backup}
	fi

	# Sondes
	printf "Sauvegarde des sondes \n";
	cp -ra /usr/lib/nagios/plugins ${rep}
				
	# Modules Perl
	#printf "Sauvegarde des modules PERL \n";
	#cp -ra /usr/lib/perl5/ ${rep}

	# Modules Python
	printf "Sauvegarde des modules Python \n";
	cp -ra /usr/lib/python2.6/ ${rep}

	# Images
	printf "Sauvegarde des images \n";
	cp -ra /usr/share/centreon/www/img ${rep}

	# Fichier RRD
	printf "Sauvegarde des fichiers RRD \n";
	cp -ra /var/lib/centreon/status ${rep}
	cp -ra /var/lib/centreon/metrics ${rep}

	# Fichiers MIBS
	printf "Sauvegarde des fichiers de MIBS \n";
	cp -ra /usr/share/snmp/mibs ${rep}

	# Fichier de conf HTTP
	printf "Sauvegarde des fichiers de configuration Apache \n";
	cp -ra /etc/httpd/conf ${rep}httpdconf/
	cp -ra /etc/httpd/conf.d ${rep}httpdconf/

	# Fichier PHP.INI
	printf "Sauvegarde du fichier php.ini \n";
	cp -ra /etc/php.ini ${rep}

	# Clé privée + publique
	#printf "Sauvegarde des clés publique et privées de root \n";
	#cp -ra /root/.ssh/ ${rep}rootkeys/

	# Compression
	printf "Compression de la sauvegarde \n";
	tar czvf ${dest} ${rep}

	# Chongement proprietaire
	Chown backup:backup ${dest}

	# Suppression des fichiers non compressé
	printf "Suppression du repertoire \n";
	rm -rf ${rep}*
		
	# Message de fin
	printf "Sauvegarde terminée, le fichier de sauvegarde est ${dest} \n";
	
.. _save_mysql:	

Sauvegarde des BDD MySQL
--------------------------

Le script ``sauv_mysql.sh``.

.. code-block:: bash

	#!/bin/bash

	if [ $# -eq 1 ]
	then
		DATE=`date +%y%m%d`
		rep=/var/spool/centreon/dump_mysql/
		dest=/home/backup/sauvegardes/dump_mysql/dump_mysql.${DATE}.tar.gz
					

		# Création du répertoire à compresser
		mkdir ${rep}
		# Suppression de l'ancienne sauvegarde
		#rm -f ${dest}

		# On compte le nombre d'archives presentes dans le dossier
		NbArchive=$(ls -A /home/backup/sauvegardes/dump_mysql/ |wc -l)
		# S'il y a plus de 4 archives, on supprime la plus ancienne
		if [ ${NbArchive} -gt 7 ]
		then
			# On recupere l'archive la plus ancienne
			Old_backup=$(ls -lrt /home/backup/sauvegardes/dump_mysql/ |grep ".tar.gz" |head -n 1 | cut -d":" -f 2 | cut -d" " -f 2)
			# On supprime l'archive la plus ancienne
			rm -f /home/backup/sauvegardes/dump_mysql/$Old_backup
		fi

		# On met les DB a sauvegarder dans une variable:
		names=( centreon centreon_storage centreon_status )	

		# On bloque les tables le temps de la replication
		# A FAIRE

		for name in ${names[@]}
		do
			printf "Sauvegarde de la BDD : $name \n";
			mysqldump -u backup --single-transaction -p$1 $name > ${rep}$name.sql
		done

		# Compression
		printf "Compression de la sauvegarde \n";
		tar czvf ${dest} ${rep}

		# Changement de proprietaire
		chown backup:backup ${dest} 

		# Suppression des fichiers non compressé:
		printf "Suppression du repertoire \n";
		rm -rf ${rep}

		# Message de fin
		printf "Sauvegarde terminée, le fichier de sauvegarde est ${dest} \n";
	else
		printf "Utilisation : ./sauv_mysql.sh <mdp_mysql_backup>, \nil est preferable d'executer ce script avec l'utilisateur root";
	fi


Ensuite, nous planifions une tache "cron".
Cron est un programme qui permet aux utilisateurs des systèmes Unix d’exécuter automatiquement des scripts, des commandes ou des logiciels à une date et une heure spécifiées à l’avance, ou selon un cycle défini à l’avance.

Pour vérifier, les taches en cours il suffit de taper la commande suivante:

.. code-block:: bash

	# crontab -L

Pour l'éditer nous devons taper:

.. code-block:: bash

	# crontab -e

Cette commande va ouvrir l'éditeur de texte VIM pour éditer les taches.

Voici les taches en cours sur le serveur au moment de l'écriture de la documentation:

.. code-block:: bash

	# m h dom mon dow command
	#Sauvegarde conf centreon tous les jours à 00h01
	01 00 * * * /home/<user>/script_sauvegarde/sauvegarde.sh
	# Sauvegarde MySQL tous les jours à 00h30
	30 00 * * * /home/<user>/script_sauvegarde/sauv_mysql.sh backup 


Récupération des sauvegardes
-----------------------------

Disons que nous voulons récupérer cette sauvegarde depuis un poste client Windows (oui, oui ça peut arriver !!)

Pour pouvoir faire cette sauvegarde nous allons tout d'abord créer un compte, sur le serveur qui ne pourra faire que cela.


Coté serveur
~~~~~~~~~~~~~

Sur le serveur:

.. code-block:: bash

	# useradd backup 
	# passwd backup

On crée l'utilisateur backup et on lui assigne comme mot de passe 'backup'

Ensuite, nous créons un bash restreint ('rbash'), si pas déjà créé.

.. code-block:: bash

	# cp /bin/bash /bin/rbash


Voici les restrictions de ``rbash``:

	#. Changing directories with cd
	#. Setting or unsetting the values of SHELL, PATH, ENV, or BASH_ENV
	#. Specifying command names containing /
	#. Specifying a file name containing a / as an argument to the . builtin command
	#. Specifying a filename containing a slash as an argument to the -p option to the hash builtin command
	#. Importing function definitions from the shell environment at startup
	#. Parsing the value of SHELLOPTS from the shell environment at startup
	#. Redirecting output using the >, >|, , >&, &>, and >> redirection operators
	#. Using the exec builtin command to replace the shell with another command
	#. Adding or deleting builtin commands with the -f and -d options to the enable builtin command
	#. Using the enable builtin command to enable disabled shell builtins
	#. Specifying the -p option to the command builtin command
	#. Turning off restricted mode with set +r or set +o restricted

On donne des droits plus élevés à nos fichiers de configuration de l'utilisateur pour qu'il ne puisse pas les modifier:

.. code-block:: bash

	# chown root. /home/backup/.bash_profile
	# chmod 755 /home/backup/.bash_profile
	# chown root. /home/backup/.bashrc
	# chmod 755 /home/backup/.bashrc

Puis on change sa variable d'environnement pour qu'il ne puisse utiliser aucune commande:

.. code-block:: bash

	vim /home/backup/.bash_profile

On change la ligne:

.. code-block:: bash

	PATH=$PATH:$HOME/bin

Par:

.. code-block:: bash

	PATH:$HOME/bin

L'utilisateur n'a plus accès aux commandes système, il ne peut rien faire.


Coté client
~~~~~~~~~~~~~

ur le poste Windows, nous allons créer des dossiers pour accueillir les sauvegardes.
Dans le dossier Documents de l'utilisateur nous créons les dossiers suivant:
sauvegardes/centreon
sauvegardes/mysql

Dans le dossier "centreon", nous récupérerons les sauvegardes des fichiers de configuration, des plugins … (dump_centreon…tar.gz)
Dans le dossier "mysql", les sauvegardes des BDD (dump_mysql….tar.gz)

Le script de sauvegarde est dans le dossier, "Documents/scripts/Script_sauvegarde"
Pour ce script, nous utilisons le programme "pscp.exe" qui se trouve sur le "Bureau".

Voici le script ``sauv_centreon.bat``.

.. code-block:: batch

	cd "C:\Documents and Settings\<user>\Documents\sauvegardes\centreon"

	"C:\Documents and Settings\<user>\Bureau\pscp.exe" –unsafe –scp –pw backup backup@IP:/home/backup/sauvegarde/dump_centreon/* .

	cd "C:\Documents and Settings\<user>\Documents\sauvegardes\mysql"

	"C:\Documents and Settings\<user>\Bureau\pscp.exe" –unsafe –scp –pw backup backup@IP:/home/backup/sauvegarde/dump_mysql/* .



Tâche planifiée
~~~~~~~~~~~~~~~~

Maintenant, nous allons créer une tâche planifiée sur le client windows.

Menu démarrer ==> Panneau de configuration ==> Tâches planifiées ==> Création d'une tâche planifiée.

L'assistant pour les tâches planifiées apparait:
	* Cliquer sur "Suivant"
	* Parcourir…
	* Récupérer le script de sauvegarde sur le Documents/scripts/Script_sauvegarde/sauv_centreon
	* Puis on choisi d'éxécuter cette tâche tous les jours
	* Cliquer sur "Suivant"
	* Heure de début = 01:00
	* Cliquer sur "Suivant"
	* Entrer le mot de passe de session

La tâche est opérationnelle et sera lancé tous les jours à 1h00 du matin.
