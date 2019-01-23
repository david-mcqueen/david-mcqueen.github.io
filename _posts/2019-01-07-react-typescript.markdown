---
layout: post
title:  "React(ing) to Typescript"
date:   2019-01-23 18:59:00 +0000
categories: blog posts
---

# How do you React?

I'm not a _React_ developer, let me start off by saying that, just so we're all on the same page and you don't think that I'm trying to hoodwink y'all. I've played around with it on a personal project (which is still under wraps, although it's the next best thing since sliced bread... you heard it here first) and have used other Javascript libaries to have a good enough understanding on whats what - most languages are just semantics. However my credentials are of absolutly no importance here, because we're not really going to be looking at what React is, how it works, or why its used by a handful of people.

No No. Today (or evening, wherever you are - I don't want to be time-ist) I'm going to be talking about converting a _Create React App_ project, to use Typescript files & syntax. _Wooohhhooooo_ (in the sound of a mid 90's TV game show audience seeing the prizes up for grabs - not the sound of a Woo Girl...).

Prior to this weekend I was using React _"out of the box"_ with whatever [Create React App](https://reactjs.org/docs/create-a-new-react-app.html) decided was best for me. After all, that was the setup which the React team themselves defined. And it worked. Why wouldn't it? Literally within minutes I was up and running, writing some React code. Shiny. I don't know if I missed some config flag for _Create React App_ which would have made it TypeScript from the start, but that doesn't really matter any more as it is currently Javascript, and I want Typescript.


# More Types

The thing which I've always found to be a bit of a bugbear with javascript code is that there is no enforced structure to it. You can literally write anything and the first that you'll know about any bugs is when you get a runtime error for spelling your variable "_numbreOfKittens_" instead of the intended "_numberOfKittens_". Don't get me wrong, I do like the ability to just throw together some code for a PoC or quick script, but for any serious projects I like structure, more akin to C#. 

This is where [Typescript](https://www.typescriptlang.org/) fits nicely. But does it even play nicely with React? And what about the project I'm already in the middle of, can I make that use Typescript?

Well the answer, as you might expect is no. Sorry. I just wanted to see how long I could keep you reading for..... Oh you got me! Of course the answer is Yes! Woooo (Ok, that one was a Woo Girl).


# Make my React app use Typescript
As it happens, its incredibly easy. Again, there is quite a bit of hand holding by the team who created Create React App, [they even made a handy guide](https://facebook.github.io/create-react-app/docs/adding-typescript) which I want to expand on a little, as I had a couple more steps than are highlighted there.


Lets get to it!

#### Numero Uno

Ensure you have the correct version of _react-scripts_, which needs to be 2.1.0 or higher. Have a gander at your package.json file, at the React project root and find the entry for _react-scripts_, and change it to something >= 2.1.0, for examples;

```javascript 
    ...
    "react-scripts": "2.1.3",
    ...
```

#### Numero Due

Install typescript and corresponding type files;

```
npm install --save typescript @types/node @types/react @types/react-dom @types/jest
```

Because Typescript is strongly typed, it needs to have knowledge of the structure for each object - including 3rd party objects. This is a fairly trivial process though, with most popular 3rd party libs making the typing files available. The build console will throw an error if it doesn't know about a 3rd party library, and will generally give you a hint of what to run to install it;

```javascript
Type error: Could not find a declaration file for module 'react-infinite-scroller'. './node_modules/react-infinite-scroller/index.js' implicitly has an 'any' type.
  Try `npm install @types/react-infinite-scroller` if it exists or add a new declaration (.d.ts) file containing `declare module 'react-infinite-scroller';`  TS7016
```

This error is because I don't have the Type declaration for _react-infinite-scroll_, running the hinted command gets the d.ts file and pops it in _'./node_modules/@types'_.

You package.json file will contain an entry for each of the dependencies which you need typings for.

```javascript
    ...
    "@types/is-url": "^1.2.28",
    "@types/jest": "^23.3.11",
    "@types/node": "^10.12.18",
    "@types/react": "^16.7.18",
    "@types/react-dom": "^16.0.11",
    "@types/react-infinite-scroller": "^1.2.0",
    "@types/react-router-dom": "^4.3.1",
    ...
```

It's important to note that this is alongside the entry for the dependency itself, for example _react-infinite-scroller_ will have two entries - one for the package, and one for the type declaration;

```javascript
    ...
    "react-infinite-scroller": "^1.2.2",
    "@types/react-infinite-scroller": "^1.2.0",
    ...
```



#### Numero Tre

Convert all of your project javascript files to be typescript files (_.tsx_). Now my project wasn't massive so there were only a handful of folders, I just went into each directory and ran the following command;

```
ren *.js *.tsx
```

This changes all of the _.js_ extensions to have the _.tsx_ extension. If you've a bigger project then you may want to tweak this slightly so it recursively drills down into each folder without having to manually go into each one.

#### Numero Quattro

Restart your development server and wait for stuff to blow up.


#### Numero Cinque
You'll likely have multiple build errors, as your previous JS code doesn't conform to the strongly typed nature of Typescript (it's currently the same code, just with a different file extension). It's necessary to go through and port your code; This will involve things such as creating classes and defining your variable types. Depending on the size of your project it may be quite lobourious, however its worth a few hours of pain for many more hours of joy filled strongly typed fun.


#### Numero Sei

Jobs a good un. Your React app should now be fully TypeScript, and life just got oh so much better. Reward yourself with some Limoncello.