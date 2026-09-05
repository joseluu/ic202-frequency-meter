# Fréquencemètre IC-202

Projet KiCad transcrit du schéma manuscrit, selon la description de
`docs/specifications.md`. Deux versions du schéma existent dans `docs/` :
`Schema.jpg`, la première photo, et `Schema_propre.jpg`, une reprise plus
lisible qui a servi à corriger plusieurs valeurs mal lues sur la première.

Chaîne de mesure : entrée RF → prédiviseur MB506 → étage de mise en forme à
transistor alimenté en 3 V → broche D5 d'un Arduino Pro Mini → afficheur LCD
3 digits 7 segments multiplexé, déporté au bout d'une nappe.

## Contenu

| Fichier | Rôle |
|---|---|
| `kicad/ic202-frequencemetre.kicad_pro` | Projet KiCad 10 |
| `kicad/ic202-frequencemetre.kicad_sch` | Schéma (feuille A3 unique) |
| `kicad/ic202-frequencemetre.kicad_pcb` | Carte, encore vide |
| `kicad/IC202.kicad_sym` | Symboles créés pour ce projet |
| `kicad/sym-lib-table` | Enregistrement de la bibliothèque de symboles `IC202` (portée projet) |
| `kicad/frequencemetre.pretty/` | Empreintes créées pour ce projet |
| `kicad/fp-lib-table` | Enregistrement de la bibliothèque d'empreintes `frequencemetre` (portée projet) |
| `kicad/ic202-frequencemetre.pdf` / `.svg` | Schéma exporté |
| `kicad/bom.csv` | Nomenclature |
| `kicad/erc.rpt` | Rapport ERC |
| `docs/Schema.jpg`, `docs/Schema_propre.jpg` | Schéma manuscrit d'origine et sa reprise lisible |
| `docs/LCD.png` | Fiche technique de l'afficheur LCD |

## État de vérification

- ERC : **0 erreur, 0 avertissement**.
- Netlist vérifiée broche à broche par export `kicadxml` après chaque retouche.
- `check_label_rotations.py` et `check_symbol_overlap.py` : OK.
- `lint_offgrid` : aucune coordonnée hors grille 1,27 mm.

## Organisation du schéma

La disposition reprend celle du schéma papier : alimentation en bandeau haut,
prédiviseur à gauche, mise en forme au centre, Arduino à droite, afficheur en
bas à droite. Les liaisons courtes à l'intérieur d'un bloc sont des fils ; les
rails d'alimentation et les signaux entre blocs passent par étiquettes de net.

## Règles de lisibilité appliquées

