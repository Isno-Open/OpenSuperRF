# Identification de la carte par le firmware

Décision du 2026-09-10. Contrainte posée par Olivier : **un firmware unique pour toute
la gamme**, et **la Super v3 est déjà partie en production**, donc elle ne peut plus
recevoir le moindre composant.

## Le problème

Un firmware unique doit savoir sur quelle carte il tourne, pour choisir la bande, le
nombre de radios et les broches. Deux cas ne sont pas devinables par le logiciel seul :

- **Super contre Solo** : devinable en interrogeant la seconde radio sur `CSN2` en SPI.
- **Solo 433 contre Solo 868** : **indevinable**. Le module E07 a la même empreinte, le
  même brochage et le même code produit vu du CC1101, qui ne connaît pas sa propre
  bande. C'est ce cas qui impose un identifiant matériel.

## Les broches, et pourquoi celles-là

Les 15 GPIO libérées sur la Solo par le retrait du mikroBUS et de la 2e radio
(`MB_*`, `GDO0B`, `GDO2B`, `CSN2`, `nFAULT_M5`, `EXT_RX`, `EXT_TX`) sont **toutes
utilisées sur la Super**. Aucune ne peut donc servir d'identifiant commun à la gamme.

Neuf broches de `U1` n'ont de net **ni sur la Super ni sur la Solo** : 7, 18, 27, 28,
30, 37, 38, 41, 44. Sept sont à écarter :

> ⚠️ **CORRIGÉ le 2026-09-10 au soir.** Une première version de ce document annonçait
> `ID0 = GPIO4 (U1.7)` et `ID1 = GPIO15 (U1.18)`. **C'était faux** : la correspondance
> broche vers IO avait été lue avec une expression régulière qui appariait un nom avec
> le mauvais numéro. La datasheet du module (pin 4 = IO0, pin 5 = IO1, pin 6 = IO2,
> **pin 7 = IO3**) et une relecture du symbole avec un analyseur qui compte les
> parenthèses donnent le même résultat. Et `GPIO3` est une **broche de strapping**
> (source du signal JTAG), donc à écarter. Ne pas se fier à une copie antérieure.

Brochage réel des neuf broches libres, vérifié deux fois :

| Broche | IO réel | Verdict |
|---|---|---|
| U1.7 | **GPIO3** | ⚠️ **strapping** : source du signal JTAG. Écartée |
| U1.18 | **GPIO14** | libre, sans rôle au démarrage. **Retenue** |
| U1.27 | **GPIO47** | libre. **Retenue** |
| U1.28 | GPIO33 (`SPIIO4`) | réservée sur les modules à PSRAM octale, à ne pas figer |
| U1.30 | **GPIO48** | libre. **Réserve** pour un troisième bit |
| U1.37 | GPIO41 (`MTDI`) | TDI du JTAG |
| U1.38 | GPIO42 (`MTMS`) | TMS du JTAG, perdre le débogage matériel serait cher |
| U1.41 | GPIO45 | ⚠️ **strapping** : tension `VDD_SPI` |
| U1.44 | GPIO46 | ⚠️ **strapping** : mode de démarrage et impression ROM |

Les quatre broches de strapping de l'ESP32-S3 sont `GPIO0`, `GPIO3`, `GPIO45` et
`GPIO46`. Trois d'entre elles figurent dans les broches libres : c'est précisément
pourquoi il fallait vérifier plutôt que de se fier au numéro de broche.

Restent les deux retenues :

- **`ID0` = GPIO14 (U1.18)**, aussi `TOUCH14`, `ADC2_CH3` et `FSPIWP`
- **`ID1` = GPIO47 (U1.27)**, aussi `SPICLK_P`, utilisé seulement avec une flash ou une
  PSRAM octale, absente de ce module

Toutes deux sont de vrais GPIO, **sans rôle au démarrage**, et libres sur **toute** la
gamme. `GPIO48` (U1.30) est gardée en réserve si un troisième bit devenait nécessaire.

## Le schéma retenu : deux straps, lus en trois états

Une empreinte de résistance 10 kΩ **vers GND** par broche, non montée par défaut :
`R30` sur `ID0`, `R31` sur `ID1`. Deux empreintes au total, une seule montée par
variante.

Les désignateurs `R30` et `R31` sont libres sur **les deux** cartes : la Super va
déjà jusqu'à `R29`. Un même repère désigne donc la même chose dans toute la gamme,
ce qui compte pour une documentation et un firmware partagés. `R20` et `R21`
auraient été un piège, ils existent sur la Super et y désignent autre chose.

La lecture ne se contente pas d'un niveau. Le firmware lit **deux fois**, une fois avec
le pull-up interne, une fois avec le pull-down :

