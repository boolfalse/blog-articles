
## Execution Context, Տվյալների պահպանում (մաս 3/7)

### NGNTJS - մաս 3

---

NGNTJS հոդվածաշարի նախորդ [մաս 2](https://medium.com/@boolfalse/javascript-engine-%D5%B6%D5%B7%D5%A1%D5%B6%D5%A1%D5%AF%D5%B8%D6%82%D5%A9%D5%B5%D5%B8%D6%82%D5%B6%D5%A8-%D6%87-%D5%AF%D5%A1%D5%BC%D5%B8%D6%82%D6%81%D5%BE%D5%A1%D5%AE%D6%84%D5%A8-d900993ba01b)֊ում ներկայացվեց JavaScript engine֊ների նշանակությունը և կառուցվածքը։
Այս մասում կանցնենք JavaScript֊ի աշխատանքի ուսումնասիրմանը։ Կներկայացնենք Execution context֊ը և տվյալների՝ փոփոխականների և ֆունկցիաների (first-class citizens) պահպանումը։ Ընթացում կառնչվենք այնպիսի տերմինների, ինչպիսիք են call stack֊ը, փոփոխականների տիպերը, hoisting֊ը և այլն։

<img src="https://i.imgur.com/rAtZenX.png" style="width:100%;">
 
[Նկարը՝ [unsplash.com](https://unsplash.com/photos/a-room-with-wooden-walls-and-black-curtains-iiFnnqiPhFw)]

JavaScript-ը single-threaded լեզու է, համարելով այստեղ thread֊ը որպես ղեկավարման հրամանների գծային հերթականությամբ (single sequential flow of control)։
JavaScript֊ը նաև synchronous language է, բայց ունի asynchronous հնարավորություններ, որն ստացվում է out-of-the-box։
Thread֊ը որը ստեղծվում է JavaScript կոդի աշխատեցման ժամանակ, կիրառում է Call Stack (Last In First Out սկզբունքով աշխատող հրամանների պահոց) և Memory Heap (հիշողության տարածք array֊ների, object֊ների, function֊ների համար)։

Call Stack-ը դա տվյալների կառուցվածք է (data structure), որում էլեմենտները պահվում են LIFO սկզբունքով։

Javascript code֊ը աշխատեցնելուց ստեղծվում է միջավայր, որտեղ կատարվում է կոդի տրանսֆորմացիան (interpretation֊ը և մնացածը) և execution֊ը։ Այդ միջավայրը կոչվում է Global Execution Context: Նրա մեջ է գտնվում աշխատող ընթացիկ կոդը և այն ամենը ինչ կապված է դրա հետ։ Բացի Global EC֊ից, յուրաքանչյուր կանչված JavaScript ֆունկցիա ունենում է իր EC֊ը՝
Function Execution Context։

Ցանկացած EC ստեղծվելուց (Global EC կամ Function EC) տեղի է ունենում հաջորդական երկու փուլ։

A. **Memory creation phase** (Հիշողության ստեղծման փուլ)

- Ստեղծվում է գլոբալ օբյեկտ։ Օրինակ՝ V8 engine֊ով աշխատող Chrome֊ում գլոբալ օբյեկտը window֊ն է, իսկ Node.js֊ում՝ global֊ը։
- Ստեղծվում է this օբյեկտը և դրան որպես արժեք տրվում է գլոբալ օբյեկտի հղումը։ Օրինակ՝ Chrome֊ում ցանկացած էջում կանչելով this֊ը՝ կստանանք նույն window֊ի օբյեկտը։
- Պահվում են գլոբալ EC֊ում գտնվող բոլոր փոփոխականները (որպես undefined) և ֆունկցիաների հղումները (ողջ ֆունկցիան պահվում է memory heap-ում)։

B. **Execution phase** (Աշխատեցման փուլ)

- Կոդը աշխատեցվում է տող առ տող։
- Տող առ տող աշխատեցման ժամանակ ցանկացած ֆունկցիայի  համար ստեղծվում է համապատասխան EC։

Որպես օրինակ վերցնենք հետևյալ կոդ բլոկը և ուսումնասիրենք Global Execution Context֊ի 2 փուլերը․

```javascript
var x = 2;
var y = 3;
function multiply(a, b) {
  var m = a * b;
  return m;
}
var z = multiply(x, y);
```

Բացատրենք այս օրինակը։
Ինչպես արդեն ասվեց, սկզբում կստեղծվի Global EC (այն "կտեղադրվի" Call Stack֊ում): Creation phase֊ում x փոփոխականի համար կազատվի հիշողության տարածք (memory allocation) և memory heap֊ում որպես արժեք կստանա undefined։
y փոփոխականի համար կազատվի հիշողության տարածք և memory heap֊ում որպես արժեք կստանա undefined։
multiply ֆունկցիայի համար կազատվի հիշողության տարածք և memory heap֊ում որպես արժեք կստանա ողջ ֆունկցիան: Այսինքն՝ համարելով որպես key:value pair, ֆունկցիայի անունը կպահվի որպես հղում memory heap֊ի համապատասպխան ֆունցիայի վրա։
z փոփոխականի համար կազատվի հիշողության տարածք և memory heap֊ում որպես արժեք կստանա undefined։

Երկրորդ (կատարման) փուլում առաջին տողից սկսած x-ը կստանա 2 արժեք, y-ը կստանա 3 արժեք։ Ֆունկցիային հասնելով ոչ մի execution տեղի չի ունենա, իսկ վերջին տողում multiply ֆունկցիան invoke կարվի (կկանչվի)։ Հետևաբար կստեղծվի Function EC:

multiply ֆունկցիայի EC֊ում նմանատիպ անալոգիայով սկզբում տեղի կունենա memory creation phase֊ը՝ a և b, այնուհետև m փոփոխականների համար հիշողություն կտրամադրվի և նրանց կտրվի undefined արժեք։ Հետո տեղի կունենա execution phase֊ը, որտեղ a-ն կստանա 2 արժեք, b֊ն կստանա 3 արժեք, այնուհետև m֊ը կստանա 6 արժեք (2*3): Ֆունկցիան կվերադարձնի արժեքը Function EC֊ից դեպի Global EC և այդ Function EC֊ն կջնջվի Call Stack֊ից։ Վերադառնալով Global EC, z֊ը կստանա վերադարձված 6 արժեքը։ Վերջապես Call Stack֊ից կջնջվի նաև Global EC֊ը և այսպիսով ծրագիրը կավարտվի (չենք խոսում [garbage collector](https://v8.dev/blog/trash-talk)֊ի աշխատանքի մասին)։

Կարելի է վերջինս նկարագրել հետևյալ պատկերով․

<img src="https://i.imgur.com/3Lr0qgJ.gif" style="width:100%;">
 
[JavaScript Execution Context֊ի աշխատանքի օրինակ]

Ժամանակակից web browser֊ներում օգտագործելով debugger֊ը մենք կարող ենք տրված կոդի execution phase֊ին հետևել, քայլ առ քայլ անցնելով կոդի հրամանների վրայով։ Օրինակ Chrome-ում դա կարելի է տեսնել ինչպես պատկերված է ստորին նկարում․

<img src="https://i.imgur.com/sxiYysL.png" style="width:100%;">
 
[Chrome֊ում execution փուլի աշխատանքի օրինակ]

Ինչպես երևում է նկարում նշված հատվածում՝ Call Stack֊ը ներքևում պարունակում է (anonymous) էլեմենտ, որը Global EC֊ն է։ Իսկ multiply ֆունկցիան իր հերթին բացել է նոր Function EC, որը ավելացել է Call Stack֊ում, և նկարում պատկերվածը հենց այն պահն է, երբ thread֊ը աշխատում է ֆունկցիայի execution context֊ի վրա։

Համենայն դեպս Call Stack֊ը ունի սահման, և այդ սահմանված չափը անցնելով կստանանք _Uncaught RangeError_, ինչպես օրինակ ստորին նկարում է․

<img src="https://i.imgur.com/PwFpapL.png" style="width:100%;">
 
[Uncaught RangeError: Maximum call stack exceeded]

Նշենք նաև, որ կարող են լինել դեպքեր, երբ կոդի execution֊ի ժամանակ ունենանք error, այդ դեպքում ավտոմատ տեղի կունենա Call Stack֊ի դատարկում։ Ստորև նկարում ներկայացված է console֊ում call stack֊ի այն պահը, որտեղ գրանցվել է error֊ը․

<img src="https://i.imgur.com/CgvZpKp.png" style="width:100%;">
 
[Uncaught Error - call stack֊ի ավտոմատ մաքրում]

Այժմ, որպեսզի ուսումնասիրենք execution context֊ի յուրահատկությունը այլ օրինակի վրա, կարող ենք ձևափոխել կոդը և դիտարկել հետևյալ դեպքը․

```javascript
console.log(x, y, multiply);
var x = 2;
var y = 3;
var z = multiply(x, y)
function multiply(a, b) {
  var m = a * b;
  return m;
}
```

Ինչպես տեսնում ենք՝ multiply ֆունկցիան կանչվում է նախքան դրա հայտարարումը։ Սա կարող է լինել տարօրինակ, բայց իրականում քանի որ մենք արդեն գիտենք execution context֊ի երկու phase֊երի մասին, ապա կարող ենք հասկանալ որ այս տեսքում խնդիր չի կարող առաջանալ, քանի որ առաջին փուլում initialize են արվում x֊ը, y֊ը և multiply֊ը, ապա դրանից հետո նոր կատարվում է execution փուլը, որտեղ x-ը և y֊ը ունեն undefined արժեք, իսկ multiply ֆունկցիան արդեն initialize է եղել։
Ստացվում է, որ console.log֊ի կտպի հետևյալ արդյունքը․

<img src="https://i.imgur.com/BZrkpMK.png" style="width:100%;">
 
[Նկարված է Chrome֊ի Inspect֊ից]

Այժմ ունենալով պատկերացում Execution Context֊ի մասին, կարելի է խոսել **Hoisting**֊ի մասին։

Ինչպես նկատեցինք՝ օրինակում գրված կոդում x և y փոփոխականները հայտարարված են var֊ով։ Փորձենք ևս մեկ անգամ ձևափոխել կոդը և x և y փոփոխականները հայտարարելիս var֊ի փոխարեն օգտագործենք let.

```javascript
console.log(multiply);
console.log(x, y);
let x = 2;
let y = 3;
var z = multiply(x, y);
function multiply(a, b) {
  var m = a * b;
  return m;
}
```

Այս դեպքում browser֊ը մեզ կվերադարձնի _Uncaught ReferenceError_ (չնայած որ multiply֊ը օգտագործելուց այն խնդիր չի տա), նշելով որ error֊ը գրանցվել է ինչպես արդեն գիտենք <anonymous> կոչվող execution context֊ում, 2 տողի 13 սիմվոլից սկսած․

<img src="https://i.imgur.com/JLWUA5k.png" style="width:100%;">
 
[Նկարված է Chrome֊ի Inspect֊ից]

Փաստորեն մենք ստացանք error հենց երկրորդ փուլում՝ execution phase֊ում JavaScript ինտերպրետատորը փորձում է log անել արգումնետում գրվածը, փորձում է գտնել դեռ առաջին (initialize֊ի) փուլում x անունով գրանցված փոփոխական՝ այդպիսին չկա, հետո փորձում է գտնել x անունով գրանցված ֆունկցիայի հղում (reference)՝ դա էլ չկա, չգտնելով ոչ մի հղում, այն հետ է ուղարկում _ReferenceError_:

Այս պահին դեռևս չխոսալով Hoisting֊ի մասին, կարող ենք հաստատել, որ գոյություն ունի տարբերություն var֊ի և let֊ի միջև՝ առնվազն մինչ այս նկարագրված փաստի առումով։

let֊ը և const֊ը JavaScript լեզվում են՝ սկսած ES6֊ից (ECMAScript 2015): Դրանց և var֊ի տարբերություններից մեկն հենց hoisting֊ն է: var փոփոխականները creation (կամ կարելի է ասել initialization) phase֊ում hoist են արվում Global scope֊ում և արդեն execution phase֊ում հասանելի են լինում՝ արդեն իսկ վերագրված undefined արժեքով:
Չնայած որ շատ աղբյուրներում նշվում է որ let/const փոփոխականները hoist չեն արվում, սակայն իրականում դրանք hoist արվում են, բայց դեպի _Script scope_, և այդ պատճառով execution փուլում հասանելի չեն լինում։ Դրա համար էլ, ի տարբերություն var֊ի` let֊ը և const֊ը block-scoped փոփոխականներ են։
Ստորև այն օրինակն է, երբ մենք execution phase֊ում ունենք միացրած debugger, որը տվյալ տողի վրա (կանգնեցված է և դեռ չի փորձել execute անել այդ տողը) արդեն արձանագրում է որ մեր let֊ով հարտարարված փոփոխականներերը hoist են եղած _Script scope_֊ում: Եվս մեկ անգամ արձանագրենք՝ դրանց հասանելությունը execution phase֊ում փակ է, այդ պատճառով մենք ունենում ենք հետևյալ error֊ը՝
_caught ReferenceError: Cannot access 'x' before initialization at index.html:13:13_

<img src="https://i.imgur.com/ndQwAKa.png" style="width:100%;">
 
[Նկարված է Chrome֊ի Inspect֊ից]

Այսպիսով, մենք ունեցանք որոշակի պատկերացում JavaScript hoisting֊ի մասին՝ նախօրոք իմանալով execution phase֊երի մասին։
Իրականում let/const և var փոփոխականները ունեն շատ ուրիշ տարբերություններ ևս, որոնք մասին կարելի է կարդալ հետևյալ [freeCodeCamp հոդվածում](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/) և [հետևյալ stackoverflow հոսքում](https://stackoverflow.com/a/11444416/7574023):

Այժմ խոսենք այն մասին, թե ինչպես է JavaScript֊ը վարվում տվյալների հետ, և որտեղ են դրանք պահպանվում։

Կախված տվյալ արժեքից՝ primitive կամ reference, այն պահվում է համապատասխանաբար stack֊ում (Call Stack-ի հետ չշփոթել), կամ heap֊ում։

Այնպիսի լեզուներում, համեմատաբար low-level, ինչպիսիք են C֊ն և C++֊ը, developer֊ը ինքն է ընտրում թե ինչ հիշողություն կարելի է allocate անել այս կամ այն օբյեկտի համար։ Համեմատաբար ավելի high-level լեզուներում, ինչպիսին է JavaScript֊ը և Python֊ը, այստեղ ավտոմատ հիշողություն է allocate արվում երբ ստեղծվում է օբյեկտ, այնուհետև ավտոմատ մաքրում է այդ memory֊ն երբ դրանց կարիքը էլ չի լինում։ Դա կատարվում է ներդրված գործիքի միջոցով՝ garbage collector։ Սկզբում նշված լեզուները այսպես ասած non-garbage-collected լեզուներ են, և այդ դեպքում հիշողությունը պետք է կատարվի մանուալ կերպով, հակառակ դեպքում որոշ դեպքերում դա կարող է հանգեցնել memory leak֊ի։ Ամեն դեպքում նույնիսկ ժամանակակից JavaScript֊ում պետք է խուսափել անցանկալի reference֊ների պահպանումից, որը նույպես կարող է բերել memory leak֊ի։

Հայտնի է որ JavaScript ստեղծողը՝ Brendan Eich֊ը իր 10֊օրյա հայտնի սպրինտի ավարտից հետո demo տարբերակում չի ունեցել գրած garbage collector, որը ավելացել է հետագայում։

Ինչպես երևի հայտնի է՝ modern JavaScript֊ը ունի տվյալների 7 պրիմիտիվ տիպեր և 3 հղման տիպեր․

**Primitive Types**: String, Boolean, Number, undefined, Symbol, BigInt
պահվում է stack֊ում և կարելի է access ստանալ հենց իրենով (առանց հղման)

**Reference Types**: Array, Function, Object
արժեքը պահվում է heap֊ում իսկ stack֊ում պահվում է հղումը, որի միջոցով կարելի է access ստանալ

Այժմ, արդեն իմանալով որ կոնկրետ օրինակ ֆունկցիայի դեպքում անձամբ ֆունկցիան պահվում է heap֊ում, իսկ ֆունկցիայի վրա հղումը (որով կարելի է access ունենալ այդ ֆունկցիայի վրա) պահվում է stack-ում, կարող ենք ավելի պատկերավոր հասկանալ նախկինում գրված Hoisting֊ի օրինակը: Այնտեղ նկարագրված էր մի օրինակ, թե ինչպես է ստացվում, որ execution context֊ի initialization/creation phase֊ում ֆունկցիան գրանցվում ու արդեն execution phase֊ում հասանելի լինում, չնայած որ var և let/const փոփոխականները իրենց այլ կերպ էին դրսևորում։

Այժմ դիտարկենք օրինակ, որտեղ ունենք օբյեկտ, ու դրա վրա կատարում ենք որոշակի մանիպուլյացիաներ․

```javascript
// variables case
let a = 2;
let b = a;
a = 5;
console.log(b); // ---------------------- 2
```

```javascript
// objects case
let car = {
  maxSpeed: 200,
  wheelsInch: 17
};
let anotherCar = car;
car.maxSpeed = 240;
console.log(anotherCar.maxSpeed); // ---- 240
anotherCar.wheelsInch = 18;
console.log(car.wheelsInch); // --------- 18
```

Ինչպես տեսնում ենք՝ փոփոխականի դեպքում տվյալը փոխում է հենց իր արժեքը, իսկ օբյեկտը (հղում ունեցող տվյալը) կառավարվում է իր վրա ունեցած հղումով։ Օրինակ, երբ ստեղծում ենք նոր օբյեկտ՝ նրան վերագրելով նախկինում ունեցած օբյեկտը, ապա այս դեպքում deep copy չի արվում (հին օբյեկտի կոնտենտը), այլ ընդամենը ստեղծվում է նոր հղում, որը ունի նույն access֊ը տվյալ օբյեկտի վրա, ինչպես ուներ նախկինում հայտարարվածը։ Ստացվում է, որ car֊ն էլ, anotherCar֊ն էլ երկուսն էլ զուտ heap֊ում եղած օբյեկտի վրա հասանելիություն ունենալու հնարավորություն են։
Նկարագրված օրինակի համար կունենանք հետևյալ պատկերը․

<img src="https://i.imgur.com/IPHaiWJ.png" style="width:100%;">
 
[Նկարում այն սպեցիֆիկ պահն է, որտեղ դեռ վերագրումներ չեն արվել]

Վերջում, միգուցե որպես լրացուցիչ նյութ, ունենանք JavaScript֊ում աշխատանքին նվիրված լեկցիայի [հղում](https://www.youtube.com/watch?v=Aa_OWn03mDo&list=PLEzQf147-uEpvTa1bHDNlxUL2klHUMHJu&index=116&t=256s), որը կարող է լինել հանգամանք, որը թույլ կտա JavaScript ֆունկցիաներին "նայել այլ հայացքով":

<img src="https://i.imgur.com/JmjCmvW.png" style="width:100%;">
 
[[Douglas Crockford](https://www.crockford.com/): Fun With Functions]

Ստորև այն օգտակար ռեսուրսների հղումներ են, որոնք ուղղակիորեն տեղ չգտան այս մասում, բայց կապված են տվյալ թեմայի հետ․

- [JavaScript Execution Context](https://luigicruz.dev/blog/execution-context) [Apr, 2021]
- [JavaScript Execution Context - How JS Works Behind The Scenes](https://www.freecodecamp.org/news/execution-context-how-javascript-works-behind-the-scenes/) [Feb, 2022]

NGNTJS հոդվածների շարքի այս մասում ներկայացվեց JavaScript֊ում Execution Context֊ը, դրա memory creation և execution phase֊երը, նկարագրվեց call stack֊ի աշխատանքը գործնական օրինակների միջողով, ինչպես նաև փոփոխականների և հղումների տիպերը:
Հաջորդ՝ [մաս 4](https://medium.com/@boolfalse/libuv-event-loop-%D5%AE%D5%A1%D5%B6%D5%B8%D5%A9%D5%B8%D6%82%D5%A9%D5%B5%D5%B8%D6%82%D5%B6-886772ab3df)֊ում կդիտարկենք libUV գրադարանը, ինչպես նաև կլինի ծանոթություն event-loop մեխանիզմի աշխատանքի հետ։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
