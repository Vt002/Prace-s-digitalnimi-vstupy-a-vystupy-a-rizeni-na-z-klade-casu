<!-- 
Knihovny pro kiCAD v repozitáři na Githubu - JVintera
https://github.com/JVintera/KiCAD-library
 -->


<!--
Doplnit 

Výpočet předřadného rezistoru LED, schéma, zapojení na nepájivém poli a proměření

-->


## Digitální výstupy 

**Digital Output (DO), Binary Output (BO), Digitaler Ausgang (A), Output (Kvůli záměně O za 0 se nahrazuje písmenem Q)**

Slouží k připojení spínaných součástek, strojů, apod., např. LED, elektromagnetů a motorů. U mikrořadičů se používají k realizaci softwarových sběrnic, pak přes ně lze připojit i např. některé displeje a senzory.
V automatizační technice se obvykle setkáme s pojmy **sink** a **source digital output**, u mikropočítačů s **kladnou** a **zápornou/inverzní logikou**, ale ve výsledku jde o naprosto identická zapojení. 

![Zapojení LED k MCU. Při log. 1 svítí (kladná logika).](/schemata/LED_kladnaLog.png )
*Obr. 1: Zapojení LED k MCU. Při log. 1 svítí (kladná logika).*

![Zapojení zátěže, např. LED k PLC. Při log. 1 svítí (kladná logika).](/schemata/load_kladnaLog.png)
*Obr. 2: Zapojení zátěže, např. LED k PLC. Při log. 1 svítí (kladná logika).*

![Zapojení LED k MCU. Při log. 0 svítí (záporná logika).](/schemata/LED_zapornaLog.png )
*Obr. 3: Zapojení LED k MCU. Při log. 0 svítí (záporná logika).*

![Zapojení zátěže, např. LED k PLC. Při log. 0 svítí (záporná logika).](/schemata/load_zapornaLog.png)
*Obr. 4: Zapojení zátěže, např. LED k PLC. Při log. 0 svítí (záporná logika).*



V praxi se můžeme setkat ještě s dalšími názvy a zkratkami, lteré se týkají této problematiky. Více viz DI.md

REALPARS. Sinking and Sourcing PLC Outputs Explained. Online. 2021. Dostupné z: https://youtu.be/oyaItbhqoW0?feature=shared. [cit. 2025-01-16].
        
        
        
