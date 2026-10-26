## Úlohy k MCU


> <mark>U klasifikovaných úloh nejde jen o jejich vypracování, ale především o pochopení programu a seznámení se s pojmy.</mark>
> U úloh můžete vypracovat bonusové otázky i případně si zadání rozšířit a poupravit. Nezapomeňte si pak říct o odměnu.

### 3. Digitální vstup a neblokující řízení LED na MCU (průvodce)

*Časová dotace: 15 minut | :bangbang: Klasifikovaná úloha ve stylu průvodce s postupným doplňováním kódu*

> **Použitá platforma** Raspberry Pi Pico W (čip RP2040)/Raspberry Pi Pico W2 (čip RP2350), 
> **programovací jazyk** MicroPython (pro CircuitPython, Arduino language - C++, případně jiný jazyk je třeba přemigrovat příklad a zvojit vhodný simulátor nebo IDE), 
> **webový simulátor**, např. [Wokwi.com](https://wokwi.com/), 
> nebo **IDE** (např. Thonny...) **a fyzický vývojový kit s potřebnými součástkami**.

<!--
Další simulátory
https://youtu.be/_HsZzkSYao0?si=dEr8ZAH__jxF9Y1n
https://velxio.dev/
https://app.cirkitdesigner.com/
-->

**Cíl úlohy:** Zapojit indikační LED a ovládací tlačítko. Nejprve demonstrovat kritickou nevhodnost blokujícího čekání `time.sleep()`, a následně program přepsat na deterministické neblokující časování pomocí vnitřního hardwarového časovače `time.ticks_ms()`.

#### Krok 1 – Hardwarová konfigurace
1. V simulátoru nebo IDE založte projekt pro **Raspberry Pi Pico W (MicroPython)**.
2. V simulátoru nebo na nepájivém poli zapojte svítivou LED s odporem $220\ \Omega$ na **GPIO15** (katoda na GND, viz [zapojení LED](DO.md) kladná logika).
3. Připojte tlačítko mezi **GPIO14** a **GND** (využijeme vnitřní `Pin.PULL_UP`, viz [zapojení tlačítka](DI.md)).

#### Krok 2 – Analýza fatální chyby s blokující funkcí `sleep()` (4 min)
Prohlédněte si následující blokující kód:

```python
# UKÁZKA ŠPATNÉ PRAXE – Blokující čekání time.sleep()
from machine import Pin
import time

led = Pin(15, Pin.OUT)
button = Pin(14, Pin.IN, Pin.PULL_UP)

while True:
    # LED bliká s periodou 2 sekundy
    led.on()
    time.sleep(1.0)   # ⚠️ CPU zamrzne v čekací smyčce
    led.off()
    time.sleep(1.0)   # ⚠️ CPU opět zamrzne

    # Dotaz na tlačítko se provede pouze 1x za 2 sekundy po dobu pár mikrosekund!
    if button.value() == 0:
        print("Tlačítko bylo zachyceno!")
```

*Didaktická otázka:* Spusťte kód a stiskněte tlačítko na krátkou dobu (např. 200 ms). Co se stane a proč je tento přístup v průmyslovém řízení nepřípustný?
`...`

#### Krok 3 – Doplňovačka: Neblokující smyčka s `ticks_ms()` a `ticks_diff()` (5 min)
Upravte kód tak, aby procesor obsluhoval tlačítko nepřetržitě v každém průchodu hlavní smyčky a časování LED bylo řízeno diferenciálním odpočtem systémového času.

Doplňte chybějící části kódu označené `???`:

```python
# SPRÁVNÉ ŘEŠENÍ – Neblokující časovač
from machine import Pin
import time

led = Pin(15, Pin.OUT)
button = Pin(14, Pin.IN, Pin.PULL_UP)

BLINK_INTERVAL_MS = 500  # Interval přepnutí stavu LED (500 ms)
last_blink_time = time.ticks_ms()

while True:
    now = time.ticks_ms()

    # 1. Časovaná neblokující obsluha LED:
    if time.ticks_diff(now, last_blink_time) >= BLINK_INTERVAL_MS:
        led.???()                # 💡 DOPLŇTE: metoda pro změnu stavu pinu (toggle)
        last_blink_time = ???    # 💡 DOPLŇTE: aktualizace referenčního času

    # 2. Okamžitá odezva na stisknutí tlačítka (bez zpoždění):
    if button.value() == ???:    # 💡 DOPLŇTE: logická úroveň při stisku s pull-up (0 nebo 1)
        print("Okamžitá detekce stisku bez blokování!")
```

*Vaše doplnění:*
- Metoda pro změnu stavu: `led.` `...` `()`
- Aktualizace referenční proměnné: `last_blink_time =` `...`
- Testovaná hodnota při sepnutí na zem: `button.value() ==` `...`

<details>
<summary> :bulb: Nápověda v jazyce C++ (Arduino Core): </summary>
<p>V Arduino jazyce odpovídá MicroPython funkci <code>ticks_ms()</code> funkce <code>millis()</code> a neblokující podmínka vypadá takto: <code>if (millis() - previousMillis >= interval) { previousMillis = millis(); digitalWrite(ledPin, !digitalRead(ledPin)); }</code>.</p>
</details>

#### Krok 4 – Vzájemné testování (Peer testing protokol) (3 min)
Vyměňte si odkaz na simulátor se sousedním studentem, případně HW (Student A testuje projekt Studenta B):

| Číslo testu | Testovací scénář testera | Očekávané chování systému | Zjištěný stav testerem | Splněno (OK / Chyba) |
| :--- | :--- | :--- | :--- | :--- |
| **T1.1** | Klidový stav bez dotyku tlačítka | LED nepřetržitě pravidelně bliká (perioda 1 s) | `...` | `[ ]` |
| **T1.2** | Rychlý trojitý klik na tlačítko | Každý stisk vyvolá okamžitý výpis do konzole | `...` | `[ ]` |
| **T1.3** | Trvalé držení stisknutého tlačítka | Blikání LED se nesmí zastavit ani zpomalit | `...` | `[ ]` |

> ⚠️ **Častá chyba začátečníků:** Zapomenutí aktualizace proměnné `last_blink_time = now`. V takovém případě podmínka po uplynutí prvního intervalu platí v každém dalším cyklu a LED začne kmitat na frekvenci cyklu smyčky (desítky kHz), což lidské oko vnímá pouze jako trvalý poloviční svit.

> :key: **Vysvětlení pojmů a odborné zdroje:**
> - **IDE**: (Integrated Development Environment, česky _integrované vývojové prostředí_) je program, který na jednom místě sdružuje všechno, co potřebujete k psaní a nahrávání programů do mikrokontroléru.
> - **Deterministický běh smyčky (Super-loop architecture):** Mikrokontrolér má jednu nekonečnou smyčku, která pořád dokola dělá totéž: zjistí, co se děje (zmáčkl někdo tlačítko? jaká je teplota?), a podle toho zareaguje. Je to jako hlídač, který neustále obchází budovu a kontroluje jedny dveře za druhými. „**Deterministický**” znamená, že každý průchod smyčkou trvá vždy zhruba stejně dlouho a předem víme, za jak dlouho se na všechno dostane. Aby mikrokontrolér nic nezmeškal (třeba krátký stisk tlačítka), musí být jeden průchod velmi rychlý, jen zlomek milisekundy.
> - **Přetečení čítače (Timer Wrap-around):** Funkce `time.ticks_diff()` správně ošetřuje aritmetické přetečení hardwarového 30bitového čítače mikrosekund, ke kterému v MicroPythonu dochází přibližně každých 12,4 dne.
>   *MicroPython documentation: time – time related functions* [online]. 2025 [cit. 2026-10-14]. Dostupné z: https://docs.micropython.org/en/latest/library/time.html

:star2: **Bonusová otázka k úloze 3:**
Co by se stalo, kdybychom namísto konstrukce `time.ticks_diff(now, last_time) >= INTERVAL` použili pouhé matematické porovnání `now - last_time >= INTERVAL` v okamžiku, kdy systémový čítač přeteče z maximální hodnoty zpět na nulu?

*Vaše odpověď:*
`...`

---

### 4. Hardwarové potlačení zákmitů kontaktů (Debouncing) a čítač událostí

*Časová dotace: 15 minut | :bangbang: Klasifikovaná úloha - průvodce s praktickým rozborem mechanických chyb*

> **Průmyslová paralela:** V průmyslových třídicích linkách s indukčními nebo mechanickými koncovými spínači způsobují zákmity kontaktů falešné vícenásobné načtení výrobků, což vede k rozpadu synchronizace taktu dopravníku nebo chybnému vyřazení kusů.

**Cíl úlohy:** Prozkoumat jev mechanických zákmitů (contact bounce), změřit chybovost surového čtení pinu a naprogramovat deterministický softwarový debouncing pomocí časového filtru bez zpoždění odezvy.

```
Fyzické sepnutí tlačítka:
Signál na pinu: 111110100100000000000000 (zákmity trvající typicky 5 až 20 ms)
                          <-- 20 ms -->
Požadovaný signál: 1111111110000000000000000 (čistá sestupná hrana)
```

#### Krok 1 – Zkouška surového vstupu bez filtru (4 min)
Spusťte program, který při každé detekci úrovně log. 0 okamžitě inkrementuje čítač:

```python
# Program BEZ debouncingu
from machine import Pin
import time

btn = Pin(14, Pin.IN, Pin.PULL_UP)
count = 0
prev_state = 1

while True:
    current_state = btn.value()
    # Detekce sestupné hrany (uvolněno -> stisknuto):
    if prev_state == 1 and current_state == 0:
        count += 1
        print(f"Detekovan stisk cislo: {count}")
    prev_state = current_state
    time.sleep_us(100) # Rychlé vzorkování
```

Proveďte na reálném tlačítku (nebo v simulátoru) přesně 10 zřetelných mechanických stisků a zapište výsledek:
- Počet fyzických stisků: `10`
- Zobrazený počet stisků v konzoli: `...`
- Rozdíl způsobený mechanickými zákmity: `...`

#### Krok 2 – Implementace filtru potlačení zákmitů (Doplňovačka) (6 min)
Doplňte časovou podmínku `DEBOUNCE_DELAY_MS = 30`, která ignoruje změny stavu po dobu 30 ms od první detekované platné hrany:

```python
from machine import Pin
import time

btn = Pin(14, Pin.IN, Pin.PULL_UP)
count = 0
prev_state = 1
last_debounce_time = 0
DEBOUNCE_DELAY_MS = ???   # 💡 DOPLŇTE: typická filtrační doba v ms (doporučeno 30)

while True:
    now = time.ticks_ms()
    current_state = btn.value()

    # Kontrola změny stavu:
    if current_state != prev_state:
        # Zkoumáme, zda uplynul dostatečný čas od posledního platného přechodu:
        if time.ticks_diff(now, last_debounce_time) > DEBOUNCE_DELAY_MS:
            if current_state == 0:  # Potvrzený platný stisk na GND
                count += 1
                print(f"[FILTROVANO] Potvrzeny platny stisk #{count}")
            last_debounce_time = ???    # 💡 DOPLŇTE: uložení aktuálního času
        prev_state = ???                # 💡 DOPLŇTE: aktualizace předchozího stavu
```

*Vaše doplnění:*
- Nastavená filtrační konstanta: `DEBOUNCE_DELAY_MS =` `...`
- Aktualizace času filtru: `last_debounce_time =` `...`
- Uložení předchozího stavu: `prev_state =` `...`

<details>
<summary> :bulb: Nápověda v jazyce ST pro PLC (Funkční blok TOF): </summary>
<p>V PLC systémech se debouncing řeší zapojením funkčního bloku zpožděného odpadu <code>TOF</code> s parametrem <code>PT := T#30ms</code>. Vstupem je syrový signál <code>DI</code> a výstupem je stabilizovaný signál bez zákmitů.</p>
</details>

<details>
<summary> :sparkles: :bulb: Návrh nápovědy pro MCU a IoT (Debouncing v C++ / Arduino Core): </summary>
<p>V programování jednočipů v C++ (např. Arduino pro RP2040 nebo ESP32) se stejný časový filtr řeší pomocí <code>millis()</code> bez blokování:</p>
<pre><code class="language-cpp">if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != buttonState) {
        buttonState = reading;
        if (buttonState == LOW) {
            count++;
        }
    }
}
</code></pre>
<p>Základním pravidlem je neblokovat běh procesoru funkcí <code>delay()</code>, aby systém stíhal další úlohy (např. měření senzorů nebo síťovou komunikaci).</p>
</details>

