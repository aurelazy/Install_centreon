.. _ref_mysql:

***********************************
Création utilisateur MySQL
***********************************

Pour que la sauvegarde des BDD fonctionne, nous devons (pour des raisons de sécurité) créer un utilisateur qui n'aura que des droits pour la sauvegarde.
Il suffit de taper les commandes suivantes:

.. code-block:: bash

	$ mysql – u root –p
	Enter password : <mot_de_passe_root_MySQL>

.. code-block:: mysql

	MariaDB [(none)]> CREATE USER 'backup'@'localhost' IDENTIFIED BY 'backup';
	MariaDB [(none)]> GRANT SHOW DATABASES, SELECT, LOCK TABLES, RELOAD ON *.* TO 'backup'@'localhost';
	MariaDB [(none)]> FLUSH PRIVILEGES;
	MariaDB [(none)]> exit;
