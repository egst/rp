V tomto dokumentu popisuji vše, co se týká syntaxe jazyka.
Sémantiku tu rozebírám jen okrajově. Podrobněji se jí věnuje jiný dokument.
Po definici gramatiky následují poznámky a dále komentáře k feedbacku.

```
-- komentář
-- v komentářích se může vyskytovat:
[*i] -- odkaz na poznámku číslo i - odpovídající poznámka je uvedena níže a označná (*i)

<x>                  -- neterminál gramatiky
<x> -> a             -- pravidlo gramatiky
<x> -> a \n -> b     -- zkrácený zápis několika pravidel se stejnou levou stranou
<x> -> a \\ <y> -> b -- když překladač narazí na a, dynamicky přidá pravidlo <y> -> b
<x> ~> a             -- pravidlo odpovídající konstruktu, který zatím nebudu implementovat,
                     -- ale nechám tomu prostor do budoucna a můžu to používat v příkladech, pokud to třeba zkrátí zápis apod.

-- na pravé straně pravidel se mohou vyskytovat:
t                    -- terminál t - vždy oddělen mezerami v pravidle [*0]
<x>                  -- neterminál x
<x.a>                -- neterminál x s kategorií a - kategorie slouží pouze k následnému odkázání v komentářích
<x#a>                -- neterminál x s názvem a - název jednoznačně určuje nějaký jeden neterminal v pravidle a slouží k odkázání na něj
`r`                  -- regulární výraz r vložený mezi neterminály a terminály nebo samostaně
                     -- slouží to jen ke zkrácení zápisu, potom samozřejmě bude potřeba to přepsat jinak
-- nevím, v jakém tvaru bude potřeba gramatiku definovat, zatím to neřeším, píšu to tak, aby to bylo lidsky dobře čitelné
-- některé neterminály mají jen jedno přepisovací pravidlo,
-- slouží jen jako takové "šablony" posloupností terminálů a neterminálů k opakovanému použití v dalších pravidlech
```

## Gramatika

