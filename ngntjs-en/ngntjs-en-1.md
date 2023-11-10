
## Browsers; JavaScript Creation (part 1/7)

### NGNTJS - part 1

---

The introduction to the NGNTJS article series ([part 0](https://medium.com/@boolfalse/not-great-not-terrible-javascript-part-0-7-c33d667a9c13)) describes the preparation, content, and title selection of this article series.
In this part, we will briefly present the history of browsers, the foundations of the emergence of JavaScript, the emergence of the ECMA standard.

<img src="https://i.imgur.com/0ThxayE.png" style="width:100%;">

[WorldWideWeb: [The first browser](https://home.cern/science/computing/birth-web)'s visual [(rebuilt in 2019)](https://worldwideweb.cern.ch/)]

Before entirety collecting this material, I considered it important to start from the historical aspect itself: the history of browsers, the need for the creation of JavaScript, and the chronology. But a passage from [Douglas Crockford's "How JavaScript works"](https://www.crockford.com/books.html) changed my mind.

<img src="https://i.imgur.com/0i3JEu4.png" style="width:100%;">

[Image from the book [How JavaScript Works](https://www.howjavascriptworks.com/)]

I think as humorous as this may be, it's worth being self-critical from the beginning and watching this short [video](https://www.destroyallsoftware.com/talks/wat) flavored with stand-up style elements. I highly recommend to watch that before continuing reading this.
You can find another [article](https://medium.com/dailyjs/the-why-behind-the-wat-an-explanation-of-javascripts-weird-type-system-83b92879a8db) about that video.

So, before moving on to JavaScript, let's start with a brief history of browsers, since JavaScript was originally created as a way to work with browsers. Before the creation of JavaScript, engineers considered using [scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) as a browser language, as the author later [notes](https://youtu.be/qKJP93dWn40?t=32). But the story was arranged as it was arranged.
[Netscape](https://en.wikipedia.org/wiki/Netscape), the company where JavaScript was to be created, had such big competitors as Microsoft. In general and especially in 1994-1995, the company was faced with the task of maintaining and increasing the number of Netscape browser users. In the rapidly changing times of browsers, a comfortable environment was needed. And as a quick solution to the problem, Netscape creates a preliminary version of the JavaScript language.
As the author [notes](https://youtu.be/krB0enBeSiE?t=2362), the initial version of JavaScript had to be prepared as soon as possible, and in the initial version it had many vulnerabilities, not to mention that there was not even a [garbage collector](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management), which often led to memory problems (running out of memory). After the language was introduced, it received various criticisms, such as how to write a script in between the markup language, or even more, it is injected as an inline comment.
However, over time it is developed and becomes usable even by competing companies.

And so:

- In 1990, the first web browser, [WorldWideWeb](https://en.wikipedia.org/wiki/History_of_the_web_browser), was released by Tim Berners-Lee (as well as the first web server), later renamed [Nexus](https://first-website.web.cern.ch/first-website/node/45.html).
- In 1993, the web browser [Mosaic](https://timeline.web.cern.ch/first-pre-release-mosaic-browser) was released.
- In 1994, Mosaic employees founded a new company, Netscape, and under that company released a new web browser called [Netscape Navigator](https://en.wikipedia.org/wiki/Netscape_Navigator). Basically, this is considered to be the very first high-performance browser, because it became quite popular. At this point of time, web browsers still lacked dynamics and were not interactive. This is the reason to create a scripting language that would solve this problem.
- In 1995, Netscape released a new scripting language, [JavaScript](https://www.javascript.com/). There is a story about this company employee [Brendan Eich](https://twitter.com/BrendanEich) releasing the first version in just 10 days.

<img src="https://i.imgur.com/qH5kNsY.jpg" style="width:100%;">
 
[[Brendan Eich: JavaScript, Firefox, Mozilla, and Brave | Lex Fridman Podcast #160](https://youtu.be/krB0enBeSiE?t=2176)]

- In the same year 1995, but after Netscape, Microsoft released its web browser, [Internet Explorer](https://en.wikipedia.org/wiki/Internet_Explorer_version_history#Microsoft_Internet_Explorer_1.x). This leads to competition between companies. At that time, Microsoft needed a JavaScript-like tool.
- In 1996, Microsoft (according to some sources using reverse-engineering) released the JScript interpreter, which was the JavaScript analogue for Internet Explorer.

However, at this time there were still no standardization rules and mechanisms to control the development of JavaScript and JScript. At that time the major players in the web browser market Netscape and Microsoft were developing their own interpreters and other tools independently, this led to more and more conflicts, resulting in a page that worked in one browser and not working the same way in another browser.
In late 1996, Netscape announced [ECMA (European Computer Manufacturers Association)](https://www.ecma-international.org/about-ecma/history/) global standardization, much of which was to follow standard rules for web browser scripting languages. That means, a common theoretically official language ECMAScript is emerging, which has certain standards, and following which other companies will align their scripting languages. Thus, JavaScript becomes a practical scripting language based on ECMA standards, while ECMAScript can be considered a theoretical scripting language. To ensure the balancing of the developing language by different large companies, an international technical community is formed, and officially an open committee (Technical Committee) [TC-39](https://tc39.es/), which will follow [proposals](https://github.com/tc39/proposals) and update the current [language specifications](https://tc39.es/ecma262/).
Over the years, along with the development of JavaScript, the ECMA standard [Versions of ECMAScript](https://en.wikipedia.org/wiki/ECMAScript_version_history) has also been developed:

- 1997 - ECMAScript 1
- 1998 - ECMAScript 2
- 1999 - ECMAScript 3
- *** ECMAScript 4 has not been released
- 2009 - ECMAScript 5
- 2015 - ECMAScript 2015 (ES6)
- 2016 - ECMAScript 2016
- … so basically one release every year since 2015

ES6 (or ECMAScript2015) is what we call **modern JavaScript**.
In summary, ECMAScript is a scripting language that implements ECMA ([ECMA-262 language specification](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/)).

Below are links to useful resources that did not appear directly in this section, but are related to this topic:

- [The History of Web Browsers](https://www.mozilla.org/en-US/firefox/browsers/browser-history/)

In this part of the NGNTJS article series were briefly presented the history of browsers, the origins of JavaScript, and the emergence of the ECMA standard.
Next, in [part 2](https://medium.com/@boolfalse/javascript-engine-meaning-and-structure-7d904697cd97), we will present the meaning and structure of JavaScript engines.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
