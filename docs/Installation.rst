**************
Installation
**************

Nous n'allons pas expliquer pas à pas l'installation de Centreon (leur `site <http://documentation-fr.centreon.com/docs/centreon/fr/2.5.x/guide_utilisateur/01a.html>`_ le fait très bien).
Nous allons voir les ajouts que nous avons fait, les changements…

Nous verrons dans les prochains chapitres:

	* :ref:`La création d'un hôte <ref_hote>`
	* :ref:`La création d'un service <ref_service>`
	* :ref:`Les graphiques <ref_graph>`
	* Les plugins :ref:`Les plugins <ref_plugins>`
	* Etc ...

Sur le serveur le compte par défaut est ``root``. Nous allons ajouter un autre utilisateur.
Nous arrons donc 2 utilisateurs:

	* root
	* aurelazy # Biensûr vous le nommer comme vous voulez

La BDD contient déja 2 utilisateurs, ``root``, ``centreon``.
Pour notre projet, nous créerons 1 utilisateur en plus pour les sauvegardes, nous le nommerons ``backup``.

Donc voici nos différents comptes pour MySQL:

	* root # Attention à bien ajouter un mot de passe, car vide
	* centreon
	* backup


