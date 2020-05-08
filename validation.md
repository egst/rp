# Validace složených příkazů v roli výrazů

V tomto dokumentu rozebírám sémantiku složených příkazů a příkazu return.
Následně popisuji algoritmy k validaci příkazů i výrazů, které ověří typy,
a zda je příkaz použitelný jako výraz, což zároveň řeší ověření, zda funkce vždy vrací.
Defunuji zde strukturu některých prvků AST konfiguračního jazyka.
Potom v dalších dokumentech budu popisovat překlad z AST konfiguračního jazyka na AST jazyka C.
Aby bylo jasné o kterém jazyce mluvím, budu prozatím konfiguračnímu jazyku říkat jazyk X.
Omezuji se zde na nějakou základní podmnožinu syntaktických konstruktů uvedených v dokumentu `grammar.md`.
Aktuální plán je vyřešit kompletní proces překladu pro nějakou rozumnou podmnožinu
a až po tom přidávat další potřebné konstrukty do jazyka X.

Složený příkaz <cstmt> je tvořen:
* `{ ... }`                 blockem
* `if ( ... ) ...`          podmínkou `if`
* `if ( ... ) ... else ...` podmínkou `if..else`

Složené příkazy je možné použít v roli výrazu:

```
<type> <cstmt>
<type> <ident> = <cstmt>
<funh> <cstmt>
<afunh> <cstmt>
return <cstmt>
<type> if ( ... ) <cstmt>
<type> if ( ... ) <cstmt> else ...
<type> if ( ... ) ... else <cstmt>
<type> if ( ... ) <cstmt> else <cstmt>
```

Použití `if` bez `else` v roli výrazu je povoleno z pohledu gramtiky.

int i = if (c) 1;

Takový kód se zparsuje a výsledné AST bude vypadat nějak takto:

```
decl {
    type: int,
    name: "i",
    value: if {
        cond: i"c",
        then: l"1",
        else: null
    }
}
```

`decl`, `if`, `i` a `l` jsou prvky (vrcholy stromu) AST s nějakou předem danou strukturou:

```
decl { type: <type>, name: "", value: (Expr | EStmt)? } : Stmt       -- deklarace s možným přiřazením
if { cond: Expr, then: Expr | EStmt, else: (Expr | EStmt)? } : EStmt -- if s možným else
i { value: "" } : Expr                                               -- identifier
l { value: "" } : Expr                                               -- literál

Stmt         -- Příkaz
Expr  : Stmt -- Výraz
EStmt : Stmt -- Příkaz použitelný v roli výrazu ("expression statement")
```

Každý prvek AST má pojmenované a otypované položky uvedené v `{}`.
Za `{} :` je uvedena kategorie prvku.
`C : D` určuje hierarchii kategorií `C` a `D` - `C` je podkategorie `D`.
Typ položky může být:
* `<type>` typ definovaný v jazyce X
* `""`     string
* `[t]`    seznam typu t
* `t | u`  jeden z typů t nebo u
* `t?`     nepovinná položka typu t - neuvedená hodnota se explicitně zapíše jako null
* `Cat`    podpoložka kategorie Cat
* `node`   podpoložka node

V zápisu AST definuju prvek s položkami:
* `node { elem: value, ... }` plným zápisem
* `node ( value, value )`     pořadovým zápisem bez uvedení názvů položek
* `node value`                zkráceným zápisem v případě jediné položky

Např.:
```
l"i"                  -- l { val: "i" }
l("i")                -- to samé
decl (int, "i", l"1") -- decl { type: int, name: "i", value: l { value: "1" } }
decl (int, "i")       -- decl { ..., value: null }
```

Zpět k příkladu:

```
decl {
    type: int,
    name: "i",
    value: if {
        cond: i"c",
        then: l"1",
        else: null
    }
}
```

`decl` je jeden z prvků, které umožňují použití podpoložky v kategorii jak `Expr` tak `EStmt`.
`EStmt` (nebo ES, "expression statement") jsou příkazy použitelné v roli výrazu.
V opačném případě jde o PS ("pure statement"), což jsou příkazy, které nikdy nelze použít jako výraz.
Syntakticky je možné použít libovolné ES, tedy:
* `if`
* `if..else`
* `{}`
Sémanticky ale například `if` bez `else` nemá žádný význam jako výraz. Stejně tak `{}`, který nikdy nevrací.
Zavádním proto pojem "validní ES", tedy ES, které je opravdu použitelné jako výraz.