Les directives du skill
[kicad-schematic-readability](https://github.com/joseluu/kicad-schematic-readability)
sont respectées, et vérifiées par deux scripts locaux au poste de travail :

```
python check_label_rotations.py  .../ic202-frequencemetre.kicad_sch   # OK
python check_symbol_overlap.py   .../ic202-frequencemetre.kicad_sch   # OK
```

- **GND indépendants** : aucun fil de masse ne traverse la feuille, chaque
  composant a son propre stub GND.
- **Alimentation en haut, GND en bas** : les découplages ont leur broche
  positive vers le haut et leur masse vers le bas. Les connecteurs suivent la
  même règle : `J1` porte le +12 V en haut et la masse en bas, `J2` a son
  blindage dirigé vers le bas. Les deux ont été retournés par un miroir, ce qui
  inverse l'ordre des broches sans déplacer la broche de signal.
- **Signaux de gauche à droite** : entrée RF à gauche, progression vers
  l'Arduino puis l'afficheur à droite.
- **Étiquettes réservées aux signaux utiles** : rails, RF_IN, D5_SIG,
  VLCD_MID et les dix lignes LCD_COM/LCD_SEG. SW1 et SW2 sont marquées non
  connectées plutôt qu'étiquetées, faute d'utilité une fois flottantes.
- **Aucun chevauchement de symboles.**

### Placement des repères et valeurs

- **Circuits intégrés rectangulaires** (`U1`, `U2`, `U3`, `A1`, `DS1`) : repère
  et valeur au centre du rectangle, l'intérieur étant libre depuis le masquage
  des noms de broches. Ces champs ne portent aucun `justify`, le défaut centré
  étant celui recherché.
- **Résistances** : valeur au centre du corps et repère sur la ligne au-dessus,
  les deux suivant l'orientation du composant. Pour une résistance verticale le
  texte se lit de bas en haut, donc la ligne au-dessus est celle de gauche.
- **Condensateurs** : repère et valeur de part et d'autre du fil du symbole,
  juste sous les armatures. Le cas vertical exige un `justify` explicite, sinon
  le texte traverse le fil. Le cas horizontal sépare par la hauteur.

### Convention d'orientation des étiquettes

Une étiquette de net a besoin des **deux** informations, et elles doivent être
cohérentes entre elles :

- l'**angle** donne l'axe de lecture, et sert d'indicateur de direction pour le
  script de contrôle ;
- le **`justify`** décide de quel côté du point d'ancrage le texte est posé.
  C'est lui seul que KiCad utilise pour placer le texte.

| Angle | `justify` | Le texte se pose |
|---|---|---|
| 0 | `left bottom` | à droite de l'ancrage |
| 90 | `left bottom` | au-dessus |
| 180 | `right bottom` | à gauche |
| 270 | `right bottom` | en dessous |

Le second mot `bottom` décale le texte perpendiculairement, pour qu'il ne soit
pas posé sur le fil.

**Piège n° 1 : pas de `justify` du tout.** Le défaut de KiCad est *centré*, donc
l'étiquette chevauche son propre point de connexion et le numéro de broche.
C'est visible surtout sur les étiquettes posées directement sur une broche, les
étiquettes en bout de stub ayant de la place autour. Chaque étiquette doit
porter un `justify` explicite.

**Piège n° 2 : `lint_schematic_cosmetic` passe `orient_labels`.** Cette passe du
serveur MCP produit une orientation à 180° de la bonne. Ne pas la relancer sur
ce schéma. Si c'est fait par erreur, réappliquer la table ci-dessus.

**Autre piège.** Dans un champ `property` d'un symbole tourné à 90°, l'angle
est relatif au symbole : il faut écrire 270 pour obtenir un texte horizontal.
C'est le cas des repères de `R7`..`R16`.

### Alimentation

`J1` (+12 V) → `U1` AMS1117-5.0 → +5 V → `U2` AMS1117 (ADJ) → +3 V.
Découplage 100 nF + 10 µF sur chacun des trois rails.

Les deux régulateurs sont en boîtier CMS SOT-223, choisis dans le stock
PartsBox de l'utilisateur plutôt que les L7805/LM317L d'origine (traversants,
non tenus en stock CMS) : mêmes coordonnées de broches, seule leur
numérotation change. Sur `U1`, l'entrée devient la broche 3 (VI) et la
sortie la broche 2 (VO) — sur `U2`, `1`=ADJ, `2`=VO, `3`=VI comme
auparavant, l'architecture (référence 1,25 V, pont diviseur) étant la même
famille que le LM317.

Le +3 V est réglé par `R1` (220 R, VO→ADJ de `U2`) et `R2` (300 R,
ADJ→GND) :

    Vout = 1,25 x (1 + 300/220) = 2,95 V

C'est proche de la tension de service du LCD (3,0 V), indiquée sur sa fiche.
Ce pont n'a pas changé de valeur : l'AMS1117 ajustable utilise la même
référence 1,25 V que le LM317L qu'il remplace.

`#FLG01` et `#FLG02` sont les PWR_FLAG des rails +12 V et GND, qui ne sont
alimentés que par un connecteur.

`J1` est un connecteur 4 broches (header 2,54 mm) : les broches 1 et 2,
adjacentes, portent toutes deux le +12 V ; les broches 3 et 4, adjacentes,
portent toutes deux le GND. Le doublement de chaque broche répartit le
courant d'alimentation sur deux points de contact.

### Prédiviseur MB506

`U3` est le Fujitsu **MB506PF**, boîtier CMS SOP-8 (208 mil, empreinte
`Package_SO:SOIC-8_5.3x6.2mm_P1.27mm`), en remplacement du MB506 traversant
DIP-8 d'origine — même brochage 1-8 (IN, VCC, SW1, OUT, GND, SW2, NC, /IN),
choisi dans le stock PartsBox de l'utilisateur.

- `J2` entrée RF, terminaison `R3` 51 R, liaison capacitive `C7` 1 nF vers IN (broche 1).
  Pas de connecteur SMA monté : un mini câble coaxial est soudé directement
  sur deux pastilles (`TestPoint:TestPoint_2Pads_Pitch2.54mm_Drill0.8mm`,
  pas 2,54 mm, perçage 0,8 mm) — âme sur la pastille 1 (IN), blindage sur
  la pastille 2 (GND).
- Entrée complémentaire /IN (broche 8) : découplée à la masse par `C8` 100 nF,
  et polarisée par `R20` 470 kΩ vers GND. C'est le réseau optionnel repéré sur
  le schéma relu (voir « Hypothèses de transcription ») ; il fixe le point de
  repos de l'entrée complémentaire quand le MB506 est piloté en simple
  extrémité (un seul signal sur IN, ~IN laissée au repos).
- VCC (broche 2) en +5 V, découplé par `C9` 100 nF.
- Sortie OUT (broche 4) : pull-down `R19` (2K2) vers GND, en parallèle de la
  liaison capacitive vers l'étage de mise en forme. C'est une terminaison
  usuelle pour une sortie de type collecteur ouvert / ECL.
- Broche 7 (NC) marquée non connectée.
- SW1 (broche 3) et SW2 (broche 6) reliées directement au rail +5 V.

D'après la table du constructeur (H = VCC, L = ouvert), SW1 et SW2 au niveau H
donnent le rapport **1/64**, câblé en dur sur cette carte :

| SW1 | SW2 | Division |
|---|---|---|
| ouvert | ouvert | 1/256 |
| ouvert | +5 V | 1/128 |
| +5 V | ouvert | 1/128 |
| **+5 V** | **+5 V** | **1/64** |

Le schéma d'origine prévoyait un connecteur `J3` pour choisir le rapport ; il
n'apparaît plus sur le schéma relu, qui laisse SW1/SW2 flottants (1/256 par
défaut). À la demande explicite, les deux broches sont ici reliées au +5 V
pour fixer la division à 1/64. Pour revenir à un autre rapport, déconnecter
l'une ou les deux broches de +5 V (les remettre flottantes redonne 1/256).

