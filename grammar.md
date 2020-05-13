## Neterminály

```
<root>  -- start symbol
<stmt>  -- statement
<estmt> -- expression statement (výrazový příkaz)
<pstmt> -- pure statement       (nevýrazový příkaz)
<rstmt> -- right-consuming (expression) statement

<type>  -- type
<id>    -- identifier

<tFun>  -- function type
<tBuff> -- buffer type
<tVec>  -- vector type

<esx>   -- expression/statement generalization
<expr>  -- expression
<rexpr> -- right-consuming expression
<e>     -- atomic expressions, parentheses
<e0>    -- reserved (možná něco jako scope resolution '::' v C++ pro namespacy)
<eCall> -- function call
<eSubs> -- subscript
<eType> -- type annotation & conversion
<ePre>  -- operations with <opPre>
<eMul>  -- operations with <opMul>
<eAdd>  -- operations with <opAdd>
<eOrd>  -- operations with <opOrd>
<eEq>   -- operations with <opEq>
<eAnd>  -- operations with <opAnd>
<eOr>   -- operations with <opOr>

<opPre> -- prefix unary operators '!' '+' '-'
<opMul> -- infix multiplicative operators '*' '/' '%'
<opAdd> -- infix additive operators '+' '-'
<opOrd> -- ordering comparison operators '<' '<=' '>' '>='
<opEq>  -- equality comparison operators '==' '!='
<opAnd> -- logical and '&&'
<opOr>  -- logical or '||'

<nLit>  -- number literal
<bLit>  -- boolean literal
<fCall> -- function call
<subs>  -- subscript
<tAnn>  -- type annotation
<rtAnn> -- right-consuming type annotation
<tCon>  -- type conversion

<exIf>  -- expression if
<rsIf>  -- right-consuming statement if
<psIf>  -- pure statement if

<fDef>  -- function definition (& declaration)
<afDef> -- anonymous function definition

<decl>  -- declaration (with optional assignment)
<assgn> -- assignment
<ret>   -- return

<FPL>   -- function parameter list
<FAL>   -- function argument list
<FTL>   -- function type list
<SL>    -- statement list
```

## Gramatika

```
-- obecná struktura programu:
<root>  -> <SL>
<stmt>  -> <pstmt> | <esx>
<esx>   -> <estmt> | <rstmt> | <expr>
<estmt> -> <block> | <esIf>
<rstmt> -> <rsIf>
<pstmt> -> '' | <ret> | <decl> | <assgn> | <fDef> | <psIf> | <when>

<id>    -> `[_a-z][_a-zA-Z1-9]*`

<type>  -> `[A-Z][_a-zA-Z1-9]*` | <tFun> | <tBuff> | <tVec>

-- složené typy:
<tFun>  -> <type> '(' <FTL>  ')'
<tBuff> -> buffer '[' <expr> ']'
<tVec>  -> <type> '[' <expr> ']'

-- jednoduché příkazy:
<decl>  -> <type> <id>
         | <type> <id> '=' <esx>
<asgn>  -> <id> '=' <esx>
<ret>   -> 'return' <esx>

-- funkce:
<fDef>  -> <type> <id> <FPL> <esx>
         |        <id> <FPL> <cstmt>
<afDef> -> <type>      <FPL> <esx>
         |             <FPL> <cstmt>

-- pomocné neterminály:
<FPL>   -> '(' ')' | '(' (<type> <id>)@',' ')'
<FTL>   -> '(' ')' | '(' (<type>)@','      ')'
<FAL>   -> '(' ')' | '(' (<expr>)@','      ')'
<SL>    -> ''      | (<stmt>)@';'

<block> -> '{' <SL> '}'

-- podmínky:
<_ifh>  -> 'if'   '(' <expr> ')'
<_whh>  -> 'when' '(' <expr> ')'
-- obsahuje jen výrazy:
<exIf>  -> <_ifh> <expr>  'else' <expr>
-- obsahuje i výrazové příkazy:
<rsIf>  -> <_ifh> <estmt> 'else' <expr>   -- končí výrazem
         | <_ifh> <esx>   'else' <rstmt>  -- končí right-consuming výrazovým příkazem
<esIf>  -> <_ifh> <esx>   'else' <estmt>  -- končí výrazovým příkazem
-- obsahuje i nevýrazové příkazy:
<when>  -> <_whh> <stmt>
<psIf>  -> <_ifh> <stmt> 'else' <pstmt>
         | <_ifh> <pstmt> 'else' <stmt>

-- obecná struktura výrazů:
<expr>  -> <eOr>   | <rexpr>
<eOr>   -> <eAnd>  | <eOr>  <opOr>  <eAnd>
<eAnd>  -> <eEq>   | <eAnd> <opAnd> <eEq>
<eEq>   -> <eOrd>  | <eEq>  <opEq>  <eOrd>
<eOrd>  -> <eAdd>  | <eOrd> <opOrd> <eAdd>
<eAdd>  -> <eMul>  | <eAdd> <opAdd> <eMul>
<eMul>  -> <ePre>  | <eMul> <opMul> <ePre>
<ePre>  -> <eType> | <opPre> <e4>
<eType> -> <eSubs> | <tAnn> | <tCon>
<eSubs> -> <eCall> | <subs>
<eCall> -> <e0>    | <fCall>
<e0>    -> <e>

-- operátory:
<opPre> -> '+'  | '-'  | '!'
<opMul> -> '*'  | '/'  | '%' |
<opAdd> -> '+'  | '-'  | 
<opOrd> -> '<'  | '<=' | '>' | '>='
<opEq>  -> '==' | '!='
<opAnd> -> '&&'
<opOr>  -> '||'

-- Jakmile je napravo right-consuming výraz, celý výraz je také right-consuming.
-- Right-consuming výrazy nemohou být levým operandem žádné binární operace.
<rexpr> -> <opPre> <rexpr>
         | <eMul> <opMul> <rexpr>
         | <eAdd> <opAdd> <rexpr>
         | <eOrd> <opOrd> <rexpr>
         | <eEq>  <opEq>  <rexpr>
         | <eAnd> <opAnd> <rexpr>
         | <eOr>  <opOr>  <rexpr>
         | <rtAnn>
         | <afDef>
         | <exIf>

-- atomy:
<e>     -> '(' <expr> ')' | <nLit> | <bLit>
<nLit>  -> `([0-9]*[.]?[0-9]+|[0-9]+.)([eE][+-]?[0-9]+)?`
<bLit>  -> 'true' | 'false'

-- specifické operace:
<fCall> -> <e0> <FAL>
<subs>  -> <eSubs> [ <expr> ]
<tAnn>  -> <type> <estmt>
<rtAnn> -> <type> <rstmt>
<tCon>  -> <type> <eType>
```