#### Krok 3 – Vzájemný test a verifikace filtru (5 min)
Protokol testera (otestujte spolužákův program):

| Zkouška  | Akce testera                                              | Očekávaný přírůstek čítače        | Skutečně zaznamenaný přírůstek | Výsledek             |
| :------- | :-------------------------------------------------------- | :-------------------------------- | :----------------------------- | :------------------- |
| **T2.1** | 10x pomalý stisk tlačítka                                 | Přesně +10                        | `...`                          | `[ ] OK`<br>`[ ] KO` |
| **T2.2** | Rychlý "dvojklik" do 20 ms                                | Pouze +1 (druhý zákmyt ignorován) | `...`                          | `[ ] OK`<br>`[ ] KO` |
| **T2.3** | Nastavení neadekvátní filtrace `DEBOUNCE_DELAY_MS = 2000` | Reaguje maximálně 1x za 2 sekundy | `...`                          | `[ ] OK`<br>`[ ] KO` |

> ⚠️ **Častá chyba v zapojení a návrhu:** Použití softwarového debouncingu na bezpečnostních vstupech rychlých optických závor nebo enkodérů otáček motoru. Filtrační zpoždění 30 ms by při frekvenci 1 kHz zcela zlikvidovalo měřený signál! Tam se používá hardwarový RC filtr s Schmittovým klopným obvodem.