```
<root>  -- celý program
<stmt>  -- příkaz
<cstmt> -- složený příkaz
<sstmt> -- jednoduchý příkaz
<expr>  -- výraz
<cexpr> -- složený výraz - výraz ukončený složeným příkazem
<sexpr> -- jednoduchý výraz
<type>  -- typ
<stype> -- jednoduchy typ
<ctype> -- slozeny typ
<ident> -- identifier

<stmt>  -> <sstmt>
        -> <cstmt>
<expr>  -> <sexpr>
        -> <cexpr>

<ident> -> `[_a-zA-Z][_a-zA-Z0-9]*`
<sstmt> -> <sexpr> -- výraz je i příkazem
<cstmt> -> <cexpr> -- [*4]

<type>  -> <stype>
        -> <ctype>
        ~> *            -- ekvivalent auto v C++ - prozatím to určitě nebudu implementovat, ale budu to používat v příkladech, kde je typ zřejmý
        ~> buffer [ * ] -- dedukce velikosti bufferu
        ~> * [ <expr> ] -- dedukce typu vectoru
        ~> * [ * ]      -- dedukce obojího - všechny tyto syntaktické konstrukty jsou jen k použití v příkladech, implementovat to teď nebudu

<stype> -> boolean
        -> integer
        -> real
        -> complex
<ctype> -> buffer [ <expr.size> ]       -- buffer velikosti size
        -> <stype.type> [ <expr.size> ] -- vector typu type a velikosti size
        -- funkce s parametry typů param a návratovou hodnotou typu return:
        -> `(` <type.return> `)?` ( `(` <type.param> `(` , <type.param> `)* )?` )
        -- funkce bez návratového typu mají typ (params, ...)
        -- funkce bez parametrů mají typ return()
        -- funkce bez návratového typu a bez parametrů má typ () [*1]

<sstmt> ~> type <ident#alias> = <stype> \\ <stype> -> #alias -- alias (ekvivalent typedefu v C) na jednoduchý typ [*2]
        ~> type <ident#alias> = <ctype> \\ <ctype> -> #alias -- alias na složený typ

<root>  -> `(` <cstmt> `|` <sstmt> <sep> `)* (` <stmt> `)?`     -- program je posloupností příkazů
<cstmt> -> { `(` <cstmt> `|` <sstmt> <sep> `)* (` <stmt> `)?` } -- klasický složený příkaz
<sep>   -> ;                                                    -- prozatim jen strednik, konce radku necham jen jako moznost do budoucna
-- příkazy jsou odděleny středníkem, ale je povolen i "trailing" středník
-- terminál } slouží jako implicitní středník před ním i po něm [*4]

<sstmt> -> return <sexpr.value> -- vrácení hodnoty value
<cstmt> -> return <cstmt.value> -- vrácení složeným příkazem [*4]

<asgnh> -> <ident.var> =                    -- hlava přiřazení do proměnné var
<wrth>  -> <sub>       =                    -- hlava zápisu do vectoru [*3]
<sstmt> -> <type> <ident.var>               -- deklarace proměnné var
<sstmt> -> <type> <asgnh> <sexpr.value>     -- deklarace proměnné s přiřazením hodnoty value
<cstmt> -> <type> <asgnh> <cstmt.value>     -- [*4]

<funh>  -> <type.return> <ident.name> ( `(` <type> <ident.param> `(` , <type> <ident.param> `)* )?` ) -- hlava definice funkce
<afunh> -> <type.return> ( `(` <type> <ident.param> `(` , <type> <ident.param> `)* )?` )              -- hlava definice anonymní funkce
<buffh> -> buffer ( `(` <type> <ident.param> `)?` )                                                   -- hlava definice anonymního bufferu
<sstmt> -> <funh> <sexpr>
<cstmt> -> <funh> <cstmt> -- [*4]

<ifh>   -> if ( <expr.condition> ) -- [*5]
<cstmt> -> <ifh> <cstmt>                       -- TODO: This can't be allowed as expression
<cstmt> -> <ifh> <cstmt> else <cstmt>

-- Pro podporu céčkových if..else bez složených příkazů:
<sstmt> ~> <ifh> <sstmt> -- [*6]
<sstmt> ~> <ifh> <sstmt> <sep> else <sstmt>
<sstmt> ~> <ifh> <cstmt> else <sstmt>
<cstmt> ~> <ifh> <sstmt> <sep> else <cstmt>

<sexpr> -> <se1> -- [*7]
<cexpr> -> <ce1>
<expr>  -> <e1>
<e1>    -> <e2>
        -> <se1>
        -> <ce1>
<e2>    -> <e3>
        -> <se2>
        -> <ce2>
<e3>    -> <e4>
        -> <se3>
        -> <ce3>
<e4>    -> <e5>
        -> <se4>
        -> <ce4>
<e5>    -> <e6>
        -> <se5>
        -> <ce5>
<e6>    -> <e7>
        -> <se6>
        -> <ce6>
<e7>    -> <e8>
        -> <se7>
        -> <ce7>
<e8>    -> <e9>
        -> <se8>
        -> <ce8>
<e9>    -> <se9>
        -> <ce9>
<se1>   -> <se2>
<ce1>   -> <ce2>
<se2>   -> <se3>
<ce2>   -> <ce3>
<se3>   -> <se4>
<ce3>   -> <ce4>
<se4>   -> <se5>
<ce4>   -> <ce5>
<se5>   -> <se6>
<ce5>   -> <ce6>
<se6>   -> <se7>
<ce6>   -> <ce7>
<se7>   -> <se8>
<ce7>   -> <ce8>
<se8>   -> <se9>
<ce8>   -> <ce9>
<se9>   -> <sef>
<ce9>   -> <cef>

<op2>   -> ||
<op3>   -> &&
<op4>   -> ==
        -> !=
<op5>   -> <
        -> <=
        -> >
        -> >=
<op6>   -> +
        -> -
<op7>   -> *
        -> /
        -> %
<op8>   -> +
        -> -
        -> !

<se1>   -> <asgnh> <se1>                    -- přiřazení do proměnné
        -> <wrth>  <se1>                    -- zápis do vectoru
        -> <ifh> <e1> else <se1>            -- ternární operátor
        -> <type> <ifh> <e1> else <se1>
        -> <type> <ifh> <cstmt> else <se1>
        -> <funh> <se1>                     -- anonymní funkce
        -> <buffh> <se1>                    -- anonymní buffer
<ce1>   -> <asgnh> <ce1>
        -> <asgnh> <cstmt>
        -> <wtrh>  <ce1>
        -> <wrth>  <cstmt>
        -> <ifh> <e1> else <ce1>
        -> <type> <ifh> <e1> else <cstmt>
        -> <type> <ifh> <cstmt> else <cstmt>
        -> <funh> <ce1>
        -> <funh> <cstmt>
        -> <buffh> <ce1>
        -> <buffh> <cstmt>
<se2>   -> <e3> <op2> <se2>
<ce2>   -> <e3> <op2> <ce2>
<se3>   -> <e4> <op3> <se3>
<ce3>   -> <e4> <op3> <ce3>
<se4>   -> <e5> <op4> <se4>
<ce4>   -> <e5> <op4> <ce4>
<se5>   -> <e6> <op5> <se5>
<se5>   -> <e6> <op5> <ce5>
<se6>   -> <e7> <op6> <se6>
<ce6>   -> <e7> <op6> <ce6>
<se7>   -> <e8> <op7> <se7>
<ce7>   -> <e8> <op7> <ce7>
<se8>   -> <op8> <se8>
<ce8>   -> <op8> <ce8>
<se9>   -> ( <expr> )                               -- závorky
        -> <sub>                                    -- subscript
        -> <e9> ( `(` <expr> `(` , <expr> `)* )?` ) -- volání funkcí
        -> <type> ( <expr> )                        -- explicitní typová konverze
        
<sub>   -> <e9> [ <expr> ]

<sef>   -> <ident>
        -> `([0-9]*[.]?[0-9]+|[0-9]+.)([eE][+-]?[0-9]+)?`  -- číselný literál
        -> true
        -> false
        -> [ `(` <sexpr.elem> `(` , <sexpr.elem> `)* )?` ] -- literál (konstruktor?) vectoru (anonymní vektor?)
<cef>   -> type <cstmt>
```

