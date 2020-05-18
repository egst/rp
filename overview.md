Popisuji zde syntaxi neformálně. Tento dokument bude zdrojem pro uživatelský manuál.
Formální definice gramatiky je v souboru `grammar.md`.
Zde při popisu syntaxe používám závorky pro označení nepovinných částí.
Specifické pojmenované části uvozuji do složených závorek.
Závorky jako symbol jazyka vždy odděluji z obou stran mezerami.
Zápis `(x y z (...))` znamená nepovinné opakování `x y z`.
Při takovém zápisu může dojít k nějakým nejednoznačnostem v interpretaci, takže spoléhám na odvození správné interpretace z kontextu.
Výsledný manuál bude ale psán v nějakém grafickém formátu, kde bude využito různých barev a stylů písma k odlišení této notace.
V blockquotech (>) uvádím detailnější důvody ke konkrétním rozhodnutím.
[^N] odkazuje na poznámku, která je vždy uvedena na konci aktuální sekce.
Sekce jsou odděleny nadpisy (##...).

## Obecná struktura programu

Celý program je posloupností **příkazů** oddělených středníky.
Příkaz je buď tzv. **nevýrazový** (expression statement), nebo **výrazový** (pure statement).
Výrazový příkaz může být tzv. **vyjádřený**.
Pžíkaz je nezávisle na "výrazovosti" tzv. **složený** (composite), nebo **jednoduchý** (simple).
Výrazový příkaz může být tzv. **right-consuming**, což ovlivňuje jen syntaxi, ne sémantiku.
Zobecněním výrazů je tzv. **ESX** [^1], což obsahuje jak **výrazy** tak výrazové příkazy.
Výraz je také příkazem.
Výrazy také mohou být syntakticky right-consuming.
I výrazy mohou být složené a jednoduché.

Výrazovost příkazu ovlivňuje možnosti jeho výskytu v kódu.
Výrazové příkazy je možné v některých kontextech použít roli výrazu - tzv. **vyjádřit** (express).
Výrazový příkaz se tak stává **vyjádřeným** (expressed statement).
Vyjádřený příkaz se může stát součástí specifických výrazů.

> Výrazové příkazy umožňují vnořenou procedurální/imperativní definici i v čistě deklarativních výrazech.
> Umožňují vytvoření nového scopu pro pomocné proměnné.
> Také nevyžadují dopřednou deklaraci cílových proměnných ve vnějším scopu,
> což vylučuje potřebu nechávat nedefinované proměnné.

Složenost příkazu i výrazu umožňuje vznik scopu.
Tento scope zahrnuje např. proměnné deklarované v blocku nebo parametry funkce.

> Termín "složený příkaz" zde má trochu nestandardní význam.
> Pro block ({}) používám jen termín "block", ne "složený příkaz", protože jde o obecnější pojem.
> Složeným příkazem je třeba i podmínka s jediným příkazem,
> což vylučuje možnost deklarovat proměnnou podmíněně,
> aniž by to bylo explicitně zakázáno u podmínek.

Right-consuming určuje syntaktické chování výrazu.
V případě příkazu určuje chování výrazu vzniklého vyjádřením příkazu.

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

Složenými příkazy jsou:

* blocky
* některé podmínky (`if`, `when`)

Ostatní příkazy jsou jednoduché.

Jediným složeným výrazem je definice anonymní funkce.

Hierarchicky to lze znázornit takto:

* příkaz
    * ESX
        * výraz
        * výrazový příkaz
            * vyjádřený příkaz
    * nevýrazový příkaz

* příkaz
    * jednoduchý
    * složený

* výraz
    * jednoduchý
    * složený

[^1] Dočasný název, vymyslím pak něco lepšího.

[^2] V dalších verzích zvažuji implementovat přiřazení jako výraz.

## Běh programu

Příkazy programu se **vykonávají** (execute) sekvenčně.
Sémantiku vykonání specifických příkazů popisuji dále podrobněji.

Výrazy se **vyhodnocují** (evaluate) podle implicitního uzávorkování,
které také popisuji dále podrobněji.

Vyjádřené příkazy se také vyhodnocují.

## Typy

Typy jsou buď **jednoduché** (simple), nebo **složené** (composite).

Vestavěnými jednoduchými typy jsou `Boolean`, `Integer`, `Real` a `Complex` [^1].
Složené typy se vytvoří z jednoduchých a reprezentují funkce, buffery a vektory. [^1]

> Některé složené typy bude možné vytvořit jen z těch jednoduchých,
> proto je takto rozděluji.

> Boolean by sice nebyl potřeba, ale ve spojení s omezením implicitních konverzí (popsáno dále)
> poskytuje dodatečné kontroly při překladu, které mohou omezit některé uživatelské chyby.

Uživatel může nadefinovat vlastní typy pomocí příkazu **typového aliasu**:

```
type {name} = {type}
```

`{name}` je název nového typu  
`{type}` je libovolný typ

Typový alias je v kontextu jen po deklaraci a v rámci svého **scopu**.

Vestavěné jednoduché typy i uživatelem nadefinované aliasy vždy začínají velkým písmenem
a pokračují posloupností malých i velkých písmen, podtržítek a číslic.

Pro vestavěné jednoduché typy jsou definovány zkratkové aliasy:

```
type Bool = Boolean;
type Int  = Integer;
```

> Integer a Boolean jsou dost zdlouhavé názvy pro tak často používané typy.
> Na druhou stranu nechci žádné vestavěné konstrukty pojmenovávat zkratkově,
> abych pak neřešil nějaké konvence, aby to bylo konzistentní.
> Je to taková maličkost, ale podle mě docela důležitá maličkost.
> Když se neřeší nějaká konzistence pojmenování, tak to pak dopadá
> jako třeba v PHP, kde je každá standardní funkce pojmenovaná s jinou konvencí.
> Proto si také myslím, že pojmenování typů s velkým písmenem bude přínosné pro udržení pojmenovávacích konvencí.

Pro automatické **odvození typu** je zaveden symbol `$`, který lze použít na místě libovolného typu. [^2]

> Chtěl jsem původně `*`, ale to by mohlo vytvořit nejednoznačnosti v gramatice,
> protože by se to chovalo jako unární prefixové `*` a potenciálně konfliktovalo s binárním infixovým násobením.
> U `+` a `-` je to stejné a `*` by měla fungovat také, ale zatím to nechci řešit.

Typový systém je statický bez žádného polymorfismu (kromě vestavěných operací).
Odvození typu neumožňuje polymorfismus, ale jen určuje vhodný typ automaticky za překladu.

[^1] Typ `Complex`, bufferové a vektorové typy budou implementovány v dalších verzích.

[^2] Odvození typů bude implementováno v dalších verzích.

## Deklarace a přiřazení jednoduchých typů

Proměnnou jednoduchého typu je možné deklarovat pomocí příkazu **deklarace**:

```
{type} {name} (= {value})
```

`{type}`  je požadovaný typ  
`{name}`  je název proměnné  
`{value}` je přiřazená hodnota daná výrazem, nebo výrazovým příkazem

Proměnná tvoří výraz odpovídajícího typu.

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

> Prozatím počítám s přiřazením jako příkazem.
> Vyjadřuje to imperativní význam této operace, takže si myslím, že konceptově to dává smysl.

Na levé straně se může vyskytovat jen název proměnné, ne výraz.

> Reference momentálně neřeším, takže nenastane situace, kdy by výraz měl hodnotu reference,
> do které by bylo možné přiřadit.

Při přiřazení může dojít k implicitní konverzi, nebo chybě, pokud to není možné.

[^1] Prozatím to nechci řešit, ale implicitně inicializovat dekalrované proměnné na nějakou výchozí hodnotu by možná bylo vhodné.

[^2] Do budoucích verzí zvažuji implementovat i konstantní (imutabilní) proměnné jednoduchých typů.

## Blocky

**Block** tvoří složený výrazový příkaz.
Block obsahuje příkazy oddělené středníkem.
Prázdný block je také povolen.

```
{ ({stmt} (; {stmt} (...))) }
```

`{stmt}` je libovolný příkaz

**Vykonání:** Příkazy v blocích se vykonávají sekvenčně.

**Vyhodnocení:** Popsáno dále.

### Podmínky

Podmínka **if** má dvě **větve**:

```
if ({cond}) {then} else {else}
```

`{cond}` je podmínka daná výrazem typu `Boolean`  
`{then}` je pozitivní větev  
`{else}` je negativní větev

Podmínka **when** má větev jen jednu:

```
when ({cond}) {then}
```

Použití podmínky if bez negativní větve je syntaktickou chybou,
stejně tak jako použití podmínky while následnované klíčovým slovem `else` s negativní větví.

> Použití when pro podmínku s jedinou větví zjednodušuje gramatiku,
> nepřidává žádné nekonzistentní výjimky a mohlo by to být i čitelnější pro uživatele
> i bez použití explicitních blocků.

Podmínka if může být výrazem nebo výrazovým i nevýrazovým příkazem v závislosti na větvích.
Větve mohou být dány výrazy i příkazy.

Má-li podmínka if obě větve dané výrazem, tvoří right-consuming výraz - tzv. **výrazové if** (jinak jde o **příkazové if**).
Jinak, má-li oba výrazy dané ESX, tvoří výrazový příkaz, který je right-consuming,
pokud je negativní větev dána výazem, nebo výrazovým right-comsuming příkazem.
Jinak (má-li alespoň jednu větev danou nevýrazovým příkazem) tvoří složený nevýrazový příkaz.

Podmínka while vždy tvoří složený nevýrazový příkaz. Větve mohou být dány libovolnými příkazy.

Všechny podmínky kromě čistě výrazového if tedy tvoří příkazy.

```
if (c) x;             // syntaktická chyba: chybí else
when (c) x;           // nevýrazový příkaz
if (c) x else y;      // výraz - obě větve jsou dány výrazem
if (c) x else {};     // výrazový příkaz - pravá větev je dána výrazovým příkazem
if (c) {} else x;     // right-consuming příkaz - pravá větev je dáva výrazem
if (c) x = y else y;  // nevýrazový příkaz - jedna z větví je dána nevýrazovým příkazem
```

Při uvedení podmínky jiného typu může dojít k implicitní konverzi, nebo chybě, pokud to není možné.

**Vykonání:**

Je-li podmínka nevyjádřeného příkazového if pravdivá, vykoná se pozitivní větev, jinak negativní.
Je-li podmínka while pravdivá, vykoná se pozitivní větev.

**Vyhodnocení:** Popsáno dále.

## Right-consuming výrazy

Většina operací mají danou **precedenci** a **asociativitu**, které určují implicitní uzávorkování.
Right-consuming operace však mají naprosto jiné syntaktické chování.

Napřed uvedu terminologii:

**Operace** jsou specifické konstrukty tvořící výrazy (např. volání funkcí, použití operátorů, apod.).
Operace mají **levé**, **pravé** a **uzavřené** operandy.
Levé operandy se vyskytují nalevo před ostatními syntaktickými částmi operace - nejsou zleva ničím ohraničeny.
Pravé operandy nejsou zprava ničím ohraničeny.
Uzavřené operandy jsou ohraničeny zleva i zprava.
Např. unární prefixové `+` má jen jeden pravý operand. Binární infixové `+` má levý a pravý operand.
Výrazové if má dva uzavřené operandy (podmínka a pozitivní větev) a jeden pravý operand (negativní větev).

**Right-consuming** operace vážou veškerý zbytek příkazu napravo na sebe
a jsou přitom bez explicitního uzávorkování použitelné v libovolné jiné operaci jako pravý operand.
Celý výraz s right-consuming pravým operandem se také stává right-consuming.

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

> Např. céčkový operátor ?: toto nemusí řešit. Má prostě definovanou nejnižší prioritu, tedy
> `a == b ? x : x + y` se uzávorkuje `(a == b) ? x : (x + y)`.
> ?: tedy není možné použít jako operand jiné operace.
> U výrazového if by to ale bylo zbytečné omezení, protože podmínka je uzavřeným operandem,
> což umožňuje použití ifu jako pravého operandu.

## Typová anotace

**Explicitní typová anotace** se provede napsáním požadovaného typu před výrazový příkaz:

```
{type} {stmt}
```

`{type}` je požadovaný typ
`{stmt}` je výazový příkaz

Explicitně typově anotovaný (right-consuming) výrazový příkaz tvoří (right-consuming) výraz (spolu s uvedeným typem).
Vlastnost right-consuming se při typové anotaci zachovává.
Dochází tak k tzv. **vyjádření příkazu**.

```
{};                        // výrazový příkaz
Int {};                    // výraz
if (c) {} else {};         // výrazový příkaz
if (c) Int {} else Int {}; // výraz (1)
Int if (c) {} else {};     // výraz se stejnou sémantikou jako výraz (1)
Int if (c) {} else x + y;  // right-consuming výraz
```

Výrazové příkazy je možné vyjádřit buď explicitní typovou anotací,
nebo v rámci jiného výrazu či příkazu, který umožňuje implicitní typovou anotaci.
Příkladem takového příkazu je deklarace s přiřazením, kde hodnota může být uvedena
samotným výrazovým příkazem, který se implicitně anotuje typem proměnné.
Takový výrazový příkaz sám o sobě netvoří syntakticky výraz, ale je vyjádřen a dochází k vyhodnocení.
Sémentika implicitní anotace se u jednotlivých příkazů mírně liší.

Vyjádření příkazu vynutí **vyhodnocení hodnoty příkazu**.

```
// vyjádřený block:
Int i = Real {}; // vyhodnocení hodnoty příkazu {} po explicitní anotaci typem Real
Int i = {};      // operace přiřazení vyhodnotí hodnotu příkazu {} po implicitní anotaci typem Int
// nevyjádřený block:
{}; // hodnota blocku zde není vyhodnocena
```

## Vyhodnocení blocků

Syntakticky je možné vyjádřit libovolný block.
Aby bylo vyjádření i sémanticky korektní, musí mít nějakou **hodnotu**.
Hodnotu vyjádřeného blocku určuje příkaz **vrácení**:

```
return {value}
```

`{value}` je hodnota blocku daná ESX.

Nevyjádřené blocky nemají hodnotu.
Příkaz vrácení se tedy vždy vztahuje na nejbližší vyjádřený block, který toto vrácení obsahuje.
Příkaz vrácení se takto tzv. **propaguje** (propagace vrácení) na odpovídající vyjádřený block.
Vrácení bez žádného vnějšího vyjádřeného blocku je chybou.
Vyjádřený block má vždy jednoznačně určený typ. Tento typ se také tzv. **propaguje** (propagace typové anotace)
na hodnotu příkazů vrácení. Dochází tak k implicitní anotaci návratových hodnot daných výrazovým příkazem,
nebo implicitní konverzi hodnot daných výrazem.
Příkaz vrácení ukončuje vyhodnocení odpovídajícího vyjádřeného blocku i vykonání vnořených nevyjádřených blocků.
Další příkazy se tedy ignorují.

```
Int x;
Int { return x };     // hodnota typu Int
Int { { return x } }; // vrácení se propaguje přes jeden vnitřní block na ten vnější vyjádřený
Int {};               // syntakticky ok, ale sémanticky dochází k chybě: vyjádřený block nevrací
Int { T y; return y } // potenciální implicitní konverze z T na Int, pokud to lze, jinak dojde k chybě
```

Podobně se chovají i jiné výrazové příkazy. Může být chybou je vyjádřit i přes to, že to syntakticky lze.
Výrazový příkaz je **vyjádřitelný**, není-li jeho vyjádření sémantickou chybou.

*Terminologie:*

**Propagace vrácení:** Příkaz vrácení se vztahuje k nejbližšímu vyjádřenému příkazu, který toto vrácení obsahuje.
Propaguje se tak do vnějšího příkazu skrze vnořené nevyjádřené příkazy.

**Propagace typové anotace:** Některé příkazy po vyjádření (explicitní či implicitní anotací) použijí daný typ
pro implicitní anotaci vnořených výrazobých příkazů, nebo implicitní konverzi výrazů.
Typ se takto propaguje do hloubky a všechny potřebné výrazové příkazy se anotují a výrazy konvertují.
Dochází tak jen k implicitním konverzím. V případě nemožné implicitní konverze dochází k chybě.

Propagaci typové anotace znázorňují následující příklady:

```
Int { return 1.0 };
// propagace:
Int { return Int 1.0 }; // chyba: implicitní konverze není možná

Int { return { return 1 } };
// propagace:
Int { return Int { return 1 } };     // implicitní anotace blocku
// další propagace:
Int { return Int { return Int 1 } }; // případná implicitní konverze
```

## Vyhodnocení podmínek

Při vyhodnocení výrazového if se jako typ výsledné hodnoty vybere typ jedné z větví,
na který je druhá větev konvertovatelná.
Je-li podmínka pravdivá, vyhodnotí se pozitivní větev, jinak negativní větev.
Hodnota větve (po případné implicitní konverzi) je výslednou hodnotou celého if.

V případě vyjádření výrazového příkazu if se jako výsledný typ použije typ daný typovou anotací
(daný explicitně, nebo implicitně propagací z vnější typové anotace).
Tento typ se použije i pro implicitní anotaci větví daných výrazovým příkazem v rámci propagace typové anotace.
Je-li jedna z větví dána výrazem, dochází k implicitní konverzi, nebo chybě, pokud to není možné.
Je-li podmínka pravdivá, vyhodnotí se pozitivní větev, jinak negativní větev (po implicitní anotaci).

```
Int i = if (c) x else Int { return y }; // if je zde výrazem
Int i = if (c) x else { return y };     // if je zde výazovým příkazem vyjádřeným v rámci přiřazení
Int if (c) x else { return y };         // if je zde výazovým příkazem vyjádřeným v rámci explicitní typové anotace
Int i = if (c) x else return y;         // chyba: příkaz vrácení není výrazovým příkazem, tedy i celý if také
```

Propagaci typové anotace znázorňují následující příklady:

```
Int if (c) x else y;
// propagace:
Int if (c) Int x else Int y; // případná implicitní konverze

Int if (c) { return 1 } else { return { return 2 } };
// propagace:
Int if (c) Int { return 1 } else Int { return { return 2 } };
// další propagace:
Int if (c) Int { return Int 1 } else Int { return Int { return 2 } };
// další propagace:
Int if (c) Int { return Int 1 } else Int { return Int { return Int 2 } };
```

## Vyhodnocení blocků s podmínkami

Použití podmínek v blocích umožňuje vrácení z podmíněné větve.
Aby byl block vyjádřitelný, musí **vracet vždy**.
Block vrací vždy, obsahuje-li alespoň jeden příkaz, který vrací vždy.
Příkaz vrácení vrací vždy.
Příkaz if vrací vždy, vracejí-li vždy obě jeho větve.

Je třeba dávat pozor na následující vlastnosti:
Příkaz when nevrací vždy.
Vrácení se nepropaguje dále z vyjádřeného příkazu.

```
Int {};                                // chyba, nikdy nevrací
Int { if (c) return x else return y }; // ok, vrací vždy
Int { if (c) return x else {} };       // chyba, negativní větev nevrací
Int { when (c) return x };             // chyba, when nevrací vždy
Int i = if (c) { return x } else {};   // chyba, negativní větev nevrací
Int { { return x } };                  // ok, vrácení se propaguje do vnějšího blocku
Int { Int { return x } };              // chyba, vrácení se nepropaguje do vněšího blocku
```

## Scope

Scope je lexikální a vytváří se složeným příkazem či výrazem.
Proměnné jsou definováni v rámci scopu nějakého složeného příkazu,
nebo v rámci globálního scopu, který je dán celým programem.
Celý program tak sémanticky představuje nevyjádřený block (s implicitním `{}`).

Proměnné jsou **viditelné** po deklaraci do konce svého scopu i uvnitř vnořených scopů.
Proměnnou je možné deklarovat znovu se stejným jménem ve vnořeném scopu.
To původní deklaraci tzv. **zastíní**.

```
Int i; // v globálním scopu
{
    Int j;                // ve scopu blocku (v roli příkazu)
    Int i;                // zastíní původní deklaraci
    Int {
        Int k;            // ve scopu blocku (v roli výazu)
    };
    if (c) Int l else {}; // ve scopu podmínky
};
Int j = i;                // použije se i z globálního scopu
```

Deklarace je možná i uvnitř podmínek. Samotná deklarace v některé větvi je však sémanticky nepoužitelná.
Překladač na to upozorní jako i u jiných nepoužitých deklarací.

## Funkce

Funkce je možné deklarovat a definovat příkazem **definice funkce**:

```
{ret} {name} ( ({type} {param} (, {type} {param} (...))) ) {body}
```

`{ret}` je návratový typ funkce [^1]  
`{name}` je název funkce  
`{type}` jsou typy parametrů  
`{param}` jsou názvy parametrů  
`{body}` je tělo funkce dané ESX

Jelikož deklarace funkce je vždy spjatá s definicí, obojí se bude označovat jako **definice funkce**.

Definice funkce tvoří složený nevýrazový příkaz.
Název funkce je sémanticky ekvivalentní proměnné složeného typu funkce.

Složený typ funkce je tvořen následujícím způsobem:

```
{ret} ( ({param} (, {param} (...))) )
```

Tělo funkce (`{body}`) je dáno výazem, nebo výrazovým příkazem.
Tělo se nevyhodnocuje při definici, ale až při zavolání funkce.
Parametry funkce jsou proměnné deklarované ve scopu definice funkce.
Rozsah tohoto scopu je určen tělem funkce. Na scopu funkce není sémanticky nic speciálního.
Proměnné z vnějšího scopu jsou tedy vždy viditelné i v těle funkce.
Parametry mohou tyto proměnné zastínit, stejně tak jako proměnné deklarované v těle funkce.

Funkci lze zavolat výrazem **volání**:

```
{fun} ( ({arg} (, {arg} (...))) )
```

`{fun}` je výraz funkčního typu  
`{arg}` jsou předané argumenty

Při volání se dané argumenty dosadí za parametry funkce a vyhodnotí se tělo funkce, což dá výslednou hodnotu volání.
Volání funkce provádí implicitní anotaci argumentů příslušnými typy.

```
Int y;
Int f (Int x) x + y; // funkce typu Int(Int) využívající proměnnou y z vnějšího scopu
Int i = f(y);        // zavolání funkce a přiřazení výsledné hodnoty
```

Tělo funkce může být dáno výrazovým příkazem.
Příkaz vrácení v tomto případě není nijak speciální sémanticky.
Jde jen o vrácení z výrazového příkazu při vyhodnocení jeho hodnoty,
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
To je konzistentní s příkazem definice funkce, který provádí obojí.

```
Int f (Int x) x;
Int g (Int x) x + x;

Int(Int) h = f; // ok
h = g;          // chyba
f = g;          // chyba

Int(Int) i; // chyba
```

[^1] Návratový typ je prozatím povinný. Funkce bez návratové hodnoty budu řešit v dalších verzích.

[^*] Rekurze prozatím není podporována. Neměl by to být problém, ale není to momentálně nutné.

## Anonymní funkce

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
$ g (Int x) Int (Int y) x + y;
```

## Hodnoty jednoduchých typů

Typ `Integer` bude implementován některým ze znaménkových celočíselných typů v C.
Přetečení nebude kontrolováno.

Typ `Real` bude implementován některým z floating-point typů v C.
Přetečení ani zaokrouhlovací chyby nebudou kontrolovány.

Typ `Boolean` má dvě hodnoty: `true` a `false`.

## Typová konverze

Uvedení typu před výrazem provádí **explicitní typovou konverzi**:

```
{type} {expr}
```

`{type}` je typ  
`{expr}` je výraz

> Typová konverze nemá syntaxi volání funkce záměrně.
> Jde totiž o koncept podobný typové anotaci, co má hlubší význam sémanticky,
> než pouhé volání uživatelské funkce.
> Typová konverze má vysokou prioritu narozdíl třeba od céčkového castu.
> To je kvůli konzistenci s typovou anotací příkazů,
> tedy např. by nedávalo smysl `Int x + y` implicitně uzávorkovat jako `Int (x + y)`,
> zatímco `Int {} + {}` je uzávorkováno jako `(Int {}) + {}`.

Při konverzi z `Integer` na `Real` může dojít k zaokrouhlovací chybě.
Při konverzi z `Real` na `Int` dojde k zaokrouhlení k nule.
Při konverzi na `Boolean` se nulové hodnoty konvertují na `false`, ostatní na `true`
Při konverzi z `Boolean` se `true` konvertuje na 0, `false` na 1.
Konverze mezi funkčním typem a jednoduchým typem není možná.
Konverze na stejný typ (**triviální konverze**) je identita.
(Tedy když se někde požaduje hodnota konvertovatelná na nějaký typ, zahrnuje to i hodnoty tohoto typu.)

Další konverze probíhají v tzv. **krocích**:

* `Boolean -> Integer`
* `Integer -> Boolean`
* `Integer -> Real`
* `Real -> Integer`

**Nezúžující** [^1] konverze probíhají ve směru `Boolean -> Integer -> Real`.
Jsou to tedy konverze které neztrácí žádnou informaci (až na zaokrouhlovací chyby).

Tedy například konverze z `Boolean` do `Real` projde dvěma kroky `Boolean -> Integer` a `Integer -> Real`.

[^1] Převzato z C++ pojmu "narrowing". Nevím ale, jak to vhodně přeložit do češtiny.

## Implicitní konverze

Implicitně mohou proběhnout jen nezúžující konverze.

Implicitní konverze probíhají při:

* přiřazení
* předání argumentu
* vrácení
* dalších vestavěných operacích

Při výběru overloadu vestavěné operace se vybere operace s typy na které je třeba nejmíň kroků implicitní konverze.

## Literály jednoduchých typů

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

## Další vestavěné operace

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

Logické operátory očekávají operandy typu `Boolean`.
Konjunkce i disjunkce jsou "short-circuit", tedy zbytečně nevyhodnocuje operandy, když jejich hodnota o té výsledné nerozhodne.

Celočíselné dělení očekává oba operandy typu `Integer` a výsledkem je `Real`.
Dělení očekává oba operandy typu `Real` a výsledkem je `Real`.

Modulo očekává levý operand typu `Integer` nebo `Real`, pravý typu `Integer` a výsledkem je `Real`.
Nezápornost pravého operandu se nekontroluje a výsledek je nedefinován.

Ostatní aritmetické operátory očekávají oba operandy typu `Integer` nebo `Real` (oba stejné) a výsledkem je stejný typ.

Porovnávací operátory očekávají oba operandy libovolného jednoduchého typu (oba stejné) a výsledkem je `Boolean`. [^1]

Při použití těchto operací na operandech jiných typů dojde k implicitním konverzím, pokud je to možné.

```
1 && 1.0 // 1 -> true, 1.0 -> 1 -> true

3 / 2   // celočíselné dělení
3.0 / 2 // dělení; 2 -> 2.0

3 % 2   // == 1
3.1 % 2 // == 1.1
3 % 2.0 // chyba; implicitní konverze Real -> Integer není možná

1 + 1.0  // 1 -> 1.0
1 + true // true -> 1
```

Všechny infixové operátory mají levou asociativitu.

Precedence operací:

0. reserved (scope resolution)
1. volání funkcí
2. reserved (subscript)
3. typová konverze
4. unární operátory
5. multiplikativní operátory (`*`, `/`, `%`)
6. aditivní operátory (`+`, `-`)
7. porovnávací opertátory v uspořádání (`<`, `<=`, `>`, `>=`)
8. porovnávací operátory v rovnosti (`==`, `!=`)
9. konjunkce (`&&`)
10. disjunkce (`||`)

> Volání funkcí má přednost před subscriptem záměrně.
> Žádné "subscriptovatelné" objekty v jazyce nebudou obsahovat složené typy, tedy ani funkce.
> Je tedy syntakticky nemožné psát `v[i](a)`.
> Je to však možné po explicitním uzávorkování `(v[i])(a)`,
> ale stále to není možné provést sémanticky na žádném vektoru ani bufferu.

> Volání funkcí má také přednost před typovou konverzí.
> `Int f(a)` je tedy implicitně uzávorkováno jako `Int (f(a))`.
> Nepřepodkládám, že budou potřeba nějaké konverze funkčních typů,
> ale i kdyby byly potřeba, tak se prostě zapíší s explicitním uzávorkováním, třeba:
> `(Int(Int) f)(a)`

[^1] Zvažuji také speciální verzi porovnávání floating-point hodnot, které by zanedbávalo rozdíl v rámci nějaké předdefinované přesnosti.

## "Zvláštnosti" jazyka

### Zúžující konverze nejsou implicitní

Např. použití hodnoty typu `Integer` v podmínce není možné bez explicitní konverze.

```
Int i = 0;
if (i) a else b;      // chyba: implicitní konverze Integer -> Boolean není možná
if (i == 0) a else b; // ok
if (Bool i) a else b; // ok
```

### Středníky za definicí funkce

Tělo funkce je dáno výrazem, nebo výrazovým příkazem. Tento příkaz může být i block.
Block však definici funkce nedělá nijak syntakticky speciální a tedy stále musí být od dalšího příkazu oddělena středníkem.

```
Int f (Int x) {
    return x + 1
};
Int i;
```

### Žádný středník uvnitř ifu

Narozdíl od jiných jazyků, kde příkaz je ukončen středníkem a tento středník je vlastně i součástí celého příkazu syntakticky,
se zde příkazy středníky oddělují a nejsou jejich součástí.
Důsledkem je, že příkazový if, který očekává příkaz v obou větvích, neočekává středník za příkazem v pozitivní větvi.

```
if (c)
    return x
else
    return y;
```

Naopak napsání středníku je zde syntaktickou chybou, protože to by ukončilo celý if bez uvedení negativní větve.

### Prázdný příkaz

I prázdný string je příkazem. To umožňuje psát libovolnou posloupnost středníků bež žádného efektu.

```
;;
Int i = 1;;;;;
i = 2;;;
;;;;
```

Umožňuje to tedy i psát poslední středník v blocku:

```
{ return x }

{
    Int i = 1;
    return x;
}
```

Také to umožňuje vynechat větve podmínek.

```
if (c) else return x;
if (c) return x else;
when (c);
```

Neumožňuje to však vynechání těla funkce, jelikož očekává ESX.

```
Int f (Int x); // chyba
```
