
## JavaScript. լեզու, runtime, engine, միջավայր; Node.js մրցակիցներ՝ Deno, Bun (մաս 7/7)

### NGNTJS - մաս 7

---

NGNTJS հոդվածաշարի նախորդ [մաս 6](https://medium.com/@boolfalse/event-loop-%D5%A8-%D5%BA%D6%80%D5%A1%D5%AF%D5%BF%D5%AB%D5%AF-%D6%85%D6%80%D5%AB%D5%B6%D5%A1%D5%AF%D5%B6%D5%A5%D6%80%D5%B8%D5%BE-%D5%A8%D5%B6%D5%A4%D5%B0%D5%A1%D5%B6%D5%B8%D6%82%D6%80-%D5%BA%D5%A1%D5%BF%D5%AF%D5%A5%D6%80-363eff47b6c9)֊ում ուսումնասիրել ենք event-loop մեխանիզմը գործնական օրինակների միջոցով և ծանոթացել ենք Reactor Pattern֊ի աշխատանքի հետ։
Այս մասում, արդեն տեղեկացված լինելով որոշ տերմինների մասին, կքննարկենք JavaScript լեզվի, engine֊ի, runtime֊ի և environment֊ի (միջավայր) մասին։ Ապա կծանոթանանք ժամանակակից JavaScript runtime/environment֊ներին, ինչպիսիք են Deno֊ն և Bun֊ը, և ոչ միայն։

<img src="https://i.imgur.com/98GtbyS.png" style="width:100%;">
 
[Նկարը՝ JSConfEU23֊ի [հետևյալ talk](https://youtu.be/aR_zxqoeBpQ?t=890)֊ից]

Մինչ այս մենք խոսել ենք JavaScript լեզվի, մասնավորապես որպես ստանդարտ ընդունված ECMAScript֊ի մասին, որը մինչև 2000֊ականների առաջին տասնամյակը համարվում էր միայն որպես browser֊ում աշխատող սկրիպտային լեզու։

Հոդվածաշարի նախորդ մասերից արդեն ունենալով տեղեկությունների որոշ պաշար, գիտենք որ browser֊ում աշխատող JavaScript (TypeScript) լեզուն բերվում է մեքենայական կոդի, որպեսզի աշխատեն այն հրամանները, որոնք նախատեսված են ցածր մակարդակների համար (մանրամասն քննարկել ենք հոդվածաշարի [մաս 2](https://medium.com/@boolfalse/javascript-engine-%D5%B6%D5%B7%D5%A1%D5%B6%D5%A1%D5%AF%D5%B8%D6%82%D5%A9%D5%B5%D5%B8%D6%82%D5%B6%D5%A8-%D6%87-%D5%AF%D5%A1%D5%BC%D5%B8%D6%82%D6%81%D5%BE%D5%A1%D5%AE%D6%84%D5%A8-d900993ba01b)֊ում)։
Սա JavaScript engine֊ի հիմնական գործն է՝ անել JavaScript կոդի ինտերպրետացիա, օգտագործելով call stack (հրամանների հաջորդականության համար օգտագործվող պահոց) և memory heap (պրիմիտիվ և հղումային տիպի փոփոխականների համար նախատեսված պահոց որը անհրաժեշտ է լինում կոդի աշխատանքի ընթացքում):

Ինչպես արդեն գիտենք՝ տարբեր ընկերություններ ունեն իրենց մշակած տարբերակները որպես JavaScript engine֊ներ։ Որպես օրինակ կարող ենք նշել Google֊ի կողմից 2008֊ին open source թողարկված [V8](https://chromium.googlesource.com/v8/v8)֊ը, որը աշխատում է Chrome browser֊ում։ Այլ browser֊ները ևս ունեն իրենց JavaScript engine֊ները, բայց ժամանակակից  բոլորն browser֊ներում ինքնին JavaScript լեզուն համապատասխանեցվում է ECMAScript ստանդարտին՝ իհարկե տարբեր ընկերությունների կողմից կրելով որոշակի վարիացիաներ։

<img src="https://i.imgur.com/HqrOUO9.png" style="width:100%;">
 
[The official mirror of the V8 Git repository on [GitHub](https://github.com/v8/v8#readme)]

Մենք գիտենք նաև, որ ժամանակակից browser֊ներում բավականին հաճախ են օգտագործում տարբեր Web API֊ներ, ինչպիսիք են հնուց հայտնի setTimeout()֊ը, setInterval()֊ը, իսկ արդեն մոդեռն JavaScript֊ում նաև համեմատաբար վերջերս ավելացված fetch()֊ը, WebSockets֊ը, WebRTC֊ն, և այլն։
Ստացվում է, որ այս հատկությունները ինքնին չեն պատկանում JavaScript լեզվին։ Եվ սա փաստորեն դուրս է JavaScript engine֊ի ազդեցության տիրույթից։
Սա դեռ մի կողմ, բացի վերում նկարագրված մեթոդներից, այդ մեթոդները կարիք ունեն օգտագործելու event-loop և դրա հետ կապված անհրաժեշտ կոմպոնենտները։
Եվ այս ամենը, ինչը դուրս է ինքնին JavaScript լեզվից (engine֊ից), իր մեջ է ներառում JavaScript runtime֊ը։
Ստացվում է, որ JavaScript runtime֊ի հիմնական բաղկացուցիչ կոմպոնենտներն են․

- **JavaScript engine**֊ը․ call stack֊ը, memory heap֊ը և ինքը լեզուն, որը ղեկավարում է դրանք։
- **WebAPI**֊ները․ setTimeout(), setInterval(), setImmediate(), fetch() և այլն։
- **Event-loop** և դրա հետ կապված մեխանիզմները․ WebAPI֊ների և շատ այլ ասինխրոն գործողությունների աշխատեցման համար նախատեսված գործիքակազմ:

2000֊ականների սկզբին, երբ JavaScript֊ը դեռ աշխատում էր միայն browser֊ներում՝ ապահովելով կայքերի դինամիկ ինտերֆեյս, ինժեներները փորձում են ստանալ JavaScript runtime֊ի standalone տարբերակ, որը կաշխատեր browser֊միջավայրից դուրս։ Կարելի է ենթադրել, որ 2008֊ին Google֊ի կողմից թողարկված V8 engine֊ը հիմք է հանդիսանում նրա, որ արդեն 2009֊ին [Ryan Dahl](https://tinyclouds.org/)֊ի կողմից թողարկվում է [Node.js](https://github.com/nodejs/node#readme)֊ը։

<img src="https://i.imgur.com/TAYYCV9.png" style="width:100%;">
 
[Node.js [official website](https://nodejs.org/en)]

Այսպիսով՝ V8֊ը C++֊ով գրված JavaScript engine է, որի հիման վրա "հավաքվել է" և ավելացվել է այն ամբողջ գործիքակազմը, որը անհրաժեշտ է JavaScript֊ը աշխատեցնելու համար browser֊միջավայրից դուրս։ Իսկ Node.js֊ը այն տեխնոլոգիան է, որը թույլ է տալիս աշխատեցնել JavaScript (այս դեպքում՝ V8-engine֊ը) browser֊միջավայրից դուրս՝ օգտագործելով այն ինչպես server-side հատվածում, այնպես էլ CLI tool֊երի կիրառման համար։
Այս իսկ պատճառով՝ Node.js֊ը համարվում է runtime environment։ Այս իմաստով՝ V8֊ը կարելի է համարել Node.js֊ի համար որպես framework։

Դեռևս NGNTJS֊ի [մաս 2](https://medium.com/@boolfalse/browser-%D5%B6%D5%A5%D6%80%D5%AB-%D5%B4%D5%A1%D5%BD%D5%AB%D5%B6-javascript-%D5%AB-%D5%BD%D5%BF%D5%A5%D5%B2%D5%AE%D5%B8%D6%82%D5%B4%D5%A8-bd7432375275)֊ում մենք նշել ենք որոշ հայտնի JavaScript engine֊ներ, այժմ այս վերնագրի տակ համապատասխան կլինի նշել նաև հետևյալը․

- Ժամանակին փորձեր են եղել նաև ունենալու Node.js֊ի անալոգները այլ engine֊ների օգտագործմամբ՝ [node-chakracore](https://github.com/nodejs/node-chakracore) (Node.js on ChakraCore) և [spidernode](https://github.com/mozilla/spidernode) (Node.js on top of SpiderMonkey), սակայն արդեն վաղուց սրանք maintain չեն արվում։
- Ընդունված է, որ եթե Node.js֊ը իր V8 engine֊ով հանդերձ կիրառում ենք սերվերում, ապա ավելի փոքր ծանրաբեռնվածության համար նախատեսված IoT սարքերում կարելի է կիրառել ավելի թեթև engine, օրինակ՝ [jerryscript](https://github.com/jerryscript-project/jerryscript) (ultra-lightweight JavaScript engine for the Internet of Things) կամ [Duktape](https://github.com/svaarala/duktape) (embeddable Javascript engine with a focus on portability and compact footprint)։

Որպես Node.js֊ի "ալտերնատիվ", ներկայումս կան այլ JavaScript runtime environment֊ներ ևս, ինչպիսին է [Deno](https://deno.land/)֊ն։

<img src="https://i.imgur.com/kwq1eV8.png" style="width:100%;">
 
[Նկարը՝ deno [official website](https://deno.land/)֊ից]

Deno֊ն ("դինո") Ryan Dahl֊ի կողմից ստեղծված երկրորդ հայտնի պրոյեկտն է, որի մասին հեղինակը պաշտոնապես հայտարարել է 2018֊ին՝ [JSConfEU talk](https://www.youtube.com/watch?v=M3BM9TB-8yA)֊ում: 2019֊ի [հետևյալ կոնֆերենսի](https://www.youtube.com/watch?v=1gIiZfSbEAE) ժամանակ հեղինակը ընդգծում է հետևյալ հիմնական հատկությունները․

- Ստեղծվել է V8 engine֊ի հիման վրա (ինչպես Node.js֊ը)։
- Այն execute է անում TypeScript և JavaScript, ունի ներդրված TypeScript compiler (Node.js֊ը TypeScript֊ի համար կիրառում է առանձին [compiler](https://www.typescriptlang.org/docs/handbook/compiler-options.html), օգտվելով [typescript](https://github.com/microsoft/TypeScript#readme) npm package֊ից)։
- Օգտագործվում է [V8 Snapshots](https://v8.dev/blog/custom-startup-snapshots) աշխատեցնելու համար TS compiler֊ը ավելի արագ։
- Սկզբում այն գրվել է Go֊ով, ապա վերագրվել է և այժմ գրված է [Rust](https://www.rust-lang.org/)֊ով (Node.js֊ը գրված է C++֊ով)։ Compiling֊ը կատարում է հետևյալ տրանսֆորմացիան՝
  JavaScript -> TypeScript -> Rust
- Event-loop֊ի համար օգտագործվում է Tokio (որպես անալոգ [libUV](https://libuv.org/)֊ին Node.js֊ում)։
- Cross-platform է (ինչպես և Node.js֊ը)։
- Աշխատում է single executable դիզայնով (Node.js֊ում այսպես չէ)։
- By default ունի secure execution մեխանիզմ արտաքին ռեսուրսները իմպորտ անելու համար, ինչպես օրինակ URL֊ների դիմում, socket֊ների բացում և այլն (չկա Node.js֊ում): ի տարբերություն Node.js֊ի՝ որտեղ օրինակ կարող էինք install անել 3-rd party ռեսուրս որը կարող էր անել անցանկալի գործողություններ (օրինակի համար կարդալ կոնֆիդենցիալ ֆայլերի պարունակությունը և ուղարկել դուրս), այստեղ by design ցանկացած գործողություն որը ունի permission֊ի կարիք, անպայման force է արվում՝ admin֊ի (տվյալ դեպում developer֊ի) կողմից ստանալու անհրաժեշտ permission֊ները։
- CLI արգումենտով պարտադիր  է նշել այն ռեսուրսները, որոնց դիմելու համար պահանջվում է permission, օրինակ՝ "--allow-read=/tmp"֊ով թույլատրում ենք կարդալ "/tmp" directory֊ին, կամ "--allow-net=google.com"֊ով թույլատրում ենք կապ հաստատել դեպի google.com (չկա Node.js֊ում)։ Node.js֊ի "npm start"֊ի փոխարեն ունենք հետևյալը (որպես sample).
  deno run https://bit.ly/deno-bronto --allow-net --allow-read
- Համեմատած Node.js֊ի՝ support են արվում ոչ բոլոր Browser API֊ները, Deno֊ի նպատակը այլ է։
- By design Deno֊ի 3-rd party մոդուլների համակարգը դիստրիբուտիվ է, քանի որ դրանք import են արվում relative կամ absolute URL֊ներից (Node.js֊ում նախօրոք install են արվում), օրինակ՝
  import { serve } from "https://deno.land/std/http/server.ts";
  սա թույլ է տալիս խուսափել "node_modules"֊ի բարդ ալգորիթմներից ինչպես նաև չունենալ package.json։
- Packaging system֊ը նման է Node.js֊ին՝ այն առաջին անգամ fetch է անում անհրաժեշտ մոդուլները, և մյուս անգամներին օգտվում է cache արվածից, քանի դեռ չենք դիմել դրանց թարմացնելուն։
- Եթե Node.js֊ում npm֊ը համարվում էր որպես add-on (հավելված), Deno֊ում npm֊ը ներդրված է որպես npm-client։
- Ի տարբերություն Node.js֊ի, որտեղ ֆայլը կարող է օգտագործվել նաև որպես module (multiple submodules in it) և export արված մոդուլից import անել կոնկրետ որոշակի submodule֊ներ, Deno֊ում խստացված է, ֆայլը միայն կարող է օգտագործվել որպես մոդուլ՝
  Module == File == URL
- Ի տարբերություն մոնոլիթ Node.js֊ի՝ Deno֊ն Rust crate֊երի հավաքածու է, որը թույլ է տալիս ունենալ standalone executable֊ներ։ Որպես sample ներկայացված է [Cargo-TypeScript integration crate](https://crates.io/crates/deno_typescript)֊ը, որը թույլ է տալիս TS compile անել V8 Snapshots֊ում և ունենալ quick startup։
- Deno թիմը ունի նաև թողարկած Deno [Fresh framework](https://github.com/denoland/fresh)֊ը։ Հակիրճ կարելի է ասել, որ այն մի փոքր ավելին է քան [express.js](https://expressjs.com/)֊ը Node.js֊ի համար։ Այն ունի ներդրված front-end rendering ապահովում, որի համար կիրառվում է [preact](https://preactjs.com/) և [JSX](https://react.dev/learn/writing-markup-with-jsx)։

<img src="https://i.imgur.com/DQiXqDw.png" style="width:100%;">
 
[Նկարը՝ [deno.com](https://deno.com/deploy)]

Բացի այս հատկությունները նշելուց, կարծում եմ՝ արժի այստեղ ունենալ նաև որոշ թարմացումներ 2023֊ի NodeCongress֊ում (JavaScript Conferences by GitNation) տեղի ունեցած [կոնֆերենսից](https://www.youtube.com/watch?v=LVEGRj3RZSA)․

- Deno 2.0֊ն ունի KV (key-value) built-in database, որը աշխատում է SQLite֊ի հիման վրա։ Սա տալիս է հնարավորություն առանց հավելյալ բազաների օգտագործման՝ ունենալ պատրաստի բազա արդեն իսկ ներդրված գործիքակազմով․
  - KV֊ում պահվում են JavaScript object֊ներ;
  - հասանելի են անհրաժեշտ get, set, list, delete հրամանները;
- Մինչ [Deno 2.0](https://github.com/denoland/deno/milestones)֊ի թողարկումը՝ KV֊ի հետ աշխատելու համար կարելի է օգտվել [հետևյալ գործիքից](https://dash.deno.com/playground/getting-started-with-kv)։
- Deno թիմը դիստրիբուտիվ գաղափարի խրախուսման համար պատրաստել է [Deno-Deploy գործիքը](https://deno.com/deploy/pricing), որով կարելի է deploy անել և ունենալ ռեալ աշխատող app։
- Deno-Deploy֊ը աշխատում է FoundationDB֊ի հիման վրա, և ունի հետևյալ առանձնահատկությունները
  - GitHub֊ի public և private repo֊ների ինտեգրացիայի հնարավորություն
  - աշխատում է աշխարհի 35+ network location֊ներում
  - տրամադրվում է անվճար *.deno.dev subdomain և custom domain֊ներ
  - անվճար և ավտոմատ HTTPS / TLS
  - անսահմանափակ production deployment֊ներ  և preview֊ներ
  - մինչև 10ms/request CPU֊ի տրամադրում

Deno֊ի փիլիսոփայությունը նաև կայանում է դիստրիբուտիվ գաղափարը խրախուսումը։ Ըստ հեղինակի խոսքի՝ Node֊ը հնարավորություն է տալիս ստեղծել օպտիմալ լոկալ սերվերներ, այնինչ Deno֊ի նպատակն է ստեղծել օպտիմալ գլոբալ սերվերներ։

Նշենք, որ ըստ Ryan Dahl֊ի՝ Deno 2.0֊ն պետք է թողարկվեր 2023֊ի ամռանը, սակայն այն արդեն իսկ ուշանում է, բայց և այնպես վերջին նկարագրված կետերը  վերաբերում են մասնավորապես Deno 2.0֊ին։

Որպես Node.js և Deno պրոյեկտների "ալտերնատիվ", վերջին ժամանակներում հայտնի դարձավ նաև այլ JavaScript runtime environment, ինչպիսին է [Bun](https://github.com/oven-sh/bun)֊ը։

<img src="https://i.imgur.com/xJwpRl2.png" style="width:100%;">
 
[Նկարը՝ bun [official website](https://bun.sh/)֊ից]

Bun֊ը ("բան") [Jarred Sumner](https://twitter.com/jarredsumner)֊ի կողմից ստեղծված open-source պրոյեկտ է: 2022֊ի [հանդիպման](https://www.youtube.com/watch?v=eF48Ar-JjT8) ժամանակ հեղինակը ընդգծում է հետևյալ հիմնական հատկությունները․
[Նշում․ քանի որ soft֊ը ժամանակի ընթացում փոփոխվում է, այդ պատճառով այստեղ չեմ նշի թե արագությամբ կոնկրետ քանի անգամ կամ քանի տոկոսով է գերազանցում Bun֊ը Node.js֊ի այս կամ այն ֆունկցիոնալին։ Հեղինակը հաճախակի թվիթում է իր էջում]

- Bun֊ը հանդիսանում է որպես ժամանակակից all-in-one JavaScript runtime environment։ Bun֊ը ունի այն հիմնական գործիքակազմը որով կարելի է start անել JS/TS կոդը արագ քան Node.js֊ում։ Այն ունի ներդրված transpiler JS/TS կոդի համար, ունի ներդրված npm package manager, front-end dev server, և իհարկե runtime։
- Իմպլեմենտացված են ավելի օպտիմիզացված [Node.js-API](https://nodejs.org/api/addons.html#node-api)֊ները, ինչպիսիք են fs.copyFile և fs.writeFile մեթոդները, TextEncoder.encode֊ը, buffer.toString('bas64')֊ը և այլն, որոնք աշխատում են ավելի արագ։
- bun install֊ի միջոցով npm package֊ները install են արվում ավելի արագ։
- bun run֊ը start է անում package.json script֊ները ավելի արագ։
- [Node.js SQLite](https://github.com/WiseLibs/better-sqlite3)֊ը ավելի արագ է Bun֊ում՝ [bun:sqlite](https://bun.sh/docs/api/sqlite)։
- Bun֊ը օգտագործում է [JavaScriptCore](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore) engine, ի տարբերբերություն Node.js֊ի, որն օգտագործում է [V8 engine](https://chromium.googlesource.com/v8/v8.git)։
- Bun֊ը մեծամասամբ գրված է [Zig](https://ziglang.org/)֊ով, ի տարբերություն Node.js֊ի, որը գրված է C++֊ով։
- Հեղինակը նախորդ երկու կետերը համարում է Bun֊ի արագ աշխատանքի հիմքերը, բայց դրանցից բացի նաև նշում է հետևյալ պատճառները․
  - Bun֊ը օգտագործում է system call֊երը շատ ավելի ճկուն;
  - Bun֊ում կիրառվում են ժամանակակից տեխնոլոգիաներ ինչպիսին է [SIMD](https://arxiv.org/abs/2109.10433)֊ը (Single Instruction Multiple Data), որը թույլ է տալիս մաքսմիալ օգտվել CPU֊ի արագության հնարավորություններից (այժմ [կիրառվում է](https://simdutf.github.io/simdutf/) նաև Node.js֊ում);
  - բավականին շատ ժամանակ է ծախսվել փորձարկումների և օպտիմիզացիայի վրա;
- Պլանավորված է [Bun 1.0](https://twitter.com/jarredsumner/status/1678424677629464576)֊ի թողարկումը այս տարի (2023֊ի սեպտեմբերին), որը կլինի stable վերսիա և կունենա հնարավորինս քիչ bug֊եր։

Ընդհանուր առմամբ, Bun֊ը, գրված լինելով Rust֊ով, իր հետ բերում է նոր փիլիսոփայություն,  ինչպես հեղինակն է նշում՝
[_"in Bun your JavaScript code isn't I/O bound, it's system-call bound."_](https://youtu.be/eF48Ar-JjT8?t=264)

Կարծում եմ՝ հոդվածաշարի այս մասում արժի նշել նաև Microsoft֊ի կողմից ստեղծված [Napa.js](https://github.com/Microsoft/napajs/wiki/introduction) JavaScript runtime֊ի մասին։

<img src="https://i.imgur.com/rX89wnz.png" style="width:100%;">
 
[[microsoft/napajs](https://github.com/microsoft/napajs)]

Թեկուզ այն 2018֊ից maintain չի արվում և դադարել է դրա վրա աշխատանքը, բայց այս runtime֊ի նշանակալից հատկությունն այն է, որ ի տարբերություն Node.js֊ի և ալտերնատիվ runtime֊ների, այն նախատեսված է եղել multiple threaded, CPU-intensive task֊երի լուծման համար։ Tech համայքներում կա [կարծիք](https://news.ycombinator.com/item?id=33351805), որ դրա maintain֊ը դադարել է այն բանից հետո, երբ [Node.js v10](https://nodejs.org/en/blog/release/v10.5.0)֊ում ներմուծվեց worker_threads֊ի գաղափարը, որը ինչ֊որ առումով "իմաստազրկեց" տվյալ պահին առանց այն էլ դեռ լայն տարածում չգտած runtime֊ի օգտավետությունը։ Համենայն դեպս արժի է իմանալ, որ Napa.js֊ում multi-thread գաղափարը կիրառվել է իրարից իզոլացված V8 engine֊ների ղեկավարմամբ՝ յուրաքանչյուրը զբաղեցնելով համապատասխան thread֊ը։ Այն հնարավոր է եղել կիրառել և որպես Node.js-ի մոդուլ, և որպես առանձին standalone սերվիս՝ չլինելով Node dependency։
napa.js֊ի հետ կապված նյութ կա [այստեղ](https://medium.com/codeinsights/starting-parallel-programming-in-node-js-with-napa-js-ef80e20ec6c2)։

Ստորև այն օգտակար ռեսուրսների  հղումներ են, որոնք ուղղակիորեն տեղ չգտան այս մասում, բայց կապված են տվյալ թեմայի հետ․

- [Node vs Deno vs Bun: The search for a better JavaScript runtime environment](https://patagonian.com/blog/node-vs-deno-vs-bun-javascript-runtime/) [Oct, 2022]
- [Choosing a JavaScript Runtime for 2023: Node vs. Bun vs. Deno](https://softwaremill.com/choosing-a-javascript-runtime-for-2023-node-vs-bun-vs-deno/) [Mar, 2023]
- [Bun vs. Node.js](https://refine.dev/blog/bun-js-vs-node/#prerequisites) [Jun, 2023]
- [What is Deno?: Deno vs Node.js](https://www.knowledgehut.com/blog/web-development/what-is-deno-difference-between-deno-nodejs) [Jul, 2023]

NGNTJS հոդվածների շարքի այս մասում քննարկեցինք JavaScript լեզվի, engine֊ի, runtime֊ի և environment֊ի մասին։ Ծանոթացանք Node.js֊ի ալտերնատիվների հետ, ինչպիսիք են Deno և Bun JavaScript runtime/environment֊ները, և ոչ միայն։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