> :key: **Vysvětlení pojmů a odborné zdroje:**
> - **Zákmity mechanických kontaktů (Contact Bounce):** Děj způsobený pružností mechanických plíšků kontaktů, kdy dochází k opakovanému rozpojování obvodu s frekvencemi stovek Hz až desítek kHz.
> - **Mechanické zákmity vs. Vzorkovací frekvence smyčky:** Pokud mikrokontrolér prochází smyčku s periodou desítek mikrosekund, mechanické sepnutí kontaktu (trvající desítky ms) zachytí jako sérii desítek rychlých náběžných a sestupných hran.
> - **Softwarový debouncing v MicroPythonu:** Princip časového okna (doba necitlivosti). Proměnná `last_debounce_time` drží časové razítko poslední platné změny. Jakákoli další změna nastalá dříve než za nastavený interval `DEBOUNCE_DELAY_MS` (typicky 20–50 ms) je filtrem ignorována.
> - **Hardwarový debouncing pro mikrokontroléry:** Zapojení dolnofrekvenčního RC článku přímo před GPIO pin (např. rezistor $10\text{ k}\Omega$ a kondenzátor $100\text{ nF}$ s časovou konstantou $\tau = R \cdot C = 1\text{ ms}$). V kombinaci s vnitřním Schmittovým klopným obvodem pinu RP2040 se hrany vyhladí bez jakéhokoliv softwarového zatížení procesoru.
>   GANSSLE, Jack. *A Guide to Debouncing* [online]. Baltimore: The Ganssle Group, 2004, rev. 2008 [cit. 2026-10-26]. Dostupné z: http://www.ganssle.com/debouncing.htm