### Mise en forme vers D5

`Q1` est le NPN RF **2SC5551AE-TD-E** (onsemi), boîtier CMS SOT-89
(30 V, 300 mA, fT = 3,5 GHz), en remplacement du 2N2369 traversant (TO-18,
fT ≈ 500 MHz) — choisi dans le stock PartsBox de l'utilisateur, largement
assez rapide pour ce rôle. Le brochage réel (1 = base, 2 = collecteur sur
la languette, 3 = émetteur) ne correspond pas à l'ordre du symbole générique
KiCad `Q_NPN_BEC` (broche 2 = émetteur, broche 3 = collecteur) : le symbole a
donc été remplacé par `Q_NPN_BCE` (broche 2 = collecteur, broche 3 =
émetteur), qui a la même géométrie de broches — seul l'étiquetage
broche/fonction change, le câblage existant reste intact — pour que la
numérotation corresponde à celle de l'empreinte SOT-89.

Sortie ECL du MB506 → `C10` 100 nF → base de `Q1`. `R4` 3,3 kΩ relie
la base au collecteur (contre-réaction continue, pas une résistance série vers
la masse comme dans une première version du schéma). Émetteur à la masse,
collecteur tiré au +3 V par `R6` 470 Ω, et c'est ce nœud collecteur qui attaque
D5. Le point de repos de `Q1` s'auto-régule par cette contre-réaction plutôt
que par un pont de base classique. Le niveau logique vu par l'Arduino est donc
bien du 3 V, d'où la présence du second régulateur (`U2`).

