.. _ref_script:

************
Les scripts
************

oici les explications des changements fait sur le plugin "/usr/lib/nagios/plugins/check_iftraffic3.pl".

Le script ne prenait pas en compte l'état administratif du port, c'est-à-dire que si le port était ouvert aucune alarme n'était remontée.
Rajout également du nom du lien dans la sortie détaillée.
Ligne 99, ajout de la variable "$snmpIfAdminStatus = '1.3.6.1.2.1.2.2.1.7'"
Ligne 107, ajout de la variable "$snmpDescrLien = '1.3.6.1.2.1.31.1.1.1.18'"
Ligne 118, ajout de la variable "if_status_admin", qui va permettre de récupérer le résultat de la commande snmp sur l'OID précédente, à la ligne 279.
Lignes 286-296, ajout d'une comparaison du résultat récupéré avec le code retour pour UP (1), si celui-ci n'est pas égal à 1 alors le service sera en OK et on sort du script car le port sera fermé DOWN (2) !
Ligne 289, ajout de la variable "$Descr_Lien", qui va récupérer le nom du lien, avec la ligne 260.

Changement dans le retour des données de performance du plugin. Les données "inAbsolut" et "outAbsolut" sortaient des graphiques incompréhensibles et non lisibles.

J'ai donc enlevé ces informations du plugin, lignes 287, 425.

NB: Si vous avez besoin du script n'hésitez pas à me demander depuis le compte `GitHub <https://github.com/aurelazy/Install_centreon>`_.
