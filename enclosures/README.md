# Boîtiers 3D — Isno RF

Boîtiers imprimables (FDM) pour les deux cartes, **générés paramétriquement depuis le PCB**
(contour, trous M3, position/mur des connecteurs extraits des `.kicad_pcb`).

| Fichier | Carte | Dimensions |
|---|---|---|
| `boitier_super.stl` | Super Qwiic (4 couches, dual 868+433, mikroBUS v3 — R série) | 72,2 × 60,1 × ~21 mm |
| `boitier_rf868.stl` | RF868 Qwiic (2 couches, mono 868) | 53,8 × 48,1 × ~21 mm |

Boîte + capot **snap-fit, posé libre** (pas de vis pour le capot). PCB vissé sur 4 plots
M3 (avant-trou Ø2,7, vis autotaraudeuse M3×6/8). Capot gravé du **logo Isno RF**.

## Régénérer

```sh
# depuis la racine du repo, KiCad 10 (flatpak) pour extraire la géométrie si respin :
python3 enclosures/gen_enclosure.py enclosures/encl_super.json boitier_super
python3 enclosures/gen_enclosure.py enclosures/encl_rf868.json boitier_rf868
# -> build/boitier_*.scad  (+ *_fit.scad = maquette carte-dans-boîte)
openscad -o boitier_super.stl build/boitier_super.scad
```

`encl_*.json` = géométrie extraite du PCB (bbox, trous, ports). Après un **respin de carte**,
régénérer ce JSON (positions connecteurs / trous) puis relancer le générateur.

## Cotes verrouillées (datasheet)

- **SMA BWSMA-KWE-Z001 (C496551)** coudé : axe barillet **6,4 mm** au-dessus du PCB,
  Ø barillet 6,2 → perçage Ø8. Corps 9,5 mm de haut → pilote la hauteur de boîte (COMP=12).
- **USB-C HRO TYPE-C-31-M-12** : axe d'ouverture 1,63 mm au-dessus du PCB.
- **JST-SH SM04B (Qwiic)** : hauteur accouplée 1,5 mm.

## Ouvertures dans le capot

Certaines découpes traversent le **capot** (et non un mur), gérées par le champ
`"lid"` de chaque port dans les `encl_*.json` (les ports avec `"lid"` sont exclus
des ouvertures murales) :

- **D1 (LED témoin)** — *les deux boîtiers* : **puits de lumière** Ø2,6 mm dans le
  capot au-dessus de la LED → témoin vu du dessus (le 0603 émet vers le haut, un
  trou latéral ne captait qu'un glow). Insert filament transparent optionnel.
- **J2 mikroBUS** — *Super v2 uniquement* (box header 2×8 **vertical**) : fente
  rectangulaire au-dessus du connecteur → une nappe IDC **au pas 2,54 mm** se
  branche par le dessus, boîtier fermé. ⚠️ Le mikroBUS Shuttle MIKROE-2882 n'est
  **pas** compatible : son connecteur et sa nappe (MIKROE-2926) sont au pas
  1,27 mm (cf. `gen_bom.py`, entrée J2). (L'ancien 2×5 coudé sortait
  sur le côté ; la rf868 n'a pas de port d'extension.)

## Impression

FDM PLA/PETG, parois 2 mm, jeux 0,4 mm (calibré FDM — **pas résine**, trop lâche).
Capot : plaque contre le plateau, lèvre vers le haut, texte gravé ressort net. Sans support.
Conseil : tirer d'abord **une** RF868 pour valider physiquement la hauteur d'axe SMA.

## Logo

`isno-rf-logo.svg` (couleur) + `isno-rf-logo-mono.svg` (1 couleur, sérigraphie/gravure).
Onde RF = clin d'œil au swoosh du logo isno.net d'origine (2001).

**Gravé sur le capot** : le logo est **reconstruit en primitives OpenSCAD** dans
`gen_enclosure.py` (module `isno_logo_2d`) — OpenSCAD n'importe que les
remplissages SVG, or le logo est fait de traits (onde RF, cadre RF) + texte de
police. Positions du cadre « RF » calées sur les métriques Liberation Sans Bold
mesurées avec `textmetrics` d'OpenSCAD 2026 ('isno'@150 = 428px, 'RF'@96 = 178px) ;
ces valeurs sont figées dans le générateur, donc le `.scad` reste régénérable sur
n'importe quelle version d'OpenSCAD (pas besoin de `--enable=textmetrics`).