## Random myšlenky a příklady

```
// right-consuming výrazy dávájí přednost operacím napravo nehledě na jejich precedenci:
if (c) a else a + b               // if (c) a else (a + b)
int (int x) x + 1                 // int (int x) (x + 1)
if (c) u else v + if (d) w else x // if (c) u else (v + (if (d) w else x))
// je ale možné je použít jako pravý operand libovolného binárního operátoru i jako operand unárního operátoru:
1 + if (c) a else b               // 1 + (if (c) a else b)
-if (c) a else b                  // -(if (c) a else b)
f <?> int (int x) x + 1           // f <?> (int (int x) x + 1), kdybych implementoval nějaký kombinátor <?>, co by měl pracovat s funkcemi
u + v * if (c) w else x           // u + (v * (if (c) w else x))
u * v + if (c) w else x           // (u + v) * (if (c) w else x)

// Volání a subscript mají přednost před konverzí i unárními operátory:
int f(x)  // int (f(x))
int v[i]  // int (v[i])
!int f(x) // !(int (f(x)))

// Volat je možné jen atomické či uzávorkované výrazy, nebo již zavolanou funkci:
f(x)(y)      // (f(x))(y)
(f <?> g)(x)
// Subscriptovaný vector/buffer volat bez uzávorkování nelze:
v[i](x)      // syntax error
(v[i])(x)    // syntakticky ok, ale vector nikdy nebude obsahovat funkci
// Naopak subscriptovat zavolanou funkci lze:
f(x)[i]      // (f(x))[i]

// Tyová anotace i konverze váže silněji, než libovolný operátor:
int x + int { return y } + z // (int x) + (int {return y}) + z

// možná nejednoznačnost?
int f (int x) // deklarace funkce, nebo volání funkce s konverzí parametru a výSLedku?
// pokud jsem nic nepřehlédl, nejednoznačnost by neměla nastávat, jakmile se přečte zbytek příkazu:
int f (int x);                 // volání
int f (int x) + 1;             // volání
int f (int x) x + 1;           // deklarace
int f (int x) { return x + 1 } // deklarace
// kdybych pak chtěl implicitní středníky na koncích řádků ve stylu JavaScriptu:
int f (int x)
{
    return x + 1;
}
// nebylo by zde jasné, zda jde o volání funkce náSLedované blockem:
int f (int x) (;)
{
    return x + 1;
}
// nebo deklaraci funkce blockem:
int f (int x) {
    return x + 1;
}

// if nemůže být použit jako výraz, pokud nemá else:
int i = if (c) x; // syntax error

// také pokud libovolná z větví není použitelná jako výraz:
int i = if (c) if (d) x else y; // syntax error
// v tomto případě je nejednoznačné uzávorkování, ale v obou případech by šlo o syntax error:
if (c) (if (d) x else y) // vnější if nemá else
if (c) (if (d) x) else y // pozitivní větev obsahuje if bez else

// použití if bez else je ale možné v kontextu příkazu:
int f (bool c) {  // vnější SLožený příkaz
    if (c) {      // vnitřní SLožený příkaz
        return 1; // return se vztahuje na vnější block, jelikož ten vnitřní není použit jako výraz
    }             // takový SLožený příkaz nemusí vrátit, to se ale nekontroluje parserem, takže k syntax erroru nedojde
}
int f (bool c) {
    int if (x) {  // vnitřní SLožený příkaz použitý jako výraz
        return 1; // return se vztahuje na vnitřní SLožený příkaz, jelikož je použit jako výraz
    }             // syntax error - if bez else použit jako výraz
}
int f (bool c) {  // vnější SLožený příkaz
    int if (x) {
        return 1;
    } else 2;     // syntakticky ok, ale vnější SLožený příkaz (a tedy i funkce) nic nevrací
}
```
