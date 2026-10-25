[Co dodělat ]: #


# Práce s digitálními vstupy a výstupy a řízení (ovládání) na základě času

$${\color{#FFA500}E11 \space \color{Gold}S22 \space \color{#4682B4}A3 }$$

## Cíl

- **Rozlišit a popsat** integrované periferie řídicích jednotek (MCU, PLC) a vysvětlit jejich funkci s odkazem na datasheet konkrétního HW.
- **Navrhnout a zapojit** digitální vstup (DI) s pull-up/pull-down rezistorem a digitální výstup (DO) bez/s předřadným rezistorem – podložit výpočtem odporu, výkonu a volby hodnoty z řady E.
- **Implementovat neblokující časování** pomocí hardwarového timeru (TON/TOF v PLC; `ticks_ms()` v MicroPythonu) a zdůvodnit, proč nelze ve smyčce použít `delay()` / `WAIT`.
- **Odladit a otestovat** funkčnost zapojení a programu (měření multimetrem, výpisy na sériový monitor, vzájemné testování mezi studenty).
- **Diagnostikovat** nefunkční zapojení nebo program a identifikovat příčinu chyby (zákmity, záměna logiky, špatná hodnota R, blokující smyčka).

## Ověření cílů

Práce s digitálními vstupy a výstupy a řízení (ovládání) na základě času

1. Vysvětlete, co to jsou integrované a externí periferie
2. A na předloženém HW vysvětlete, jak se připojí tlačítko a kontrolka k DI a DO
3. Vysvětlete pojmy timer a counter
4. A uveďte příklady řízení na základě času

---
## Úlohy

### 1. Klíčové pojmy a principy číslicových I/O periferií

*Časová dotace: 10–15 minut | Lehká orientační a opakovací úloha*

Doplňte do níže uvedené tabulky význam klíčových zkratek, úrovně napětí a příklady nasazení společné pro automatizaci (PLC) i internet věcí (MCU / SoC). První řádek slouží jako vzorové vyplnění.

| Zkratka / Pojem         | Co pojem / zkratka znamená                                      | Typické napěťové úrovně (log. 0 / log. 1)                         | Hardwarový výskyt (MCU / PLC / periferie)                              | Typické praktické využití                                       |
| :---------------------- | :-------------------------------------------------------------- | :---------------------------------------------------------------- | :--------------------------------------------------------------------- | :-------------------------------------------------------------- |
| **GPIO (VZOR)**         | General Purpose Input/Output (Univerzální vstupně-výstupní pin) | 0 V / 3,3 V (LVTTL logika)                                        | Raspberry Pi Pico W, ESP32, STM32                                      | Připojení indikační LED, čtení tlačítka, generování PWM signálu |
| **DI (Digital Input)**  | `...`                                                           | PLC: `...` V / `...` V (ČSN EN 61131-2)<br>MCU: `...` V / `...` V | PLC svorkovnice, binární vstupy senzorů                                | `...`                                                           |
| **DO (Digital Output)** | `...`                                                           | Tranzistor: `...` V<br>Relé: beznapěťový kontakt (až 230 V AC)    | Tranzistorové karty (MOSFET), reléové výstupy                          | `...`                                                           |
| **Pull-up rezistor**    | Rezistor svazující signálovou linku s napájecím napětím         | Uvolněný vstup: `...` V<br>Sepnutý na GND: `...` V                | Interní výbava GPIO (Pico W), sběrnice I²C, 1-Wire                     | `...`                                                           |
| **Pull-down rezistor**  | Rezistor svazující linku s referenčním potenciálem (GND)        | Uvolněný vstup: `...` V<br>Sepnutý na VCC: `...` V                | Vstupy digitálních logických obvodů, tlačítka spínaná na kladné napětí | `...`                                                           |
| **Sink vstup (NPN)**    | `...`                                                           | Proud teče **DO** vstupní svorky / ze svorky ven na GND: `...`    | Průmyslové PLC karty, Japonský standard                                | `...`                                                           |
| **Source vstup (PNP)**  | `...`                                                           | Vstup očekává přivedení kladného napětí (+24 V): `...`            | Evropský průmyslový standard (EN 61131-2)                              | Připojení optických závor a indukčních snímačů PNP              |
| **Timer (Časovač)**     | Hardwarový čítač taktovaný krystalem s předděličkou             | Nezávislý na cyklu CPU (v ST např. blok TON/TOF)                  | Periferie čipu RP2040, systémové časovače PLC                          | `...`                                                           |
| **Counter (Čítač)**     | Inkrementální / dekrementální registr reagující na vnější hrany | Pulzní signál (v ST funkční blok CTU/CTD)                         | Vstupy rychlého čítání (HSC), interní registry                         | `...`                                                           |

>  :key: **Vysvětlení pojmů a odborné zdroje:**
> - **GPIO (General Purpose Input/Output):** Programově konfigurovatelný číslicový pin čipu umožňující definovat směr toku dat (vstup s vysokou impedancí nebo nízkoimpedanční výstup) i vnitřní předpětí (pull-up / pull-down rezistor).
>   GPIO. In: *Wikipedia: the free encyclopedia* [online]. San Francisco (CA): Wikimedia Foundation, 2024, 2024-05-18 [cit. 2026-10-14]. Dostupné z: https://en.wikipedia.org/wiki/General-purpose_input/output
> - **Logické úrovně průmyslových vstupů dle ČSN EN 61131-2**. ČESKÝ NORMALIZAČNÍ INSTITUT. *ČSN EN 61131-2 ed. 2: Programovatelné řídicí jednotky – Část 2: Požadavky na zařízení a zkoušky*. Praha: ÚNMZ, 2008.
> - **Sink a Source zapojení:** U zapojení Source dodává řídicí jednotka (nebo napájecí zdroj) proud do snímače; u zapojení Sink vtéká proud ze vstupu do záporného pólu snímače.
>   REALPARS. *Sinking and Sourcing PLC Inputs Explained* [online video]. 2021 [cit. 2026-10-14]. Dostupné z: https://youtu.be/B65detMhnoc

<details>
<summary> :bulb: Tip k doplnění tabulky: </summary>
<p>Pamatujte, že vstupy průmyslových PLC a mikrokontrolérů se liší napěťovou hladinou: mikrokontrolér Raspberry Pi Pico W pracuje s logickými úrovněmi 0 V (log. 0) a 3,3 V (log. 1), přičemž překročení 3,6 V vede k nevratnému zničení pinu. Naopak průmyslové PLC (např. Teco Foxtrot 2) používá odolné vstupy na 24 V DC s galvanickým oddělením optočleny.</p>
</details>

:star2: **Bonusová otázka k úloze 1:**
Proč se u bezpečnostních prvků (např. tlačítko nouzového zastavení E-STOP, bezpečnostní koncový spínač krytu stroje) povinně používá **rozpínací kontakt (NC - Normally Closed)** a nikoliv kontakt spínací (NO)? Co se stane v řídicím systému v případě přerušení přívodního vodiče nebo ztráty napájení?

*Vaše odpověď:*
`...`

---
### 2. Dimenzování předřadného rezistoru LED a elektrické zapojení I/O

*Časová dotace: max. 15 minut | Mírně náročnější úloha propojující teorii, výpočty a zapojení*

Při zapojování číslicových výstupů s LED indikací a vstupů se spínači je nutné správně spočítat a zvolit pasivní součástky, aby nedošlo k tepelnému přetížení GPIO pinu ani polovodičového přechodu diody.

#### Část A – Návrh a výpočet předřadného rezistoru LED

Pro indikaci stavu použijte červenou LED s parametry propustného napětí $U_f = 2{,}0\text{ V}$ a požadovaným pracovním proudem $I_f = 20\text{ mA}$ ($0{,}02\text{ A}$). 
Pin mikrokontroléru Raspberry Pi Pico W poskytuje výstupní napětí $U_{CC} = 3{,}3\text{ V}$.
Svorka PLC Foxtrot CP-20xx poskytuje výstupní napětí $U_{CC} = 24\text{ V}$.

Zapojení viz [Příklad zapojení LED.](DO.md)

1. **Výpočet úbytku napětí na rezistoru podle II. Kirchhoffova zákona:**
   $$U_R = U_{CC} - U_f = \dots\text{ V} - 2{,}0\text{ V} = \dots\text{ V}$$

2. **Výpočet teoretického odporu předřadného rezistoru podle Ohmova zákona:**
   $$R = \frac{U_R}{I_f} = \frac{\dots\text{ V}}{0{,}02\text{ A}} = \dots\ \Omega$$

3. **Volba normalizované hodnoty z řady E24 a zdůvodnění:**
   Doplňte tabulku a vyberte správnou součástku z řady E24 (tolerance ±5 %):

| Vypočítaná teoretická hodnota | Nejbližší nižší hodnota v E24 | Nejbližší vyšší hodnota v E24 | Zvolená hodnota do obvodu | Fyzikální zdůvodnění volby (proč ne nižší hodnota) |
| :--- | :--- | :--- | :--- | :--- |
| `...` $\Omega$ | `...` $\Omega$ | `...` $\Omega$ | **`...` $\Omega$** | `...` |

4. **Výpočet ztrátového výkonu na rezistoru:**
   $$P_R = U_R \cdot I_f = \dots\text{ V} \cdot 0{,}02\text{ A} = \dots\text{ W} = \dots\text{ mW}$$
   *Ověření pouzdra součástky:* Můžeme použít miniaturní SMD rezistor velikosti 0402 s maximální výkonovou zatížitelností $P_{max} = 62{,}5\text{ mW}$?
	   - `[ ] Ano`
	   - `[ ] Ne`
	   - Zdůvodnění: `...`

5. **Jednoduchá zkoušečka:**

Podle předešlého výpočtu si z dostupných součástek vytvořte jednoduchou zkoušečku digitálních výstupů (pamatujte na to, zda používáte PLC nebo MCU). **Zkoušečku otestujte na reálném HW.**

#### Část B – Tester digitálního vstupu

Vymyslete způsob, jak odzkoušet funkčnost digitálního vstupu aniž byste použili tlačítko, nebo senzor s digitálním výstupem. Řešení odzkoušejte na reálním HW. 

Zapojení viz [Příklad zapojení tlačítka.](DI.md) 

Doplňte očekávanou logickou hodnotu na pinu mikrokontroléru:
- Tlačítko **rozepnuto (uvolněno)**: 
	- Naměřené napětí = `...` V $\rightarrow$ Logická hodnota v programu
		- `[ ] log. 0` 
		- `[ ] log. 1`
- Tlačítko **stisknuto**: 
	- Naměřené napětí = `...` V $\rightarrow$ Logická hodnota v programu
		- `[ ] log. 0`
		- `[ ] log. 1`
- Jak se nazývá tato logika? `...` *(pozitivní / negativní / invertovaná)*


>  :key: **Vysvětlení pojmů a odborné zdroje:**
> - [Příklad zapojení tlačítka.](DI.md) 
> - [Příklad zapojení LED.](DO.md)

> :key: **Vysvětlení pojmů a odborné zdroje:**
> - **Řady jmenovitých hodnot odporů (řada E):** Mezinárodní geometrické řady normalizovaných hodnot elektrotechnických pasivních součástek definované normou ČSN IEC 60063. Řada E24 rozděluje jednu dekádu na 24 logaritmicky rovnoměrných kroků.
>   ČESKÝ NORMALIZAČNÍ INSTITUT. *ČSN IEC 60063: Preferované řady pro rezistory a kondenzátory*. Praha: ÚNMZ, 2016.
> - **Výpočet předřadného rezistoru:** Zásady dimenzování polovodičových světelných diod a chladicích parametrů.
>   MALINA, Václav. *Poznáváme elektroniku I*. 4. vyd. Praha: Kopp, 2003. ISBN 80-7232-198-4.

<details>
<summary> :bulb: Tip k volbě rezistoru: </summary>
<p>Pokud vám výpočtem vyjde hodnota například 108,3 Ω, v řadě E24 existují nominály 100 Ω, 110 Ω a 120 Ω. Vždy volíme nejbližší <strong>vyšší</strong> jmenovitou hodnotu (zde 110 Ω nebo 120 Ω). Volba nižšího odporu by způsobila překročení povoleného proudu diodou i výstupním tranzistorem GPIO budiče.</p>
</details>

:star2: **Bonusová otázka k úloze 2:**
Proč nemůžeme přímo na GPIO pin mikrokontroléru (s limitem $12\text{ mA}$) připojit výkonové relé se stejnosměrnou cívkou 12 V / 100 mA? Jaké minimálně dva prvky musí tvořit spínací rozhraní mezi pinem a cívkou?

*Vaše odpověď:*
`...`

---

## Další úlohy jsou specializovány pro určitý typ hardwaru

[pro PLC](PLC/ulohy_PLC.md)

> :key: **PLC**
>
> Programovatelný logický automat
> Programovatelný logický automat. Online. In: Wikipedia: the free encyclopedia. San Francisco (CA): Wikimedia Foundation, 2024, 23. 2. 2024 v 09:19. Dostupné z: https://cs.wikipedia.org/wiki/Programovateln%C3%BD_logick%C3%BD_automat. [cit. 2024-12-28].

[pro MCU](MCU/ulohy_MCU.md)

> :key: **MCU**
>
> Mikrokontroler (anglicky microcontroller), mikrořadič, jednočipový počítač (hovorově jednočip), µC
> V článku níže se hovoří o architektuře Von Neuman a Harvard. U moderních mikrokontrolerů se setkáme s modifikacemi Harvardské architektury. O architektuře 
> Jednočipový počítač. Online. In: Wikipedia: the free encyclopedia. San Francisco (CA): Wikimedia Foundation, 2024, 6. 11. 2024 v 19:16. Dostupné z: https://cs.wikipedia.org/wiki/Jedno%C4%8Dipov%C3%BD_po%C4%8D%C3%ADta%C4%8D. [cit. 2024-12-28].




