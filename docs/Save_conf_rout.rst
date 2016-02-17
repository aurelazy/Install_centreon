.. _ref_save_conf_rout:

*************************************
Sauvegarde configuration routeurs
*************************************

Pour la sauvegarde de la configuration des routeurs, nous avons 2 possibilités:
	* Sauvegarde automatique
	* Sauvegarde manuelle


Pour notre projet, une sauvegarde manuelle sera effectué. En effet, la configuration de notre réseau ne nous permet pas d'installer les paquets, modules nécessaire pour une sauvegarde automatique, et de plus cela permettra de travailler avec la commande ``expect``.
Pour sauvegarder les configurations, nous devrons lancer manuellement le script puis entrer les mots de passe.


Voici le pas à pas:
	#. :ref:`Création d'un fichier host <file_host>`
	#. :ref:`Ajout d'un nouvel hôte <add_host>`
	#. :ref:`Ajout des paquets utiles au script <paquet>`
	#. :ref:`Création d'un script <create_script>`
	#. :ref:`Lancement du script <send_script>`

.. _file_host:

Création d'un fichier host
---------------------------

Pour le script, nous avons besoin d'un fichier contenant tous les routeurs de notre réseau, pour cela, nous allons utiliser un fichier présent dans tous les systèmes d'exploitation GNU/Linux, le fichier ``/etc/hosts``

.. code-block:: bash

	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
	192.168.10.121		TOTO

	# Liste Routeurs
	192.168.0.1	RT_2001
	192.168.0.2	SW_2002
	192.168.0.3	RT_2003
	192.168.0.7	RT_2007
	192.168.0.8	RT_2008
	192.168.0.9	RT_2009
	192.168.0.11	RT_2011
	192.168.0.12	RT_2012
	192.168.0.13	RT_2013
	192.168.0.15	RT_2015
	192.168.0.16	RT_2016
	192.168.0.17	RT_2017
	192.168.0.18	RT_2018
	192.168.0.21	RT_2021
	192.168.0.22	RT_2022
	192.168.0.23	RT_2023
	192.168.0.24	RT_2024
	192.168.0.27	RT_2027
	192.168.0.28	RT_2028
	192.168.0.33	RT_2033
	192.168.0.34	RT_2034
	192.168.0.35	RT_2035
	192.168.0.36	RT_2036
	#192.168.0.37	RT_2037
	192.168.0.39	RT_2039
	192.168.0.40	RT_2040
	192.168.0.43	RT_2043
	192.168.0.47	RT_2047
	192.168.0.48	RT_2048

On peut voir dans ce fichier que nous avons l'adresse IP qui est en relation avec le nom du routeur.
Lorsque nous faisons un ``ping RT_2001`` la requête aboutie, (normal en même temps !)


.. _add_host:

Ajout d'un nouvel hôte
-----------------------

Lors de l'ajout d'un nouveau routeur/commutateur, il est important de remplir le fichier précédent de la façon suivante:
Avec VIM

.. code-block:: bash

	# vim /etc/hosts

Pour accéder à la fin du fichier il suffit de taper:

.. code-block:: bash

	Ctrl+G
	o (la lettre o en minuscule)

Nous entrons en mode ``INSERTION``
Puis taper:

.. code-block:: bash

	192.168.0.xxx	RT_20xx

.. warning:: 

	L'espace est une tabulation

Puis enregistrer le fichier:

.. code-block:: bash

	Echap ==> :x


.. _paquet:

Ajout des paquets utiles au script
-----------------------------------

Pour le bon fonctionnement du script, nous avons eu besoin d'installer les paquets suivant
	* tcl
	* expect
	* telnet

Tcl est une dépendance du paquet expect, c'est un langage de script.

"Expect est un outil d'automation et de tests de non-regression, …, comme extension au langage de script Tcl pour tester des applications interactives comme telnet, passwd, fsck, rlogin, ssh ou bien d'autres." [1]_


C'est-à-dire qu'il va permettre de répondre, d'une façon interactive, aux demandes de login et mot de passe lors de notre connexion en telnet à nos équipements.

Telnet est un protocole qui permet de communiquer entre équipements, attention ce protocole n'est pas du tout sécurisé, il est recommandé d'utiliser SSH à la place, mais bon pour certains projet, on a pas le choix ...

Pour installer ces paquets il suffit de récupérer ceux-ci sur internet dans les dépôts officiels de notre système d'exploitation et de lancer la commande suivante:

.. code-block:: bash

	# rpm –ivh tcl-8.5.7-6.el6.x86_64.rpm

Et idem pour les 2 autres.

.. _create_script:

Création d'un script
---------------------

Ce script demande les mots de passe pour se connecter aux routeurs, ainsi que le mot de passe ``enable`` pour les équipements CISCO.
Il récupère le nom des routeurs dans le fichier ``/etc/hosts``,
Puis il communique avec l'équipement en telnet et envoie les informations de la configuration dans un fichier au nom de l'équipement dans le dossier ``/home/<user>/sauvegardes/routeurs/``
Enfin il édite le fichier en enlevant les informations superflues.

