
## event-loop֊ը պրակտիկ օրինակներով։ Ընդհանուր պատկեր (մաս 6/7)

### NGNTJS - մաս 6

---

NGNTJS հոդվածաշարի նախորդ [մաս 5](https://medium.com/@boolfalse/thread-pool-%D5%A1%D5%B6%D5%A1%D5%AC%D5%B8%D5%A3-%D5%BC%D5%A5%D5%A1%D5%AC-%D5%AF%D5%B5%D5%A1%D5%B6%D6%84%D5%AB%D6%81-8fb6f5f4bfd7)֊ում ծանոթացել ենք thread pool գաղափարի հետ, և դրա գործնական օրինակը պրոյեկտել ենք ռեալ կյանքից վերցված դեպքի վրա։
Այս մասում կդիտարկենք տարբեր օրինակներ (քայլ առ քայլ), և այդ օրինակների միջոցով կփորձենք հասկանալ թե ինչպես է աշխատում event-loop֊ը գործնականում։

<img src="https://i.imgur.com/Cr2NSPb.png" style="width:100%;">
 
[Նկարը՝ [unsplash.com](https://unsplash.com/photos/UEX8WkrdQgE)]

Մինչ այս հոդվածների շարքը և հատկապես այս մասը պատրաստելը ուսումնասիրվել են բազում նյութեր, որտեղ հիմնականում նկատվում է հետևյալ տրամաբանական հաջորդականությունը՝ սկզբում սահմանվում է թե ինչ է event-loop֊ը, այնուհետև՝ ինչպես է այն աշխատում։
Իհարկե այստեղ ևս միանգամից անցում չկա (նախորդ՝ [մաս 5](https://medium.com/@boolfalse/thread-pool-%D5%A1%D5%B6%D5%A1%D5%AC%D5%B8%D5%A3-%D5%BC%D5%A5%D5%A1%D5%AC-%D5%AF%D5%B5%D5%A1%D5%B6%D6%84%D5%AB%D6%81-8fb6f5f4bfd7)֊ում և [մաս 4](https://medium.com/@boolfalse/libuv-event-loop-%D5%AE%D5%A1%D5%B6%D5%B8%D5%A9%D5%B8%D6%82%D5%A9%D5%B5%D5%B8%D6%82%D5%B6-886772ab3df)֊ում կա որոշակի ծանոթություն libUV֊ի և event-loop֊ի հետ կապված), բայց որպես հեղինակ՝ կարծում եմ որ տվյալ դեպքում արժի սկսել event-loop֊ի ուսումնասիրությունը գործնական օրինակներով (հասկանալով թե ինչպես է այն աշխատում), այնուհետև ավելի մեծ պատկերով կհասկանանք թե ինչ է այն իրականում:

Օրինակ 1․ microtask and timer queues

```javascript
setTimeout(() => console.log("setTimeout 1"), 0);
setTimeout(() => {
  console.log("setTimeout 2");
  process.nextTick(() => console.log("setTimeout 2 nextTick"))
}, 0);
setTimeout(() => console.log("setTimeout 3"), 0);

process.nextTick(() => console.log("nextTick 1"));
process.nextTick(() => {
  console.log("nextTick 2");
  process.nextTick(() => console.log("nextTick 2 nextTick"));
});
process.nextTick(() => console.log("nextTick 3"));

Promise.resolve().then(() => console.log("Promise 1"));
Promise.resolve().then(() => {
  console.log("Promise 2");
  process.nextTick(() => console.log("Promise 2 nextTick"));
});
Promise.resolve().then(() => console.log("Promise 3"));
```

Արդյունքը կլինի հետևյալը՝

```log
nextTick 1
nextTick 2
nextTick 3
nextTick 2 nextTick
Promise 1
Promise 2
Promise 3
Promise 2 nextTick
setTimeout 1
setTimeout 2
setTimeout 3
setTimeout 2 nextTick
```

Օրինակ 2․ I/O queue and timer queue anomaly

```javascript
const fs = require("fs");

setTimeout(() => console.log("setTimeout"), 0);
fs.readFile(__filename, () => console.log("readFile"));
// for (let i = 0; i < 1000000000; i++) {}
```

Արդյունքը կլինի հետևյալը՝

```log
setTimeout
readFile
// անոմալիա
readFile
setTimeout
```

Օրինակ 3․ I/O queue and I/O polling

```javascript
const fs = require("fs");

fs.readFile(__filename, () => {
  console.log("readFile");
});
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));

for (let i = 0; i < 1000000000; i++) {}
```

Արդյունքը կլինի հետևյալը՝

```log
nextTick
Promise
setTimeout
setImmediate
readFile
```

Օրինակ 4․ check queue & timer queue anomaly

```javascript
setTimeout(() => console.log("setTimeout"));
setImmediate(() => console.log("setImmediate"));
```

Արդյունքը կլինի հետևյալը՝

```log
setTimeout
setImmediate
// անոմալիա
setImmediate
setTimeout
```

Օրինակ 5․ Close queue in the end

```javascript
const fs = require("fs");

const readableStream = fs.createReadStream(__filename);
readableStream.on("close", () => {
  console.log("readableStream");
});
readableStream.close();

setImmediate(() => console.log("setImmediate"));
setTimeout(() => console.log("setTimeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
process.nextTick(() => console.log("nextTick"));
```

Արդյունքը կլինի հետևյալը՝

```log
nextTick
Promise
setTimeout
setImmediate
readableStream
```

Օրինակները կարող եք գտնել նաև [GitHub Gist](https://gist.github.com/boolfalse/5f2c7160a6478da492634facc0c416e9)֊ում։

Այս օրինակներում հերթով անցում է կատարվում microtask (nextTick և Promise), timer, I/O (I/O polling), check, close queue֊ներով։

Այժմ քանի որ այստեղ արդեն հասկանում ենք՝ ինչ է իրենից ներկայացնում event-loop մեխանիզմը, և գիտենք աշխատանքի փուլերը, կարելի է նայել Node.js ճարտարապետությանը "մեծ" պատկերով։
Ցանցում "Node.js architecture" բանալի֊բառը կիրառելով կարելի է գտնել բազմաթիվ վիզուալներ, որոնք կազմված են տարբեր հեղինակների կողմից ըստ որոշ պահանջների՝ ներկայացնելու իրենց ուզած այս կամ այն բաղադրիչը կամ աշխատանքի բնույթը։
Ստորև կտեղադրեմ իմ կողմից պատրաստված վիզուալը, որտեղ ներկայացված են որոշ կտորներ, որոնց փոխազդեցություններով պայմանավորված է աշխատանքը Node.js Runtime֊ում։ Այն մակերեսային է, և թերևս միտված է ավելի տրամաբանական շղթային, քան թե առանձին ծրագրային բլոկներին։

<img src="https://i.imgur.com/sHsjTvh.png" style="width:100%;">
 
[libUV֊ի event-loop և մյուս կոմպոնոնտների փոխազդեցությունը Node.js runtime֊ում]

Հակիրճ ներկայացնենք վիզուալում նշված նոր (հոդվածների շարքում դեռևս չմեկնաբանված) տերմինների նշանակությունը․

- **event-demultiplexer** - պատասխանատու է տարբեր I/O աղբյուրների մոնիտորինգի և event-ները ընդունելու համար: Եթե I/O աղբյուրը պահանջում է blocking operation, ապա event-DEMUX-ը տվյալ task֊ը կփոխանցի դեպի C/C++ դաշտ, որտեղ կկատարվի տվյալ task֊ը։ Task֊ը ավարտվելուն պես, C/C++ կողմը կտեղեկացնի event-dispatcher֊ին, որն իր հերթին այն կուղարկի դեպի event-loop֊ի կոնկրետ queue֊ում ավելացնելու:
- **event-dispatcher** - պատասխանատու է event֊ների ղեկավարման՝ ստացման֊ուղարկման համար։
- **kqueue** - BSD-based մեխանիզմ event notification֊ների համար (macOS, FreeBSD, NetBSD)
- **IOCP** (input/output completion ports) - Windows OS ընտանիքի մեխանիզմ event notification֊ների համար
- **event ports** - Solaris-specific մեխանիզմ event notification֊ների համար (Oracle Solaris, OpenSolaris)
- **epoll** - Linux-specific մեխանիզմ event notification֊ների համար

Որպես հավելում, կարելի նշել նաև, որ վերոգրյալից բացի libUV֊ն տրամադրում է օգտագործման համար այլ կարևոր feature֊ներ նույսպես․

- **Asynchronous I/O** - libUV-ն ապահովում է ընդհանուր ինտերֆեյս տարբեր OS֊ներում ասինխրոն I/O գործառնությունների համար:
- **Signal handling** - libUV-ն տրամադրում է API․ OS signal֊ների ղեկավարման համար գրանցվում են signal handler֊ներ, ինչպիսիք են SIGINT-ը կամ SIGTERM-ը:
- **File system operations** - libUV-ն տրամադրում է API․ ֆայլային համակարգի գործողություններ կատարելու համար, ինչպիսիք են file writing֊ը կամ file reading֊ը:
- **Network I/O** - libUV-ն ապահովում է API․ Network I/O operation֊ները կատարելու համար, ինչպիսիք են TCP/UDP-socket֊ների ստեղծումն ու կառավարումը:
- **Child processes management** - libUV-ն տրամադրում է API․ child_process֊ների ստեղծման և ղեկավարման համար։
- ...

Հասնելով այս պահին՝ կարող ենք մեջբերել Reactor Pattern֊ի գաղափարը:

<img src="https://i.imgur.com/WytzyaP.png" style="width:100%;">
 
[Նկարը՝ [Chernobyl](https://www.imdb.com/title/tt7366338/) մինի֊սերիայից]

NGNTJS հոդվածների շարքի այս մասում ուսումնասիրեցինք event-loop մեխանիզմը գործնական օրինակների միջոցով, ծանոթացանք Reactor Pattern֊ի աշխատանքի հետ։
Հաջորդ՝ [մաս 7](https://medium.com/@boolfalse/javascript-%D5%AC%D5%A5%D5%A6%D5%B8%D6%82-runtime-engine-%D5%B4%D5%AB%D5%BB%D5%A1%D5%BE%D5%A1%D5%B5%D6%80-node-js-%D5%B4%D6%80%D6%81%D5%A1%D5%AF%D5%AB%D6%81%D5%B6%D5%A5%D6%80-deno-bun-82cf3222e94b)֊ում կքննարկենք JavaScript լեզվի, engine֊ի, runtime֊ի և environment֊ի (միջավայր) մասին։ Ապա կծանոթանանք ժամանակակից JavaScript runtime/environment֊ներին, ինչպիսիք են Deno֊ն և Bun֊ը, և ոչ միայն։

***

Եթե հավանեցիք այս նյութը, ազատ կարող եք հետևել ինձ այստեղ։ 😇

Ժամանակակից տարբեր տեխնոլոգիաներով աշխատող պրոյեկտներ ուսումնասիրելու համար կարող եք հետևել իմ [GitHub](https://github.com/boolfalse) էջին, որտեղ ես ակտիվորեն հանրայնացնում եմ իմ կատարած աշխատանքի զգալի մասը։

Այլ ինֆոի համար կարող եք այցելել իմ օֆիցիալ կայք` [https://boolfalse.com/](https://boolfalse.com/)
