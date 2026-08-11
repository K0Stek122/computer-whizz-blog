+++
date = '2026-08-11T09:26:15+01:00'
draft = false
title = 'React pt. 1: The Footwork'
+++
[Next Post -->](https://themindpalace.bearblog.dev/react-pt2-more-on-components/)

I have to say straight away that i absolutely hate React, Vue, and Angular. There has to be a better solution for responsive UI/UX design than cramming everything into javascript. Javascript in itself is a mess with library updates every 3 minutes, and so. many. security issues.  
That said, React is the de-facto standard in website design		, so it is useful to master it. And that is what i am planning to do as i read through the book [Learning React](https://www.amazon.co.uk/Learning-React-Kirupa-Chinnathambi/dp/0134546318) and make a full guide for you.  
My last computer science post is a massive monolithic creature, so let us jump straight in.

You can try out the examples here at my repository: [GitHub/K0Stek122/React-Tutorial](https://github.com/K0Stek122/React-Tutorial)

## The Theory
There is definitely less theory than in other topics, but there is still a few things to know.  
Old school websites will have separate pages for each topic. Imagine that you have a navigation bar. Each item that you click will take you to a separate website, and each time you do that, the whole page refreshes.  
To make pages more responsive, new website design uses **Single-Page App (SPA) Models**. These models work by dynamically replacing the content of the page within one single page. Because it's dynamic, javascript is required.

While this approach makes websites more responsive, it introduces three major issues:
1. Keeping all the data in-sync is difficult. For example, every time you press a button in a navigation bar, the data you inputted into a search bar changes. Should the data stay? Or be deleted? How should it be deleted?
2. DOM Manipulation is handled fully by React, and DOM Manipulation is very slow. Despite React optimisations, the websites still suffer from this limitation.
3. Using templates becomes harder, since everything is a component.

## Working With React
React works best when all of the elements on a web page are split into tiny **components**. Each component builds onto the previous one, creating a hierarchical network of them.  
Think of them like tiny classes.

```plaintext
                             ┌──────────────┐                                         
                             │Main Component│                                         
                          ┌──┴──────────────┴────┐                                    
                          │                      │                                    
                  ┌───────▼───────┐      ┌───────▼───────┐                            
                  │Child Component│      │Child Component│                            
          ┌───────┴───────────────┴───┬──┴───────────────┴───────────────┐            
          │                           │                                  │            
          │                           │                                  │            
┌─────────▼─────────────┐  ┌──────────▼────────────┐         ┌───────────▼───────────┐
│Smaller Child Component│  │Smaller Child Component│         │Smaller Child Component│
└───────────────────────┘  └───────────────────────┘         └───────────────────────┘
```

It is common for those hierarchies to span up-to even 10 sub-components. It's also important to *not overdo this*. Too many components will lead to the code being completely unreadable, and javascript is already barely readable without React.

## JSX
React merges javascript and html, it does so using a "language" called **JSX**. JSX allows converting between javascript and html markup. We will omit how JSX works in detail for now.

React allows you to write JSX, which looks like HTML but is actually JavaScript:
```javascript
	ReactDOM.render(React.createElement(
		"h2",
		null,
		"I am a man because i err"
	), document.querySelector("#container2"));
```
This does the exact same thing as `<h2>I am a man because i err</h2>`

## Hello World
Before we start, there is one more thing. In a real-world environment, you would use node.js and debug the website there. And only then host it finished. However for debugging purposes and for simplicity we are going to use a [Content Delivery Network](https://en.wikipedia.org/wiki/Content_delivery_network).

Let's start with the simplest example we can make.
```html
<!DOCTYPE html>
<html>
    <head>
        <title>React</title>
        <script src="https://unpkg.com/react@18/umd/react.development.js"></script> <!--This is the main React Library-->
        <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script> <!--This is the library to interact with .html and .css-->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script> <!--This is the library to convert JSX to JS, It is crucial-->
    </head>

    <body>
        <div id="container"></div>
        <div id="container2"></div>
        <script type="text/babel">
            ReactDOM.render(
                <h1>Hello World!</h1>,
                document.querySelector("#container") //Notice how after every ReacDOM.render() you will put this line, to notify where the element should be.
            );

            ReactDOM.render(React.createElement(
                "h2",
                null,
                "I am a man because i err"
            ), document.querySelector("#container2"));

        </script>
    </body>
</html>
```

First, we use CDN to get the react library and react-dom. as well as **babel-core**. babel-core is a library that is used to convert between JSX and html, normally you would do this in node.js, but for learning, we are just going to import it directly into html.

Let us focus on the `<script type="text/babel">`. We specify that babel has to be used, and it is followed by a ReactDOM.render(). This function is what will actually display stuff onto the screen. We then specify that we want a header size 2 to be displayed.  
Pretty simple. Originally when i started i though it is going to be way more painful to get started with this.

## Components
The problem with this hello world code is that it completely ignores what React is about. React is about components, you are supposed to deal with small modular pieces of front-end display. To do this, components are utilised.  
This next example has a simple Hello World component. I have split the code into .js and .html for convenience.
```html
<!DOCTYPE html>
<html>
    <head>
        <title>React Components</title>
        <script src="https://unpkg.com/react@15.3.2/dist/react.js"></script> <!--This is the main React Library-->
        <script src="https://unpkg.com/react-dom@15.3.2/dist/react-dom.js"></script> <!--This is the library to interact with .html and .css-->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script> <!--This is the library to convert JSX to JS-->
    </head>

    <body>
        <div id="container"></div>
        <script type="text/babel" src="./scripts.js"></script>
    </body>
</html>
```

This is the html, nothing too complex. But look at that javascript, you better sit down for this:
```javascript
var HelloWorld = React.createClass({ //By itself it is just an empty object.
    render: function() { //This is a MANDATORY member of React Class.
        return (
            <p>Hello From a Component!</p>
        );
    }
});

//IMPORTANT! Here i am using an old version of React, since the function createClass is deprecated in newer versions. We'll get to that soon.

ReactDOM.render(
    <div>
        <HelloWorld/>
        <HelloWorld/>
        <HelloWorld/>
        <HelloWorld/>
        <HelloWorld/>
        <HelloWorld/>
    </div>,
    document.querySelector("#container")
);
```
we use React.createClass (*which is a deprecated class by now but we will ignore that*).  
Yes, you call your own "html tags" using `<HelloWorld/>`, I know it's insane; when i first looked at this i almost had a stroke.  
Let's ignore the complete insanity that this code is, and focus on the component itself. the `render: function()` is a mandatory element of this object. and in `return ();` we specify the html that we want to return. Ignoring the fact that this code is bollocks, it's quite a handy tool to be able to create your own html tags. 

## Components With Attributes
Not only can you make your own components, but you can add your own parameters to them:

```javascript
var Buttonify = React.createClass({
    render: function() {
        return (
            <div>
                <button type={this.props.behaviour}>{this.props.children}</button>
            </div>
        );
    }
});

ReactDOM.render(
    <div>
        <Buttonify behaviour="Submit">Send Data</Buttonify>
        <Buttonify behaviour="Submit">Send Photos</Buttonify>
        <Buttonify behaviour="Submit">Send Something Else</Buttonify>
    </div>,
    document.querySelector("#container")
);
```
- **Remember** that `this.props.behaviour` can be used in every React component. The code is quite self-explanatory so please take a moment see what is actually happening. The result of this code are three buttons named "Send Data", "Send Photos", "Send Something Else".

## CSS & Styling
React kind of remakes CSS in a new *javascript* way, which means that it is going to be just as insane:

```javascript
var Letter = React.createClass({
    render: function() {
        var styles = { //Notice that properties which had a dash (-) in the middle are now CamelCase.
            padding: "10em", //em suffix is in commas.
            margin: 10, //px suffix is omitted and is added automatically.
            backgroundColor: this.props.bgcolor ? this.props.bgcolor : "#ff0000", //if bgcolor is set, set it to bgcolor, otherwise, set it to red.
            color: "#333",
            display: "inline-block",
            fontFamily: "Monospace",
            fontSize: 32,
            textAlign: "center"
        };

        return (
            <div style={styles}>
                {this.props.children}
            </div>
        );
    }
});

ReactDOM.render(
    <div>
        <Letter bgcolor="#00ff00">A</Letter>
        <Letter>E</Letter>
        <Letter>I</Letter>
        <Letter>O</Letter>
        <Letter>U</Letter>
    </div>,
    document.querySelector("#container")
);
```
One thing to notice is that you do not need to add px to the end of integers. They are appended automatically unless specified otherwise.

## Conclusion
In the next part we will deal with the main problem in react, which is passing `this.props.behaviour` down the hierarchy of components, as detailed before:
```plaintext
                             ┌──────────────┐                                         
                             │Main Component│                                         
                          ┌──┴──────────────┴────┐                                    
                          │                      │                                    
                  ┌───────▼───────┐      ┌───────▼───────┐                            
                  │Child Component│      │Child Component│                            
          ┌───────┴───────────────┴───┬──┴───────────────┴───────────────┐            
          │                           │                                  │            
          │                           │                                  │            
┌─────────▼─────────────┐  ┌──────────▼────────────┐         ┌───────────▼───────────┐
│Smaller Child Component│  │Smaller Child Component│         │Smaller Child Component│
└───────────────────────┘  └───────────────────────┘         └───────────────────────┘
```
This problem occurs when there is a lot of child components. It becomes an issue to pass different variables down the hierarchy. But we will tackle this in the next part.

While a *bit* dated, like everything in the javascript community, the book: [Learning React](https://www.amazon.co.uk/Learning-React-Kirupa-Chinnathambi/dp/0134546318) is excellent. I can recommend it to anyone, and it is one of not many textbooks that i actually wanted to keep reading, since it explains everything in a fun way. I'd recommend this to anyone trying to learn react.

You can check out all the fully working projects on my [GitHub](https://github.com/K0Stek122/React-Tutorial), and try them on your own.

Thank You for reading, and see you next time.

[Next Post -->](https://themindpalace.bearblog.dev/react-pt2-more-on-components/)
