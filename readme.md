[Co dodělat ]: #

<!-- dodělat zapojení tlačítka a LED (sink source), vč. výpočtů -->

<!-- do ulohy_MCU.md přidat jako typy odkazy na info o millis, ticks_ms a interruptu  -->


    <h2 style="color:Orange">E11</h2>
    <h2 style="color:Yellow">S22</h2>
    <h2 style="color:SteelBlue">A8</h2>


# Práce s digitálními vstupy a výstupy a řízení (ovládání) na základě času

## Cíl

-	Studenti popíší nejběžnější integrované periferie u mikrokontrolerů
-   Mají přehled v nejběžnějších obvodech vstupů a výstuppů
-   K vybranému HW připojí tlačítko/spínač/senzor se spínacím/rozpínacím výstupem a kontrolka (LED)
-   Vysvětlí a podloží výpočtem předřadný rezistor
-   Vysvětlí význam pull-up rezistoru jak u tlačítka/spínače, tak i u sběrnic
-   Vysvětlí pojmy timer a counter a popíší základní princip obvodu
-   Napíší jedoduchý program využívájící timer nebo counter
-   Na příkladech vysvětlí, co je řízení podle času


## Ověření cílů

Práce s digitálními vstupy a výstupy a řízení (ovládání) na základě času

1. Vysvětlete, co to jsou integrované a externí periferie
2. A na předloženém HW vysvětlete, jak se připojí tlačítko a kontrolka k DI a DO
3. Vysvětlete pojmy timer a counter
4. A uveďte příklady řízení na základě času


## Úlohy

### 1. Integrované periferie a obvody vstupů a výstupů

1. Zjistěte, jaké jsou externí a jaké integrované periferie u běžného počítače.
2. U vámi používanému hardwaru vyhledeje technický list a zjistěte z něj jeho integrované periferie.
3. Co z toho jsou obvody vstupů a výstupů? A jaký typ vstupu/výstupu jimi realizujeme?
4. Vyhledejte v dokumentaci (technický list), jak jsou ošetřeny DI, DO, RO (pouze u PLC), AI a AO.




    <details>
        <summary> :bulb: Tip: </summary>
            Technický list = datasheet <br>
            K MCU často nalezneme jen výňatky na pár desítek stran, ale některé informace nalezneme pouze v kompletním technickém listu, který mívá několik set stran.
            U modulárních PLC se informace ke vstupům a výstupům dozvýme z technických listů jednotlivých karet.
    </details>



Další úlohy jsou specializovány pro určitý typ hardwaru:

[pro PLC](PLC/ulohy_PLC.md)

> :key: **PLC**
>
> Programovatelný logický automat
> Programovatelný logický automat. Online. In: Wikipedia: the free encyclopedia. San Francisco (CA): Wikimedia Foundation, 2024, 23. 2. 2024 v 09:19. Dostupné z: https://cs.wikipedia.org/wiki/Programovateln%C3%BD_logick%C3%BD_automat. [cit. 2024-12-28].


[pro MCU](MCU/ulohy_MCU.md)

> :key: **MCU**
>
> Mikrokontrolér (anglicky microcontroller), mikrořadič, jednočipový počítač (hovorově jednočip), µC
> V článku níže se hovoří o architektuře Von Neuman a Harvard. U moderních mikrokontrolerů se setkáme s modifikacemi Harvardské architektury. O architektuře 
> Jednočipový počítač. Online. In: Wikipedia: the free encyclopedia. San Francisco (CA): Wikimedia Foundation, 2024, 6. 11. 2024 v 19:16. Dostupné z: https://cs.wikipedia.org/wiki/Jedno%C4%8Dipov%C3%BD_po%C4%8D%C3%ADta%C4%8D. [cit. 2024-12-28].
