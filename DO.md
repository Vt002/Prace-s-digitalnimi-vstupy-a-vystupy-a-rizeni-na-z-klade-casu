<!-- 
Knihovny pro kiCAD v repozitáři na Githubu - JVintera
https://github.com/JVintera/KiCAD-library
 -->



## Digitální výstupy 

**Digital Output (DO), Binary Output (BO), Digitaler Ausgang (A), Output (Kvůli záměně O za 0 se nahrazuje písmenem Q)**

Slouží k připojení spínaných součástek, strojů, apod., např. LED, elektromagnetů a motorů. U mikrořadičů se používají k realizaci softwarových sběrnic, pak přes ně lze připojit i např. některé displeje a senzory.
V automatizační technice se obvykle setkáme s pojmy **sink (NPN)** a **source (PNP) digital output**, u mikropočítačů s **kladnou** a **zápornou/inverzní logikou**, ale ve výsledku jde o naprosto identická zapojení. 

Na následujících elektrických schématech jsou vedle sebe naprosto identická zapojení, jen různě nakreslená. V praxi se používají nejen tyto varianty, vždy však musí splňovat pravidla pro kreslení elektrickcých schémat, která jsou daná normami. Je tedy jedno, jak budete schémata kreslit, ale musí splňovat tyto normy.

![Zapojení LED k MCU. Při log. 1 svítí (kladná logika).](/schemata/LED_kladnaLog.png )
*Obr. 1: Zapojení LED k MCU. Při log. 1 svítí (kladná logika).*

LED se zapojují s tzv. předřadným rezistorem, na kterém se zmaří přebytečná energie a tím se chrání LED. Ukažme si to na následujícím příkladu:

Vezměme si LED s následujícími parametry U<sub>f</sub> = 2,1 V a I<sub>f</sub> = 20 mA. Připojíme ji na pin GPIO10 k desce raspberry pi pico, tedy U<sub>CC</sub> = 3,3 V (pokud bychom použili např. Arduino UNO, U<sub>CC</sub> by bylo 5 V). Podle 2. Kirchhoffova zákona vypočítáme napětí, které bude na rezistoru:

U<sub>CC</sub> = U<sub>R</sub> + U<sub>f</sub> <br>
U<sub>R</sub> = U<sub>CC</sub> - U<sub>f</sub> <br>
U<sub>R</sub> = 3,3 - 2,1 <br>
U<sub>R</sub> = <u>1,2 V</u> <br>

Z tohoto napětí dopočítáme pomocí Ohmova zákona hodnotu rezistoru:

R = U<sub>R</sub> / I<sub>f</sub> <br>
R = 1,2 V / 20 mA <br>
R = <u>60 Ohmů</u> <br>

Skutečnou hodnotu rezistoru ale zjistíme až z následující tabulky. Rezistory se vyrábí v tzv. řadách a jim odpovídajícím přesnostem. Nejčastěji se používá řada E24 s přesností +-5 %. Hodnota 60 Ohmů se nevyrábí. Jelikož rezistor má ochrannou funkci, tedy má omezit napětí na LED a tím i proud, který diodou teče, a zároveň by neměl příliš snížit jas, je třeba zvolit nejbližší vyšší hodnotu. Ta vychází 62 Ohmů v řadě E24, 68 Ohmů v řadě E12, atd.

![Řady rezistorů](/MCU/ryad-nominalov-radiodetaley-3.jpg)
Jaké jsou rozsahy označení rádiových komponent. Online. In: . 2019, 02.03.2019. Dostupné z: https://our.electricianexp.com/cs/ryady-nominalov-radiodetalej.html. [cit. 2025-02-03].


Ještě je tu otázka, jaké by měl mít rezistor pouzdro, resp. na jaký výkon by měl být minimálně dimenzovaný. Nejprve vypočítáme výkon, jaký by měl být na rezistoru zmařený:

P<sub>R</sub> = U<sub>R</sub> * I<sub>f</sub> <br>
P<sub>R</sub> = 1,2 V * 20 mA <br>
P<sub>R</sub> = 0,024 W = <u>24 mW</u> <br>

Pokud nahlédneme do katologu rezistorů na některém e-shopu, zjistíme, že nám stačí SMD (pro povrchovou montáž) rezistor (např. v pouzdru 0402) se jmenovitým zatížením 0,031 W. Pokud chceme zapojení realizovat na nepájivém poli, potřebujeme rezistor v pouzdře s drátovými vývody, tedy THT, narazíme např. na pouzdro se jmenovitým zatížením 0,4 W.


![Zapojení zátěže, např. LED k PLC. Při log. 1 svítí (kladná logika).](/schemata/load_kladnaLog.png)
*Obr. 2: Zapojení zátěže, např. LED k PLC. Při log. 1 svítí (kladná logika).*

![Zapojení LED k MCU. Při log. 0 svítí (záporná logika).](/schemata/LED_zapornaLog.png )
*Obr. 3: Zapojení LED k MCU. Při log. 0 svítí (záporná logika).*

![Zapojení zátěže, např. LED k PLC. Při log. 0 svítí (záporná logika).](/schemata/load_zapornaLog.png)
*Obr. 4: Zapojení zátěže, např. LED k PLC. Při log. 0 svítí (záporná logika).*



V praxi se můžeme setkat ještě s dalšími názvy a zkratkami, lteré se týkají této problematiky. Více viz DI.md

REALPARS. Sinking and Sourcing PLC Outputs Explained. Online. 2021. Dostupné z: https://youtu.be/oyaItbhqoW0?feature=shared. [cit. 2025-01-16].
        
        
        