Voici le script ``sauv_conf_routers.sh``:
"En commentaire", les explications.

.. code-block:: bash

	#!/bin/bash
	#sauv_conf_routers.sh

	echo "Veuillez donner le mot de passe pour telnet"
	# Permet de cacher la saisie du mot de passe
	stty –echo
	read pwdTelnet
	# Reviens au mode saisie normal
	stty echo

	echo "Veuillez donner le mot de passe enable"
	stty -echo
	read pwdEnable
	stty echo

	export telnet='./telnet.sh'
	# On recupere le nom des routeurs
	export routeurs=`cat /etc/hosts | sed -e '/^#\|^127\|::1\|10\.241\.103\.221\|^$/d'| cut -f2`
	export pwdTelnet
	export pwdEnable
	export temp='./tmp_routeur.log'


	for routeur in $routeurs
	do
		rm -f $telnet
		echo $routeur
		export routeur

		case $routeur in
		# On prend les routeurs Juniper (pas meme commande telnet), IL FAUT REMPLIR CETTE LIGNE A CHAQUE AJOUT DE ROUTEUR JUNIPER !!!
			RT_2035|RT_2036|RT_2047|RT_2048)
				# On cree un fichier tenet.sh avec les commandes a envoyer au routeur
				echo 'expect 2>&1 << EOF'>> $telnet
				echo 'spawn telnet $routeur'>> $telnet
				echo 'sleep 5' >> $telnet
				echo 'expect "login:"' >> $telnet
				echo 'send "admin\r"' >> $telnet
				echo 'expect "Password:"' >> $telnet
				echo 'send "${pwdTelnet}\r"' >> $telnet
				echo 'expect ">"' >> $telnet
				echo 'send "show configuration | no-more\r"' >> $telnet
				echo 'expect ">"' >> $telnet
				echo 'send "exit\r"' >> $telnet
				echo 'expect "closed"' >> $telnet
				echo 'exit' >> $telnet
				echo 'EOF' >> $telnet
				;;
			\*)
				# Ici tous les routeurs CISCO
				echo 'expect 2>&1 << EOF'>> $telnet
				echo 'spawn telnet $routeur'>> $telnet
				echo 'sleep 5' >> $telnet
				echo 'expect "Password:"' >> $telnet
				echo 'send "$pwdTelnet\r"' >> $telnet
				echo 'expect ">"' >> $telnet
				echo 'sleep 5' >> $telnet
				echo 'send " enable\r"' >> $telnet
				echo 'expect "Password:"' >> $telnet
				echo 'send "${pwdEnable}\r"' >> $telnet
				echo 'expect "#"' >> $telnet
				# Pour ne pas avoir le resultat coupe par un "MORE", reviens a la normal apres reconnexion
				echo 'send " terminal length 0\r"' >> $telnet
				echo 'expect "#"' >> $telnet
				echo 'send " show running-config\r"' >> $telnet
				echo 'expect "#"' >> $telnet
				echo 'send " exit\r"' >> $telnet
				echo 'expect "closed"' >> $telnet
				echo 'exit' >> $telnet
				echo 'EOF' >> $telnet
				;;
		esac
		# On change les droits sur le fichier pour pouvoir le lancer
		chmod 755 $telnet
		# On lance la commande et on met les résultats dans le fichier $routeur
		$telnet > ../sauvegardes/routeurs/$routeur 2>&1
															    
		# On nettoie le fichier
		eraseDebut=`grep -n version ../sauvegardes/routeurs/$routeur | cut -d":" -f1`
		sed -i "1,${eraseDebut}d" ../sauvegardes/routeurs/$routeur
		eraseFIN=`grep -n exit ../sauvegardes/routeurs/$routeur | cut -d":" -f1`
		sed -i ${eraseFIN},10000d ../sauvegardes/routeurs/$routeur
	done

.. _send_script:

Lancement du script
--------------------

Pour lancer le script, il suffit de se connecter sur le serveur avec Putty (si PC Windows), sur le serveur 168.192.10.121
Ensuite se rendre sur ``/home/<user>/script_sauvegarde``

On lance le script de la façon suivante:

.. code-block:: bash

	# ./sauv_conf_routeurs.sh

Lorsque l'on lance la commande, celle-ci demande le mot de passe Telnet:

.. code-block:: bash

	Veuillez donner le mot de passe pour telnet

 
Puis le mot de passe pour le mode ``enable``:

.. code-block:: bash

	Veuillez donner le mot de passe enable


Ensuite le script tourne tout seul et récupère les configurations de nos routeurs/commutateurs.

Les fichiers de sauvegarde se trouvent dans le dossier suivant:

.. code-block:: bash

	/home/<user>/sauvegardes/routeurs/

Et les fichiers sont au nom de l'équipement, ex: RT_2001



.. [1] Tiré de la page Wikipédia.