```
int i = return 1;         // syntax error, použití PS v roli výrazu
int j = {                 // ok, validní ES
    return { int k = 5; } // error, použití nevalidního ES v roli výrazu (ale syntakticky ok)
}
```

AST uvedeného příkladu bude obsahovat další typy prvků:

```
block { stmts: [Stmt] } : Cstmt    -- block příkazů v {}
ret { value: Expr | EStmt } : Stmt -- návratová hodnota
```

Příkaz return se vztahuje na nejbližší ES, které ho obsahuje.
Např.:

```
int i = {         // ES
    if (c) {      // PS, oba returny se vztahují na ES
        return 1;
    } else {
        return 2;
    }
}

int i = {         // vnější ES
    return        // vztahuje se na vnější ES
    {             // vnitřní ES
        return 5; // vztahuje se na vnitřní ES
    }
}
```

Zpět k příkladu:

```
decl {
    type: int,
    name: "i",
    value: cstmt [
        ret (block [
            decl (int, "j", l"5")
        ])
    ]
}
```

Prvek `ret` obsahuje ES. Je tedy potřeba ověřit, zda je daný block opravdu validním ES.

Popíšu hodně abstraktně (místy spíše deklarativně než imperativním algoritmem) nástin algoritmů k validaci ES.

Zavedu pojem "příkaz (vždy) vrací".
Příkaz return vždy vrací.
Zda příkaz vrací neovlivňují vnořené ES, ale jen PS.

```
int i = { return 5; }   // Celý příkaz přiřazení nevrací.
if (cond) { return 5; } // Celá podmínka vrací.

int j = {                           // Vnější block
    int k = { return 1; }           // Celý příkaz přiřazení nevrací, tedy evaluace vnějšího bloku zde nekončí.
    int l = if (cond) { return 1; } // Příkaz if vrací, ale v rámci celého příkazu přiřazení jde o ES,
            else      { return 2; } // tedy příkaz přiřazení nevrací a vnejší block zde taky nekončí.
    if (cond) { return 1; }         // Celý příkaz if vrací, tedy evaluace vnějšího bloku zde končí a celý block vrací 1.
}
```

## Algoritmus `check_return` - ověření návratové hodnoty složeného příkazu:

Vstup:
* AST podstrom příkazu `s`.

Pro příkaz return `s ~ ret x`:
`s` vrací vždy. Jeho návratová hodnota je `x`.

Pro block `s ~ block [s_1, ...]`:
Algoritmus se pustí rekurzivně na každý `s_*`.
Pokud existuje `s_i`, který vždy vrací, `s_j` pro `j > i` jsou zahozeny v AST, a celý block vždy vrací.
Návratové hodnoty všech příkazů `s_*` jsou návratovými hodnotami celého blocku.

Pro podmínku if `s ~ if (cond, [s_1, ...])`:
Platí to samé, co pro block, ale celá podmínka může vracet jen podmíněně, tedy nevrací vždy.
(Kdybych kontroloval, zda se dá podmínka `cond` evaluovat při překladu, mohl bych i určit, zda if vrací vždy, ale to zatím neřeším.)

Pro podmínku if..else `s ~ if (cond, [s_1, ...], [z_1, ...])`:
Algoritmus se pustí rekurzivně na všecha `s_*` i `z_*`.
Pokud existuje `s_i`, který vrací vždy, další příkazy v této větvi jsou zahozeny. Stejné platí pro `z_i`.
Existují-li nějaké `s_i` i `z_j`, co vrací vždy, celá podmínka vrací vždy.
Návratové hodnoty všech `s_*` i `z_*` jsou návratovými hodnotami celé podmínky.

Jinak příkaz `s` nevrací.

Výstup:
* modifikované AST příkazu `s` se zahozenými příkazy
* vrací-li příkaz `s` vždy
* návratové hodnoty příkazu `s` dané jejich AST podstromem

