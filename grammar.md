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