:star2: **Bonusová otázka k úloze 4:**
Jaké součástky a v jakém zapojení musíte přidat k tlačítku, pokud chcete zákmity odstranit čistě **hardwarovou cestou**, aniž by procesor musel spotřebovávat výpočetní čas na softwarový debouncing?

*Vaše odpověď:*
`...`

---

### 5. Časové sekvenční řízení: Dopravní semafor jako konečný stavový automat (FSM)

*Časová dotace: 20–25 minut | :bangbang: Klasifikovaná úloha - komplexnější úloha ve stylu průvodce*

> **Průmyslová paralela:** Sekvenční řízení technologických celků (např. automatická mycí linka, dávkování chemického reaktoru, přepínání hvězda-trojúhelník u asynchronního motoru) se v praxi projektuje výhradně jako deterministický stavový automat (Finite State Machine – FSM) v souladu s normou ISA-88.

**Cíl úlohy:** Sestavit program pro řízení přechodu pro chodce. Systém pracuje ve standardním cyklu a reaguje na stisk tlačítka chodce s garancí minimální doby zelené pro automobily.

#### Krok 1 – Návrh stavového automatu (FSM) (5 min)
Systém má 4 stavy. Doplňte tabulku stavů a logických úrovní na výstupech ($1 = \text{svítí}, 0 = \text{nesvítí}$):

