
## JavaScript Engine․ նշանակությունը և կառուցվածքը (մաս 2/7)

### NGNTJS - մաս 2

---

NGNTJS հոդվածաշարի նախորդ [մաս 1](https://medium.com/@boolfalse/browser-%D5%B6%D5%A5%D6%80%D5%AB-%D5%B4%D5%A1%D5%BD%D5%AB%D5%B6-javascript-%D5%AB-%D5%BD%D5%BF%D5%A5%D5%B2%D5%AE%D5%B8%D6%82%D5%B4%D5%A8-bd7432375275)֊ում ներկայացվել է browser֊ների պատմությունը, JavaScript֊ի առաջացման հիմքերը, ECMA ստանդարտի առաջացումը։
Այս մասում կներկայացնենք JavaScript engine֊ների նշանակությունը և կառուցվածքը։

<img src="https://i.imgur.com/20M5uGG.png" style="width:100%;">

["Check Engine"]

Բոլոր web browser-ները ունեն JavaScript Engine: Դա ծրագրային բաղադրիչ է, որը օպտիմիզացնում և աշխատեցնում է JavaScript կոդը։ Web browser֊ների պատմության սկզբում այդ բաղադրիչը իրենից ներկայացնում էր հիմնականում միայն interpreter, բայց ժամանակակից engine֊ները շատ ավելին են քան interpreter-ը, դրանք ունեն հավելյալ կոմպոնենտներ, որոնք կատարում են օպտիմիզացիա, ինչպես նաև օգտագործում են JIT compilation, որը լավացնում է performance-ը։
JavaScript engine֊ի ֆունկցիոնալն է աշխատեցնել JavaScript կոդը անկախ նրանից՝ այն run է լինում browser֊ում, Node.js֊ում, CLI environment֊ում թե որևէ IoT device֊ում։
JavaScript engine֊ների մասին ավելի մանրամասն կդիտարկենք [մաս 7](https://medium.com/@boolfalse/javascript-%D5%AC%D5%A5%D5%A6%D5%B8%D6%82-runtime-engine-%D5%B4%D5%AB%D5%BB%D5%A1%D5%BE%D5%A1%D5%B5%D6%80-node-js-%D5%B4%D6%80%D6%81%D5%A1%D5%AF%D5%AB%D6%81%D5%B6%D5%A5%D6%80-deno-bun-82cf3222e94b)֊ում, իսկ այս մասում կփորձենք ստանալ բավարար ընդհանրական ինֆորմացիա։

Ներկայումս տարբեր browser֊ներ աշխատում են տարբեր engine֊ներով։ Դրանցից հայտնիներից են՝

- [**SpiderMonkey**](https://spidermonkey.dev/) - երբևէ ստեղծված առաջին engine֊ը նույն Brandon Eich֊ի կողմից, որը սկզբում օգտագործել է Netscape Navigator֊ի կողմից, այնուհետև այն դարձավ [open source](https://searchfox.org/mozilla-central/source/js/src), այժմ աշխատում է Mozilla Firefox֊ի վրա։
- [**V8**](https://v8.dev/) - Google֊ի կողմից թողարկված [open source](https://github.com/v8/v8) engine է ([issue֊ների էջը](https://bugs.chromium.org/p/v8/issues/list))։ Սա կարող է կիրառվել standalone, կամ կարող է embed լինել այլ C++ application֊ներում, ինչպիսին է Node.js runtime֊ը (որը աշխատեցնում է JavaScript կոդը սերվերում), որը նույնպես օգտվում է սրանից։
- [**ChakraCore**](https://github.com/chakra-core/ChakraCore) - Microsoft֊ի կողմից ստեղծած սեփական [open source](https://github.com/chakra-core/ChakraCore) engine-ն է, որն օգտագործում են նախկին IE-ն և այժմվա Edge-ը (մինչ ECMA ստանդարտները՝ լեզուն կոչվել է JScript)։ Սրա յուրահատկություններից է այն, որ սա կոմպիլացնում է սկրիպտները առանձին CPU core֊ի վրա, web browser-ի հետ parallel կերպով։
- [**JavaScriptCore** (JSC)](https://github.com/WebKit/WebKit/tree/main/Source/JavaScriptCore) - [Apple֊ի կողմից](https://developer.apple.com/documentation/javascriptcore) ստեղծված [open source](https://github.com/WebKit/WebKit/tree/main/Source/JavaScriptCore) (սեփական [repo](https://opensource.apple.com/source/JavaScriptCore/)֊ն) engine է։ Սա օգտագործում է Safari֊ին, ինչպես նաև այլ Apple֊ի բոլոր [WebKit engine](https://webkit.org/project/)֊ները (rendering engine)։ Ինչպես V8֊ի դեպքում, նման ձևով JavaScriptCore֊ը առանձին կարող է կիրառվել Swift, Objective-C և C-սպեցիֆիկ application֊ներում։ Որպես engine այն օգտագործվում է նաև Bun֊ում (այս մասին կխոսենք հետագայում)։

Ներկայումս կան այլ open source JavaScript engine֊ներ ևս, որոնք կարող են կիրառվել տարբեր միջավայրերում․

- [Boa](https://github.com/boa-dev/boa) - Boa is an embeddable and experimental Javascript engine written in Rust. Currently, it has support for some of the language.
- [Hermes](https://github.com/facebook/hermes/) - A JavaScript engine optimized for running React Native.
- [Yantra](https://github.com/yantrajs/yantra) - JavaScript Engine for .NET Standard.

Թեկուզև տարբեր browser֊ների մոտեցումները իրենց JavaScript engine֊ներում թեթևակի տարբեր են, ըստ էության՝ բոլորը անում են միևնույն գործը։ Թեկուզ լուծումները կարող են ունենալ որոշ տարբերություններ, այնուհանդերձ նրանց տարբեր լինելը առաջացնում է բնական մրցակցություն, որը ժամանակի ընթացքում բերել է էական փոփոխությունների և զարգացման։
Համենայն դեպս նկարագրենք ժամանակակից JavaScript Engine-ի գործողությունների ընդհանրական պատկերը․

<img src="https://i.imgur.com/l8jD0SY.png" style="width:100%;">

[Պատկերը կառուցված է [draw.io](https://app.diagrams.net/)֊ի օգնությամբ]

Նկարում JavaScript engine-ի միայն շատ ընդհանրական պատկերն է, այն նկարագրում է թե ինչպես է տեղի ունենում JavaScript code֊ը աշխատանքը։ Բացի այն, որ JavaScript engine-ները իրարից տարբերվում են, դրանց տարբերությունները նույպես փոփոխվում  են ժամանակի ընթացքում, քանի որ զարգանալու ընթացքում այն որոշակի փոփոխություններ է կրում։ Բայց զուտ հասկանալու համար կարելի է դիտարկել վերևի նկարում պատկերված սխեման։

**Source code**

1֊ին քայլում ունենք JavaScript/TypeScript կոդ։
Սա կարող է լինել տարբեր միջավայրերի համար նախատեսված կոդ, ինչպիսին է web browser֊ը, server֊ը, CLI֊ը կամ որևէ IoT սարք։

**Parser**

Parser֊ը նախատեսված է վերափոխելու (transforming) կոդը աբստրակտ սինտաքս ծառի։ Parser-ը ստուգում է կոդի սինտաքսը, և եթե այնտեղ կա սխալ, այն հետ է վերադարձնում error, և դրանով արգելակում է հետագա աշխատանքը։
Ճշգրիտ (valid) սինտաքսի դեպքում Parser֊ը կառուցում է սինտաքս ծառը (Abstract Syntax Tree):
V8֊ում Parser֊ի աշխատանքի մասին արժի լսել JSConf EU 2017֊ում V8 թիմի աշխատակցի կողմից [հետևյալ talk](https://www.youtube.com/watch?v=Fg7niTmNNLg)֊ը (համապատասխան [պրեզենտացիան](https://docs.google.com/presentation/d/1b-ALt6W01nIxutFVFmXMOyd_6ou_6qqP6S0Prmb1iDs/))։ Այստեղ նա խոսում է parser֊ների անհրաժեշտության մասին, օրինակի վրա բացատրում է parser֊ի աշխատանքը, խոսում է caching֊ի, parsing֊ի lazy և eager mode֊երի մասին, parsing mode֊երի աշխատանքի մասին function context֊ում, parsing փուլում առաջացած խնդիրների և այլնի մասին։

**Abstract Syntax Tree (AST)**

AST֊ն կիրառվում է նաև շատ այլ ծրագրավորման լեզուների և գործիքների սինտաքս ծառը կառուցելու համար (CSS, HTML, GraphQL, Java, JSON, Markdown, PHP, Scala, SQL, Regular Expressions, և այլն): Սրա իմպլեմենտացիայի վիզուալ օրինակ կարելի է տեսնել այստեղ՝ [AST explorer](https://github.com/fkling/astexplorer):

**Interpreters/Compilers**

> _Հուշում․ անհրաժեշտության դեպքում interpreter֊ի, compiler֊ի, JIT-compiler֊ի հետ կապված կարող եք ծանոթանալ ներքևում։_

Ստանալով աբստրակտ սինտաքս ծառը, այն փոխանցվում է դեպի 4֊րդ քայլ, որտեղ պետք է ինտերպրետատորը մշակի այդ ծառը և դրանից ստանա միջանկյալ/նախնական bytecode տեսք (intermediate representation):
Bytecode֊ը JavaScript engine֊ի աշխատանքի արդյունքի այն վիճակն է, երբ կոդը ներկայացված է որպես հրամանների հաջորդականություն (instruction set): Սա այն փուլն է, երբ ծրագրավորողը չունի անմիջական ազդեցություն։ Այս փուլում սկրիպտ կոդի փոփոխականները և ֆունկցիաները պահվում և օգտագործվում են register (հիմնական հիշողություն) և accumulator (ընթացիկ հիշողություն) կոչվող պահոցներում, և դրանք օգտագործվում են instruction set֊ի պահանջների կատարման համար։

**Intermediate Representation, a.k.a. Bytecode**

Ինտերպրետատորի կողմից ստացված bytecode֊ը իրականում այն տեսքն է, որը հետագայում պետք է transform արվի մեքենայական կոդի (7֊րդ բլոկում), ու այնուհետև փոխանցվի անմիջականորեն դեպի CPU:
Բայց, փաստորեն, տեսնում ենք որ ինտերպրետատորը 4֊րդ բլոկից այն անմիջապես չի փոխանցում 7֊րդ բլոկին, այսինքն՝ աբստրակտ ծառը անմիջապես transform չի արվում մեքենայական կոդի։ Պատճառն այն է, որ մինչև մեքենայական կոդ սարքելը, այստեղ կարիք կա ունենալու որոշակի միջանկյալ տեսք (IR, կամ որ հայտնի  է որպես bytecode), որի վրա պետք է կատարվի որոշակի աշխատանք, ինչպիսին է JIT compiling֊ը (բլոկ 6֊ով)։
Bytecode֊ը սպեցիֆիկ է տվյալ engine֊ին, այսինքն տարբեր engine֊ներ ունեն իրենց սեփական bytecode syntax֊ը և instruction sets֊ը։

Գրված JS կոդի bytecode֊ը տեսնելու միջոցներից է node cli֊ի "--print bytecode" արգումենտի օգտագործումը։ Զուտ պատկերացում ունենալու համար դիտարկենք մի օրինակ։ Ունենք index.js ֆայլ հետևյալ կոնտենտով․

```javascript
function sum() {
  var first = 10;
  var second = 5;
  var third = first + second;
  return third;
}

console.log(sum());
```

Որպեսզի տեսնենք sum ֆունկցիայի հետ կապված bytecode կտորը (ամբողջ bytecode֊ը բավական մեծ է), կարող ենք աշխատեցնել հետևյալը․

```shell
node --print-bytecode --print-bytecode-filter=sum index.js
```

արդյունքում կստանանք հետևյալը․

```yaml
[generated bytecode for function: sum (0x095c31356c51 <SharedFunctionInfo sum>)]
Bytecode length: 13
Parameter count 1
Register count 3
Frame size 24
OSR urgency: 0
Bytecode age: 0
   31 S> 0x95c31357690 @    0 : 0d 0a             LdaSmi [10]
         0x95c31357692 @    2 : c4                Star0 
   50 S> 0x95c31357693 @    3 : 0d 05             LdaSmi [5]
         0x95c31357695 @    5 : c3                Star1 
   67 S> 0x95c31357696 @    6 : 0b f9             Ldar r1
   73 E> 0x95c31357698 @    8 : 39 fa 00          Add r0, [0]
         0x95c3135769b @   11 : c2                Star2 
   98 S> 0x95c3135769c @   12 : a9                Return 
Constant pool (size = 0)
Handler Table (size = 0)
Source Position Table (size = 12)
0x095c313576a1 <ByteArray[12]>
15
```

Արտաբերված տվյալներից կարելի է հասկանալ հետևյալը (մուգ ֆոնտը տառերի նշանակությունն է)․

- LdaSmi [10]
  **l**oa**d** **sm**all **i**nteger **10** into the **a**ccumulator
- Star0
  **st**ore **a**ccumulator value to the **r0** register
- LdaSmi [5]
  **l**oa**d** **sm**all **i**nteger **5** into the **a**ccumulator
- Star1
  **st**ore **a**ccumulator value to the **r1** register
- Ldar r1
  **l**oad **r1** **r**egister value into the **a**ccumulator
- Add r0, [0]
  **add** whatever it is in **r0** **r**egister into the **0** index of the **a**ccumulator
- Star2
  **st**ore **a**ccumulator value to the **r2** register
- Return
  **return** the current value of the accumulator

Ստորև աղյուսակում ցուցադրենք instruction֊ների հաջորդականությունը և յուրաքանչյուր  քայլում accumulator֊ի և register֊ների արժեքները․

```markdown
| Instruction | Accumulator | Register r0 | Register r1 | Register r2 |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| <START>     |             |             |             |             |
| LdaSmi [10] |     10      |             |             |             |
| Star0       |     10      |     10      |             |             |
| LdaSmi [5]  |      5      |     10      |             |             |
| Star1       |      5      |     10      |      5      |             |
| Ldar r1     |      5      |     10      |      5      |             |
| Add r0, [0] |     15      |     10      |      5      |             |
| Star2       |     15      |     10      |      5      |      15     |
| Return      |     15      |     10      |      5      |      15     |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| <END>       |     15      |             |             |             |
```

bytecode֊ի մասով հետաքրքիր հոդված կա գրված հենց V8 թիմի աշխատակցի կողմից՝ [Understanding V8's Bytecode](https://www.fhinkel.rocks/posts/Understanding-V8-s-Bytecode), իսկ [այստեղ](https://www.alibabacloud.com/blog/javascript-bytecode-v8-ignition-instructions_599188) բավականին մանրամասն դիտարկվում է V8֊ի Ignition interpreter֊ի bytecode֊ը տարբեր օրինակների վրա։

**JIT (just-in-time) compiler**

Չնայած նրան որ 6֊րդ քայլը պարտադիր չէ, սակայն ներկայումս հիմնականում բոլոր JavaScript engine֊ները ունեն JIT compiler բաղադրիչ։ Bytecode֊ի վրա տարբեր engine֊ներ կիրառում են իրենց optimization compiler֊ները, որոնք կարող են լինել մեկից ավելի, և օպտիմիզացնում են աշխատանքը։

<img src="https://i.imgur.com/dtfApv4.png" style="width:100%;">

[Նկարը՝ ստորև բերված հղումներից]

Դրանց մասին կարելի է ավելի մանրամասն ծանոթանալ սեփական թիմերի աշխատակիցների կողմից՝

- **V8**: [Fanziska Hinkelmann - Introduction to JavaScript engines and performance](https://www.youtube.com/watch?v=WBkMm19ziUI)
- **JSC**: [Michael Saboff - JavaScriptCore, many compilers make this engine perform](https://www.youtube.com/watch?v=mtVBAcy7AKA)

Հակիրճ նկարագրությամբ՝ JIT compiler֊ը գործիք է, որը անմիջապես bytecode֊ից սարքում է machine code, կատարելով օպտիմիզացիա։ Ներկայումս, օրինակ V8 engine֊ը որպես JIT compiler կիրառում է [TurboFan](https://v8.dev/docs/turbofan), SpiderMonkey engine֊ը որպես JIT compiler կիրառում է [WarpMonkey](https://hacks.mozilla.org/2020/11/warp-improved-js-performance-in-firefox-83/) (նախկինում՝ IonMonkey):

**Machine Code**

Հարկավոր է հաշվի առնել այն հանգամանքը, machine code֊ը, լինելով [assembly language](https://en.wikipedia.org/wiki/Assembly_language), սպեցիֆիկ է մեքենայի architecture֊ին (ARM, Intel-based կամ այլ processor֊ների դեպքում), այսինքն՝ մեքենայական կոդը տարբեր ճարտարապետություն ունեցող մեքենաների համար լինելու է տարբեր, այդ իսկ պատճառով անհրաժեշտ է ունենալ այդ միջանկյալ կոչվող bytecode֊ը։

<img src="https://i.imgur.com/ZFV6zzf.jpg" style="width:100%;">

[Նկարը՝ [unsplash.com](https://unsplash.com/photos/QndYCQc_a3g)]

**Compiler, Interpreter, JIT-compiler; անալոգներ ռեալ կյանքից**

- **_Compiler (կոմպիլյատոր)_**
  Սա ծրագրային ապահովում է, որը բարձր մակարդակի լեզվով ([high-level language](https://en.wikipedia.org/wiki/High-level_programming_language)) գրված source code֊ը (օրինակ՝ C++ լեզուն) "միանգամից և ամբողջությամբ" թարգմանում է մեքենայական կոդի (այն սպեցիֆիկ է տվյալ մեքենայի ճարտարապետությանը)։
  Թարգմանելու համար այն կատարում է ողջ source code֊ի ստատիկ անալիզ, ստուգում է error֊ների առկայությունը և գեներացնում է executable կամ binary ֆայլ, որը կարող է execute լինել մեքենայի կողմից՝ compiler֊ից անկախ։ Այդ ֆայլն արդեն կարելի է դիտարկել որպես առանձին օբյեկտ, որը հետագայում execute անելու համար compile անելու կարիք չի լինի։
- **_Interpreter (ինտերպրետատոր)_**
  Սա մեծամասամբ ինտերպրետացվող լեզվի հետ կամ JavaScript֊ի դեպքում դրա engine֊ի հետ արդեն իսկ ներդրված ծրագիր է։ Դասական դեպքում (երբ ամեն տողում գրված է մեկ instruction) այն տող առ տող կարդում է յուրաքանչյուր հրամանը, թարգմանում այն մեքենայական լեզվի և execute անում այդ հրամանին համապատասխանող մեքենայական կոդը (JavaScript֊ի դեպքում մի փոքր տարբերությամբ)։
  JavaScript֊ի դեպքում տարբերությունն այն է, որ interpreter֊ը source code֊ը տող առ տող վերածում է ոչ թե միանգամից մեքենայական լեզվի, այլ միջանկյալ լեզվի (IR - intermediate representation)։ Երբ source code֊ն ամբողջությամբ ինտերպրետացվել է միջանկյալ լեզվի (սա հենց այդ bytecode֊ն է), այն հետո ամբողջությամբ թարգմանվում է մեքենայական լեզվի և execute է արվում։
  Ժամանակակից interpreter֊ները անալիզ են անում source code֊ը ոչ ամբողջապես, այլ հերթական հրամանը ստանալու սկզբունքով, և հերթական հրամանը ստանալու պահին կարող են կատարել օպտիմիզացիա։
  Java լեզվում նույնպես տեղի է ունենում source code֊ի թարգմանություն դեպի bytecode֊ի (օրինակ՝ javac compiler֊ի միջոցով), ապա bytecode֊ը interpreter֊ի և JIT compiler֊ի միջոցով թարգմանվում է մեքենայական կոդի՝ տվյալ միջավայրում JVM֊ի համար, որը պետք է execute արվի։
- **_JIT (just-in-time) compiler (JIT կոմպիլյատոր)_**
  JIT compiler֊ը կարելի է համարել որպես compiler֊ի և interpreter֊ի հիբրիդ։ Սա ծրագրային ապահովում է, որը դինամիկ կերպով անալիզ է անում source code֊ը և մինչև execute լինելը բլոկ առ բլոկ կոմպիլացնում է մեքենայական լեզվի՝ բլոկների ընթացքում կատարելով օպտիմիզացիա։ Երբ source code֊ը ամբողջությամբ կոմպիլացվել է, և արդեն կա ամբողջական մեքենայական կոդը, տեղի է ունենում execution։

<img src="https://i.imgur.com/u5Pts3Y.png" style="width:100%;">
 
[Նկարը՝ [americangirl.com](https://www.americangirl.com/products/american-girl-baking-cookbook-dtd87)]

- **_Compiler - անալոգ ռեալ կյանքից_**
  Պատկերացրեք՝ ունեք տորթ թխելու բաղադրատոմս, որը գրված է ձեզ անհասկանալի լեզվով: Դուք բաղադրատոմսը տալիս եք թարգմանչին, ով կարդում է ամբողջ բաղադրատոմսը, հասկանում է հրահանգները և թարգմանում դրանք ձեզ հասկանալի լեզվով: Այնուհետև թարգմանված բաղադրատոմսը տալիս է ձեզ, և դուք կարող եք պատրաստել տորթը՝ հետևելով հրահանգներին: Այսպիսով, դուք կարող եք պատրաստել տորթը, ամբողջությամբ ունենալով ողջ թարգմանված հրահանգները։
- **_Interpreter - անալոգ ռեալ կյանքից_**
  Պատկերացրեք՝ ունեք տորթ թխելու բաղադրատոմս, որը գրված է ձեզ համար անհասկանալի լեզվով, իսկ ձեր ընկերը տիրապետում է այդ լեզվին։ Դուք այս անգամ դիմում եք ձեր ընկերոջը օգնության։ Ձեր ընկերը տող առ տող կարդում է հրահանգները և հերթականորեն թարգմանում է դրանք ձեզ: Ամեն հրահանգի թարգմանությունը լսելուց հետո դուք կատարում եք ընթացիկ հրահանգը, և արդյունքում տորթը պատրաստվում է հաջորդական արված քայլերի արդյունքում։
  Ոչ պարտադիր, բայց ժամանակակից կյանքում հնարավոր է, որ ձեր ընկերը ունենա նաև որոշակի խոհարարական գիտելիքներ։ Այս դեպքում, երբ օրինակ N֊րդ հրահանգն է _"ավելացնել 3 գդալ շաքար"_, իսկ N+1֊րդ հրահանգն է _"ավելացնել ևս 2 գդալ շաքար"_, ձեր ընկերը այս երկու հրահանգները համարում է որպես մեկ հրահանգ և ձեզ համար ձևակերպում է օպտիմիզացված հրահանգը՝ _"ավելացնել 5 գդալ շաքար"_՝ դրանով խնայելով գործողությունների քանակը և կրճատելով պատրաստման ժամանակը։
- **_JIT compiler - անալոգ ռեալ կյանքից_**
  Պատկերացրեք՝ ունեք բաղադրատոմս, որը գրված է անհասկանալի լեզվով, բայց ունեք անձնական օգնական, ով կարող է սինխրոն թարգմանել այդ լեզուն, ինչպես նաև ունի գերազանց խոհարարական գիտելիքներ։ Ձեր օգնականը բլոկ առ բլոկ կարդում է այն, թարգմանում այն և գրի է առնում, դրա վրա կատարում է որոշ օպտիմիզացիաներ (ավելի ակտիվ, քան նախորդ՝ ընկերոջ դեպքում), և ասում է ձեզ առաջիկա մի քանի հրահանգների օպտիմիզացված տարբերակը։ Ամեն հաջորդ հրահանգի ժամանակ, մինչ դուք կատարում եք հրահանգը, ձեր օգնականը շարունակում է ինքն իր տետրում անել գրառումներ, կատարել հաշվարկներ, մտածել, և պատրաստվում է ասել ձեզ հաջորդ հրահանգների բլոկը։
  Օրինակ, ձեր օգնականը կարող է ասել _"լվացրու 50 հատ ելակ, հեռացրու պոչերը, մեջտեղից կտրատիր դրանք և այդ 100 կիսաելակները կտրված մակերեսով տեղադրիր տորթի շերտի վրա"_, այնինչ՝ ոչ օպտիմիզացված դեպքում դուք պետք է լվայիք 1 ելակ, հեռացնեիք պոչը, մեջտեղից կտրատեիք այն և կտրված մակերեսով 2 կիսաելակները տեղադրեիք տորթի շերտի վրա, և այս հրահնագների շարքը պետք է անեիք 50 անգամ, քանի որ դուք չգիտեք ապագա հրահանգները։ Այնինչ, եթե իմանում եք, որ հետագա 49 քայլերում նույպես պետք է լվաք ելակ, դուք այդ 50 ելակները լվանում եք մի տարայի մեջ միանգամից։
  Թեորետիկ դեպքում, կարելի է պատկերացնել, որ 20 հրահանգ ունեցող բաղադրատոմսը ձեր օգնականը ձեզ համար կարող է թարգմանել և ներկայացնել օրինակ 5 հրահանգի տեսքով։
  Այսպիսով՝ ձեր օգնականը օգնում է ձեզ հաշվարկների օպտիմիզացիայի, ռեսուրսների խնայման և ժամանակի կրճատման հարցում։

Ստորև այն օգտակար ռեսուրսների հղումներ են, որոնք ուղղակիորեն տեղ չգտան այս մասում, բայց կապված են տվյալ թեմայի հետ․

- [An Introduction to Speculative Optimization in V8](https://ponyfoo.com/articles/an-introduction-to-speculative-optimization-in-v8) [Nov, 2017]
- [Attacking JS engines: Fundamentals for understanding memory corruption crashes](https://www.sidechannel.blog/en/attacking-js-engines) [Apr, 2023]
- [How JavaScript Works: Under the Hood of the V8 Engine](https://www.freecodecamp.org/news/javascript-under-the-hood-v8/) [Aug, 2020]
- [The Mysterious Realm of JavaScriptCore](https://www.cyberark.com/resources/threat-research-blog/the-mysterious-realm-of-javascriptcore) [Mar, 2021]
- [Talks by fhinkel on YouTube](https://www.youtube.com/playlist?list=PL65pp6Tpk690HkOh324FqtFYyxFVnEFEX)
- [How JavaScript works: Optimizing the V8 compiler for efficiency](https://blog.logrocket.com/how-javascript-works-optimizing-the-v8-compiler-for-efficiency/) [Sep, 2019]
- [Just In Time (JIT) Compilers - Computerphile](https://www.youtube.com/watch?v=d7KHAVaX_Rs) [Nov, 2022]
- [Ode to the V8](https://medium.com/@lucygibbmar/ode-to-the-v8-ff2a4c394ac8) [Apr, 2019]

> Հավելում․ Հավելում․ նրանց համար, ովքեր հետաքրքրված են ցածր մակարդակի հետ կապված (compiler֊ներ, interpreter֊ներ, և այլն), YouTube֊ում հասանելի օգտակար ռեսուրսներից են՝ [MIT 6.172 Performance Engineering of Software Systems, Fall 2018](https://www.youtube.com/playlist?list=PLUl4u3cNGP63VIBQVWguXxZZi0566y7Wf) և [Compilers and Interpreters 2022](https://www.youtube.com/playlist?list=PLlUGRprNEgH-FVK5NekKPg__7ySjeI3gn) դասախոսությունները, իսկ պրոֆեսիոնալ մոտեցման համար՝ [DragonBook֊ի 2֊րդ հրատարակությունը](https://www-2.dc.uba.ar/staff/becher/dragon.pdf)։

NGNTJS հոդվածների շարքի այս մասում ներկայացվեց JavaScript engine֊ների նշանակությունը և կառուցվածքը։
Հաջորդ՝ [մաս 3](https://medium.com/@boolfalse/execution-context-%D5%BF%D5%BE%D5%B5%D5%A1%D5%AC%D5%B6%D5%A5%D6%80%D5%AB-%D5%BA%D5%A1%D5%B0%D5%BA%D5%A1%D5%B6%D5%B8%D6%82%D5%B4-4bc4fc2ebd4c)֊ում կանցնենք JavaScript֊ի աշխատանքի ուսումնասիրմանը։ Կներկայացնենք Execution context֊ը և տվյալների՝ փոփոխականների և ֆունկցիաների (first-class citizens) պահպանումը։ Ընթացում կառնչվենք այնպիսի տերմինների, ինչպիսիք են call stack֊ը, փոփոխականների տիպերը, hoisting֊ը և այլն։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