## Poznámky

### (*0)

Terminály budou keywordy `[a-zA-Z]+`, identifiery `[_a-zA-Z][_a-z_A-Z]*`, číselné literály `([0-9]*[.]?[0-9]+|[0-9]+.)([eE][+-]?[0-9]+)?`,
nebo některé z předem daných posloupností speciálních symbolů (`+`, `==`, `{` apod.).

Mezi keywordy a identifiery musí být samozřejmě mezera.
Po číslech před keywordy a identifiery by mezera být teoreticky nemusela,
jelikož nemám v číselných literálech znaky určující typ, jako v C (1f apod.).
Radši s ní ale prozatím budu počítat.
Před čísly po identifierech mezera být musí, protože povoluji čísla v identifierech.
Před čísly po keywordech musí být také, protože by nebylo jasné, jestli jde o keyword a číslo, nebo identifier.
Kolem posloupností speciálních symbolů prozatím počítám, že mezery být nemusí.
Budu ale muset ověřit, zda nevzniknou nějaké nejednoznačnosti.

To bude ale práce lexeru. Parser pak bude pracovat podle pravidel gramatiky s tokeny, takže mezery v gramatice neřeším.

### (*1)

Pokud to bude problém, zavedu speciálná typ void.
Není to jednoduchý ani složený typ. Nelze ho použít prakticky nikde až na definici typu funkce:
<ctype> -> `(` <type> `|` void `)?` ( `(` <type> `(` , <type> `)*` `|` void `)` )
Potom typ funkce bez parametrů je return(void), bez návratové hodnoty void(params, ...) a bez obojího void(void).
Při pokusu o přiřazení z výsledku funkce vracející void by doělo k chybě při překladu.

### (*2)

Nevím, jestli to zbytečně nezkomplikuje implementaci, ale příjde mi to jako dost užitečný nástroj.
Hlavě kvůli často používaným typům funkcí reprezentujících signály, jako real(integer), real(real) apod.

V příkladech budu počítat s následujícími aliasy:

```
type int     = integer;
type intsig  = real(int);
type realsig = real(real);
type nullsig = real();
```

### (*3)

Přiřazení z jednoduchého typu přiřadí kopii, ale ze složeného typu referenci.
Reference na jednoduché typy neexistují, takže přiřazení jednoduchého typu do výrazu nemá smysl.
Přiřazení do složeného typu vždy vyžaduje na levé straně proměnnou, která potom odkazuje na danou hodnotu.
Proto na levé straně stačí identifier, ne výraz.