| Název stavu | Popis situace na křižovatce | Červená AUTA | Žlutá AUTA | Zelená AUTA | Červená CHODCI | Zelená CHODCI | Doba trvání stavu |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `S_AUTA_VOLNO` | Auta jedou, chodec čeká | `0` | `0` | `1` | `1` | `0` | Min. 5 s nebo do stisku tlačítka |
| `S_AUTA_POZOR` | Příprava aut na zastavení | `...` | `...` | `...` | `...` | `...` | Pevně 2 sekundy |
| `S_CHODCI_VOLNO` | Auta stojí, chodec přechází | `...` | `...` | `...` | `...` | `...` | Pevně 5 sekund |
| `S_VYKLIZENI` | Vyklizení přechodu (červená všem) | `1` | `0` | `0` | `1` | `0` | Pevně 2 sekundy |

#### Krok 2 – Implementace neblokujícího stavového automatu (12 min)
Doplňte chybějící části kódu stavového automatu v MicroPythonu:

```python
from machine import Pin
import time

# Definice výstupních pinů semaforu:
led_car_r = Pin(10, Pin.OUT)
led_car_y = Pin(11, Pin.OUT)
led_car_g = Pin(12, Pin.OUT)
led_ped_r = Pin(13, Pin.OUT)
led_ped_g = Pin(14, Pin.OUT)
btn_pedal = Pin(15, Pin.IN, Pin.PULL_UP)

# Pomocná funkce pro nastavení světel:
def set_lights(car_r, car_y, car_g, ped_r, ped_g):
    led_car_r.value(car_r)
    led_car_y.value(car_y)
    led_car_g.value(car_g)
    led_ped_r.value(ped_r)
    led_ped_g.value(ped_g)

# Proměnné stavového automatu:
state = "S_AUTA_VOLNO"
state_start_time = time.ticks_ms()
pedestrian_waiting = False

while True:
    now = time.ticks_ms()
    elapsed = time.ticks_diff(now, state_start_time)

    # Čtení požadavku chodce:
    if btn_pedal.value() == 0:
        pedestrian_waiting = True

    # Vyhodnocení stavů automatu:
    if state == "S_AUTA_VOLNO":
        set_lights(0, 0, 1, 1, 0)
        # Podmínka přechodu: stisknuto tlačítko a auta jela minimálně 5 sekund
        if pedestrian_waiting and elapsed >= 5000:
            pedestrian_waiting = False
            state = "S_AUTA_POZOR"
            state_start_time = ???       # 💡 DOPLŇTE: reset časovače pro nový stav

    elif state == "S_AUTA_POZOR":
        set_lights(0, 1, 0, 1, 0)
        if elapsed >= ???:               # 💡 DOPLŇTE: doba žluté 2000 ms
            state = "S_CHODCI_VOLNO"
            state_start_time = now

    elif state == "S_CHODCI_VOLNO":
        set_lights(1, 0, 0, 0, 1)
        if elapsed >= 5000:
            state = "S_VYKLIZENI"
            state_start_time = now

    elif state == "S_VYKLIZENI":
        set_lights(1, 0, 0, 1, 0)
        if elapsed >= 2000:
            state = ???                  # 💡 DOPLŇTE: návratový stav pro auta
            state_start_time = now
```

*Vaše doplnění:*
- Reset časovače stavu: `state_start_time =` `...`
- Doba trvání žluté výstrahy: `elapsed >=` `...`
- Výchozí stav po vyklizení křižovatky: `state =` `...`

