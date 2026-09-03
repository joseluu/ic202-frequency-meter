# Fréquencemètre IC-202

Projet KiCad transcrit du schéma manuscrit `docs/Schema.jpg`, selon la description
de `docs/specifications.md`.

Chaîne de mesure : entrée RF → prédiviseur MB506 → étage de mise en forme à
transistor alimenté en 3,3 V → broche D5 d'un Arduino Pro Mini → afficheur
3 digits 7 segments multiplexé.

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

- ERC : **0 erreur**.
- Netlist vérifiée pin à pin par export `kicadxml`, identique à la netlist de
  référence après chaque retouche cosmétique.
- `check_label_rotations.py` et `check_symbol_overlap.py` : OK.
- `lint_offgrid` : aucune coordonnée hors grille 1,27 mm.
- Reste un avertissement `lib_symbol_mismatch` sur `DS1`. Les deux définitions
  sont sémantiquement identiques (diff effectué). C'est l'avertissement
  cosmétique connu de KiCad sur une bibliothèque locale au projet, sans effet
  sur la netlist.

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
- **Étiquettes réservées aux signaux utiles** : rails, RF_IN, SW1/SW2, D5_SIG
  et les lignes D2..D12 vers l'afficheur.
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

`J1` (+12 V) → `U1` L7805 → +5 V → `U2` LM317L → +3,3 V.
Découplage 100 nF + 10 µF sur chacun des trois rails.

Le +3,3 V est réglé par `R1` (100 R, VO→ADJ) et `R2` (160 R, ADJ→GND) :

    Vout = 1,25 x (1 + 160/100) = 3,25 V

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
+3,3 V par `R6` 1 k. Le collecteur attaque D5. Le niveau logique vu par
l'Arduino est donc bien du 3,3 V, d'où la présence du régulateur LM317L.

### Arduino et afficheur

L'Arduino Pro Mini est la version **3,3 V / 8 MHz**, alimentée sur sa broche VCC
depuis le +3,3 V (le régulateur de la carte est contourné, RAW non connectée).

| Broche | Fonction |
|---|---|
| D5 | entrée de comptage depuis l'étage transistor |
| D2, D3, D4, D6, D7, D8, D9 | segments A, B, C, D, E, F, G via `R7`..`R13` (220 R) |
| D10, D11, D12 | communs DIG1, DIG2, DIG3 via `R14`..`R16` (100 R) |

D13 est laissé libre à cause de la LED intégrée. TXO/RXI restent disponibles
pour la programmation par le connecteur FTDI de la carte.

## Hypothèses de transcription

La photo du schéma manuscrit n'est pas assez nette pour toutes les annotations.
Les points suivants ont été déduits ; ils sont à confronter à l'original.

| Élément | Retenu | Raison |
|---|---|---|
| `R2` (ADJ→GND du LM317L) | 160 R | Seule valeur E24 donnant ≈3,3 V avec `R1` = 100 R. La lecture directe était ambiguë. |
| `Q1` | 2N2369 | Lecture la plus probable de l'annotation ; transistor de commutation rapide cohérent avec l'usage. Tout NPN rapide équivalent convient. |
| `R4`, `R5`, `R6` | 1 k, 100 k, 1 k | Valeurs de polarisation usuelles pour cet étage ; chiffres illisibles sur la photo. |
| `R3` | 51 R | Terminaison 50 Ω d'entrée RF. |
| `R7`..`R16` | 220 R et 100 R | Le papier montre une résistance sur chacune des 10 lignes. Les valeurs manuscrites se lisaient « 100R » ou « 100K » selon les zones. |
| Brochage `DS1` | 10 broches, 1..5 en bas, 6..10 en haut | Le papier montre 5 broches par rangée, ce qui correspond à 7 segments + 3 communs, sans point décimal. |

**Point d'attention électrique.** Les communs de digits sont attaqués
directement par les broches de l'Arduino, comme sur le papier. Avec 7 segments
allumés simultanément, le courant cumulé dans une broche commune dépasse la
valeur recommandée pour l'ATmega328P. Si la luminosité obtenue ne convient pas,
il faut intercaler trois transistors de commutation sur DIG1..DIG3 plutôt que
de diminuer les résistances de segments.

## Ce qui reste à faire

1. **Empreintes de `A1` et `DS1`.** Aucune empreinte standard KiCad ne
   correspond au Pro Mini ni à un afficheur 3 digits 10 broches. Il faut les
   créer avant de router la carte.
2. **Vérifier le brochage réel de l'afficheur** et corriger les numéros de
   broches du symbole `IC202:7SEG_3DIGIT_CC` si nécessaire. Les noms de
   fonction (A..G, DIG1..DIG3) sont corrects, seuls les numéros sont supposés.
3. **Confirmer les valeurs du tableau d'hypothèses** sur le schéma d'origine.
4. **Implantation et routage** de la carte, encore vide.
5. **Firmware.** Comptage sur D5, division par 256 à compenser dans le calcul,
   multiplexage des 3 digits.

## Ouvrir le projet

```
"C:/Program Files/KiCad/10.0/bin/kicad.exe" kicad/ic202-frequencemetre.kicad_pro
```

La bibliothèque `IC202` est enregistrée en portée projet ; elle est trouvée
automatiquement à l'ouverture.
