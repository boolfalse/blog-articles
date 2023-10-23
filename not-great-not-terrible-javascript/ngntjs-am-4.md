
## libUV, event-loop ծանոթություն (մաս 4/7)

### NGNTJS - մաս 4

---

NGNTJS հոդվածաշարի նախորդ [մաս 3](https://medium.com/@boolfalse/execution-context-%D5%BF%D5%BE%D5%B5%D5%A1%D5%AC%D5%B6%D5%A5%D6%80%D5%AB-%D5%BA%D5%A1%D5%B0%D5%BA%D5%A1%D5%B6%D5%B8%D6%82%D5%B4-4bc4fc2ebd4c)֊ում խոսել ենք Execution Context֊ի, creation (ինիցիալիզացիայի) և execution (աշխատեցման) փուլերի, Hoisting֊ի, պրիմիտիվների և հղումային տվյալների (7 primitives + 3 referenced data-types) պահպանման մասին։
Այս մասում կդիտարկենք libUV գրադարանը, ինչպես նաև կլինի ծանոթություն event-loop մեխանիզմի աշխատանքի հետ։

<img src="https://i.imgur.com/2YlhS1x.png" style="width:100%;">
 
[Նկարը վերցված է հետևյալ [NodeCongress23 talk](https://youtu.be/aR_zxqoeBpQ?t=339)֊ից]

Այժմ մենք գիտենք, որ JavaScript֊ը single-threaded լեզու է, որտեղ thread֊ում "բացվում է" execution context (Global EC և անհրաժեշտության դեպքում Function EC֊ներ), որն իր primitive֊ները և referenced տվյալները hoist է անում համապատասխանաբար stack֊ում և heap֊ում։
Բացի այդ, մենք գիտենք նաև որ JavaScript֊ը synchronous լեզու է, այսինքն՝ execution֊ը կատարվում է հերթով հրամաններով անցնելով, յուրաքանչյուր հրամանի կատարումը block լինելով։

Սակայն, ինչպես գիտենք՝ web browser֊ում JavaScript engine֊ից բացի (engine֊ը այն մասն է, որը երբեմն նույնացվում է լեզվի հետ) կան նաև այլ բաղկացուցիչ մասեր, որոնք ապահովում են asynchrounous գործողությունների իրականացումը։
JavaScript֊ի և այդ async ապահովող մասի միջև կան որոշ "կամուրջներ" (Web APIs), որպեսզի հնարավոր լինի մեկից մյուսը տվյալներ փոխանցել՝ որոշակի async գործողություններ կատարելու համար:
Հաճախ այդ ասինխրոն գործողությունների իրականացման համար կիրառվող հրամանները շփոթմունքի տեղիք են տալիս, և երբեմն ոմանք կարծում են թե դրանք JavaScript֊ի մասն են, այնինչ դրանք Web API֊ներ են, որոնք ներդրված են browser֊ում` այդ asynchronous, non-blocking գործողությունների կատարման և պատասխանների ետ ուղարկման համար։

Գոյություն ունեն բավականին շատ Web API-ներ, որոնք օգտագործում են event-loop մեխանիզմը` browser֊ում ասինխրոն գործողություններ վարելու համար: Web API-ների ցանկը մշտապես ընդլայնվում և փոփոխվում է, քանի որ նոր տեխնոլոգիաներ են մշակվում և ընդունվում, այնուամենայնիվ, ստորև նշենք դրանցից տարածներից մի քանիսը․

- setTimeout()
- setInterval()
- XMLHttpRequest (XHR)
- fetch()
- requestAnimationFrame()
- addEventListener()
- geolocation API
- WebSockets
- Web Workers
- IndexedDB
- FileReader API
- WebRTC
- Web Audio API
- Service Worker API
- Notification API
- MediaDevices.getUserMedia()
- Intersection Observer API
- Resize Observer API
- Mutation Observer API
- …

<img src="https://i.imgur.com/5O0ETjD.png" style="width:100%;">
 
[[https://github.com/libuv/libuv#readme](https://github.com/libuv/libuv#readme)]

[**libUV**](https://github.com/libuv/libuv)֊ն (UV = Unicorn Velociraptor) C լեզվով գրված (multi-platform) open source գրադարան է, որը նախատեսված է հիմնականում asynchrounous I/O գործողությունների համար։ Ի սկզբանե այն գրվել է Node.js֊ի համար (օգտագործվում է այլ միջավայրերում ևս), որպեսզի ապահովի async operation֊ները և ապահովի non-blocking execution։ Վերջինս ունենալու համար այստեղ կիրառվում են thread-pool, event-loop և այլ մեխանիզմներ։
Event-loop մեխանիզմը աշխատում է ինչպես browser միջավայրում, այնպես էլ Node.js runtime֊ում, և դրանց աշխատանքը ունի որոշակի ընդհանրություն։
Այնուամենայնիվ Node.js-ում և browser֊ներում (կարելի է որպես օրինակ դիտարկել հենց Chrome browser֊ը, քանի որ ինչպես Node.js֊ում, այստեղ նույնպես ներդրված է V8 engine)  event-loop մեխանիզմների միջև հիմնական տարբերություններից է այն, որ browser֊ներում event-loop֊ը ինտեգրված է նույն thread֊ում, որտեղ կատարվում է նաև browser UI֊ի աշխատանքը, մինչդեռ Node.js-ում event-loop֊ը աշխատում է առանձին thread֊ով։
Ընդհանուր առմամբ՝ թեև կան տարբերություններ Node.js-ի և browser֊ների event-loop մեխանիզմների միջև, այնուամենայնիվ դրանք երկուսն էլ ծառայում են նույն նպատակին՝ թույլ տալով non-blocking, asynchronous օպերացիաների կատարումը:

YouTube֊ում կա հետաքրքիր [conference-talk](https://www.youtube.com/watch?v=8aGhZQkoFbQ), որտեղ խոսվում է event-loop մեխանիզմի մասին browser֊ներում, որտեղ կարելի է օրինակների միջոցով, բավականին պատկերավոր տեսնել և հասկանալ event-loop֊ի  browser֊ական հատվածը։
[Տեսանյութի կոնֆերենսը եղել է դեռևս 2014֊ին, սակայն՝ կարծում եմ այն արժի դետել թեկուզև որպես բավական հաջող ստացված talk]
Որպեսզի այստեղ չստացվի կրկնություն, կամ զուտ թարգմանություն, կարծում եմ՝ արժի այն դիտարկել որպես event-loop թեմայի մի մասը (browser֊ներում)։
Բայց զուտ ժամանակ խնայելու համար էստեղ ունենանք այն հատվածները, որտեղ պատկերավոր ներկայացվում է event-loop մեխանիզմը browser֊ում․

<img src="https://i.imgur.com/bvTCpKn.gif" style="width:100%;">
 
[առանց event-loop֊ի (sync operations), call stack֊ի օգտագործման օրինակ]

<img src="https://i.imgur.com/pGfYNBD.gif" style="width:100%;">
 
[event-loop կիրառման օրինակ՝ setTimeout() timer֊տեսակի Web API միջոցով]

Հեղինակը ներկայացրել է այլ օրինակներ ևս, որոնք կարելի է կիրառել [latentflip/loupe](https://github.com/latentflip/loupe) գործիքի օգնությամբ։ Գործիքին առընչվող ևս մեկ հասցե [CodeSandbox](https://codesandbox.io/s/6lgw1)֊ում։

Այս հոդվածների շարքը գրելիս ուսումնասիրվել են թեմային վերաբերվող տարբեր հեղինակների կողմից պատրաստված նյութեր, որտեղ հանդիպում են event-loop֊ի վերաբերյալ որոշ վիզուալներ, ինչպիսիք են՝

<img src="https://i.imgur.com/oSB2AP3.jpg" style="width:100%;">
 
[Event Loop and the Big Picture - NodeJS Event Loop Part 1](https://blog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810) [Apr, 2017]

<img src="https://i.imgur.com/4cRiGqN.jpg" style="width:100%;">
 
[The JavaScript Event Loop: Explained](https://towardsdev.com/event-loop-in-javascript-672c07618dc9) [Apr, 2021]

<img src="https://i.imgur.com/tSpcWxW.jpg" style="width:100%;">
 
[Event Loop in NodeJS - Visualized](https://medium.com/@mmoshikoo/event-loop-in-nodejs-visualized-235867255e81) [Nov, 2021]

<img src="https://i.imgur.com/LvqWWhm.jpg" style="width:100%;">
 
[What you should know to really understand the Node.js Event Loop](https://medium.com/the-node-js-collection/what-you-should-know-to-really-understand-the-node-js-event-loop-and-its-metrics-c4907b19da4c) [Jul, 2017]

<img src="https://i.imgur.com/7sxaDki.png" style="width:100%;">
 
[Synchronous vs Asynchronous JavaScript - Call Stack, Promises, and More](https://www.freecodecamp.org/news/synchronous-vs-asynchronous-in-javascript/) [Sep, 2021]

<img src="https://i.imgur.com/iziFRGx.png" style="width:100%;">
 
[NodeConf EU | A deep dive into libuv - Saul Ibarra Coretge](https://youtu.be/sGTRmPiXD4Y?t=640) [Nov, 2016]

<img src="https://i.imgur.com/fHp38vG.png" style="width:100%;">
 
[Node.js Tutorial - 42 - Event Loop](https://www.youtube.com/watch?v=L18RHG2DwwA&list=PLC3y8-rFHvwh8shCMHFA5kWxD9PaPwxaY&index=43) [Jan, 2023]

Այսպիսի վիզուալներով նկարներ կարելի է հանդիպել շատ տեղերում։ Չնայած որ առաջին հայացքից սրանք տարբերվում են իրարից, բայց իրականում դրանց տրամաբանությունը նույնն է՝ ցուցադրել event-loop֊ի աշխատանքի ընթացքը։ Որպես ավելի հարմար պատկեր (ըստ սուբյեկտիվ հանգամանքի) հաջորդ մասում մենք կօգտագործենք վերջին վիզուալին նման պատկեր։

event-loop֊ը C լեզվով գրված libUV գրադարանի feature֊ներից մեկն է, բայց ոչ միակը։ Սա մեխանիզմ է, որի խնդիրն է ղեկավարել JavaScript asynchronous օպերացիաները։ Ակնհայտ է, որ վերջին մեկնաբանությունը բավականին ընդհանրական է, և դրանով դժվար է խորը պատկերացում կազմել դրա աշխատանքի մասին։

Կարծում եմ՝ այս հատվածում, չհեռանալով բուն թեմայից, կարելի է նաև նշել, որ բացի libUV֊ից՝ ժամանակակից tech աշխարհում կան նաև event-loop֊ի գափափարն իմպլեմենտացնող այլ ծրագրային ապահովումներ ևս։ Հենց նույն Node.js֊ի հեղինակ [Ryan Dahl](https://tinyclouds.org/)֊ի կողմից ստեղծված մյուս հայտնի պրոյեկտը՝ [Deno](https://deno.land/)֊ն, C֊ով գրված libUV֊ի փոխարեն օգտագործում է այլ asynchronous event-driven platform, Rust֊ով գրված [Tokio](https://tokio.rs/)։

Deno֊ի, ինչպես նաև այլ ժամանակակից ալտերնատիվների մասին ավելի մանրամասն դեռ կանդրադառնանք NGNTJS հոդվածաշարի [մաս 7](https://medium.com/@boolfalse/javascript-%D5%AC%D5%A5%D5%A6%D5%B8%D6%82-runtime-engine-%D5%B4%D5%AB%D5%BB%D5%A1%D5%BE%D5%A1%D5%B5%D6%80-node-js-%D5%B4%D6%80%D6%81%D5%A1%D5%AF%D5%AB%D6%81%D5%B6%D5%A5%D6%80-deno-bun-82cf3222e94b)֊ում, իսկ այս մասում սահմանափակվենք այսքանով։ Համարելով, որ ունենք որոշակի ներածական նյութ even-loop֊ի մասին browser միջավայրում, հետագա մասերում կուսումնասիրենք event-loop մեխանիզմը արդեն Node.js runtime֊ում (V8 engine + libUV)։

NGNTJS հոդվածների շարքի այս մասում մենք դիտարկեցինք libUV գրադարանը, ծանոթացանք event-loop մեխանիզմի աշխատանքի հետ։
Հաջորդ՝ [մաս 5](https://medium.com/@boolfalse/thread-pool-%D5%A1%D5%B6%D5%A1%D5%AC%D5%B8%D5%A3-%D5%BC%D5%A5%D5%A1%D5%AC-%D5%AF%D5%B5%D5%A1%D5%B6%D6%84%D5%AB%D6%81-8fb6f5f4bfd7)֊ում կլինի ծանոթություն thread pool գաղափարի հետ, և դրա գործնական օրինակը կպրոյեկտենք ռեալ կյանքից վերցված դեպքի վրա։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