<details>
<summary> :bulb: Stavový automat a asynchronní architektura: </summary>
<p>V programování vestavných systémů a IoT se stavy automatu v MicroPythonu často definují jako celočíselné konstanty namísto textových řetězců, což šetří RAM a zrychluje porovnávání:</p>
<pre><code class="language-python">S_AUTA_VOLNO = const(0)
S_AUTA_POZOR = const(1)
S_CHODCI_VOLNO = const(2)
S_VYKLIZENI = const(3)
</code></pre>
<p>Klíčové je, že v žádném stavu nesmí být použita blokující funkce <code>time.sleep()</code>. Časové přechody jsou řízeny výhradně rozdílem systémového času <code>time.ticks_diff(now, state_start_time)</code>, takže procesor může mezitím kdykoliv zachytit stisk tlačítka chodce nebo odesílat telemetrická data přes Wi-Fi.</p>
</details>

#### Krok 3 – Protokol vzájemné oponentury (Peer testing) (5 min)
Tester ověří reakce modelu v simulátoru Wokwi:

| Testovací bod | Provedená akce testerem            | Očekávaná reakce programu                                         | Zjištěný stav testerem | Závěr                |
| :------------ | :--------------------------------- | :---------------------------------------------------------------- | :--------------------- | :------------------- |
| **T3.1**      | Bez stisku tlačítka po dobu 20 s   | Svítí trvale zelená pro auta, červená pro chodce                  | `...`                  | `[ ] OK`<br>`[ ] KO` |
| **T3.2**      | Okamžitý stisk ihned po náběhu     | Auta mají zelenou ještě garantovaných 5 s, teprve pak žlutá       | `...`                  | `[ ] OK`<br>`[ ] KO` |
| **T3.3**      | Bezpečnostní audit: kolize signálů | Ani v jednom stavu nesmí současně svítit zelená pro auta a chodce | `...`                  | `[ ] OK`<br>`[ ] KO` |

> ⚠️ **Kritická bezpečnostní chyba:** Pokud programátor zapomene zhasnout předchozí LED při přechodu do nového stavu, může nastat situace, kdy svítí zelená pro chodce i pro auta současně. V reálných řadičích křižovatek tuto situaci jistí nezávislý bezpečnostní hardwarový monitor napětí na žárovkách (Green Conflict Monitor), který při chybě okamžitě shodí křižovatku do blikající žluté.

> :key: **Vysvětlení pojmů a odborné zdroje:**
> - **Konečný stavový automat (Finite State Machine - FSM):** Model diskrétního řízení složený z konečného počtu stavů, přechodů mezi nimi a akcí. Zabraňuje nekonzistentním mezistavům programu.
>   Konečný automat. In: *Wikipedie: Otevřená encyklopedie* [online]. San Francisco (CA): Wikimedia Foundation, 2024 [cit. 2026-10-14]. Dostupné z: https://cs.wikipedia.org/wiki/Kone%C4%8Dn%C3%BD_automat
> - **Konečný stavový automat v Embedded systémech:** Způsob strukturování kódu do diskrétních stavů (`state`), v nichž se vykonávají pouze definované akce (např. nastavení výstupů funkcí `set_lights()`). Přechod do dalšího stavu nastává splněním tranzice (časový limit nebo událost z DI).
> - **Eliminace souběhu (Race Conditions):** Na rozdíl od blokujícího kódu s postupným voláním `sleep()` zaručuje stavový automat deterministické chování – procesor v každém okamžiku přesně ví, v jakém stavu se nachází, a nehrozí uvíznutí při externím podnětu.

:star2: **Bonusová otázka k úloze 5:**
Jak byste stavový automat rozšířili o noční režim, kdy semafor po obdržení signálu z nadřazeného systému (nebo po uplynutí reálného času z RTC) přejde do stavu trvale blikající žluté s frekvencí 1 Hz?

*Vaše odpověď:*
`...`

---
### 6. Čtení IoT senzoru teploty DS18B20 na 1-Wire sběrnici a časované vzorkování

*Časová dotace: 20–25 minut | :bangbang: Klasifikovaná úloha - průvodce prací s digitálními senzory a časovaným alarmem*

> **Průmyslová paralela:** V potravinářských chladírnách (HACCP) nebo rozvodnách VN je nepřípustné vyhlásit poplach při jednorázovém záchvěvu čidla způsobeném napěťovou špičkou. Používá se časová integrace: alarm se aktivuje pouze tehdy, pokud překročení kritické meze trvá nepřetržitě déle než stanovená doba (např. 15 sekund).