### Arduino

L'Arduino Pro Mini est la version **3,3 V / 8 MHz**, alimentée sur sa broche VCC
depuis le rail +3 V (le régulateur de la carte est contourné, RAW non connectée).
L'ATmega328P à 8 MHz fonctionne sans réserve jusqu'à 2,7 V.

D13 est laissé libre à cause de la LED intégrée. TXO/RXI restent disponibles
pour la programmation par le connecteur FTDI de la carte.

**Montage sur picots.** Le Pro Mini n'est pas soudé directement : il se
branche sur deux barrettes de picots femelles soudées sur le PCB principal
(empreinte `frequencemetre:Arduino_Pro_Mini_Header_2x12`). Il se trouve donc surélevé
par rapport à la carte, et le volume dégagé en dessous est disponible pour
d'autres composants. Cette empreinte ne modélise que les deux courtyards des
barrettes (pas de contour de carte complet ni de courtyard unique englobant
tout le module), justement pour ne pas signaler ce volume comme occupé.

Géométrie vérifiée contre l'empreinte open source
[`Arduino_Pro_Mini_Socket`](https://github.com/Alarm-Siren/arduino-kicad-library)
(2 rangées de 12 broches au pas de 2,54 mm, 15,24 mm entre rangées) : 24
broches au total, pas de connecteur de programmation FTDI modélisé. Les
broches A4/SDA et A5/SCL du Pro Mini réel ne sont pas sur ces barrettes — ce
sont des pastilles au dos du module, atteignables seulement par un fil volant
— et ont été retirées du symbole `IC202:Arduino_Pro_Mini`, qui ne comptait
d'ailleurs déjà aucune connexion sur ces deux broches.

### Afficheur LCD

`docs/LCD.png` donne la fiche du composant :

| Caractéristique | Valeur |
|---|---|
| Technologie | TN, polariseur réflectif positif |
| Multiplexage | 1/4 duty, 1/2 bias |
| Tension de service | 3,0 V |
| Angle de lecture | 6 heures |
| Broches | 10, au pas de 2,00 mm |

Référence commerciale :
<https://fr.aliexpress.com/item/1005003745628043.html>

Le 1/4 duty impose **4 communs**, et les 10 broches se répartissent donc en
4 communs plus **6 segments**. Cela adresse 4 × 6 = 24 segments, dont 21 pour
les trois chiffres.

C'est un afficheur à cristaux liquides : il ne consomme aucun courant et ne
supporte pas de tension continue permanente, qui le détruirait par
électrolyse. Il n'y a donc **pas de résistance de limitation** ; l'attaque se
fait en alternatif, par inversion de polarité à chaque trame.

**Attaque en 1/2 bias par tri-état.** Chaque ligne a besoin de trois niveaux :
0 V, 1,5 V et 3 V. Les broches de l'Arduino en fournissent deux, et le
troisième vient de la mise en haute impédance : la ligne rejoint alors la
référence à mi-tension `VLCD_MID` à travers sa résistance de 100 kΩ.

| Élément | Rôle |
|---|---|
| `R7`..`R16` (100 kΩ) | une par ligne, vers `VLCD_MID` |
| `R17`, `R18` (10 kΩ) | pont générant `VLCD_MID` = 3 V / 2 |
| `C12` (1 µF) | découplage de la référence |

Le pont est dix fois plus raide que les résistances de ligne, pour que la
référence ne bouge pas quand plusieurs broches commutent.

**Cavaliers de coupure devant `J4`.** Un cavalier à souder ouvert par défaut
(`JP1`..`JP10`, empreinte `Jumper:SolderJumper-2_P1.3mm_Open_Pad1.0x1.5mm`)
est inséré en série entre chaque ligne (`R7`..`R16`) et sa broche de `J4` —
un par ligne LCD. Chaque ligne passe donc par deux nœuds électriques
distincts : le nœud côté Arduino/résistance (nom de net inchangé,
`LCD_COM1`..`LCD_SEG6`) et le nœud côté `J4` (nommé explicitement
`J4_COM1`..`J4_SEG6`), reliés par le cavalier. Ouverts (comportement par
défaut, sans étain), ils isolent la carte de la nappe LCD — utile pour
tester l'étage Arduino/résistances seul avant de raccorder l'afficheur.
Les fermer au fer à souder rétablit la liaison normale.

**Deux bugs de câblage trouvés et corrigés après coup, à l'occasion d'une
tentative de routage** (voir plus bas) :

1. Les 10 fils reliant chaque cavalier à sa broche de `J4` passaient par un
   point de coude intermédiaire à une abscisse commune (`x=330.2`) pour
   rester orthogonaux. Plusieurs de ces segments verticaux se chevauchaient
   partiellement sur cette même abscisse (les nappes de lignes LCD ont des
   pas différents côté résistances — 5,08 mm — et côté `J4` — 2,54 mm — donc
   leurs coudes intermédiaires ne tombent pas aux mêmes hauteurs), ce qui a
   **court-circuité entre elles plusieurs lignes LCD** (`COM2`-`COM3`-`COM4`-`SEG1`
   d'un côté, `SEG3`-`SEG4` de l'autre). L'ERC ne signalait qu'un avertissement
   discret (« multiple_net_names »), pas une erreur — c'est une relecture du
   netlist exporté par `kicad-cli` (source de vérité) qui a révélé l'ampleur
   réelle du problème. Corrigé en remplaçant les 10 chemins coudés par un
   fil diagonal unique par ligne (pas de point partagé entre lignes
   différentes, donc aucun risque de chevauchement accidentel).
2. Après correction du point 1, le nom de net explicite (`J4_COM1` etc.) a
   dû être ajouté sur chaque broche de `J4` : sans lui, `sync_schematic_to_board`
   (l'outil MCP qui importe le schéma dans le PCB) n'arrivait pas à
   résoudre les nets anonymes auto-générés par KiCad (`Net-(J4-Pin_X)`) et
   laissait ces broches et celles des cavaliers **sans net du tout** côté
   PCB — invisibles pour le ratsnest et l'autorouteur, silencieusement, sans
   qu'aucun outil ne le signale comme une erreur. Un nom de net explicite
   sur chaque broche contourne cette limite.

**Affectation des broches de l'Arduino :**

| Broche | Ligne LCD |
|---|---|
| D5 | entrée de comptage depuis l'étage transistor |
| D2, D3, D4, D6 | COM1, COM2, COM3, COM4 |
| D7, D8, D9, D10, D11, D12 | SEG1 à SEG6 |

**Liaison par nappe.** Le LCD est déporté au bout d'une nappe de 10
conducteurs. `J4` (header mâle au pas de 2,54 mm) est le seul connecteur
présent sur la carte ; `DS1`, l'afficheur, n'est **pas monté sur ce PCB**,
il se trouve à l'autre extrémité du câble. Le symbole `DS1` est marqué
« hors carte » (`on_board: no`) : il reste dans le schéma et la nomenclature
pour documenter le système complet, mais `sync_schematic_to_board` ne lui
crée pas d'empreinte sur cette carte.

## Intention de conception pour le placement/routage

Le schéma porte, en plus de la connectivité, une propriété masquée
`Placement` sur chaque composant dont la position physique finale importe
au-delà du simple raccordement électrique — suivant le skill Claude Code
`conception-electronique-kicad`. Cette propriété n'apparaît pas sur le
schéma ; elle se lit via `get_schematic_component` ou en filtrant le
`.kicad_sch` sur `(property "Placement"`. **Elle ne s'applique pas toute
seule** : au moment du placement/routage, il faut demander explicitement de
la respecter.

**Découplage documenté** (préfixe `LOCAL`) :

| Réf. | Rôle |
|---|---|
| `C1`, `C2` | HF / bulk BF, entrée `J1` et broche 3 (VI) de `U1` |
| `C3`, `C4` | HF / bulk BF, nœud +5 V partagé broche 2 (VO) de `U1` / broche 3 (VI) de `U2` |
| `C5`, `C6` | HF / bulk BF, broche VO (sortie) de `U2` |
| `C9` | HF, broche VCC de `U3` |
| `C8` | HF de la broche ~IN (signal, pas alimentation) de `U3`, avec `R20` |
| `C11` | HF du rail +3 V près de `R6`, point le plus éloigné de `U2` |
| `C12` | référence `VLCD_MID`, point milieu du pont `R17`/`R18` |

Vérification faite : les trois circuits intégrés actifs (`U1`, `U2`, `U3`)
ont chacun un découplage local documenté sur leur(s) broche(s)
d'alimentation. `A1` (Arduino Pro Mini) n'est pas inclus dans cette
vérification : c'est un module complet avec son propre découplage embarqué,
pas un CI nu à décrire ici.

**Boucles critiques documentées** (préfixe `LOOP CRITIQUE`, hors
découplage) :

- `J2`-`R3`-`C7` → broche IN de `U3` : chemin RF d'entrée, jusqu'à 2,4 GHz,
  surface minimale impérative.
- `U3`(OUT)-`R19`-`C10`-`R4`-`R6`-`Q1` → `D5_SIG` (broche D5 de `A1`) : chemin
  de comptage, le signal que l'instrument existe pour mesurer — à éloigner
  des sources de bruit.

### Placement initial (fait)

Carte **80 × 50 mm**, paysage (hauteur réduite de 70 à 50 mm après le
passage aux boîtiers CMS ci-dessous, qui a libéré de la place). Emplacements
imposés par l'utilisateur :

- `J1` (+12 V IN) : bord haut, au milieu, rangée de broches **parallèle**
  au bord (rotation 90° par rapport à l'orientation par défaut du
  footprint, qui les mettait perpendiculaires).
- `J2` (RF IN, coax à souder) : bord gauche, rangée de broches parallèle au
  bord (même correction de rotation).
- `J4` (nappe LCD, header 2,54 mm) : bord droit.
- `A1` (Arduino Pro Mini sur picots) : décalé de 3 mm vers la gauche par
  rapport au premier placement, toujours à droite et proche de `J4`,
  orientation paysage, extrémité USB/FTDI côté gauche (rotation 90°, vérifiée
  par la mesure des pads comme au premier placement).
- `R7`..`R16` (échelle de polarisation LCD) : **sous `A1`**, dans le volume
  dégagé entre les deux barrettes de picots — l'empreinte de `A1` ne porte
  que les deux courtyards des barrettes (voir « Arduino » ci-dessus), donc
  cet espace n'est pas signalé occupé.

L'orientation « parallèle au bord le plus proche » pour `J1`/`J2` est
désormais une règle par défaut du skill `kicad-pcb-placement` : elle
s'applique à tout connecteur proche d'un bord, sauf orientation contraire
explicitement demandée.

Le reste (bloc d'alimentation, chaîne RF/prédiviseur, chemin de comptage,
cavaliers `JP1`-`JP10`, `R17`/`R18`/`C12`) est réparti par proximité en
suivant les groupes `Placement` du schéma. Vérifié via `run_drc`
(kicad-cli, source de vérité du projet) : **0 erreur**, 3 avertissements
cosmétiques silkscreen. Méthode suivie : skill Claude Code
`kicad-pcb-placement`.

### Routage (fait)

Autoroutage via Freerouting (`autoroute`, 2 couches F.Cu/B.Cu) sur ce
placement : **16,5 s**, 278 pistes + 29 vias. Deux passes de nettoyage
après coup :

- 11 pistes exportées à 0,15 mm par le pipeline KiCad→DSN→Freerouting alors
  que la carte impose 0,2 mm minimum (limite connue de ce pipeline,
  reproductible même en relançant l'autoroutage plusieurs fois de suite —
  ne se corrige pas tout seul). Corrigées une par une : piste supprimée puis
  retracée au même endroit avec une largeur explicite de 0,2 mm.
- Un essai de placement alternatif via l'outil externe **FD-Autoplacer**
  (`github.com/joseluu/FD-Autoplacer`, optimisation par descente de
  gradient) a été comparé sur ce même schéma : passage clone → exécution →
  import direct, sans succès retenu (chevauchement de courtyard réel,
  plusieurs composants débordant du bord de carte) — le placement à la main
  ci-dessus a été conservé.

Résultat final vérifié via `run_drc` : **0 erreur**, 10 avertissements
cosmétiques (7 vias non connectées des deux côtés, 1 chevauchement
silkscreen, 2 silkscreen sur cuivre).

### Bug critique trouvé et corrigé après coup : broches actives sans net

En inspectant le PCB routé, une broche visiblement connectée à trop
d'endroits (broche 1 de `U3`) a révélé un bug bien plus large : **5 nets
sans label explicite** (`Net-(U3-IN)`, `Net-(U3-OUT)`, `Net-(U3-~{IN})`,
`Net-(Q1-B)`, `Net-(U2-ADJ)`) — électriquement valides au niveau du schéma,
l'ERC ne signale rien — mais que `sync_schematic_to_board` échoue à
résoudre silencieusement, laissant les broches concernées **sans net du
tout côté PCB**. Conséquence réelle : la sortie du prescaler (`U3` OUT) et
la base de `Q1` n'ont jamais été reliées à la chaîne de comptage, sur les
deux cartes, malgré un DRC à 0 erreur — une broche sans net n'ayant par
ailleurs aucune obligation de clearance, ce qui explique l'apparence de
connexion multiple constatée.

Corrigé : les 5 nets renommés explicitement (`MB506_IN`, `MB506_OUT`,
`MB506_NIN`, `Q1_BASE`, `U2_ADJ`), resynchronisés vers les deux PCB, la
règle de clearance globale remontée de 0 mm (!) à 0,2 mm — qui a d'ailleurs
mis au jour des croisements de pistes que le clearance nul masquait — puis
carte principale entièrement reroutée : **0 erreur `run_drc`** au final
(327 pistes, 40 vias, même limite de 0,15 mm sur quelques pistes corrigée
comme précédemment). Règle ajoutée au skill `conception-electronique-kicad`
pour détecter ce cas systématiquement avant tout premier routage.

**Piège outil découvert à cette occasion :** `check_courtyard_overlaps`
(MCP) évalue le chevauchement de courtyard d'une empreinte à partir de sa
*bounding box globale*, pas des polygones réels — pour une empreinte à
plusieurs rectangles de courtyard disjoints comme celle de `A1` (justement
conçue pour laisser un volume libre au milieu), l'outil signale à tort un
chevauchement dès qu'un autre composant entre dans cette bounding box, y
compris pile au centre du volume libre. Vérifié en plaçant une résistance
au centre exact de `A1` et en comparant : `check_courtyard_overlaps` la
signale en conflit, `run_drc` (qui lit les vrais polygones KiCad) ne
signale rien. Pour toute empreinte de ce genre, se fier à `run_drc`, pas à
`check_courtyard_overlaps`.

## Hypothèses de transcription

La photo du schéma manuscrit n'est pas assez nette pour toutes les annotations.
Les points suivants ont été déduits ; ils sont à confronter à l'original.

| Élément | Retenu | Raison |
|---|---|---|
| `Q1` | 2N2369 | Lecture la plus probable de l'annotation ; transistor de commutation rapide cohérent avec l'usage. Tout NPN rapide équivalent convient. |
| `R17`, `R18` | 3,3 kΩ chacune | Valeur clairement lisible sur le schéma relu. |
| Affectation des 10 broches de `DS1` | 1..4 = COM1..COM4, 5..10 = SEG1..SEG6 | La fiche donne le nombre de broches et le multiplexage, pas leur ordre. **À confirmer avant fabrication.** |

Le schéma manuscrit relu (`docs/Schema_propre.jpg`) est nettement plus lisible
que la première photo et a permis de corriger plusieurs valeurs mal lues, ainsi
que d'ajouter des composants absents de la première transcription. Le tableau
ci-dessous liste les changements :

| Élément | Ancienne valeur | Valeur corrigée |
|---|---|---|
| `R1` | 100 R | 220 R |
| `R2` | 160 R | 300 R |
| `R6` | 1 k | 470 R |
| `C8` | 1 n | 100 n |
| `R17`, `R18` | 10 k | 3,3 k |
| ancien `R5` (100 k, base→GND) | présent | supprimé, absent du schéma relu |
| ancien `J3` (sélecteur SW1/SW2) | présent | supprimé, SW1/SW2 laissés flottants |
| `R19` (2K2, OUT MB506→GND) | absent | ajouté |
| `R4` | 1 k, en série cap→base | 3,3 k, en contre-réaction base→collecteur |
| `R20` (470 kΩ, broche 8 ~IN→GND) | absent | ajouté |

Deux de ces corrections ont été apportées directement dans KiCad par
l'utilisateur puis relues par la suite : le rebranchement de `R4` en
contre-réaction base→collecteur (la première transcription le plaçait par
erreur en série entre `C10` et la base), et l'ajout de `R20`, le réseau
repéré près de la broche complémentaire (~IN, broche 8) du MB506, qui polarise
cette broche vers la masse quand le composant est piloté en entrée simple.

## Ce qui reste à faire

1. **Obtenir la table de correspondance segments du LCD**, c'est-à-dire quel
   couple (COM, SEG) allume quel segment de quel chiffre. Elle n'est pas dans
   la fiche et conditionne le firmware. Sans elle, on peut router la carte mais
   pas afficher un chiffre juste.
2. **Confirmer l'affectation des 10 broches** en communs et segments.
3. **Empreinte de `A1`.** `DS1` n'a pas besoin d'empreinte, il n'est pas
   monté sur ce PCB (voir « Afficheur LCD »). `J4` est un header 1×10 au
   pas de 2,54 mm (`Connector_PinHeader_2.54mm:
   PinHeader_1x10_P2.54mm_Vertical`).
4. **Confirmer les valeurs du tableau d'hypothèses** sur le schéma d'origine.
5. ~~Routage de la carte~~ — fait (voir « Routage » ci-dessus), 0 erreur DRC.
6. **Firmware.** Comptage sur D5, division par 256 à compenser, puis génération
   des trames LCD en 1/4 duty et 1/2 bias : chaque broche prend tour à tour
   l'état haut, bas ou haute impédance, et la polarité s'inverse à chaque trame
   pour que la tension moyenne sur chaque segment reste nulle.

## Ouvrir le projet

```
"C:/Program Files/KiCad/10.0/bin/kicad.exe" kicad/ic202-frequencemetre.kicad_pro
```

La bibliothèque `IC202` est enregistrée en portée projet ; elle est trouvée
automatiquement à l'ouverture.
