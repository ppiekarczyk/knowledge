# Behind The Scenes of A Browser

There's a lot that goes into actually having something appear on your website. The browser can request multiple forms of content whether it would be HTML, CSS, JS, images, other types of content, etc.

> Browsers like Chrome make `predictive optimizations` by observing network traffic. What it'll do is build a predictive model in order to improve performance.
> What that means is the second you start looking for a certain URL, like Facebook, Chrome will dispatch a speculative DNS lookup and even start the TCP handshake before you hit `enter`

While browsers will have different engines that handle rendering differently, there are simlarities. They will have a User Interface, Browser Engine, Rendering Engine, Network Engine, Javascript Engine and the UI backend.

- `User Interface` - What the User sees. Includes Address Bar, Search, Bookmarks, etc.
- `Browser Engines` - Middleman between user interface and the rendering engine
- `Rendering Engine` - Takes care of the page rendering process
- `Network Engine` - Handles network requests like HTTP and other requests
- `UI Backend` - draws basic widgets and exposes generic interface that is not platform specific
- `Javascript Interpreter` - Parses and executes JS
- `Data Storage` - Persistence layer to store stuff like cookies and now things like `localStorage`, `IndexedDB`, `WebSQL` and `FileSystem`

Once you initiate the TCP handshake with the browser, the network engine will request 8kb chunks at a time. All modern browsers are multi-threaded, which means they have the abililty to download in paralell, usually around 6 streams but there are ways an individual can increase this limit.

The main steps that take place in the process from the request to rendering the page on your browser:

1. Parse HTML to construct DOM tree
2. Render Tree construction
3. Layout of Render Tree
4. Paint Render tree

As the network is transferring over data, a parser will start running through the HTML. This is an interactive proces, meaning that a web page will start displaying content on the page even before it's completely finished transferring. There are multiple reasons for why it was designed this way:

- User Experience - the faster the user starts seeing something on the screen, the better
- Changes in HTML - there are a variety of factors that can change the HTML structure - javascript being one of them.

The `parser` will ask the `lexer` for a `token` and if its valid, it will append it to the `parse tree`. If it's not valid, that means there are errors in the process and the document becomes invalidated. There are many reasons why a document can become invalidated, mostly because of human error.

The browser does a great job of handling these errors as well, which we will discuss in further detail below.

`Parsing` - The process of converting human readable code into something the computer can use.

`Lexical Analysis` - Breaks input into `tokens`, the building blocks of the specific ruleset
`Syntax Analysis` - The process of applying those rules

`Lexer` - Knows how to stripe unwanted information like whitespace and line breaks
`Parser` - Constructs `parse tree` based on the rules provided

There are different kinds of parsers.

`Top Down` - High level overview approach that looks for a match
`Bottom Up` - Processes input after input and transforms them into syntax rules until all rules are met

`Webkit` uses `Flex` for the Lexer and `Bison` for the Parser

There are huge differences between parsing HTML and everything else. The reason is because humans make enormous mistakes. CSS and JS use context free grammar. The human is an example of what is NOT a context free grammar and neither is HTML. HTML has soft syntax which means not all elements need to close, elements can be created on the fly (but not necessarily allowed).

HTML needs to be more forgiving me because of its need to render out the page error-free (in terms of what the user sees). Browsers have very advanced error handling techniques that will close unclosed brackets, move elements that aren't syntatically correct and default made-up elements with real ones. Eg, `<myblock>` would become a normal `<div>`

`Parse Tree` - Collection of `DOM` elements and nodes
`DOM` - Document Object Model. Objective representation and interface of HTML representation
`Document` - root of Parse Tree
`Context-Free Grammar` - specific rules and follow a specific format.

HTML can't use a top down or bottom up parser because of potentially unknown factors. Document.write() can be called once javascript is executed that could totally void what previously took place. The HTML has the ability to change on the fly and that's how the system is meant to work.

The tokenization of HTMl elements is the lexical analysis.

What happens is that the second before the browser starts parsing the HTML it enters a `data state`. As soon as it sees a `<` it will enter a `TAG OPEN STATE` and individual [a-z] characters are parsed. The browser is now looking for a specific element like `<html>`. It's in a `TAG NAME STATE` until it reaches the final `>`  As soon as `<HTML>` is created, the browser is back in data state until it reaches another `<` to start the process over again. Once the browser hits a `/` it will enter a `TAG CLOSE STATE` to close out the tag

## CSS Specificity

CSS Specificity determines which rules are applied by the browser. Defined By [http://www.w3.org/TR/css3-selectors/#specificity](W3 CSS).

Set of rules that make up specificity:

- 1 if declaration is from `style` attribute rather than a rule with a selector, 0 otherwise (a)
- # of ID attributes on selector (b)
- # of other attributes and psuedo-classes in the selector (c)
- # of elelemtn names and psuedo elements in selector

Here are some examples:

Ruleset | inline styles | IDs | classes, attributes, psuedo classes | elements, psuedo elements

| rule                | a | b | c | d |
|---------------------|---|---|---|---|
| *                   | 0 | 0 | 0 | 0 |
| p                   | 0 | 0 | 0 | 1 |
| p:last-child        | 0 | 0 | 0 | 2 |
| ul li               | 0 | 0 | 0 | 2 |
| p + p               | 0 | 0 | 0 | 2 |
| #form               | 0 | 1 | 0 | 0 |
| .red                | 0 | 0 | 1 | 0 |
| p.red.large         | 0 | 0 | 2 | 1 |
| style="color:black" | 1 | 0 | 0 | 0 |

Some of you may wonder about `!important` but that is not something you should be using. `!important` does override CSS but it should only be used on foreign css for example. Look for a way to handle the issue using specificity instead.


## Resources
- [http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)
- [http://www.aosabook.org/en/posa/high-performance-networking-in-chrome.html](http://www.aosabook.org/en/posa/high-performance-networking-in-chrome.html)
- [http://www.w3.org/TR/css3-selectors/#specificity](http://www.w3.org/TR/css3-selectors/#specificity)
- [http://www.smashingmagazine.com/2007/07/27/css-specificity-things-you-should-know/](http://www.smashingmagazine.com/2007/07/27/css-specificity-things-you-should-know/)
