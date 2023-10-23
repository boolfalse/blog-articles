
## Browser֊ների մասին; JavaScript֊ի ստեղծումը (մաս 1/7)

### NGNTJS - մաս 1

---

NGNTJS հոդվածաշարի նախաբանում ([մաս 0](https://medium.com/@boolfalse/not-great-not-terrible-javascript-ce7b3d97fd0b)) ներկայացված է այս հոդվածաշարի պատրաստման, բովանդակության և վերնագրի ընտրման մասին։
Այս մասում կներկայացնենք browser֊ների պատմությունը, JavaScript֊ի առաջացման հիմքերը, ECMA ստանդարտի առաջացումը։

<img src="https://i.imgur.com/0ThxayE.png" style="width:100%;">
 
[WorldWideWeb․ Նկարում [առաջին browser](https://home.cern/science/computing/birth-web)֊ի ժամանակակից [վիզուալն է (վերահավաքված 2019֊ին)](https://worldwideweb.cern.ch/)]

Մինչ այս նյութը ամբողջությամբ հավաքելը կարևոր էի համարում սկսել հենց պատմական ասպեկտից՝ browser֊ների պատմությամբ, JavaScript֊ի ստեղծման անհրաժեշտության և ժամանակագրության մասին։ Բայց [Douglas Crockford֊ի "How JavaScript works"](https://www.crockford.com/books.html) գրքից մի հատված փոխեց որոշումս։

<img src="https://i.imgur.com/0i3JEu4.png" style="width:100%;">
 
[Նկարված է [How JavaScript Works](https://www.howjavascriptworks.com/) գրքից]

Կարծում եմ՝ որքան էլ սա լինի հումորային, հենց սկզբից արժի լինել ինքնաքննադատական և դիտել stand-up ոճի տարրերով համեմված այս կարճ [տեսանյութը](https://www.destroyallsoftware.com/talks/wat)։
WAT! talk֊ի մասին կարող եք գտնել այլ [հոդված](https://medium.com/dailyjs/the-why-behind-the-wat-an-explanation-of-javascripts-weird-type-system-83b92879a8db) ևս։

Կարծում եմ՝ WAT! տեսանյութը դիտելուց հետո, մինչ JavaScript֊ին անցնելը, կարելի է սկսել browser֊ների մասին հակիրճ պատմությամբ, քանի որ ի սկզբանե JavaScript֊ը ստեղծվել է որպես browser֊ների հետ աշխատելու միջոց։ Մինչ JavaScript֊ի առաջ գալը, ինժեներները մտածում էին որպես browser լեզու կիրառել [scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)), ինչպես հետագայում [նշում է հեղինակը](https://youtu.be/qKJP93dWn40?t=32)։ Բայց պատմությունը դասավորվեց այնպես ինչպես դասավորվեց։
[Netscape](https://en.wikipedia.org/wiki/Netscape) ընկերությունը, որտեղ պետք է ստեղծվեր JavaScript֊ը, ուներ այնպիսի խոշոր մրցակիցներ, ինչպիսին էր Microsoft֊ը։ Ընդհանրապես և հատկապես 1994–1995 թվականներին ընկերության առաջ դրված էր խնդիր՝ պահպանելու և ավելացնելու Netscape browser֊ի օգտատերերի քանակը։ Browser֊ների արագ ձևափոխվող  ժամանակներում անհրաժեշտ էր հարմարավետ միջավայր։ Եվ որպես խնդրի արագ լուծում, Netscape֊ում ստեղծվում է JavaScript լեզվի նախնական տարբերակը։
Ինչպես [նշում է հեղինակը](https://youtu.be/krB0enBeSiE?t=2362)՝ JavaScript֊ի նախնական տարբերակը պետք է պատրաստվեր հնարավորինս շուտ, և սկզբնական տարբերակում այն ուներ շատ խոցելիություններ, չհաշված այն որ նույնիսկ չկար [garbage collector](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management), որը հաճախ բերում էր հիշողության հետ կապված խնդիրների (running out of memory): Լեզուն ներկայացնելուց հետո այն արժանանում է տարբեր քննադատությունների՝ թե ինչպես կարելի է markup լեզվի արանքում գրել սկրիպտ, կամ որ ավելին՝ այն inject է արվում որպես inline comment:
Սակայն ժամանակի ընթացքում այն մշակվում է և դառնում օգտագործելի նույնիսկ մրցակից ընկերությունների կողմից։

Եվ այսպես․

- 1990֊ին թողարկվեց առաջին web browser-ը՝ [WorldWideWeb](https://en.wikipedia.org/wiki/History_of_the_web_browser), Tim Berners-Lee֊ի կողմից (ինչպես նաև առաջին web server֊ը), որը հետագայում վերանվանվեց [Nexus](https://first-website.web.cern.ch/first-website/node/45.html):
- 1993֊ին թողարկվեց web browser֊ը՝ [Mosaic](https://timeline.web.cern.ch/first-pre-release-mosaic-browser)֊ը:
- 1994֊ին Mosaic֊ի աշխատակիցները հիմնում են նոր ընկերություն՝ Netscape, և այդ ընկերության անվան տակ թողարկում նոր web browser, որը կոչվում էր [Netscape Navigator](https://en.wikipedia.org/wiki/Netscape_Navigator): Հիմնականում սա համարվում է հենց առաջին լավ արդյունավետությամբ աշխատող browser֊ը, քանի որ այն դարձավ բավականին հայտնի։ Այս փուլում web browser-ները դեռ չունեին դինամիկություն և ինտերակտիվ չէին։ Սա հանդիսանում է պատճառ՝ ստեղծելու մի սկրիպտային լեզու, որը կլուծեր այդ խնդիրը։
- 1995֊ին Netscape ընկերությունը թողարկում է նոր սկրիպտային լեզու՝ [JavaScript](https://www.javascript.com/)-ը։ Այս մասին կա պատմություն, որ ընկերության աշխատակից [Brendan Eich](https://twitter.com/BrendanEich)֊ը առաջին վերսիան թողարկում է ընդամենը 10 օրում:

<img src="https://i.imgur.com/qH5kNsY.jpg" style="width:100%;">
 
[[Brendan Eich: JavaScript, Firefox, Mozilla, and Brave | Lex Fridman Podcast #160](https://youtu.be/krB0enBeSiE?t=2176)]

- Նույն 1995֊ին, բայց Netscape֊ից հետո, Microsft ընկերությունը թողարկում է իր web browser֊ը՝ [Internet Explorer](https://en.wikipedia.org/wiki/Internet_Explorer_version_history#Microsoft_Internet_Explorer_1.x)։ Սա առաջ է բերում ընկերությունների միջև կոնկուրենցիայի։ Այդ ժամանակ Microsoft-ը կարիք ուներ ունենալու JavaScript֊ին համանման գործիքի։
- 1996֊ին Microsoft֊ը (ըստ որոշ աղբյուրների՝ օգտագործելով reverse-engineering) թողարկում է JScript ինտերպրետատորը, որը JavaScript֊ի անալոգն էր Internet Explorer֊ի համար։

Սակայն այս ժամանակներում դեռևս չկային ստանդարտիզացիայի կանոններ և մեխանիզմներ, որոնք կվերահսկեին JavaScript֊ի և JScript֊ի զարգացումները։ Եվ հաշվի առնելով այն, որ այդ պահին Netscape֊ը և Microsoft֊ը (web browser֊ի շուկայում հիմնական խաղացողները) develop էին անում իրենց ինտերպրետատորները և այլ գործիքները միմյանցից անկախ, դա գնալով ավելի շատ էր բերում հակասությունների, որի արդյունքում մի browser-ով աշխատող էջը նույն ձևով չէր աշխատում մյուս browser֊ի վրա։
1996֊ի վերջում Netscape֊ը հայտարարում է [ECMA (European Computer Manufacturers Association)](https://www.ecma-international.org/about-ecma/history/) գլոբալ ստանդարտիզացիայի մասին, որի մեծ մասը լինում է հետևելը web browser֊ների սկրիպտային լեզուների ստանտարտ կանոններին։ Այսինքն՝ առաջանում է մի ընդհանուր տեսականորեն օֆիցիալ լեզու ECMAScript, որը ունի որոշակի ստանդարտներ, և որին հետևելով մյուս ընկերությունները կհամապատասխանեցնեն իրենց սկրիպտային լեզուները։ Այսպիսով՝ JavaScript֊ը դառնում է ECMA ստանդարտների հիման վրա գործնական սկրիպտային լեզու, մինչդեռ ECMAScript֊ը կարելի է համարել տեսական սկրիպտային լեզու։ Ապահովելու համար տարբեր խոշոր ընկերությունների կողմից զարգացող լեզվի բալանսավորումը՝ ձևավորվում  է միջազգային տեխնիկական համայնք, իսկ պաշտոնապես՝ բաց կոմիտե (Technical Commitee) [TC-39](https://tc39.es/), որը կհետևի [առաջարկություններին](https://github.com/tc39/proposals) և կթարմացնի ընթացիկ [language specification֊ները](https://tc39.es/ecma262/)։
Տարիների ընթացքում JavaScript֊ի զարգանալուն զուգընթաց develop է արվում նաև ECMA ստանդարտի [ECMAScript֊ի վերսիաները](https://en.wikipedia.org/wiki/ECMAScript_version_history)՝

- 1997 - ECMAScript 1
- 1998 - ECMAScript 2
- 1999 - ECMAScript 3
- *** ECMAScript 4 չի թողարկվել
- 2009 - ECMAScript 5
- 2015 - ECMAScript 2015 (ES6)
- 2016 - ECMAScript 2016
- … այսպես հիմնականում ամեն տարի մի թողարկում 2015֊ից սկսած

ES6֊ը (ECMAScript 2015) այն է, ինչ մենք անվանում ենք **modern JavaScript**:
Ամփոփելով՝ ECMAScript֊ը scripting language է, որը implement է անում ECMA֊ն ([ECMA-262 language specification](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/))։

Ստորև այն օգտակար ռեսուրսների հղումներ են, որոնք ուղղակիորեն տեղ չգտան այս մասում, բայց կապված են տվյալ թեմայի հետ․

- [The History of Web Browsers](https://www.mozilla.org/en-US/firefox/browsers/browser-history/)

NGNTJS հոդվածների շարքի այս  մասում, հակիրճ ներկայացվեց browser֊ների պատմությունը, JavaScript֊ի առաջացման հիմքերը, ECMA ստանդարտի առաջացումը։
Հաջորդ՝ [մաս 2](https://medium.com/@boolfalse/javascript-engine-%D5%B6%D5%B7%D5%A1%D5%B6%D5%A1%D5%AF%D5%B8%D6%82%D5%A9%D5%B5%D5%B8%D6%82%D5%B6%D5%A8-%D6%87-%D5%AF%D5%A1%D5%BC%D5%B8%D6%82%D6%81%D5%BE%D5%A1%D5%AE%D6%84%D5%A8-d900993ba01b)֊ում, կներկայացնենք JavaScript engine֊ների նշանակությունը և կառուցվածքը։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
