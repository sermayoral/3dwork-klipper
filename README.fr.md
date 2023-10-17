* * *

## description : Pack de macros, paramètres et autres utilitaires pour Klipper

# Pack Tondeuse 3Dwork

> [!AVERTISSEMENT]**GUIDE EN COURS !!! Bien que les macros soient entièrement fonctionnelles, elles sont en développement continu.****Utilisez-les à vos risques et périls!!!**</mark>

Depuis**Vos excuses**Nous avons compilé et affiné un ensemble de macros, de paramètres machines et électroniques, ainsi que d'autres outils pour une gestion simple et puissante de Klipper.

Une grande partie de ce package est basée sur[**Les rats**](https://os.ratrig.com/)améliorer les parties que nous jugeons intéressantes, ainsi que d'autres contributions de la communauté.

## Installation

Pour installer notre package pour Klipper, nous suivrons les étapes suivantes

### Télécharger depuis le référentiel

Nous allons nous connecter à notre hôte via SSH et émettre les commandes suivantes :

```bash
cd ~/printer_data/config
git clone https://github.com/3dwork-io/3dwork-klipper.git
```

> [!AVERTISSEMENT]Si votre répertoire de configuration Klipper est personnalisé, n'oubliez pas d'ajuster la première commande en conséquence pour votre installation.

Dans les nouvelles installations :

Puisque Klipper n'autorise pas l'accès aux macros tant qu'il n'a pas un fichier Printer.cfg correct et qu'il ne se connecte pas à un MCU, nous pouvons "tromper" Klipper avec les étapes suivantes qui nous permettront d'utiliser les macros de notre bundle pour, par exemple, lancer le Macro de compilation du firmware Klipper si nous utilisons des électroniques compatibles :

-   Nous nous assurons d'avoir notre[hôte comme deuxième MCU](raspberry-como-segunda-mcu.md)
-   Ensuite, nous ajouterons un fichier Printer.cfg, rappelez-vous que ces étapes sont destinées à une installation propre où vous n'avez pas de fichier Printer.cfg et que vous souhaitez lancer la macro pour créer un firmware, comme celui que vous pouvez voir ci-dessous :

```django
[mcu]
serial: /tmp/klipper_host_mcu

[printer]
kinematics: none
max_velocity: 1
max_accel: 1

[gcode_macro PAUSE]
rename_existing: PAUSE_BASE
gcode:
  M118 Please install a config first!

[gcode_macro RESUME]
rename_existing: RESUME_BASE
gcode:
  M118 Please install a config first!

[gcode_macro CANCEL_PRINT]
rename_existing: CANCEL_BASE
gcode:
  M118 Please install a config first!
  
[idle_timeout]
gcode:
  {% raw %}
{% if printer.webhooks.state|lower == 'ready' %}
    {% if printer.pause_resume.is_paused|lower == 'false' %}
      M117 Idle timeout reached
      TURN_OFF_HEATERS
      M84
    {% endif %}
  {% endif %}
{% endraw %}
# 2 hour timeout
timeout: 7200

[temperature_sensor raspberry_pi]
sensor_type: temperature_host

[skew_correction]

[input_shaper]

[virtual_sdcard]
path: ~/printer_data/gcodes

[display_status]

[pause_resume]

[force_move]
enable_force_move: True

[respond]
```

Avec cela, nous pouvons démarrer Klipper pour nous donner accès à nos macros.
{% indice de fin %}

### Utiliser Moonraker pour être toujours à jour

Grâce à Moonraker nous pouvons utiliser sa mise à jour_manager pour pouvoir rester au courant des améliorations que nous pourrions introduire à l'avenir.

Depuis Mainsail/Fluidd nous éditerons notre moonraker.conf (il doit être à la même hauteur que votre imprimante.cfg) et nous ajouterons à la fin du fichier de configuration :

```django
[include 3dwork-klipper/moonraker.conf]
```

{% indice style="avertissement" %}<mark style="color:orange;">**Pensez à faire l'étape d'installation au préalable, sinon Moonraker générera une erreur et ne pourra pas démarrer.**</mark>

**En revanche, si le répertoire de votre configuration Klipper est personnalisé, pensez à ajuster le chemin en fonction de votre installation.**{% finint %}

## Macro

Nous avons toujours dit que RatOS est l'une des meilleures distributions Klipper, avec prise en charge des modules Raspberry et CB1, en grande partie grâce à ses configurations modulaires et à ses excellentes macros.

Quelques macros ajoutées qui nous seront utiles :

### **Macros à usage général**

| Macro                                                                                  | Description                                                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PEUT ÊTRE_MAISON**                                                                   | Cela nous permet d'optimiser le processus de référencement uniquement en l'exécutant sur les axes qui ne sont pas référencés.                                                                                                                                           |
| **PAUSE**                                                                              | Grâce aux variables associées, cela nous permet de gérer une pause avec un stationnement de tête plus polyvalent que les macros normales.                                                                                                                               |
| <p><strong>SET_PAUSE_AT_LAYER</strong><br><strong>SET_PAUSE_AT_NEXT_LAYER</strong></p> | <p>Une macro très utile que Mainsail intègre dans son UI pour pouvoir faire une pause à la demande dans un calque spécifique... au cas où nous l'aurions oublié lors du laminage.<br>Nous en avons également un autre pour exécuter la pause sur le calque suivant.</p> |
| **CONTINUER**                                                                          | Amélioré car il nous permet de détecter si notre buse n'est pas à la température d'extrusion afin de le résoudre avant qu'elle ne montre une erreur et n'endommage notre impression.                                                                                    |
| **ANNULER_IMPRIMER**                                                                   | Ce qui permet d'utiliser le reste des macros pour effectuer correctement une annulation d'impression.                                                                                                                                                                   |

-   **En pause lors du changement de calque**, des macros très intéressantes qui nous permettent de mettre en pause un calque ou de lancer une commande au démarrage du calque suivant. \\![](<../../.gitbook/assets/image (6) (5) (1) (2).png>)![](<../../.gitbook/assets/image (1) (1) (8).png>)\\
    De plus, un autre avantage est qu'ils sont intégrés à Mainsail, nous aurons donc de nouvelles fonctions dans notre interface utilisateur, comme vous pouvez le voir ci-dessous :\\![](<../../.gitbook/assets/image (3) (15).png>)![](<../../.gitbook/assets/image (29) (1) (2).png>)

### **Macros de gestion d'impression**

<table><thead><tr><th width="170">Macro</th><th>Descripción</th></tr></thead><tbody><tr><td><strong>START_PRINT</strong></td><td>Nos permitirá poder iniciar nuestras impresiones de una forma segura y al estilo Klipper. Dentro de esta encontraremos algunas funciones interesantes como:<br>- precalentado de nozzle inteligente en el caso de contar con sensor probe<br>- posibilidad de uso de z-tilt mediante variable<br>- mallado de cama adaptativo, forzado o desde una malla guardada<br>- línea de purga personalizable entre normal, línea de purgado adaptativa o gota de purgado<br>- macro segmentada para poder personalizarse tal como os mostraremos más adelante</td></tr><tr><td><strong>END_PRINT</strong></td><td>Macro de fin de impresión donde también disponemos de segmentación para poder personalizar nuestra macro. También contamos con aparcado dinámico del cabezal.</td></tr></tbody></table>

-   **Cadre de lit adaptatif**Grâce à la polyvalence de Klipper, nous pouvons faire des choses qui semblent aujourd'hui impossibles... un processus important pour l'impression est d'avoir un maillage d'écarts par rapport à notre lit qui nous permet de les corriger pour avoir une parfaite adhérence des premières couches. \\
    À de nombreuses reprises, nous effectuons ce maillage avant l'impression pour nous assurer qu'il fonctionne correctement et cela se fait sur toute la surface de notre lit.\\
    Avec le maillage de lit adaptatif, cela se fera dans la zone d'impression, ce qui le rend beaucoup plus précis que la méthode traditionnelle... dans les captures d'écran suivantes, nous verrons les différences entre un maillage traditionnel et un maillage adaptatif.\\![](<../../.gitbook/assets/image (6) (12) (1).png>)![](<../../.gitbook/assets/image (2) (1) (4).png>)

### **Macros de gestion des filaments**

| Macro                  | Description                                                                                                                        |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **M600**               | Cela nous permettra la compatibilité avec le gcode M600 normalement utilisé dans les plastifieuses pour le changement de filament. |
| **DÉCHARGER_FILAMENT** | Configurable à travers les variables, il nous permettra une décharge assistée du filament.                                         |
| **CHARGER_FILAMENT**   | Identique au précédent mais lié à la charge du filament.                                                                           |

### **Macros de configuration des machines**

| Macro                                                                                          | Description                                                                                                                                                                                                                                                                                  |
| ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **COMPILER_MICROLOGICIEL**                                                                     | <p>Avec cette macro, nous pouvons compiler le firmware Klipper de manière simple, rendre le firmware accessible depuis l'interface utilisateur pour plus de simplicité et pouvoir l'appliquer à notre électronique.<br>Ici vous avez plus de détails sur l’électronique prise en charge.</p> |
| **CALCULER_LIT_ENGRENER**                                                                      | Une macro extrêmement utile pour calculer la surface de notre maillage car cela peut parfois être un processus compliqué.                                                                                                                                                                    |
| <p><strong>PID_ALL</strong><br><strong>PID_EXTRUDEUSE</strong><br><strong>PID_BED</strong></p> | Ces macros, où nous pouvons transmettre les températures au PID sous forme de paramètres, nous permettront d'effectuer l'étalonnage de la température de manière extrêmement simple.                                                                                                         |
| <p><strong>TEST_VITESSE</strong><br><strong>TEST_SPEED_DELTA</strong></p>                      | Macro originale du compagnon[Élise](https://github.com/AndrewEllis93)Ils nous permettront de manière assez simple de tester la vitesse à laquelle nous pouvons déplacer notre machine avec précision et sans perte de pas.                                                                   |

-   **Firmware compilé pour les appareils électroniques pris en charge**, pour faciliter le processus de création et de maintenance de notre firmware Klipper pour nos MCU, nous avons la macro COMPILE_FIRMWARE qui, une fois exécuté, nous pouvons utiliser notre électronique comme paramètre pour faire uniquement cela, Klipper compilera pour toute l'électronique prise en charge par notre bundle :\\![](<../../.gitbook/assets/image (7) (5) (1).png>)\\
    Nous les trouverons facilement accessibles depuis notre interface Web dans le répertoire du firmware_binaires dans notre onglet MACHINE (si nous utilisons Grand-voile) :\\![](../../.gitbook/assets/telegram-cloud-photo-size-4-6019366631093943185-y.jpg)\\
    Vous trouverez ci-dessous la liste des appareils électroniques pris en charge :

{% indice style="avertissement" %}**IMPORTANTE!!!**

-   Ces scripts sont prêts à fonctionner sur un système Raspbian avec un utilisateur pi, si ce n'est pas votre cas vous devrez l'adapter.
-   Les firmwares sont générés pour être utilisés avec une connexion USB, ce qui est toujours ce que nous recommandons. De plus, le point de montage USB est toujours le même, donc la configuration de votre connexion MCU ne changera pas si elle est générée avec notre macro/script.
-   **Pour que Klipper puisse exécuter des macros shell, une extension doit être installée, grâce au compagnon**[**arc sinus**](https://github.com/Arksine)**, cela le permet.**

        <mark style="color:green;">**Dependiendo de la distro de Klipper usada pueden venir ya habilitadas.**</mark>

        ![](<../../.gitbook/assets/image (1179).png>)

        La forma más sencilla es usando [**Kiauh**](../instalacion/#instalando-kiauh) donde encontraremos en una de sus opciones la posibilidad de instalar esta extensión:

        ![](../../.gitbook/assets/telegram-cloud-photo-size-4-5837048490604215201-x\_partial.jpg)

        También podemos realizar el proceso a mano copiaremos manualmente el plugin para Klipper[ **gcode\_shell\_extension**](https://raw.githubusercontent.com/Rat-OS/RatOS/master/src/modules/ratos/filesystem/home/pi/klipper/klippy/extras/gcode\_shell\_command.py) dentro de nuestro directorio _**`~/klipper/klippy/extras`**_ usando SSH o SCP y reiniciamos Klipper.

    {% finint %}

{% onglets %}
{% tab title="Bigtreetech" %}
| Electronique | Nom du paramètre à utiliser dans la macro |
\| ------------------ \| ----------------------------------- \|
| Poulpe Pro (446) | pieuvre_pro_446 |
| Poulpe Pro (429) | pieuvre_pro_429 |
| Poulpe v1.1 | pieuvre_11 |
| Poulpe v1.1 (407) | pieuvre_11_407 |
| SKR Pro v1.2 | skr_pro_12 |
| 3 SEK | btt_skr_3 |
| 2 SKR (429) | skr_2_429 |
| SKR Mini E3 v3 | btt_skr_mini_ez_30              |

| Tête d'outil (CAN) | Nom du paramètre à utiliser dans la macro |
| ------------------ | ----------------------------------------- |
| EBB42 v1           | btt_reflux42_10                           |
| EBB36v1            | btt_reflux36_10                           |
| EBB42 v1.1         | btt_reflux42_11                           |
| EBB36 v1.1         | btt_reflux36_11                           |
| EBB42 v1.2         | btt_reflux42_12                           |
| EBB36 v1.2         | btt_reflux36_12                           |

{% perte finale %}

{% titre de l'onglet="MKS/ZNP" %}
| Electronique | Nom du paramètre à utiliser dans la macro |
\| -------------------- \| ----------------------------------- \|
| ZNP Robin Nano DW v2 | znp_rouge-gorge_nano_dw_v2 |
{% onglet de fin %}

{% tab title="Doux" %}
| Tête d'outil (CAN) | Nom du paramètre à utiliser dans la macro |
\| ----------------- \| ----------------------------------- \|
| Mellow FLY SHT 42 | moelleux_voler_merde_42 |
| Mellow FLY SHT 36 | moelleux_voler_merde_36 |
{% perte finale %}

{% tab title="Fysetc" %}
| Electronique | Nom du paramètre à utiliser dans la macro |
\| ------------- \| ----------------------------------- \|
| Araignée Fysetc | fysetc_araignée                      |
{% onglet de fin %}
{% onglets de fin %}

### Ajout de macros 3Dwork à notre installation

Depuis notre interface, Mainsail/Fluidd, nous allons éditer notre imprimante.cfg et ajouter :

{% code title="printer.cfg" %}

    ## 3Dwork standard macros
    [include 3dwork-klipper/macros/macros_*.cfg]
    ## 3Dwork shell macros
    [include 3dwork-klipper/shell-macros.cfg]

{%endcode%}

{% indice style="info" %}
Il est important d'ajouter ces lignes à la fin de notre fichier de configuration... juste au dessus de la section pour que s'il y a des macros dans notre cfg ou include elles soient écrasées par les nôtres :\\#\*# &lt;---------------------- SAUVEGARDER_CONFIGURATION ------------>
{% indice de fin %}

{% indice style="avertissement" %}
Les macros normales ont été séparées de**shell de macros**car**Pour les activer, il est nécessaire d'effectuer manuellement des étapes supplémentaires, en plus du fait qu'elles sont actuellement en cours de test.**et**Ils peuvent avoir besoin d'autorisations supplémentaires pour attribuer des autorisations d'exécution pour lesquelles les instructions n'ont pas été indiquées puisqu'ils tentent d'automatiser.**\\<mark style="color:red;">**Si vous les utilisez, c'est à vos propres risques.**</mark>{% finint %}

### Configuration de notre plastifieuse

Puisque nos macros sont dynamiques, elles extrairont certaines informations de la configuration de notre imprimante et de la plastifieuse elle-même. Pour ce faire, nous vous conseillons de configurer vos plastifieuses comme suit :

-   **démarrer le gcode DÉBUT_IMPRIMER**, en utilisant des espaces réservés pour transmettre dynamiquement les valeurs de température du filament et du lit :

{% onglets %}
{% tab title="PrusaSlicer-SuperSlicer" %}**Trancheuse Prusa**

```gcode
M190 S0 ; Prevents prusaslicer from prepending m190 to the gcode ruining our macro
M109 S0 ; Prevents prusaslicer from prepending m109 to the gcode ruining our macro
SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count] ; Provide layer information
START_PRINT EXTRUDER_TEMP=[first_layer_temperature[initial_extruder]] BED_TEMP=[first_layer_bed_temperature] PRINT_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} PRINT_MAX={first_layer_print_max[0]},{first_layer_print_max[1]}
```

**SuperSlicer**- nous avons la possibilité d'ajuster la température de l'enceinte (CHAMBRE)

```gcode
M190 S0 ; Prevents prusaslicer from prepending m190 to the gcode ruining our macro
M109 S0 ; Prevents prusaslicer from prepending m109 to the gcode ruining our macro
SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count] ; Provide layer information
START_PRINT EXTRUDER_TEMP=[first_layer_temperature[initial_extruder]] BED_TEMP=[first_layer_bed_temperature] CHAMBER=[chamber_temperature] PRINT_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} PRINT_MAX={first_layer_print_max[0]},{first_layer_print_max[1]}
```

![Ejemplo para PrusaSlicer/SuperSlicer](<../../.gitbook/assets/image (210).png>){% perte finale %}

{% tab title="Bambu Studio/OrcaSlicer" %}

```gcode
M190 S0 ; Prevents prusaslicer engine from prepending m190 to the gcode ruining our macro
M109 S0 ; Prevents prusaslicer engine from prepending m109 to the gcode ruining our macro
SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count] ; Provide layer information
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[first_layer_bed_temperature] CHAMBER=[chamber_temperature] PRINT_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} PRINT_MAX={first_layer_print_max[0]},{first_layer_print_max[1]}
```

<figure><img src="../../.gitbook/assets/image (2) (1) (9) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Cura" %}

```gcode
START_PRINT EXTRUDER_TEMP={material_print_temperature_layer_0} BED_TEMP={material_bed_temperature_layer_0} PRINT_MIN=%MINX%,%MINY% PRINT_MAX=%MAXX%,%MAXY%
```

{% indice style="avertissement" %}
Il va falloir installer le plugin[**Plugin de post-traitement (par frankbags)**](https://gist.github.com/frankbags/c85d37d9faff7bce67b6d18ec4e716ff)du menu_**Aide/Afficher**_Dossier de configuration... nous copierons le script du lien précédent dans le dossier du script. \\
On redémarre Cura et on ira à_**Extensions/Post-traitement/Modifier le G-Code**_et nous sélectionnerons_**Taille d'impression du maillage**_,
{% finint %}
{% indtab %}

{% tab title="IdeaMaker" %}

```gcode
START_PRINT EXTRUDER_TEMP={temperature_extruder1} BED_TEMP={temperature_heatbed}
```

{% perte finale %}

{% tab title="Simplify3D" %}

```gcode
START_PRINT EXTRUDER_TEMP=[extruder0_temperature] BED_TEMP=[bed0_temperature]
```

{% perte finale %}
{% de perte finale %}

{% indice style="info" %}
Los**les espaces réservés sont des "alias" ou des variables que les plastifieurs utilisent pour que lors de la génération du gcode, ils soient remplacés par les valeurs configurées dans le profil**d'impression.

Dans les liens suivants, vous pouvez en trouver une liste pour :[**Trancheuse Prusa**](https://help.prusa3d.com/es/article/lista-de-placeholders_205643),[**SuperSlicer**](https://github.com/supermerill/SuperSlicer/wiki/Macro-&-Variable-list)(en plus de ceux ci-dessus),[**Studio Bambou**](https://wiki.bambulab.com/en/software/bambu-studio/placeholder-list)et[**Traitement**](http://files.fieldofview.com/cura/Replacement_Patterns.html).

L'utilisation de ceux-ci permet à nos macros d'être dynamiques.
{% indice de fin %}

-   **gcode de final END_IMPRIMER**, dans ce cas, en n'utilisant pas d'espaces réservés, il est commun à toutes les plastifieuses

```gcode
END_PRINT
```

### Variables

Comme nous l'avons déjà mentionné, ces nouvelles macros nous permettront d'avoir des fonctions très utiles comme nous l'avons listé ci-dessus.

Pour les ajuster à notre machine nous utiliserons les variables que nous trouverons dans les macros/macros_était_globals.cfg et que nous détaillons ci-dessous.

#### Langue des messages/notifications

Étant donné que de nombreux utilisateurs aiment recevoir des notifications de macros dans leur langue, nous avons conçu un système de notification multilingue, actuellement en espagnol (es) et en anglais (en). Dans la variable suivante, nous pouvons l'ajuster :

<table><thead><tr><th width="189">Variable</th><th width="247">Descripción</th><th width="163">Valores posibles</th><th>Valor por defecto</th></tr></thead><tbody><tr><td>variable_language</td><td>Nos permite seleccionar el idioma de las notificaciones. En el caso de no estar bien definido se usará en (inglés)</td><td>es / en</td><td>es</td></tr></tbody></table>

#### Extrusion relative

Cela nous permet de contrôler quel mode d'extrusion nous utiliserons à la fin de notre START.\_IMPRIMER La valeur dépendra de la configuration de notre plastifieuse.

{% indice style="succès" %}
Il est conseillé de configurer votre plastifieuse pour utiliser l'extrusion relative et de définir cette variable sur True.
{% indice de fin %}

| Variable                   | Description                                                                    | Valeurs possibles | Valeur par défaut |
| -------------------------- | ------------------------------------------------------------------------------ | ----------------- | ----------------- |
| variable_relatif_extrusion | Il nous permet d'indiquer le mode d'extrusion utilisé dans notre plastifieuse. | Vrai faux         | Vrai              |

#### Vitesses

Pour gérer les vitesses utilisées dans les macros.

| Variable                      | Description                       | Valeurs possibles | Valeur par défaut |   |
| ----------------------------- | --------------------------------- | ----------------- | ----------------- | - |
| variable_macro_voyage_vitesse | Vitesse de transfert              | numérique         | 150               |   |
| variable_macro_Avec_vitesse   | Vitesse de transfert pour l'axe Z | numérique         | 15                |   |

#### Retour à destination

Ensemble de variables liées au processus de référencement.

| Variable | Description | Valeurs possibles | Valeur par défaut |
| -------- | ----------- | ----------------- | ----------------- |
|          |             |                   |                   |

#### Chauffage

Variables liées au processus de chauffage de notre machine.

| Variable                                              | Description                                                                                      | Valeurs possibles | Valeur par défaut |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ----------------- | ----------------- |
| variable_Préchauffer_extrudeuse                       | Permet de préchauffer la buse à la température indiquée en variable_Préchauffer_extrudeuse_temp. | Vrai faux         | Vrai              |
| variable_Préchauffer_extrudeuse_temp.                 | Température de préchauffage de la buse                                                           | numérique         | 150               |
| variable_commencer_imprimer_chaleur_chambre_lit_temp. | Température du lit pendant le processus de chauffage de notre enceinte                           | numérique         | 100               |

{% indice style="succès" %}
Avantages de l'utilisation d'une buse préchauffée :

-   Cela nous laisse du temps supplémentaire pour que le lit puisse atteindre sa température de manière uniforme.
-   Si nous utilisons un capteur inductif sans compensation de température, cela permettra à nos mesures d'être plus cohérentes et précises.
-   Permet de ramollir tout filament restant dans la buse, ce qui signifie que, dans certaines configurations, ces restes n'affectent pas l'activation du capteur.
    {% indice de fin %}

#### Filet de lit

Pour contrôler le processus de mise à niveau, nous disposons de variables qui peuvent être très utiles. Par exemple, nous pouvons contrôler le type de nivellement que nous souhaitons utiliser en créant toujours un nouveau maillage, en chargeant un maillage précédemment stocké ou en utilisant un maillage adaptatif.

| Variable                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Valeurs possibles                                                   | Valeur par défaut |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------- |
| variable_étalonner_lit_engrener | <p>Cela nous permet de sélectionner le type de maillage que nous utiliserons dans notre START_PRINT :<br>- nouveau maillage, il maillera chaque impression<br>- storemesh, chargera un maillage stocké et n'effectuera pas le sondage du lit<br>- adaptatif, il fera un nouveau maillage mais adapté à la zone d'impression, améliorant souvent nos premières couches<br>- nomesh, au cas où nous n'aurions pas de capteur ou si nous utilisons un maillage pour ignorer le processus</p> | <p>nouveau maillage / maillage stocké / adaptatif /<br>des noms</p> | adaptative        |
| variable_lit_engrener_profil    | Le nom utilisé pour notre maillage stocké                                                                                                                                                                                                                                                                                                                                                                                                                                                 | texte                                                               | défaut            |

{% indice style="avertissement" %}
Nous vous conseillons d'utiliser le nivellement adaptatif puisqu'il ajustera toujours le maillage à la taille de notre impression, vous permettant ainsi d'avoir une zone de maillage ajustée.

Il est important que nous ayons dans notre[démarrer le gcode de notre plastifieuse](../empezamos/configuracion-klipper-en-laminadores.md#configurando-nuestro-laminador-para-usar-nustras-macros-start_print-y-end_print), dans l'appel à notre START_IMPRIMER, IMPRIMER les valeurs_MAX et IMPRIMER_Homme.
{% finint %}

#### purgé

Une phase importante de notre démarrage d'impression est une purge correcte de notre buse pour éviter les restes de filament ou que ceux-ci pourraient endommager notre impression à un moment donné. Ci-dessous vous avez les variables qui interviennent dans ce processus :

| Variable                             | Description                                                                                                                                                                                                                                                                                                                                                                                                           | Valeurs possibles                                                                   | Valeur par défaut              |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------ |
| variable_buse_amorçage               | <p>Nous pouvons choisir entre différentes options de purge :<br>- primeline tracera la ligne de purge typique<br>- primelineadaptative générera une ligne de purge qui s'adapte à la zone de la pièce imprimée en utilisant variable_nozzle_priming_objectdistance comme marge<br>- primeblob nous fera une goutte de filament dans un coin de notre lit, très efficace pour nettoyer la buse et facile à retirer</p> | <p>ligne principale /</p><p>primelineadaptatif /<br>goutte principale /<br>FAUX</p> | lignes principales adaptatives |
| variable_buse_amorçage_objetdistance | Si nous utilisons une ligne de fond perdu adaptative, ce sera la marge à utiliser entre la ligne de fond perdu et l'objet imprimé.                                                                                                                                                                                                                                                                                    | numérique                                                                           | 5                              |
| variable_buse_prime_commencer_X      | <p>Où nous voulons localiser notre ligne de purge :<br>- min le fera à X=0 (plus une petite marge de sécurité)<br>- max le fera à X=max (moins une petite marge de sécurité)<br>- le numéro sera la coordonnée X où localiser la purge</p>                                                                                                                                                                            | <p>minutes /<br>maximum /<br>nombre</p>                                             | maximum                        |
| variable_buse_prime_commencer_et     | <p>Où nous voulons localiser notre ligne de purge :<br>- min le fera à Y=0 (plus une petite marge de sécurité)<br>- max le fera à Y=max (moins une petite marge de sécurité)<br>- le numéro sera la coordonnée Y où localiser la purge</p>                                                                                                                                                                            | <p>minutes /<br>maximum /<br>nombre</p>                                             | min                            |
| variable_buse_prime_direction        | <p>L'adresse de notre ligne ou de notre dépôt :<br>- vers l'arrière la tête se déplacera vers l'avant de l'imprimante<br>- les avants se déplaceront vers l'arrière<br>- auto ira vers le centre en fonction de variable_nozzle_prime_start_y</p>                                                                                                                                                                     | <p>automatique /<br>en avant /<br>en arrière</p>                                    | auto                           |

#### Chargement/déchargement de filaments

Dans ce cas, ce groupe de variables nous permettra de gérer plus facilement le chargement et le déchargement de notre filament utilisé en émulation du M600 par exemple, ou lors du lancement des macros de chargement et déchargement du filament :

| Variable                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                        | Valeurs possibles | Valeur par défaut |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ----------------- |
| variable_filament_décharger_longueur | De combien rétracter le filament en mm, ajustez à votre machine, normalement la mesure de votre buse aux engrenages de votre extrudeuse en ajoutant une marge supplémentaire.                                                                                                                                                                                                                                                                      | nombre            | 130               |
| variable_filament_décharger_vitesse  | Vitesse de rétraction du filament en mm/sec, normalement une vitesse lente est utilisée.                                                                                                                                                                                                                                                                                                                                                           | nombre            | 5                 |
| variable_filament_charger_longueur   | Distance en mm pour charger le nouveau filament... ainsi qu'en variable_filament_décharger_longueur, nous utiliserons la mesure de votre équipement à l'extrudeuse en ajoutant une marge supplémentaire, dans ce cas cette valeur supplémentaire dépendra de la quantité que vous souhaitez purger... normalement vous pouvez lui donner plus de marge que la valeur précédente pour garantir que la l'extrusion du filament précédent est propre. | nombre            | 150               |
| variable_filament_charger_vitesse    | Vitesse de chargement du filament en mm/sec, normalement une vitesse plus rapide est utilisée que la vitesse de déchargement.                                                                                                                                                                                                                                                                                                                      | nombre            | 10                |

{% indice style="avertissement" %}
Un autre paramètre nécessaire pour votre section\[extruder] se indique el[<mark style="color:green;">**maximum_extruder_seulement_distance**</mark>](https://www.klipper3d.org/Config_Reference.html#extruder)...la valeur recommandée est généralement >101 (si elle n'est pas définie, utilisez 50) pour, par exemple, permettre des tests d'étalonnage typiques d'une extrudeuse. \\
Vous devez ajuster la valeur en fonction de ce qui a été mentionné précédemment concernant le test ou la configuration de votre**variable_filament_décharger_longueur**je**variable_filament_charger_longueur**,
{% finint %}

#### Parking

Dans certains processus de notre imprimante, comme en pause, il est conseillé de garer la tête. Les macros de notre bundle ont cette option en plus des variables suivantes à gérer :

| Variable                                      | Description                                                                                                                                                                                                                                                                                                               | Valeurs possibles                  | Valeur par défaut |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ----------------- |
| variable_commencer_imprimer_parc_dans         | Emplacement où garer la tête pendant le préchauffage.                                                                                                                                                                                                                                                                     | <p>dos /<br>centre /<br>devant</p> | dos               |
| variable_commencer_imprimer_parc_Avec_hauteur | Hauteur Z pendant le préchauffage                                                                                                                                                                                                                                                                                         | nombre                             | 50                |
| variable_fin_imprimer_parc_dans               | Emplacement où garer la tête lors de la fin ou de l’annulation d’une impression.                                                                                                                                                                                                                                          | <p>dos /<br>centre /<br>devant</p> | dos               |
| variable_fin_imprimer_parc_Avec_houblon       | Distance à monter en Z en fin d'impression.                                                                                                                                                                                                                                                                               | nombre                             | 20                |
| variable_pause_imprimer_parc_dans             | Emplacement où garer la tête lors d’une pause d’impression.                                                                                                                                                                                                                                                               | <p>dos /<br>centre /<br>devant</p> | dos               |
| variable_pause_inactif_temps mort             | Valeur, en secondes, de l'activation du processus d'inactivité de la machine qui libère les moteurs et provoque la perte des coordonnées,**Une valeur élevée est conseillée afin que lors de l'activation de la macro PAUSE, il faille suffisamment de temps pour effectuer une action avant de perdre les coordonnées.** | nombre                             | 43200             |

#### Inclinaison en Z

Tirer le meilleur parti de notre machine pour qu'elle s'auto-nivelle et veiller à ce que notre machine soit toujours dans les meilleures conditions est essentiel.

**Z-TILT est essentiellement un processus qui nous aide à aligner nos moteurs Z par rapport à notre axe/portique X (Cartésien) ou XY (CoreXY).**. Avec ça**nous veillons à ce que notre Z soit toujours parfaitement aligné, précisément et automatiquement**.

| Variable                            | Description                                                                        | Valeurs possibles | Valeur par défaut |
| ----------------------------------- | ---------------------------------------------------------------------------------- | ----------------- | ----------------- |
| variable_étalonner_Avec_inclinaison | Permet, si activé dans notre configuration Klipper, le processus de réglage Z-Tilt | Vrai faux         | FAUX              |

#### Fausser

L'utilisation de[FAUSSER](../../guias-impresion-3d/calibracion_3d.md#7.-pasos-ejes)Pour la correction ou l'ajustement précis de nos imprimantes, il est extrêmement conseillé si nous avons des écarts dans nos impressions. En utilisant la variable suivante, nous pouvons autoriser l'utilisation dans nos macros :

| Variable                | Description                                                                                                                                                                                                                      | Valeurs possibles | Valeur par défaut  |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ------------------ |
| variable_fausser_profil | Cela nous permet de prendre en compte notre profil de biais qui sera chargé dans notre macro START_IMPRIMER Pour l'activer, nous devons décommenter la variable et utiliser le nom du profil asymétrique de notre configuration. | texte             | mon_fausser_profil |

### Personnalisation des macros

Notre module pour Klipper utilise le système de configuration modulaire utilisé dans RatOS et profite des avantages de Klipper dans le traitement séquentiel de ses fichiers de configuration. C'est pourquoi l'ordre des inclusions et des paramètres personnalisés que l'on souhaite appliquer à ces modules est essentiel.

{% indice style="info" %}
Lorsqu'elles sont utilisées en tant que module, les configurations 3Dwork NE PEUVENT PAS être modifiées directement à partir du répertoire 3dwork-klipper dans votre répertoire de configuration Klipper car elles seront en lecture seule pour des raisons de sécurité.

C'est pourquoi il est très important de comprendre le fonctionnement de Klipper et comment personnaliser nos modules pour votre machine.
{% indice de fin %}

#### **Personnalisation des variables**

Normalement, ce sera ce que nous devrons ajuster, faire des ajustements aux variables que nous avons par défaut dans notre module**Vos excuses**para Falaises.

Simplement, il suffit de coller le contenu de la macro\[gcode_macroGLOBAL_VARS] que l'on peut trouver dans les macros/macros_était_globals.cfg dans notre imprimante.cfg.

Nous vous rappelons ce que nous avons mentionné précédemment sur la façon dont Klipper traite les configurations de manière séquentielle, il est donc conseillé de le coller après les inclusions que nous avons mentionnées.[ici](3dwork-klipper-bundle.md#anadiendo-las-macros-3dwork-a-nuestra-instalacion).

Nous aurons quelque chose comme ceci (c'est juste un exemple visuel) :

<pre class="language-django"><code class="lang-django">### 3Dwork Klipper Includes
[include 3dwork-klipper/macros/macros_*.cfg]

### USER OVERRIDES
<strong>## VARIABLES 3DWORK
</strong>[gcode_macro GLOBAL_VARS]
description: GLOBAL_VARS variable storage macro, will echo variables to the console when run.
# Configuration Defaults
# This is only here to make the config backwards compatible.
# Configuration should exclusively happen in printer.cfg.

# Possible language values: "en" or "es" (if the language is not well defined, "en" is assigned by default.)
variable_language: "es"                         # Possible values: "en", "es"
...

<strong>
</strong>#*# &#x3C;---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
</code></pre>

{% indice style="avertissement" %}
Les trois points (...) dans les exemples précédents ont simplement pour but d'indiquer que vous pouvez avoir plus de configurations entre les sections... en aucun cas il ne faut les ajouter.
{% indice de fin %}

{% indice style="info" %}

-   Nous vous conseillons d'ajouter des commentaires comme vous le voyez dans le cas précédent pour identifier ce que fait chaque section.
-   Bien qu'il ne soit pas nécessaire de toucher à toutes les variables, nous vous conseillons de copier tout le contenu de\[gcode_macroGLOBAL_Année]
    {% finint %}

#### Personnalisation des macros

Les macros ont été configurées de manière modulaire afin de pouvoir être facilement ajustées. Comme nous l'avons mentionné précédemment, si nous voulons les ajuster, nous devrons procéder de la même manière que pour les variables, copier la macro en question dans notre imprimante.cfg (ou une autre inclusion de notre choix) et nous assurer qu'elle est bien après l'inclusion où nous avons ajouté notre module 3Dwork pour Klipper.

Nous avons deux groupes de macros :

-   Macros pour ajouter des paramètres utilisateur, ces macros peuvent être facilement ajoutées et personnalisées car elles ont été ajoutées afin que tout utilisateur puisse personnaliser les actions à sa guise dans certaines parties des processus effectués par chaque macro.

**COMMENCER_IMPRIMER**

<table><thead><tr><th width="400">Nombre Macro</th><th>Descripción</th></tr></thead><tbody><tr><td>_USER_START_PRINT_HEAT_CHAMBER</td><td>Se ejecuta justo después que nuestro cerramiento empiece a calentar, si CHAMBER_TEMP se pasa como parámetro a nuestro START_PRINT</td></tr><tr><td>_USER_START_PRINT_BEFORE_HOMING</td><td>Se ejecuta antes del homing inicial de inicio de impresión</td></tr><tr><td>_USER_START_PRINT_AFTER_HEATING_BED</td><td>Se ejecuta al llegar nuestra cama a su temperatura, antes de _START_PRINT_AFTER_HEATING_BED</td></tr><tr><td>_USER_START_PRINT_BED_MESH</td><td>Se lanza antes de _START_PRINT_BED_MESH</td></tr><tr><td>_USER_START_PRINT_PARK</td><td>Se lanza antes de _START_PRINT_PARK</td></tr><tr><td>_USER_START_PRINT_AFTER_HEATING_EXTRUDER</td><td>Se lanza antes de _START_PRINT_AFTER_HEATING_EXTRUDER</td></tr></tbody></table>

**FIN_IMPRIMER**

| Nombre Macro                                          | Description                                                                                   |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| \_UTILISATEUR_FIN_IMPRIMER_AVANT_CHAUFFAGES_DÉSACTIVÉ | Il est exécuté avant d'éteindre les radiateurs, avant_FIN_IMPRIMER_AVANT_CHAUFFAGES_DÉSACTIVÉ |
| \_UTILISATEUR_FIN_IMPRIMER_APRÈS_CHAUFFAGES_DÉSACTIVÉ | Il est exécuté après l'arrêt des radiateurs, avant_FIN_IMPRIMER_APRÈS_CHAUFFAGES_DÉSACTIVÉ    |
| \_UTILISATEUR_FIN_IMPRIMER_PARC                       | Il est exécuté avant que la tête ne soit garée, avant_FIN_IMPRIMER_PARC                       |

**IMPRIMER_LES BASES**

| Nombre Macro                      | Description                  |
| --------------------------------- | ---------------------------- |
| \_UTILISATEUR_PAUSE_COMMENCER     | Exécuté au début d'une PAUSE |
| \_UTILISATEUR_PAUSE_FIN           | Exécuté à la fin d'une PAUSE |
| \_UTILISATEUR_CONTINUER_COMMENCER | Exécuté au début d'un RESUME |
| \_UTILISATEUR_CONTINUER_FIN       | Exécuté à la fin d'un CV     |

-   Les macros internes sont des macros permettant de diviser la macro principale en processus et sont importantes pour cela. Il est conseillé que si des ajustements sont nécessaires, ils soient copiés tels quels.

**COMMENCER_IMPRIMER**

<table><thead><tr><th width="405">Nombre Macro</th><th>Descripción</th></tr></thead><tbody><tr><td>_START_PRINT_HEAT_CHAMBER</td><td>Calienta el cerramiento en el caso de que el parámetro CHAMBER_TEMP sea recibido por nuestra macro START_PRINT desde el laminador</td></tr><tr><td>_START_PRINT_AFTER_HEATING_BED</td><td>Se ejecuta al llegar la cama a la temperatura, después de _USER_START_PRINT_AFTER_HEATING_BED. Normalmente, se usa para el procesado de calibraciones de cama (Z_TILT_ADJUST, QUAD_GANTRY_LEVELING,...)</td></tr><tr><td>_START_PRINT_BED_MESH</td><td>Se encarga de la lógica de mallado de cama.</td></tr><tr><td>_START_PRINT_PARK</td><td>Aparca el cabezal de impresión mientras calienta el nozzle a la temperatura de impresión.</td></tr><tr><td>_START_PRINT_AFTER_HEATING_EXTRUDER</td><td>Realiza el purgado del nozzle y carga el perfil SKEW en caso de que así definamos en las variables</td></tr></tbody></table>

## Imprimantes et électronique

Comme nous travaillons avec différents modèles d'imprimantes et d'électronique, nous ajouterons ceux qui ne sont pas directement supportés par RatOS, qu'il s'agisse de contributions de notre part ou de la communauté.

-   imprimantes, dans ce répertoire nous aurons toutes les configurations d'imprimantes
-   cartes, nous trouverons ici les cartes électroniques

### Paramètres et broches

Notre module pour Klipper utilise le système de configuration modulaire utilisé dans RatOS et profite des avantages de Klipper dans le traitement séquentiel de ses fichiers de configuration. C'est pourquoi l'ordre des inclusions et des paramètres personnalisés que l'on souhaite appliquer à ces modules est essentiel.

{% indice style="info" %}
Lorsqu'elles sont utilisées en tant que module, les configurations 3Dwork NE PEUVENT PAS être modifiées directement à partir du répertoire 3dwork-klipper dans votre répertoire de configuration Klipper car elles seront en lecture seule pour des raisons de sécurité.

C'est pourquoi il est très important de comprendre le fonctionnement de Klipper et comment personnaliser nos modules pour votre machine.
{% indice de fin %}

Comme nous l'avons expliqué dans "[personnalisation des macros](3dwork-klipper-bundle.md#personalizando-macros)"Nous utiliserons le même processus pour ajuster les paramètres ou les broches en fonction de nos besoins.

#### Paramètres de personnalisation

Tout comme nous vous conseillons de créer une section dans votre imprimante.cfg appelée USER OVERRIDES, placée après les inclusions de nos configurations, pour pouvoir ajuster et personnaliser n'importe quel paramètre utilisé dans celles-ci.

Dans l'exemple suivant, nous verrons comment dans notre cas nous souhaitons personnaliser les paramètres de notre nivellement de lit (lit_mesh) en ajustant les points de sonde_count) par rapport à la configuration que nous avons par défaut dans les configurations de notre module Klipper :

{% code title="printer.cfg" %}

```django
### 3Dwork Klipper Includes
[include 3dwork-klipper/macros/macros_*.cfg]

### USER OVERRIDES
## VARIABLES 3DWORK
[gcode_macro GLOBAL_VARS]
...

## PARAMETERS 3Dwork
[bed_mesh]
probe_count: 11,11
...

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
```

{%endcode%}

{% indice style="avertissement" %}
Les trois points (...) dans les exemples précédents ont simplement pour but d'indiquer que vous pouvez avoir plus de configurations entre les sections... en aucun cas il ne faut les ajouter.
{% indice de fin %}

Nous pouvons utiliser ce même processus avec n’importe quel paramètre que nous souhaitons ajuster.

#### Personnalisation de la configuration des broches

Nous procéderons exactement comme nous l'avons fait précédemment, dans notre zone USER OVERRIDES, nous ajouterons les sections de broches que nous souhaitons ajuster à notre guise.

Dans l'exemple suivant, nous allons personnaliser quelle est la broche de notre ventilateur électronique (contrôleur_ventilateur) pour l'attribuer à un autre que celui par défaut :

{% code title="printer.cfg" %}

```django
### 3Dwork Klipper Includes
[include 3dwork-klipper/macros/macros_*.cfg]

### USER OVERRIDES
## VARIABLES 3DWORK
[gcode_macro GLOBAL_VARS]
...

## PARAMETERS 3Dwork
[bed_mesh]
probe_count: 11,11
...

## PINS 3Dwork
[controller_fan controller_fan]
pin: PA8

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
```

{%endcode%}

{% indice style="avertissement" %}
Les trois points (...) dans les exemples précédents ont simplement pour but d'indiquer que vous pouvez avoir plus de configurations entre les sections... en aucun cas il ne faut les ajouter.
{% indice de fin %}