Příklad:
```
int foo ()
{ // pustím algoritmus manuálně
    int i = // nevrací
        // ale můžu pustit zase manuálně k ověření přiřazené hodnoty:
        if (cond) {         // vždy vrací něco z [1, 2]
            return 1;       // vždy vrací [1]
        } else {
            return 2;       // vždy vrací [2]
        }
    if (cond1) {            // nemusí vrátit, ale když vrátí, tak něco z [1, 2.5]
        if (cond2) {        // vždy vrací [1, 2.5]
            return 1;       // vždy vrací [1]
        } else {
            return 2.5;     // vždy vrací [2.5]
            int j = 1;      // zahodím
        }
    }
}                           // => celý block nemusí vrátit [1, 2, 1, 2.5]
```

Z takového výstupu vidím, že 1) block nemusí vrátit, tedy nejde o validní ES
a 2) i kdyby vrátil, tak vrací hodnoty jak typu integer, tak typu real,
takže by muselo dojít k nějaké konverzi, aby bylo možné výsledek vyhodnotit jako požadovaný typ int.

Další příkad:
```
int {
    return { return 1 } // vrací vždy [{ return 1 }]
}
```

V tomto případě mi algoritmus vrátí komplikovanější podstrom AST - nějaký block.
Tento block bych musel opět vyhodnotit stejným algoritmem, abych zjistil, zda je to validní ES.

### Algoritmus `validate_es` - validace ES

Vstup:
* požadovaný typ `t`
* AST podtrom složeného příkazu `s`

Algoritmem `check_return` se zjistí:
* zda příkaz `s` vrací vždy
* seznam možných návratových hodnot `[r_1, ...]`

Nevrací-li `s` vždy, došlo někde k chybě a příkaz `s` není validním ES.

Jinak, pro každou hodnotu `r_i`:
* Je-li to ES, pustím na něj rekurzivně tento algoritmus se stejným typem `t`. Pokud nevrátí chybu, `r_i` je ok.
* Jinak na něj pustím algoritmus `validate_expr` (definovaný dále) s typem `t`. Pokud nevrátní chybu, `r_i` je ok.

Pokud nějaká hodnota `r_i` neprošla validací, došlo k typové chybě.

Výstup:
* potvrzení validity, nebo chyba

### Algoritmus `validate_expr` - validace výrazu

Vstup:
* Požadovaný typ `t`
* AST podstrom výrazu `e`

Výraz `e` s nějakými operandy bude mít přesně dány typy operandů a výsledku, nebo seznam více možností
v případě overloadovaných built-in operací (int + int, float + float apod).

Pro každou možnost typů operandů `t1, ...` se provede:

Pro každý operand `o_i` výrazu `e`:
* Jde-li o ES, pustím na něj algoritmus `validate_es` s typem `t_i`. Pokud nevrací chybu, `o_i` je ok.
* Je-li typ operandu znám (jde třeba o literál, nebo jiný atomický výraz),
  porovná se s `t_i` (případně s nějakými implicitními konverzemi) a pokud jsou stejné (nebo nějak kompatibilní), `o_i` je ok.
* Jinak na něj pustím rekurzivně algoritmus `validate_expr` s typem `t_i`. Pokud nevrací chybu, `o_i` je ok.

Pokud všechny `o_i` byly vpořádku validovány, tato konfigurace typů `t1, ...` je vhodným overloadem.

Nakonec se vybere nějaký nejvhodnější overload a pro něj správný výstupní typ.
Pokud se tento typ neshoduje (případně po implicitních konverzích) s požadnovaným typem `t`, došlo k typové chybě.
Pokud neexistuje žádný vhodný overload, nastala typová chyba.

(S overloady se bude pracovat jen u vestavěných operací, u kterých bude jasně definováno nějaké pořadí preference těch overloadů.
Ještě se kouknu pro inspiraci, jak to dělají jiné jazyky. Asi se to pokusím přiblížit céčku.)

Výstup:
* potvrzení validity, nebo chyba

Validace specifických konstruktů pak bude využívat tyto tři algoritmy.
Např. pro validaci `int i = 1 + 1` pustím algoritmus `check_expr` na `1 + 1` s typem `int`.