**Cíl úlohy:** Připojit digitální teplotní senzor DS18B20 k Raspberry Pi Pico W, vyčíst 64bitovou hardwarovou ROM adresu čidla a implementovat časovaný alarm s prodlevou.

```
       +3.3V o-------+
                     |
                    [4k7] Pull-up rezistor sběrnice 1-Wire
                     |
   GPIO28 o----------+-----> DATA (žlutý/bílý vodič DS18B20)
   
   +3.3V  o----------------> VDD  (červený vodič)
   GND    o----------------> GND  (černý vodič)
```

#### Krok 1 – Hardwarové zapojení sběrnice 1-Wire (4 min)
Sběrnice Dallas 1-Wire využívá otevřený kolektor (Open-Drain).
1. Připojte senzor DS18B20 k pinu **GPIO28**.
2. **Kritický krok:** Zapojte pull-up rezistor $4{,}7\text{ k}\Omega$ mezi pin **GPIO28** a napájení **+3,3 V**.
*Kontrolní otázka:* Co se stane se sběrnicí 1-Wire, pokud rezistor $4{,}7\text{ k}\Omega$ vynecháte?
`...`

#### Krok 2 – Doplňovačka: Čtení senzoru a časovaný alarm (12 min)
Doplňte chybějící volání knihoven a časové parametry:

```python
import machine
import onewire, ds18x20
import time

# 1. Inicializace sběrnice 1-Wire na pinu GPIO28:
ow_pin = machine.Pin(28)
ds_sensor = ds18x20.DS18X20(onewire.OneWire(ow_pin))

# Vyhledání všech unikátních čipů na sběrnici:
roms = ds_sensor.scan()
if not roms:
    print("CHYBA: Zadny senzor DS18B20 nebyl nalezen! Zkontrolujte pull-up rezistor.")
    raise SystemExit
print("Nalezen senzor s ROM:", [hex(b) for b in roms[0]])

alarm_led = machine.Pin(15, machine.Pin.OUT)

TEMP_LIMIT = 28.0           # Kritická teplota ve stupních Celsia
SAMPLE_INTERVAL_MS = 2000   # Měření každé 2 sekundy
ALARM_PERSIST_MS = 6000     # Teplota musí být překročena nepřetržitě 6 sekund

last_sample_time = time.ticks_ms()
temp_exceeded_start = None  # Čas, kdy teplota poprvé překročila mez

while True:
    now = time.ticks_ms()

    # Periodické spuštění konverze a čtení:
    if time.ticks_diff(now, last_sample_time) >= SAMPLE_INTERVAL_MS:
        # Zahájení měření v čidle:
        ds_sensor.???()                     # 💡 DOPLŇTE: metoda convert_temp()
        time.sleep_ms(750)                  # Doba AD převodu při 12bitovém rozlišení
        current_temp = ds_sensor.???(roms[0]) # 💡 DOPLŇTE: metoda read_temp()
        print(f"Aktualni teplota: {current_temp:.2f} °C")
        last_sample_time = now

        # Časová analýza alarmu:
        if current_temp > TEMP_LIMIT:
            if temp_exceeded_start is None:
                temp_exceeded_start = now   # Záznam prvního překročení meze
                print("VAROVANI: Teplota prekrocena! Bezi overovaci cas...")
        else:
            temp_exceeded_start = ???       # 💡 DOPLŇTE: reset casovace pri navratu pod mez
            alarm_led.off()

    # Vyhodnocení, zda překročení trvá požadovanou dobu ALARM_PERSIST_MS:
    if temp_exceeded_start is not None:
        if time.ticks_diff(now, temp_exceeded_start) >= ALARM_PERSIST_MS:
            alarm_led.on()
            print("🚨 POPLACH: Teplota je trvale nad limitem!")
```

*Vaše doplnění:*
- Příkaz pro zahájení měření: `ds_sensor.` `...` `()`
- Příkaz pro přečtení teploty: `ds_sensor.` `...` `(roms[0])`
- Reset časovače překročení teploty: `temp_exceeded_start =` `...`