| Lecture pull-up | Lecture pull-down | État réel |
|---|---|---|
| 1 | 0 | **en l'air**, aucune résistance montée |
| 0 | 0 | tenue à **GND** par une résistance |
| 1 | 1 | tenue à **3V3** (non utilisé aujourd'hui, réservé) |

C'est ce qui rend la décision compatible avec une Super déjà fabriquée : ses deux
broches ne sont reliées à rien, et « en l'air » devient un **code positif**, mesuré,
au lieu d'une supposition sur un niveau haut.

## La table

| ID1 | ID0 | Carte | Résistance montée |
|---|---|---|---|
| en l'air | en l'air | **Super** (433 + 868, diplexeur) | aucune, rien à changer sur la carte partie en prod |
| GND | en l'air | **Solo 868** | `R31` seule |
| en l'air | GND | **Solo 433** | `R30` seule |
| GND | GND | réservé | les deux |

La différence entre une Solo 433 et une Solo 868 reste donc **deux lignes de
nomenclature** : le module (`C5844213` au lieu de `C5844212`) et la position de
l'unique résistance. Même cuivre, même empreinte, même brochage.

## Marge de manœuvre, sans figer davantage

Olivier a demandé « juste ce qu'il faut », donc on s'arrête à quatre cartes
identifiables. Mais la lecture en trois états laisse **neuf combinaisons** accessibles
sur ces deux mêmes broches, sans en réserver une troisième : il suffira d'ajouter une
empreinte vers 3V3 le jour où le Zigbee ou la Pro existeront. Aucun changement de
broche, aucun changement de firmware sur les cartes déjà vendues.

## Numéro de série : le second étage, quand le volume le justifiera

Ajouté le 2026-09-10 sur demande d'Olivier. Les straps disent **quelle carte c'est**.
Ils ne disent pas **laquelle**. C'est un besoin distinct, qui apparaît avec le volume :
suivi de garantie, rappel d'un lot, corrélation d'un défaut avec une série de
fabrication.

### Étage 0, gratuit et disponible tout de suite : la MAC

L'ESP32-S3 porte une **adresse MAC de base gravée en usine**, unique par pièce. Un
numéro de série lisible peut en dériver directement, sans rien graver, sans banc de
personnalisation, dès le premier prototype. C'est suffisant pour tracer les cartes
tant qu'on en produit peu.

Limite : la MAC identifie **le module**, pas la carte. Si un module est dessoudé et
remonté sur une autre carte, la traçabilité suit le module. À faible volume, c'est
sans conséquence.

### Étage 1, à graver : eFuse utilisateur

L'ESP32-S3 expose un bloc d'eFuse utilisateur programmable **une seule fois**. On y
écrit le type de carte et le numéro de série, et c'est définitif, y compris après un
effacement complet de la flash. C'est le « qu'on figera ».

Son avantage décisif ici : **c'est du logiciel**. Les 5 Super déjà en fabrication
pourront le recevoir a posteriori, alors qu'elles ne peuvent plus recevoir la moindre
résistance.

Son danger, symétrique : **une gravure fausse est définitive**. Il n'y a pas de reprise.

### Comment les deux cohabitent : le strap reste l'autorité

Règle de précédence, à ne pas inverser :

1. Le firmware lit **d'abord les straps**. C'est eux qui déterminent la bande et le
   nombre de radios, donc le comportement radio.
2. Il lit **ensuite l'eFuse**, s'il est gravé, pour le numéro de série.
3. Si l'eFuse porte aussi un type de carte et qu'il **contredit les straps**, le
   démarrage doit **échouer bruyamment**. Cette contradiction ne peut signifier que
   deux choses, toutes deux graves : une carte mal assemblée, ou une gravure erronée.
   Se replier sur l'un des deux reviendrait à choisir au hasard entre émettre en 433
   et émettre en 868.

Pourquoi le strap et pas l'eFuse comme autorité : le strap est **mesurable au
multimètre**, vérifiable en fin de chaîne d'assemblage, et il décrit la carte telle
qu'elle est réellement peuplée. L'eFuse décrit ce qu'un opérateur a *cru* graver.
Pour un comportement radio, il faut la réalité physique.

### Quand franchir l'étage 1

Pas avant que le banc de personnalisation existe et ait été éprouvé sur des cartes
sacrifiables, puisque l'opération est irréversible. Concrètement : tant que la
production tient dans une série de prototypes, la MAC suffit. Le jour où un lot part
chez un assembleur ou vers des clients, la gravure devient utile, et à ce moment-là
elle doit être **automatisée et vérifiée par relecture**, jamais tapée à la main.

### À vérifier avant la première gravure

La taille exacte du bloc utilisateur et son adressage dépendent de la révision du
silicium. Lire le manuel de référence technique de l'ESP32-S3 et confirmer avec
`espefuse.py summary` sur une carte réelle **avant** d'écrire quoi que ce soit. Ne
jamais graver sur une carte qu'on n'est pas prêt à perdre.

## Garde-fous pour l'implémentation

- Laisser un délai après chaque changement de résistance interne avant de lire, la
  broche et sa piste forment une capacité.
- Ne lire qu'**au démarrage**, puis remettre les broches en entrée sans résistance
  interne, pour ne pas consommer inutilement.
- Ne **jamais** réutiliser `ID0` et `ID1` pour autre chose : une carte doit rester
  identifiable à tout instant, y compris après une mise à jour partielle.
- Une combinaison inconnue doit faire **échouer bruyamment** le démarrage, pas se
  replier silencieusement sur une bande par défaut : émettre en 868 sur un module 433
  est un défaut radio, pas une dégradation acceptable.

## Reste à faire

Ces deux straps ne sont **pas encore sur la Solo**. À poser pendant le re-placement,
après le retour des prototypes Super : router `GPIO14` et `GPIO47` vers `R30` et `R31`,
et **retirer du schéma les 13 étiquettes de net orphelines** (`MB_*`, `GDO0B`, `GDO2B`,
`CSN2`, `nFAULT_M3`, `nFAULT_M5`, `EXT_RX`, `EXT_TX`), vestiges du mikroBUS et de la
seconde radio. C'est ce qui fait échouer le gate `pin-net-pad` aujourd'hui.