V případě zápisu do vektoru, jde o nastavení hodnot uvnitř vectoru, referenci na který může vracet i nějaká funkce.
Proto je třeba povolit libovolný výraz na levé straně před subscriptem.

Podrobněji k referencím se vyjadřuju v druhém dokumentu.

### (*4)

Složené příkazy mohou být i výrazy. Buď se musí explicitně uvést typ:

```
int { int j = 1; return j + 1 }
```

Nebo musí být součástí nějakého syntaktického konstruktu, kde je typ triviálně odvoditelný:

```
int i = { int j = 1; return j + 1 }
int foo (int x) { int j = 1; return x + j }
int foo (int x) { return { int j = 1; return x + j } }
```

Terminál } tady slouží jako implicitní středník po posledním příkazu ve složeném příkazu, i jako středník po něm.
Oba středníky je ale možné uvést explicitně, aby uživatel nemusel řešit, kam dávat středníky musí a kam nesmí.

Proto se rozlišují jednoduché a složené příkazy `<cstmt>` a `<sstmt>` i výrazy `<cexpr>` a `<sexpr>`
`<cexpr>` a `<cstmt>` jsou vždy ukončeny terminálem `}`, tedy nepotřebují explicitní středník.

Podrobněji příkaz return popisuji v druhém dokumentu.

### (*5)

Konstrukt if..else lze použít jako výraz:

```
int i = if (true) 1 else 2;
```

Při použití složeného příkazu za if nebo else je třeba uvést typ:

```
1 + int if (true) { int j = 1; return j + 1 } else 3;
```

V kontextu s triviálně odvoditelným typem to není potřeba:

```
int i = if (true) { int j = 1; return j + 1 } else 3;
int foo (bool c) if (c) { int j = 1; return j + 1 } else 3;
```

Platí stejná pravidla pro vynechávání středníku za }:

```
int if (true) 1 else { int j = 1; return j + 1 }
```

Zároveň lze if..else i samotné if použít jako příkaz, pokud se neuvede typ:

```
int foo (bool c)
    if (c) {
        return 1;
    } else {
        return 2;
    }

int foo (bool c) {
    if (c) {
        return 1;
    }
    return 2;
}
```

### (*6)

Pro použití if, či `if..else` jako příkazu, je potřeba používat složených výrazů v těle.
To mi příjde víc konzistentní s celou logikou složených výrazů, jak byla doposud nadefinována.
To samé totiž platí i u funkcí. Nelze například nadefinovat funkci takto:

```
int foo (int x) return x + 1;
```

Aby toto fungovalo u `if` a `if..else`, bylo by potřeba dodefinovat několik pravidel navíc.
Tato pravidla uvádím v gramatice se symbolem `~>`. Podobná pravidla by se dala nadefinovat i pro funkce,
aby bylo možné psát například:

```
inf foo (int[*] v) v[0] = 1;
```

### (*7)

Pro všechny základní operace platí typické vlastnosti, jako u céčka:

```
[precedence asociativita: operace s _ označujícím operandy]
1 NA:    _[_] _(_) <type>(_)
2 right: +_ -_ !_
3 left:  _*_ _/_ _%_
4 left:  _+_ _-_
5 left:  _<_ _<=_ _>_ _>=_
6 left:  _==_ _!=_
7 left:  _&&_
8 left:  _||_
9 right: _=_ if(_)_else_ <type>(_)_ buffer(_)_
```

`<type>(_)_` a `buffer(_)_` jsou definice anonymní funkce a anonymního bufferu.
`if(_)_else_` nahrazuje céčkový operátor `_?_:_`.

Nejslaběji váže přiřazení/zápis, podmínka a definice anonymní funkce a bufferu.
Přiřazení očekává na levé straně jen identifier. Na pravé straně může být libovolný výraz.

```
int i;
int j;

i = j = 1;     // ok
i + 1 = 1;     // syntax error
i = j + 1 = 2; // syntax error
```

Zápis do vectoru očekává na levé straně `<sub>`, tedy `<e9> [ <expr> ]`.

```
int[*] v = [1, 2, 3];
int[*] foo () [1, 2, 3];
int[*] bar (int[*] v) v;

v[0]         = 10; // ok
foo()[0]     = 10; // ok, i kdyz zbytečné sémanticky
bar(v)[0]    = 10; // ok, má to efekt na vectoru v
[1, 2, 3][0] = 10; // ok, zase prakticky bez efektu sémanticky

* foo () [1, 2, 3][0] = 10; // * foo () ([1, 2, 3][0])
(* foo () [1, 2, 3])[0] = 10;
```

