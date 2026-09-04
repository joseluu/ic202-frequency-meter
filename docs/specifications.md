Utilise kicad-mcp pour creer le projet ic202-frequencemetre. Crée le schema a partir de doc/schema.jpg. Description du circuit: alimentation 12V, regulateur 5V puis 3.3V. Prédiviseur MB506 suivi d'un etage a transistor alimenté en 3.3V qui pilote la broche D5 d'un arduino pro mini, le arduino pro mini pilote un affichage LCD 3 digit 7 segments multiplexé.

## modification 1
Erreur dans l'enoncé, c'est du 3V et non du 3.3V. Un information sur le LCD est dans doc/LCD.png une URL decrivant le LCD est https://fr.aliexpress.com/item/1005003745628043.html, le LCD est à l'extremité d'une nappe de fils, representer les connecteurs sur le schema.

## modification 2
Relis le schema en principe plus lisible dans doc/schema_propre.jpg et corrige le schema kicad si necessaire.

## alternative envisagee (non retenue pour l'instant) : remplacer le Arduino Pro Mini
Objectif : reduire l'encombrement de la carte. Le Pro Mini (33 x 18 mm) est
plus grand que necessaire pour ce projet, qui n'a besoin que de 11 broches
(D5 + 10 lignes LCD COM/SEG en tri-etat).

Le ESP32-C3-SuperMini a ete ecarte : seulement 11 GPIO exploitables au total,
dont 2 broches de strapping (GPIO8/9), soit une marge nulle pour les 11
broches necessaires.

Meilleure alternative identifiee : **ESP32-C3-MINI-1U** (module Espressif nu,
a souder directement, sans antenne PCB integree - remplacee par un connecteur
U.FL/IPEX laisse non connecte puisque ce projet n'utilise ni WiFi ni
Bluetooth) :
- Dimensions : 13,2 x 12,5 mm (~165 mm^2), contre 13,2 x 16,6 mm pour la
  variante MINI-1 a antenne PCB integree (~219 mm^2), et 33 x 18 mm pour le
  Pro Mini (~594 mm^2).
- 15 GPIO sorties au total (GPIO0-10, 18-21) ; 13 disponibles en gardant
  GPIO18/19 (USB natif) reserves a la programmation - large marge pour les
  11 broches necessaires.
- GPIO4/5/6/7/10 (noms "FSPI...") sont le peripherique SPI generaliste du
  composant, pas le bus interne de la flash embarquee : entierement libres.
- USB natif (USB-Serial-JTAG sur GPIO18/19) : pas de puce FTDI/CH340 a
  ajouter pour la programmation.
- Alimentation 3,0-3,6 V, compatible avec le rail +3 V deja present sur
  cette carte.
- Sans antenne sur le module, pas de zone de degagement RF a respecter sous
  le composant lors du routage.
- Source : datasheet officiel Espressif "ESP32-C3-MINI-1 & MINI-1U
  Datasheet" v2.2 (documentation.espressif.com).

Decision : conserver le Arduino Pro Mini pour l'instant. A reconsiderer si
la reduction de taille devient prioritaire ; impliquerait de changer
d'ecosysteme firmware (Arduino AVR -> Arduino-ESP32 / ESP-IDF) et de
concevoir un symbole/empreinte KiCad pour ce module.