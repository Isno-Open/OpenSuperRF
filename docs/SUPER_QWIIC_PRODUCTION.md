# Super Qwiic — paramètres de production

## Révision de référence

- PCB : `openprofalux-pcb.kicad_pcb`
- 4 couches, épaisseur finie 1,6 mm
- cuivre externe 1 oz, cuivre interne 0,5 oz
- empilage à sélectionner chez JLCPCB : **JLC04161H-7628**
- contrôle d'impédance : **oui, 50 ohms simple-ended pour les liaisons RF**

Référence fabricant : <https://jlcpcb.com/impedance>

L'empilage JLC04161H-7628 place le plan `In1.Cu` à environ 0,2104 mm de
`F.Cu`. `In1.Cu` doit rester un plan GND continu sous toute la chaîne RF.

## Routage RF

Les tronçons principaux RF sur `F.Cu` utilisent 0,405 mm au-dessus du plan
GND `In1.Cu`. Les réductions à 0,25 mm et 0,15 mm sont limitées aux fanouts
des pads 0402 et du boîtier SC-70-6 ; elles doivent rester aussi courtes que
possible et ne constituent pas des lignes 50 ohms uniformes.

Chaîne :

`U2 868 -> C9 -> U6` / `U5 433 -> C10 -> U6` / `U6 -> C11 -> J3`

Une validation VNA sur le premier prototype reste requise avant une série :
mesurer S11 à 433,92 MHz et 868,3 MHz avec l'antenne finale et le boîtier.

## Contrôles obligatoires avant commande

1. Exécuter ERC et DRC avec la même version KiCad que celle utilisée pour les exports.
2. Vérifier `0 erreur` et `0 connexion non routée`.
3. Régénérer Gerbers, BOM et CPL avec `ISNO_VARIANT=super ./gen_fab.sh`.
4. Dans JLCPCB, choisir 4 couches / 1,6 mm / JLC04161H-7628 / impédance 50 ohms.
5. Contrôler visuellement les rotations de J1, J3, J4, D1, D2, U2, U5 et U6.
6. Ne jamais substituer U3 par un AMS1117 : la BOM de référence utilise AP2114H.
7. R12 et R13 (10 kΩ) maintiennent respectivement `CSN` et `CSN2` à `+3V3` pendant
   le reset de l'ESP32-S3 : les deux radios CC1101 restent matériellement désélectionnées.
8. J3 est assemblé : **BAT WIRELESS BWSMA-KWE-Z001 / LCSC C496551**, SMA femelle
   coudé THT 50 ohms, avec footprint `BWSMA-KWE-Z001`. Toute substitution de J3 exige
   une nouvelle validation MPN ↔ datasheet ↔ footprint ↔ mécanique ↔ RF avant commande.

## Alimentation des extensions

Exigence d'alimentation du produit : **USB 5 V / 1 A minimum**. Utiliser une
source **5 V / 2 A recommandée** lorsque Qwiic et mikroBUS sont chargés
simultanément avec les deux radios. Éviter un port USB de PC limité à 500 mA et
utiliser un câble court de bonne qualité. Un chargeur plus puissant ne force pas
son courant nominal dans la carte : la carte ne prélève que le courant requis.

J2 expose `+5V_MB` et `+3V3_MB`, protégés respectivement par U8 et U7 ; J4 expose
`+3V3_QWIIC`, protégé par U6. Les SGM2553 assurent limitation de courant,
signalement de défaut et blocage du courant inverse. Ne jamais injecter une
autre alimentation simultanément sur une sortie d'extension et sur l'USB-C.

La simulation ngspice qualifie le cumul Wi-Fi TX + double RX + 50 mA sur chacun
des trois rails avec une source 4,75 V / 1 A et un câble résistif de 0,8 ohm :
VBUS reste à environ 4,31 V et +3V3 à 3,295 V. Le même cumul sur une source
limitée à 500 mA est volontairement hors spécification et peut provoquer un
brownout/reset, sans constituer un défaut de la carte.

Les avertissements DRC de sérigraphie du banc pogo et des connecteurs de bord
n'affectent pas le cuivre, les perçages, le masque ou la connectivité. Ils
peuvent être nettoyés graphiquement sans déplacer les composants.

## Contrôles DRC

Les catégories DRC de fabrication précédemment mises à `ignore` ont été réactivées :
- ces checks sont actifs (`missing_courtyard`, `npth/pth_inside_courtyard`,
  `track_not_centered_on_via`, `footprint_type_mismatch`, `footprint_filters_mismatch`,
  `tuning_profile_track_geometries`) - **0 cas** sur le board.
- `lib_footprint_mismatch` est la seule catégorie ignorée : U1, U2 et J2 sont des copies
  embarquées volontairement adaptées au board figé. Cette alerte générique ne valide ni
  leurs pads ni leur netlist ; le comparateur pin-par-pin ci-dessous couvre la divergence
  électrique qui importe. Toutes les autres catégories restent actives.

Le pipeline de fabrication compare aussi automatiquement chaque pin connectée du schéma
avec le pad correspondant du PCB et s'arrête à la moindre divergence.

DRC : **0 erreur, 0 avertissement**. ERC : **0**. Le symbole E07 est nommé
`E07-900MM10S` (= composant réel).
