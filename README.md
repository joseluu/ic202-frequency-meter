# Fréquencemètre IC-202

Projet KiCad transcrit du schéma manuscrit `docs/Schema.jpg`, selon la description
de `docs/specifications.md`.

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
| `kicad/sym-lib-table` | Enregistrement de la bibliothèque `IC202` (portée projet) |
| `kicad/ic202-frequencemetre.pdf` / `.svg` | Schéma exporté |
| `kicad/bom.csv` | Nomenclature |
| `kicad/erc.rpt` | Rapport ERC |

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
- **Étiquettes réservées aux signaux utiles** : rails, RF_IN, SW1/SW2, D5_SIG,
  VLCD_MID et les dix lignes LCD_COM/LCD_SEG.
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

`J1` (+12 V) → `U1` L7805 → +5 V → `U2` LM317L → +3 V.
Découplage 100 nF + 10 µF sur chacun des trois rails.

Le +3 V est réglé par `R1` (100 R, VO→ADJ) et `R2` (140 R, ADJ→GND) :

    Vout = 1,25 x (1 + 140/100) = 3,00 V

C'est la tension de service du LCD, indiquée sur sa fiche.

`#FLG01` et `#FLG02` sont les PWR_FLAG des rails +12 V et GND, qui ne sont
alimentés que par un connecteur.

### Prédiviseur MB506

- `J2` entrée RF, terminaison `R3` 51 R, liaison capacitive `C7` 1 nF vers IN (broche 1).
- Entrée complémentaire /IN (broche 8) découplée à la masse par `C8` 1 nF.
- VCC (broche 2) en +5 V, découplé par `C9` 100 nF.
- Sortie OUT (broche 4) vers l'étage de mise en forme.
- Broche 7 (NC) marquée non connectée.

Rapport de division réglé par `J3` selon la table du constructeur
(H = VCC, L = ouvert) :

| SW1 | SW2 | Division |
|---|---|---|
| ouvert | ouvert | 1/256 |
| ouvert | +5 V | 1/128 |
| +5 V | ouvert | 1/128 |
| +5 V | +5 V | 1/64 |

Sans cavalier, le montage divise par 256.

### Mise en forme vers D5

Sortie ECL du MB506 → `C10` 100 nF → `R4` 1 k → base de `Q1` (2N2369).
`R5` 100 k ramène la base à la masse, émetteur à la masse, collecteur tiré au
+3 V par `R6` 1 k. Le collecteur attaque D5. Le niveau logique vu par
l'Arduino est donc bien du 3 V, d'où la présence du régulateur LM317L.

### Arduino

L'Arduino Pro Mini est la version **3,3 V / 8 MHz**, alimentée sur sa broche VCC
depuis le rail +3 V (le régulateur de la carte est contourné, RAW non connectée).
L'ATmega328P à 8 MHz fonctionne sans réserve jusqu'à 2,7 V.

D13 est laissé libre à cause de la LED intégrée. TXO/RXI restent disponibles
pour la programmation par le connecteur FTDI de la carte.

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

**Affectation des broches de l'Arduino :**

| Broche | Ligne LCD |
|---|---|
| D5 | entrée de comptage depuis l'étage transistor |
| D2, D3, D4, D6 | COM1, COM2, COM3, COM4 |
| D7, D8, D9, D10, D11, D12 | SEG1 à SEG6 |

**Liaison par nappe.** Le LCD est déporté au bout d'une nappe de 10
conducteurs. `J4` représente le connecteur côté carte ; `DS1` est l'afficheur
à l'autre extrémité.

## Hypothèses de transcription

La photo du schéma manuscrit n'est pas assez nette pour toutes les annotations.
Les points suivants ont été déduits ; ils sont à confronter à l'original.

| Élément | Retenu | Raison |
|---|---|---|
| `R2` (ADJ→GND du LM317L) | 140 R | Valeur donnant exactement 3,00 V avec `R1` = 100 R. Hors série E24, à prendre en 1 % (E48). La lecture directe sur la photo était illisible. |
| `Q1` | 2N2369 | Lecture la plus probable de l'annotation ; transistor de commutation rapide cohérent avec l'usage. Tout NPN rapide équivalent convient. |
| `R4`, `R5`, `R6` | 1 k, 100 k, 1 k | Valeurs de polarisation usuelles pour cet étage ; chiffres illisibles sur la photo. |
| `R3` | 51 R | Terminaison 50 Ω d'entrée RF. |
| `R17`, `R18`, `C12` | 10 kΩ, 10 kΩ, 1 µF | Valeurs usuelles pour une référence à mi-tension : dix fois plus raide que les résistances de ligne, avec un découplage. Les chiffres du papier sont illisibles. |
| Affectation des 10 broches de `DS1` | 1..4 = COM1..COM4, 5..10 = SEG1..SEG6 | La fiche donne le nombre de broches et le multiplexage, pas leur ordre. **À confirmer avant fabrication.** |

Le papier annotait les dix résistances de ligne « 100K », ce qui est cohérent
avec une polarisation de LCD. Une première transcription les avait ramenées à
220 Ω en supposant un afficheur à LED : c'était une erreur, corrigée depuis.

## Ce qui reste à faire

1. **Obtenir la table de correspondance segments du LCD**, c'est-à-dire quel
   couple (COM, SEG) allume quel segment de quel chiffre. Elle n'est pas dans
   la fiche et conditionne le firmware. Sans elle, on peut router la carte mais
   pas afficher un chiffre juste.
2. **Confirmer l'affectation des 10 broches** en communs et segments.
3. **Empreintes de `A1` et `DS1`.** Aucune empreinte standard KiCad ne
   correspond au Pro Mini ni à ce LCD. `J4` utilise pour l'instant une
   empreinte JST-SH 10 points, à remplacer par celle du connecteur retenu.
4. **Confirmer les valeurs du tableau d'hypothèses** sur le schéma d'origine.
5. **Implantation et routage** de la carte, encore vide.
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
