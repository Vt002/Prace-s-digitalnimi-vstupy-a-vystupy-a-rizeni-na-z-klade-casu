<!-- 
Knihovny pro kiCAD v repozitáři na Githubu - JVintera
https://github.com/JVintera/KiCAD-library
 -->


## Digitální vstupy 

**Digital Input (DI), Binary Input (BI), Digitale Eingang (E), Input (I)**

Slouží k připojení součástek, senzorů, apod. se spínaným výstupem.
V automatizační technice se obvykle setkáme s pojmy **sink (NPN)** a **source (PNP) digital input**, u mikropočítačů s **pull-up** a **pull-down rezistory**, ale ve výsledku jde o naprosto identická zapojení. 

![Zapojení tlačítka k MCU s pull-up rezistorem (kladná logika).](/schemata/Rpull-up.png)
*Obr. 1: Zapojení tlačítka k MCU s pull-up rezistorem (kladná logika).*

![Zapojení tlačítka k PLC. Při log. 1 svítí (kladná logika).](/schemata/button_kladnaLog.png)
*Obr. 2: Zapojení tlačítka k PLC. Při log. 1 svítí (kladná logika).*

![Zapojení tlačítka k MCU s pull-down rezistorem (záporná logika).](/schemata/Rpull-down.png)
*Obr. 3: Zapojení tlačítka k MCU s pull-down rezistorem (záporná logika).*

![Zapojení tlačítka k PLC. Při log. 0 svítí (záporná logika).](/schemata/button_zapornaLog.png)
*Obr. 4: Zapojení tlačítka k PLC. Při log. 0 svítí (záporná logika).*

Senzory se zapojují v podstatě stejně, jako tlačítka, tedy pin BN (Brown) na Vcc, BU (Blue) na GND a BK a WH na DI. Zakresluje se to ale trochu jinak, viz např. https://www.jmtcontrol.pl/czujnik-indukcyjny-lr18xbf08dpoy#galleryName=productGallery,imageNumber=2


V praxi se můžeme setkat ještě s dalšími názvy a zkratkami, lteré se týkají této problematiky.
Např. 
**GPIO** (zkráceně GP) - General Purpose Input/Output - univerzální vstup/výstup slouží pro označení pinů MCU. Ty se občas značili i jinými písmeny.
**COM** (společný přívod), **NC** (rozpínací kontakt - normally closed) a **NO** (spínací kontakt - normally open) se zase používá ke značení pinů u některých spínačů a relé.


REALPARS. Sinking and Sourcing PLC Inputs Explained | What is the Difference? Online. 2021. Dostupné z: https://youtu.be/B65detMhnoc?feature=shared. [cit. 2025-01-16].