Zápis do anonymního (ve významu jako rvalue v C++, ale žádný takový pojem tu formálně nezavádím) vectoru
by neměl žádný význam, kdyby návratovou hodnotou zápisu byl zapsaný prvek.
Kdyby se vracel celý vector, pak by to mělo využití v modifikaci anonymních vektorů bez přiřazení do proměnné.

```
int[3] v = [1, 2, 3][0] = 10;
```

Pokud se rozhodnu pro vracení zapsané hodnoty, bude platit `v == 10`.
Pokud pro vracení celého vectoru, bude platit `v == [10, 2, 3]`.

Ternární podmínka `if..else` umožňuje vnořené podmínky.
Pro použití `if..elde` v levém operandu jiných operací, je ho potřeba uvést do `()`.

```
int i = 1;
int j = (if (i == 1) 1 else if (i == 2) 2 else 3) + 1;
```

Stejně se chová anonymní funkce a buffer:

```
int () { int j = 1; return j + 1 }() // int () ({ ... }()), což je syntax error

int i = (int () { int j = 1; return j + 1 })() + 1;
```

Místo evaluace anonymních funkcí ihned po definici se dá použít složený příkaz:

```
int i = int { int j = 1; return j + 1 } + 1;
```

V operacích je možné použít složený příkaz v roli výrazu po explicitním uvedení typu:

```
int { return 1 } + real { return 1.5 }
```

Možnost odvození tohoto typu nechám jako možnost do budoucna, zatím to řešit určitě nebudu.

Dost mi zkomplikovalo gramatiku to, že `}` představuje implicitní středník.
Libovolný výraz končící terminálem `}` se tak stává `<cexpr>`, tedy v kontextu příkazu je to `<cstmt>`.
Po `<cstmt>` není potřeba středník.

```
int i = 1 + { return 5 } // není potřeba středník, končí to symbolem }
int j = 1 + 5;           // je potřeba středník
int k = 1 + 5            // není potřeba středník, dokud je to poslední příkaz v programu nebo složeném příkazu
```

Kdyby to bylo zbytečně složité na implementaci, tak se na to vykašlu a budu středník požadovat vždy.
Hlavním důvodem, proč jsem chtěl toto povolit je to, že po definici funkce by bylo třeba psát středník také,
což by bylo zvláštní z pohledu jiných jazyků s céčkovou syntaxí:

```
int foo () {
    return 1;
};            // tady by byl nutný středník
```

Můžu to také nechat jako výjimku jen pro případ definice funkce, ale to je zase takové nekonzistentní.

## Komentáře k feedbacku

> Ja myslim ze ano, pokud bude mozne jeho velikost zadat vyrazem (ktery treba
> vola nejake uzivatelske funcke), ktery se nakonec vyhodnoti na konstantu
> (behem prekladu).

Velikost vectoru se uvádí přímo v typu `vector[n]`. Nepromyslel jsem zatím,
jak předně se bude řešit to, aby bylo možné to vyhodnotit na konstantu,
a zda to bude možné vždy. K tomu se dostanu později.

> Klidne, zapomenuty strednik na konci radku je otravny. Na druhou stranu by
> mely jit napsat dlouhe vyrazy i jinak nez na jeden radek, takze "\\\n" by
> melo rusit funkci stredniku.
>
> Mozna bych se na to v prvni verzi i vykaslal a strednik vyzadoval vzdy,
> budes to mit jednodussi (taky to zjednodusuje pripad kdy bys chtel tyto
> programy generovat nejakym programem).

Vykašlu se zatím na to. Dosavadní definice gramatiky umožňuje vynechávat středníky kolem `}`.

> > Prazdny string je take prikaz. Syntaxe prikazu je tedy podobna JavaScriptu.
>
> OK, ale je treba to promyslet kvuli jednoznacnosti gramatiky.

To jsem taky zrušil. S dosavadní definicí gramatiky by to možná fungovalo, ale nebudu to zatím řešit.
Kdyby bylo potřeba někde "nic nedělat", tak se dá použít prázdný složený příkaz `{}`.
Navíc kód jako nař. `int foo ()`; by byl matoucí pro céčkaře a vypadal by jako deklarace funkce bez definice.

