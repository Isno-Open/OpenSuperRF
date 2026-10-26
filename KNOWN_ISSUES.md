# Points ouverts, v1.0 (EVT) / Known issues, v1.0 (EVT)

Cette v1.0 est la série de **prototypes EVT**, mesurée et fonctionnelle, mais avec des points
ouverts connus, relevés après le gel et la commande des cinq cartes. Rien ici ne rend les
prototypes inutilisables ; tout ici doit être tranché avant une série. Ils sont listés pour que
qui fabrique ou étudie la carte le sache, dans l'esprit de l'open hardware honnête.

*This v1.0 is the EVT prototype batch: measured and working, with known open points found after
freeze. Listed here so anyone building or studying the board knows them.*

## ⚠️ Critique : brochage de J2 (mikroBUS)

**Un cordon IDC droit sur J2 est destructeur.** Le brochage de J2 est inversé par rapport au
HM2X8 du mikroBUS Shuttle (MIKROE-2880) : pour chaque signal, notre numéro plus le leur font 17.
Un cordon droit met notre **+5V sur leur RST** et notre **+3V3 sur leur INT**, et court-circuite
les deux alimentations sur des entrées.

Il faut **croiser dans l'adaptateur** (l'adaptateur de pas 2,54 vers 1,27, purement passif) ou
inverser sur la carte lors d'un prochain respin. Ne jamais relier J2 à une carte Shuttle par un
cordon droit.

*A straight IDC cable on J2 is destructive (it shorts +5V/+3V3 onto inputs). Use the adapter or
respin; never connect J2 to a Shuttle with a straight cable.*

## RF 433 MHz : ne pas viser pile la limite

Avec 10 dBm conduits et la limite de 10 mW ERP en 433, le budget de gain d'antenne est nul : un
dipôle demi-onde (2,15 dBi) place le produit pile sur la limite. Recommandation (labo de
certification) : plafonner le gain d'antenne déclaré **et** descendre la puissance conduite à
8 ou 9 dBm en 433. L'antenne n'est pas figée : prévoir une antenne à faible gain.

## Boîtier : ouverture mikroBUS

L'ouverture du capot au-dessus du mikroBUS est taillée pour une prise plate nue ; une prise à
serre-câble ne passe pas.

## Sérigraphie

Sur les prototypes EVT, la sérigraphie est vide : aucun connecteur n'y est nommé. Se référer au
**brochage** dans le README (`images/super-brochage.svg`).

---

Une révision suivante corrigera ou tranchera ces points. Ils sont donnés tels quels, par
transparence.
