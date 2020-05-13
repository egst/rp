Popisuji zde syntaxi neformálně. Tento dokument bude zdrojem pro uživatelský manuál.
Formální definice gramatiky je v souboru `grammar.md`.
Při popisu syntaxe používám závorky pro označení nepovinných částí.
Specifické pojmenované části uvozuji do složených závorek.
Závorky jako symbol jazyka vždy odděluji z obou stran mezerami.
Zápis `(x y z (...))` znamená nepovinné opakování `x y z`.
Při takovém zápisu může dojít k nějakým nejednoznačnostem v interpretaci, takže spoléhám na odvození správné interpretace z kontextu.
Výsledný manuál bude ale psán v nějakém grafickém formátu, kde bude použito různých barev a stylů písma k odlišení této notace.

## Obecná struktura programu

Celý program je posloupností **příkazů** oddělených středníky.
Příkaz je buď tzv. pure/**nevýrazový**, nebo **výrazový**.
Pžíkaz může být nezávisle na "výrazovosti" tzv. **složený**, nebo **jednoduchý**.
Výrazový příkaz může být tzv. **right-consuming**, což ovlivňuje jen syntaxi, ne sémantiku.
Zobecněním výrazů je tzv. **ESX** [^1], což obsahuje jak **výrazy** tak výrazové příkazy.
Výrazy také mohou být syntakticky right-consuming.
I výrazy mohou být složené a jednoduché.

*K syntaktickému konceptu "right-consuming" se dostanu dále.*

Nevýrazovými příkazy jsou:

* prázdný příkaz
* vrácení
* deklarace porměnných (s nepovinným přiřazením)
* přiřazení [^2]
* deklarace + definice funkce
* některé podmínky (`if`, `when`)

Výrazovými příkazy jsou:

* blocky (`{}`)
* některé podmínky (`if`)

Samotný výraz není ani výrazovým ani nevýrazovým příkazem.

Složenými jsou příkazy:

* blocky
* některé podmínky (`if`, `when`)

Ostatní výrazy jsou jednoduché.

Hierarchicky to lze znázornit takto:

* příkaz
    * ESX
        * výraz
        * výrazový příkaz
    * nevýrazový příkaz

* příkaz
    * jednoduchý
    * složený

* výraz
    * jednoduchý
    * složený

Výrazovost příkazu vypovídá o jeho použitelnosti v roli výrazu.
Right-consuming určuje syntaktické chování výrazu.
V případě příkazu jde jen o vlastnost, která určuje chování výrazu vytvořeného tímto příkazem.
Složenost příkazu i výrazu umožňuje vytvoření scopu.

[^1] Dočasný název, vymyslím pak něco lepšího.
[^2] V dalších verzích zvažuji implementovat přiřazení jako výraz.

### Typy

Typy jsou buď **jednoduché**, nebo **složené**.

Vestavěnými jednoduchými typy jsou `Boolean`, `Integer`, `Real` a `Complex` [^1].
Složené typy se vytvoří z jednoduchých a reprezentují funkce, buffery a vektory. [^1]

Uživatel může nadefinovat vlastní typy pomocí příkazu **typového aliasu**:

```
type {name} = {type}
```

`{name}` je název nového typu  
`{type}` je libovolný typ

Typový alias je v kontextu jen po deklaraci a v rámci svého **scopu**. *Podrobněji ke konceptu scopu se dostanu dále.*

Vestavěné jednoduché typy i uživatelem nadefinované aliasy vždy začínají velkým písmenem
a pokračují posloupností malých i velkých písmen, podtržítek a číslic.

Pro vestavěné jednoduché typy jsou definovány zkratkové aliasy:

```
type Bool = Boolean;
type Int  = Integer;
```

Pro automatické **odvození typu** je zaveden symbol `*`, který lze použít na místě libovolného typu. [^2]

Typový systém je statický bez žádného polymorfismu (kromě vestavěných operací).
Odvození typu neumožňuje polymorfismus, ale jen určuje vhodný typ automaticky za překladu.

[^1] Typ `Complex`, bufferové a vektorové typy budou implementovány v dalších verzích.
[^2] Odvození typů bude implementováno v dalších verzích.

### Deklarace a přiřazení jednoduchých typů

Proměnnou jednoduchého typu je možné deklarovat pomocí příkazu **deklarace**:

```
{type} {name} (= {value})
```

`{type}`  je požadovaný typ  
`{name}`  je název proměnné  
`{value}` je přiřazená hodnota daná výrazem, nebo výrazovým příkazem

Je možné ponechat proměnnou neinicializovanou. Překladač na to však uživatele vždy upozorní,
protože proměnná před přiřazením nemá definovanou hodnotu - není implicitně inicializovaná na žádnou výchozí hodnotu. [^1]

Názvy proměnných vždy začínají malým písmenem, nebo podtržítkem
a pokračují posloupností malých i velkých písmen, podtržítek a číslic.

Proměnná je v kontextu jen po deklaraci a v rámci svého scopu.
Proměnná jednoduchého typu tzv. **obsahuje** (nejde o referenci) uloženou hodnotu.
Životnost obsahované hodnoty je tedy spjatá se scopem proměnné.
Jakmile proměnná vychází z kontextu, končí i životnost obsahované hodnoty.

Do proměnné jednoduchého typu je možné i po deklaraci [^2] přiřadit příkazem **přiřazení**:

```
{name} = {value}
```

Na levé straně se může vyskytovat jen název proměnné, ne výraz.

Proměnná tvoří výraz odpovídajícího typu.

[^1] Prozatím to nechci řešit, ale implicitně inicializovat dekalrované proměnné na nějakou výchozí hodnotu by možná bylo vhodné.
[^2] Do budoucích verzí zvažuji implementovat i konstantní (imutabilní) proměnné jednoduchých typů.

### Blocky

**Block** tvoří složený výrazový příkaz.
Příkazy v blocích jsou odděleny středníkem.
Prázdný block je také povolen.

```
{ ({stmt} (; {stmt} (...))) }
```

`{stmt}` je libovolný příkaz

### Podmínky

Podmínka **if** má dvě **větve**:

```
if ({cond}) {then} else {else}
```

`{cond}` je podmínka daná výrazem  
`{then}` je pozitivní větev  
`{else}` je negativní větev

Podmínka **when** má větev jen jednu:

```
when ({cond}) {then}
```

Použití podmínky if bez negativní větve je syntaktickou chybou,
stejně tak jako použití podmínky while následnované klíčovým slovem `else` s negativní větví.

Podmínka if může být výrazem, výrazovým i nevýrazovým příkazem v závislosti na větvích.
Větve mohou být dány výrazy i libovolnými příkazy.

Má-li podmínka if obě větve dané výrazem, tvoří right-consuming výraz.
Jinak, má-li oba výrazy dané ESX, tvoří složený výrazový příkaz, který je right-consuming,
pokud je negativní větev dána výazem, nebo složený right-comsuming příkazem.
Jinak (má-li alespoň jednu větev danou nevýrazovým příkazem) tvoří složený nevýrazový příkaz.

Podmínka while vždy tvoří složený nevýrazový příkaz. Větve mohou být dány libovolnými příkazy.

Všechny podmínky kromě čistě výrazového if tedy tvoří složené příkazy.

```
if (c) x;             // syntaktická chyba, chybí else
when (c) x;           // nevýrazový příkaz
if (c) x else y;      // výraz - obě větve jsou dány výrazem
if (c) x else {};     // výrazový příkaz - pravá větev je dána výrazovým příkazem
if (c) {} else x;     // right-consuming příkaz - pravá větev je dáva výrazem
if (c) x = y else y;  // nevýrazový příkaz - jedna z větví je dána nevýrazovým příkazem
```

### Right-consuming výrazy

Většina operací mají danou **precedenci** a **asociativitu**, které určují implicitní uzávorkování.
*K precedenci operací se dostanu dále.*
**Right-consuming** operace však mají naprosto jiné syntaktické chování.
Right-consuming operace vážou veškerý zbytek příkazu napravo na sebe
a jsou přitom bez explicitního uzávorkování použitelné v libovolné jiné operaci jako pravý operand.

Příkladem je výraz if:

```
if (c) x else if (d) y else z;
```

Implicitní uzávorkování je následující:

```
if (c) x else (if (d) y else (z));
```

Stejné pravidlo platí i pro ostatní operace, např. binární infixové `+`:

```
x + if (c) y else y + z;
```

Implicitní uzávorkování je:

```
x + (if (c) y else (y + z));
```

Right-consuming výraz tedy není možné použít jako levý operand binární infixové operace bez explicitního uzávorkování.

### Typová anotace

**Typová anotace** se provede napsáním požadovaného typu před výrazový příkaz:

```
{type} {stmt}
```

`{type}` je požadovaný typ
`{stmt}` je výazový příkaz

Při explicitní typové anotaci se z (right-consuming) výrazového příkazu stává (right-consuming) výraz.
Vlastnost right-consuming v kontextu příkazu je syntakticky bezvýznamná.
Při převodu na výraz se ale zachovává. U příkazu tedy jen vypovídá o syntaktickém chování po potenciálním převodu na výraz.

```
{};                        // výrazový příkaz
Int {};                    // výraz
if (c) {} else {};         // výrazový příkaz
if (c) Int {} else Int {}; // výraz (1)
Int if (c) {} else {};     // výraz se stejnou sémantikou jako výraz (1)
Int if (c) {} else x + y;  // right-consuming výraz
```

Výrazové příkazy je možné použít v roli výrazu buď explicitní typovou anotací, nebo
v rámci jiného výrazu či příkazu, který to umožňuje bez uvedení typu.
Příkladem takového příkazu je samotné if, větve kterého mohou být uvedeny
samotnými výrazovými příkazy, které se pak implicitně anotují společným typem
uvedeným ještě před klíčovým slovem if.
Sémantika se u konkrétních výrazů a příkazů trochu liší (*a bude popsána dále*), ale syntaxe zůstává stejná.

Použitím nějakého syntaktického konstruktu **v roli výrazu** je myšleno použití samotného výrazu,
nebo použití hodnoty výrazového příkazu.
Použitím **v roli příkazu** je myšleo použití samotného výrazu jako příkazu,
použití nevýrazového příkazu, nebo použití výazového příkazu bez pokusu o evaluaci jeho hodnoty.
**Hodnotu výrazového příkazu** získá buď uživatel explicitní typovou anotací,
nebo nějaká specifická vestavěná operace po implicitním určení typu.
Při použití konstruktu v roli výazu dochází **evaluaci jeho hodnoty**.

```
// block v roli výrazu:
Int i = Real {}; // uživatel explicitně vyhodnotí hodnotu příkazu {} s typem Real
Int i = {};      // operace přiřazení implicitně vyhodnotí hodnotu příkazu {} s typem Int
// block v roli příkazu:
{}; // návratová hodnota zde není vyhodnocena
```

### Sémantika evaluace příkazů - návratová hodnota blocku

Syntakticky je možné použít libovolný block v roli výrazu (libovolného typu).
Aby bylo sémanticky korektní použít block jako výraz, musí mít nějakou **návratovou hodnotu**.
Návratovou hodnotu blocku v roli výrazu určuje příkaz **vrácení**:

```
return {value}
```

`{value}` je návratová hodnota blocku daná výrazem, nebo výrazovým příkazem.

Blocky použité v roli příkazu nemají hodnotu.
Příkaz vrácení se tedy vždy vztahuje na nejbližší block v roli výrazu, který toto vrácení obsahuje.
Příkaz vrácení se takto tzv. **propaguje** na odpovídající block v roli výrazu.
Takový block má jednoznačně určený typ. Hodnota příkazu vrácení musí být implicitně konvertovatelná na tento typ.
*Podrobněji k typovým konverzím se dostanu dále.*
Příkaz vrácení ukončuje evaluaci odpovídajícího blocku v roli výazu i běh vnořených blocků v roli příkazů.
Další příkazy se ignorují.

```
Int x;
Int { return x };     // návratová hodnota typu Int
Int { { return x } }; // návrat se propaguje přes jeden vnitřní block na ten vnější v roli výrazu
Int {};               // syntakticky ok, ale sémanticky dochází k chybě - block v roli výrazu nevrací
Int { T y; return y } // Potenciální implicitní konverze z T na Int, pokud to je možné; jinak dojde k chybě
```

### Sémantika evaluace podmínek

V případě použití výrazu if v roli výrazu není třeba uvádět typ. Typ se určí z daných větví.
Vybere se společný typ, na který lze obě větve implicitně konvertovat podle pravidel,
ke kterým se dostanu dále u konverzí. Na tento společný typ se obě větve konvertují.
Podle hodnoty podmínky je výslednou hodnotou jedna z větví po konverzi.
Pokud takový společný typ vybrat nelze, doje k chybě.

V případě použití výrazového příkazu if v roli výazu se typ v typové anotaci (daný explicitně, nebo implicitně jiným konstruktem)
použije pro implicitní anotaci větví daných výrazovým příkazem. Je-li jedna z větví dána výrazem,
musí být na tento typ implicitně konvertovatelná. Na tento typ se potom konvertuje.
Podle hodnoty podmínky je výslednou hodnotou jedna z větví po konverzi.

```
Int i = if (c) x else Int { return y }; // if je zde výrazem
Int i = if (c) x else { return y };     // if je zde výazovým příkazem (v roli výrazu)
Int i = if (c) x else return y;         // chyba, příkaz vrácení není výrazovým příkazem, tedy i celý if také
```

### Evaluace blocků s podmínkami

Použití podmínek v blocích umožňuje vrácení z podmíněné větve.
Aby byl block použitelný v roli výrazu, musí **vracet vždy**.
Block vrací vždy, obsahuje-li příkaz, který vrací vždy.
Příkaz vrácení vrací vždy.
Příkaz if vrací vždy, vracejí-li vždy obě jeho větve.

Je třeba dávat pozor na následující vlastnosti:
Příkaz when nevrací vždy.
Vrácení se nepropaguje dále z příkazu v roli výrazu.

```
Int {};                                // chyba, nikdy nevrací
Int { if (c) return x else return y }; // ok, vrací vždy
Int { if (c) return x else {} };       // chyba, negativní větev nevrací
Int { when (c) return x };             // chyba, when nevrací vždy
Int i = if (c) { return x } else {};   // chyba, negativní větev nevrací
Int { { return x } };                  // ok, vrácení se propaguje do vnějšího blocku
Int { Int { return x } };              // chyba, vrácení se nepropaguje do vněšího blocku
```

### Scope

Scope je lexikální a vytváří se složeným příkazem či výrazem.
Proměnné jsou definováni v rámci scopu nějakého složeného příkazu,
nebo v rámci globálního scopu, který je dán celým programem.
Celý program tak představuje nevýazový block s implicitním `{}`.

Proměnné jsou viditelné po deklaraci do konce svého scopu i uvnitř vnořených scopů.
Proměnnou je možné deklarovat znovu se stejným jménem ve vnořeném scopu.
To původní deklaraci tzv. **zastíní**.

```
Int i; // v globálním scopu
{
    Int j;               // ve scopu blocku (v roli příkazu)
    Int i;               // zastíní původní deklaraci
    Int {
        Int k;           // ve scopu blocku (v roli výazu)
    };
    if (c) Int l else {} // ve scopu podmínky
};
Int j = i;               // použije se i z globálního scopu
```

Deklarace je možná i uvnitř podmínek. Samotná deklarace v některé větvi je sémanticky nepoužitelná.
Překladač na to upozorní jako i u jiných nepoužitých deklarací.

### Funkce

Funkce je možné deklarovat a definovat příkazem **definice funkce**:

```
{ret} {name} ( ({type} {param} (, {type} {param} (...))) ) {body}
```

`{ret}` je návratový typ funkce [^1]  
`{name}` je název funkce  
`{type}` jsou typy parametrů  
`{param}` jsou názvy parametrů
`{body}` je tělo funkce dané výrazem, nebo výrazovým příkazem

Jelikož deklarace funkce je vždy spjatá s definicí, obojí se bude označovat jako **definice funkce**.

Definice funkce tvoří složený nevýrazový příkaz.
Název funkce je sémanticky ekvivalentní proměnné složeného typu funkce.

Složený typ funkce je tvořen následujícím způsobem:

```
{ret} ( ({param} (, {param} (...))) )
```

Tělo funkce (`{body}`) je dáno výazem, nebo výrazovým příkazem.
Hodnota se neevaluje při definici.
Parametry funkce jsou proměnné deklarované ve scopu definice funkce.
Rozsah tohoto scopu je určen tělem funkce.
Na scopu funkce není sémanticky nic speciálního.
Proměnné z vnějšího scopu jsou tedy vždy viditelné i v těle funkce.
Parametry mohou tyto proměnné zastínit, stejně tak jako proměnné deklarované v těle funkce.

Funkci lze zavolat výrazem **volání**:

```
{fun} ( ({arg} (, {arg} (...))) )
```

`{fun}` je výraz funkčního typu  
`{arg}` jsou předané argumenty

```
Int y;
Int f (Int x) x + y; // funkce typu Int(Int) využívající proměnnou y z vnějšího scopu
Int i = f(y);        // zavolání funkce a přiřazení výsledné hodnoty
```

Tělo funkce může být dáno výazovým příkazem.
Příkaz vrácení v tomto případě není nijak speciální sémanticky.
Jde jen o vrácení z výrazového příkazu při evaluaci jeho hodnoty,
která nastane při volání funkce.

```
Int f (Int x) { return x };
Int g (Bool c, Int x, Int y) if (c) { return x } else { return y };
Int h (Bool c, Int x) if (c) { return x } else {}; // chyba
Int i (Bool c, Int x) when (c) { return x };       // chyba
```

Syntakticky funkce také není nijak speciální. Lze ji definovat i uvnitř jiných složených příkazů.

```
Int f (Int x) {
    Int i = 1;
    Int g () { i = 5; return 0 };
    g();
    return i;
};
```

Funkci lze přiřazovat do proměnných (funkčního typu), předávat parametrem i vracet z jiné funkce.

```
Int f (Int(Int) g, Int x) g(x) + x;

Int(Int) g (Int x) {
    Int f (Int y) y + y;
    return f;
}

Int i;
Int(Int) h = g(i);
Int j = f(h, i);
```

Proměnné funkčního typu jsou **imutabilní**. Nelze do nich opakovaně přiřadit.
Z toho plyne i to, že proměnná funkčního typu musí být definována už při deklaraci.
To je konzistentní s příkazem definice funkce, který provádní oboje.

```
Int f (Int x) x;
Int g (Int x) x + x;

Int(Int) h = f; // ok
h = g;          // chyba
f = g;          // chyba

Int(Int) i; // chyba
```

[^1] Návratový typ je prozatím povinný. Funkce bez návratové hodnoty budou přidány v dalších verzích.
[^*] Rekurze prozatím není podporována. Neměl by to být problém, ale není to momentálně nutné.

### Anonymní funkce

Funkci je možné definovat i výrazem **definice anonymní funkce**:

```
{ret} ( ({type} {param} (, {type} {param} (...))) ) {body}
```

Definice anonymní funkce tvoří složený right-consuming výraz.
Vytváří tedy vlastní scope, ve kterém jsou deklarovány dané parametry.
Anonymní funkci je možné stejně jako obyčejnou funkci přiřadit do proměnné, předat parametrem i vrátit z funkce.
Na imutabilitě takových proměnných se nic nemění.

```
Int(Int) f = Int (Int x) x;

Int(Int) g (Int x) Int (Int y) x + y;
// implicitně uzávorkováno jako:
Int(Int) g (Int x) (Int (Int y) (x + y));

Int a;
Int b;
g(a)(b);
```

V budoucích verzích bude vyřešeno odvození návratového typu funkcí, což umožní vynechání opakovaného zápisu typů u curryovaných funkcí:

```
* g (Int x) Int (Int y) x + y;
```

### Hodnoty jednoduchých typů

Typ `Integer` bude implementován některým ze znaménkových celočíselných typů v C.
Přetečení nebude kontrolováno.

Typ `Real` bude implementován některým z floating-point typů v C.
Přetečení ani zaokrouhlovací chyby nebudou kontrolovány.

Typ `Boolean` má dvě hodnoty: `true` a `false`.

### Typová konverze

Použití typu před výrazem je syntakticky podobné typové anotaci, ale sémanticky jde o **typovou konverzi**:

```
{type} {expr}
```

`{type}` je typ  
`{expr}` je výraz

Při konverzi z `Integer` na `Real` může dojít k zaokrouhlovací chybě.
Při konverzi z `Real` na `Int` dojde k zaokrouhlení k nule.
Při konverzi na `Boolean` se nulové hodnoty konvertují na `false`, ostatní na `true`
Při konverzi z `Boolean` se `true` konvertuje na 0, `false` na 1.
Konverze mezi funkčním typem a jednoduchým typem není možná.

Další konverze probíhají v tzv. **krocích**:
* `Boolean -> Integer`
* `Integer -> Boolean`
* `Integer -> Real`
* `Real -> Integer`

Tedy například konverze z `Boolean` do `Real` projde dvěma kroky `Boolean -> Integer` a `Integer -> Real`.
V případě stejného počtu kroků se preferují konverze směrem `Boolean -> Integer -> Real`.

### Implicitní konverze

Implicitně mohou proběhnout libovolné konverze mezi jednoduchými typy. [^1]

Implicitní konverze probíhají při:

* přiřazení
* předání argumentu
* vrácení
* dalších vestavěných operacích

Při výběru overloadu vestavěné operace se vybere operace s typy na které je třeba nejmíň kroků implicitní konverze.

[^1] S tím počítám prozatím, ale nejspíš to v dalších verzích nějak omezím.
Prozatím nevím, jaké konverze budou vhodné a jaké by naopak mohly někde uškodit.

### Literály jednoduchých typů

Literálem typu `Int` je libovolná posloupnost číslic reprezentující číslo v desítkové soustavě
volitelně následovaná znakem `e` či `E`, za kterým pokračuje další posloupnost číslic udávající
exponent ve vědecké notaci.

```
{num}((e|E){exp})
```

Literál typu `Real` navíc umožňuje použití desetinné tečky. Implicitní nulu před, nebo za desetinnou tečkou je možné vynechat.

```
{num}.{dec}((e|E){exp})
.{dec}((e|E){exp})
{num}.((e|E){exp})
```

Literály typu `Bool` jsou stringy `true` a `false`.

Literály tvoří výraz odpovídajícího typu.

### Další vestavěné operace

Vestavěné operátory jsou buď unární prefixové, nebo binární infixové.
Budou zde označeny jednoduše jako **unární** a **binární**.

Logické operátory:

* unární **negace** `!`
* binární **konjunkce** `&&`
* binární **disjunkce** `||`

Aritmetické operátory:

* unární **plus** a **minus**: `-` `+`
* binární **sčítání** a **odečítání**: `+` `-`
* binární **násobení**, **dělení** (i celočíselné) a **modulo**: `*` `/` `%`

Porovnávací operátory:

* binární porovnání v uspořádání: `<` `<=` `>` `>=`
* binární **rovnost** a **nerovnost**: `==` `!=`

Logické operátory očekávají operandy typu `Boolean`. Operandy jiných typů konvertují na `Boolean`.
Konjunkce i disjunkce jsou "short-circuit", tedy zbytečně neevalují operandy, když jejich hodnota o té výsledné nerozhodne.

Celočíselné dělení očekává levý operand typu `Integer`, pravý typu `Integer` nebo `Real` a výsledkem je `Real`.
Dělení očekává oba operandy typu `Real` a výsledkem je `Real`.

Modulo očekává levý operand typu `Integer` nebo `Real`, pravý typu `Integer` a výsledkem je `Real`.
Nezápornost pravého operandu se nekontroluje a výsledek je nedefinován.

Ostatní aritmetické operátory očekávají oba operandy typu `Integer` nebo `Real` (oba stejné) a výsledkem je stejný typ.

Porovnávací operátory očekávají oba operandy libovolného jednoduchého typu (oba stejné) a výsledkem je `Boolean`.