> OK, tak tedy if z Algolu misto ?:

Nevím, jak funguje `if` v Algolu, ale `if`y a celkově složené výrazy jsem musel promyslet trochu víc do hloubky, aby to takhle dávalo smysl.
S aktuální definicí si myslím, že to docela dává smysl. Umí to i čistě céčkovou syntaxi
(v některých případech ale nevím, jestli se sem ta céčková syntaxe hodí - jako např. jeden příkaz za `if`em bez uvození do složeného výrazu)
a umožňuje to používat složené příkazy (klasický `{}` i konstrukty jako `if..else`) jako výrazy a řeší to odvozování typů v triviálních případech
a naopak nutí uvést typ v netriviálních případech.

> A tohle jaky neterminalni symbol gramatiky?  <*:expression> ?  Muzu tedy
> napsat a=b=c jako v Ccku a vyhodnoti se to jako a=(b=c)?

Předtím jsem nechtěl dělat z přiřazení výraz, ale asi není důvod, proč to neumožnit.
Na pravé straně přiřazení je teď výraz `<expr>`, na levé identifier `<ident>`
a samotné přiřazení je výraz, tedy je možné psát a = b = c s pravou asociativitou.

> A jak resis to ze prazdny string je prikaz je prikaz je tez vyraz --- to bys
> pak mohl napsat if a=10 a nebude jasne jestli tim myslis

Prázdný výraz jsem zahodil a v `if`u jsou závorky povinné. Je to pak myslím konzistentnější s definicí funkce, která má podobnou syntaxi.

> A jak ji pak pojmenujes? To chces psat
>    foo = (a,b,c) -> body

To jsem chtěl původně, ale neumožňuje to nijak hezky stručně napsat typy aniž by se musely aspoň někde odvozovat.
Teď se funkce deklarují normálně céčkově s možností vynechání názvu k definici anonymní funkce.
Názvy těch "normálních" funkcí pak nefungujou jako klasické proměnné, nelze do nich přiřadit jinou funkci.
Ty anonymní by bylo možné přiřadit do proměnné, předat jiné funkci apod. s tím, že by pak implementačně šlo o pointer na funkci.
Nebo alternativně by se nějak omezilo přiřazování anonymních funkcí do proměnných.
To ještě promyslím, abych neimplementoval zbytečnosti navíc.

> ? Taky mi neni jasne jak se specifikuji typy parametru.

Předtím jsem chtěl typy odvozovat při překladu, ale možnost uvést typy explicitně byla také, jenom to popisuju dál v tom dokumentu.
Na to odvozování typů jsem se vykašlal až na opravdu triviální případy, kde není důvod uživatele nutit třeba psát ten samý typ víckrát.

> Neni to trochu moc syntaktickeho cukru? Ber to tak ze kazada takova vyjimka
> te bude stat spoustu casu pri implementaci a testovani. Casu ktery bys mohl
> vyuzit pro praci na skutecnych problemech.

U té syntaxe se šipkama se to možná hodilo, ale s "céčkovější" syntaxí to rozhodně není potřeba. Závorky budou explicitní a žádné zkratky.

> Tady se bojim ze ti to hrozne zkomplikuje parser. Taky by bylo dobry aby
> gramatika byla nejak prirozene jednoznacna a nemusel jsi pouzivat semantiku
> na disambiguaci gramatiky jak se to deje v prirozenych jazycich (pokud uz,
> tak radsi jen omezene a v pripadech kdy to opravdu usetri spoustu psani --
> ma to i dalsi nevyhody krome pracnosti -- mnohem hur se pak generuji chybove
> hlasky, protoze spatne muze byt velmi mnoho veci).

Tu jednoznačnost jsem zatím neřešil nijak rigorózně, ale takhle na první pohled to vypadá, že v té aktuální definici gramatiky
by neměly být nejednoznačnosti. Trochu se obávám problému u typových signatur funkcí
(`(int)` odpovídá céčkovému `void(int)` a `()` céčkovému `void(void)`, abych nezaváděl zvlášť typ `void`)
ale kdyby to byl problém, tak mám promyšlenou alternativu se zavedením speciálního typu `void`.
Haskellovské volání funkcí jsem zahodil a implicitní volání nulárních funkcí jen uvedením názvu taky.