<details>
<summary> :bulb: Časová integrace alarmu a IoT telemetrie: </summary>
<p>V mikrokontrolérech a IoT aplikacích se naměřená hodnota neposílá na cloud (MQTT / REST API) při každém záchvěvu. Kód implementuje dva časovače bez blokování smyčky:</p>
<ol>
  <li><strong>Vzorkovací interval (Sample Interval):</strong> Pravidelné čtení hodnoty každé 2 s funkcí <code>time.ticks_diff(now, last_sample_time)</code>.</li>
  <li><strong>Časová hystereze (Alarm Persistence):</strong> Proměnná <code>temp_exceeded_start</code> zaznamená první okamžik překročení limitu. Teprve po setrvání nad mezí déle než 6 s se rozsvítí indikační LED a odešle se varovná zpráva (např. MQTT zpráva do Home Assistantu). Pokud teplota klesne dříve, časovač se resetuje na <code>None</code>.</li>
</ol>
</details>

#### Krok 3 – Protokol vzájemného přezkoušení (Peer testing) (5 min)
Tester otestuje funkčnost v simulátoru Wokwi (kde lze kliknutím na DS18B20 plynule měnit teplotu jezdcem):

| Krok testu | Činnost testera                                             | Očekávané chování systému                                   | Naměřená odezva | Hodnocení            |
| :--------- | :---------------------------------------------------------- | :---------------------------------------------------------- | :-------------- | :------------------- |
| **T4.1**   | Nastavení teploty na 25 °C                                  | LED alarmu nesvítí                                          | `...`           | `[ ] OK`<br>`[ ] KO` |
| **T4.2**   | Zvýšení teploty na 32 °C po dobu 3 sekund a návrat na 25 °C | Poplach se nesmí aktivovat (krátkodobý výkyv ignorován)     | `...`           | `[ ] OK`<br>`[ ] KO` |
| **T4.3**   | Trvalé zvýšení teploty na 32 °C                             | Přesně po 6 sekundách od překročení se rozsvítí červená LED | `...`           | `[ ] OK`<br>`[ ] KO` |

> :key: **Vysvětlení pojmů a odborné zdroje:**
> - **Sběrnice 1-Wire:** Průmyslová asynchronní sběrnice vyvinutá firmou Dallas Semiconductor (nyní Analog Devices). Komunikuje po jediném datovém vodiči a zemi, každé zařízení má celosvětově unikátní 64bitové sériové číslo (ROM kód).
> - **Knihovní moduly `onewire` a `ds18x20`:** V MicroPythonu tvoří dvě vrstvy – `onewire` řídí přesné mikrosekundové časování bitů po sběrnici (bit-banging), zatímco `ds18x20` poskytuje vysokoúrovňové metody pro čtení teplotních datových registrů.
> - **Topologie sběrnice s otevřeným drainem (Open-Drain):** Pin čidla DS18B20 aktivně stahuje linku pouze k zemi (GND). Logickou úroveň 1 zajišťuje externí pull-up rezistor $4{,}7\text{ k}\Omega$ připojený na 3,3 V. Bez tohoto rezistoru zůstává linka trvale na nulovém napětí a komunikace selže.
> - **Metoda `.convert_temp()` a doba převodu:** Odesílá všem čidlům příkaz ke spuštění interního převodu. Při 12bitovém rozlišení potřebuje integrovaný Sigma-Delta A/D převodník v senzoru dobu až 750 ms, teprve poté lze číst platná data metodou `.read_temp(rom)`.
>   ANALOG DEVICES. *DS18B20: Programmable Resolution 1-Wire Digital Thermometer* [online]. Rev. 6. Norwood (MA): Analog Devices, 2019 [cit. 2026-10-26]. Dostupné z: https://www.analog.com/media/en/technical-documentation/data-sheets/DS18B20.pdf

:star2: **Bonusová otázka k úloze 6:**
Proč vyžaduje převod teploty v senzoru DS18B20 při maximálním rozlišení 12 bitů dobu až 750 ms, zatímco čtení digitálních dat po sběrnici trvá jen několik milisekund? Jaký fyzikální princip převodníku čip uvnitř využívá?

*Vaše odpověď:*
`...`

---