> Co je to funkce ktera ocekava konstantu? To chces rozlisovat funkce ktere
> lze aplikovat na promennou nebo vysledek vyrazu od funkci, ktere se apliguji
> na konstantu?

Nedefinoval jsem, co myslím konstantou. Myslel jsem tím to, čemu teď říkám jednoduchý typ (je to popsáno někde v definici gramatiky),
tedy `boolean`, `integer`, `real` a `complex`. Chtěl jsem se trochu inspirovat Rkem, ve kterém je možné některé funkce aplikovat jak na skaláry, tak na vektory,
bez použití nástrojů jako `map` apod. Ale Rko je celé postavěné vektorově, tam i skaláry jsou jednoprvkové vektory.
V tom mém kontextu to moc smysl nedává. Takže zatím nic takového řešit nebudu a potom, jak říkáš, tak třeba přidám nějaké funkce jako `map`, `reduce`,
nebo nějaké operátory na skládání funkcí apod.

> Neni to trochu zbytecne definovat specialni syntax na skladani funkci s
> jednim argumentem, kdyz pri normalnim pouziti muzu napsat proste g(f(x)) a
> pokud bych to chtel predat jako funkci nekam tak napsat neco jako
> λx.g(f(x))?

Zrovna u toho sčítání, násobení signálů apod. si myslím, že by mohly být nějaké nástroje na zkrácení zápisu.
Psát to lambda funkcí samozřejmě jde, ale vzhledem k tomu povinnému uvádění typů je to trochu krkolomné.

```
real(real) (real x) g(g(x))
```

Ale OK, zatím to neřeším. Jsou to už opravdu jen detaily navíc, které určitě nemusí být nějak v základu vestavěné do jazyka.
Takže kdyby to náhodou bylo potřeba, vymyslím to potom.

> OK, mozna by se hodilo mit jeste komplexni cisla (i pro buff a list), ale
> spis je to zase overkill, takze v prvni verzi se na to vykasli.

Přidal jsem typ `complex` do gramatiky, ale nic s ním zatím neřeším. Bude potřeba vymyslet literály, operace a samotnou implementaci.
Každopádně s tím počítám do budoucna.

> OK, mozna by se hodilo mit jeste komplexni cisla (i pro buff a list), ale
> spis je to zase overkill, takze v prvni verzi se na to vykasli.

S tím určitě počítám. Ve specifikaci jazyka ani nebudu uvádět konkrétní typy pro implementaci, nechám to otevřené nějakému experimentování a tweakování.
Možná by se hodilo i umožnit uživateli si vybrat, jestli chce float, nebo double?

> Tohle je mozna az overkill. Draty bych delal jen jednoho typu. Zjednodusi to
> implementaci.

OK. `buffer`y nechám se spojitými vzorky. U `vector`ů umožním použití libovolného jednoduchého typu (`boolean`, `integer`, `real` a `complex`).
Zatím ale počítám s tím, že v nějaké první verzi bych se mohl obejít i bez `vector`ů.

> Uvaz jestli to neni overkill. Tyhle dopocitavane typy maji vyznam, pokud mas
> typove polymorfni jazyk, nebo kdyz ten jazyk podporuje sablony. Tady ale
> budou typy vetsinou fixni a pak je lepsi, kdyz aspon u vystupu funkce
> uzivatele nutis uzivatele typ psat, protoze to umoznuje kompilatoru poznat
> chybu.

Odvozování typu jsem nechal jenom pro opravdu triviální případy, kdy je ten typ uživatelem už uveden,
ale ne nutně u daného výrazu. Je to vestavěné do gramatiky.

> Pro usporu tve prace a zlepseni ucici krivky uzivatelu bych taky
> doporucil vybrat si jen jednu syntax jak se te bude zapisovat.

To, že mi tam vzniklo víc syntaxí pro tu samou věc bylo spíš výsledkem toho, jak jsem to postavil.
Nebylo to tak, že bych chtěl explicitně do jazyka zabudovat víc alternativních syntaxí.
S aktuální definicí to určení typů už probíhá jen jedním způsobem.

```
int foo (int x) { ... }
```

V případě anonymní funkce by se musel typ uvést víckrát v případě přiřazení do proměnné.

```
int(int) foo = int (int x) { ... }
```

Ale nevím, jestli vůbec bude potřeba přiřazovat funkce do proměnných. Mělo by úplně stačit předávání funkcí parametry.

> Uvedomujes si, ze tim ze v tomto pripade kompilatoru prozradis typ funkce
> sinc az v prikazu return, hodne komplikujes overovani semanticke spravnosti
> programu? Bude o dost pracnejsi napsat kompilator.

Tím jsem nechtěl říct, že by bylo možné rozhodnout o výsledném typu za běhu podle nějakého ifu třeba.
Původní představa byla taková, že složený příkaz by vracel hodnoty jen jednoho typu.
Ten typ by musel odvodit překladač. Kdyby se vracely různé typy, které by se nedaly implicitně konvertovat,
došlo by k chybě při překladu.
Teď ale požaduju explicitní uvedení typu složeného příkazu (nebo někde v celém příkzu, kterého je součástí).
Pokus o vrácení hodnoty jiného typu (nebo spíš typu, který by nebylo možné implicitně konvertovat na ten správný)
by vedl k chybě při překladu.

> Ok. Uvaz ovsem jak by to vypadalo kdybys pro pole udelal comprehense

To je dobrý nápad, nenapadlo mě to. Zatím jsem to nezvažoval.
Funkci subscriptu zatím vubec nedefinuji, takže věci typu f[0:size] si ještě nechám projít hlavou
a zkusím promyslet tu komprehenzi, jestli nebude moc komplikovaná na implementaci.
Rozhodně by ale byla dost užitečná.

> Ted koukam ze mozna jeste uplne nechapu jak chces presne delat ty funkce s
> buffery. Vypada to ze povolujes jen funkce typu (buffer,buffer,other)->cont
> a vzdy vyzadujes explicitni prirazeni takove funkce k bufferove promenne.
>
> Asi to nesnizuje expresivitu jazyka, ale myslim ze to je trochu
> restriktivni, protoze pak vlastne nemuzu napsat neco jako
>
> buff1 << generator1
> buff2 << generator2
> return moduluj(buff1, zpracuj(buff1, buff2))

(Pozn.: U `buffer`ů jsem nahradil operátor `<<` klasickým přiřazením `=`, protože přiřazení do `buffer`u v klasickém významu stejně nemá smysl.)

Nevím, jestli jsem tě pochopil správně, ale beru to tak, že ti jde o vracení `buffer`ů z funkcí a lokální deklaraci `buffer`ů.
To aktuální gramatika dovoluje a přidal jsem navíc možnost definovat anonymní `buffer` rovnou s definicí generátoru:

```
buffer (real t) a[-1] * b[-2]
```

Nebo "jakoby" typovou konverzí:

```
buffer(generator)
```

To popíšu podrobněji v druhém dokumentu.

> terminologicka otazka? Jak se lisi kompilace od transpilace (krome toho ze
> transpile neni anglicke slove kdezto compile je).

Asi je to jedno, ale jsem zvylký na to, že překladu z high level jazyka do jiného high level jazyka se říká transpilace.
Třeba z TypeScriptu (extenze JavaScriptu) se transpiluje do JavaScriptu, ne do nějakého mezikódu ani do procesorových instrukcí.
Překladu do procesorových instrukcí (nebo maximálně nějakého assembly) se pak říká kompilace.
Ale je to úplně jedno a i kdyby se to rozlišovalo, tak ta hranice je docela vágní.
Chtěl jsem ale zdůraznit, že celý překlad toho mého jazyka projde dvěma fázemi: nejdřív transpilace (do céčka), pak kompilace (do strojového kódu).

> Proc globalne?

Ve většině případů by se to dalo rovnou inlinovat. Ale kdyby bylo potřeba něco definovat jako funkci zvlášť,
tak jinak než globálně by se to dalo nadeklarovat jedine s GCC rozšířením, co povoluje ty lokální funkce,
ale to je myslím docela omezené co se týče vracení z těch funkcí a tak. Nevím, musel bych se na to kouknout, nikdy jsem to nepoužíval.

Nemyslel jsem to ale globálně jako z pohledu uživatele jazyka. Ten by to normálně viděl lokálně.
V tom céčkovém kódu by se to pojmenovalo nějakým mangled názvem, aby nedocházelo ke kolizím.
