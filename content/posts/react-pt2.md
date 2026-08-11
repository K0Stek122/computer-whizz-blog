+++
date = '2026-08-19T09:30:22+01:00'
draft = false
title = 'React pt. 2: More on components'
+++
[<-- Previous Post](https://themindpalace.bearblog.dev/react-pt1-the-footwork/)

Hello again! In this part, we will take a closer look at components and how to pass props down the component hierarchy.

But first, let us get a better grip on how components work. Let's create a simple colour card. It will include the colour name and the colour itself. This exercise is to solidify our understanding of how props and components work. The HTML code will always stay the same, so I will only include the JavaScript:
```javascript
var Square = React.createClass({ //This is the color
    render: function() {
        var style = {
            height: 150,
            backgroundColor: this.props.color
        };

        return (
            <div style={style}>

            </div>
        );
    }
});

var Label = React.createClass({ //This is the Text
    render : function() {
        var style = {
            fontFamily: "Monospace",
            fontWeight: "bold",
            fontSize: 16,
            padding: 13,
            margin: 0,
            textAlign: "center"
        };

        return (
            <p style={style}>{this.props.color}</p>
        );
    }
});

var Card = React.createClass({ //This is the "parent" component
    render: function() {
        var style = {
            height: 200,
            width: 150,
            padding: 0,
            margin: 5,
            backgroundColor: "#eee",
            WebkitFilter: "drop-shadow(0px 0px 5px #666)",
            filter: "drop-shadow(0px 0px 5px #666)"
        };

        return (
            <div style={style}>
                <Square color={this.props.color}/>
                <Label color={this.props.color}/>
            </div>
        );
    }
});

ReactDOM.render(
    <div>
        <Card color="#ff6663"/>
        <Card color="#89ff21"/>
        <Card color="#9054ef"/>
    </div>,
    document.querySelector("#container")
);
```
Take a moment to review what the code does and try it yourself.

Let's continue to something new.
## Spread Operators
Let's tackle the main issue that React presents to us. When working in a more complex environment, there will be components nested within components. Here is a simplified example of this problem:
```javascript
var Display = React.createClass({
    render: function() {
        return (
            <div>
                <p>{this.props.color}</p>
                <p>{this.props.num}</p>
                <p>{this.props.size}</p>
            </div>
        );
    }
});

var Label = React.createClass({
    render: function() {
        return (
            <Display color={this.props.color} num={this.props.num} size={this.props.size}/>
        );
    }
});

var Shirt = React.createClass({
    render: function() {
        return (
            <Label color={this.props.color} num={this.props.num} size={this.props.size}/>
        );
    }
});

ReactDOM.render(
    <div>
        <Shirt color="Blue" num="12" size="medium"/>
    </div>,
    document.querySelector("#container")
);

```
You need to pass the props in every single component. This becomes a big issue in larger projects with many nested components. Here is an example of how this problem is solved with spread operators:
```javascript
var Display = React.createClass({
    render: function() {
        return (
            <div>
                <p>{this.props.color}</p>
                <p>{this.props.num}</p>
                <p>{this.props.size}</p>
            </div>
        );
    }
});

var Label = React.createClass({
    render: function() {
        return (
            <Display {...this.props}/>
        );
    }
});

var Shirt = React.createClass({
    render: function() {
        return (
            <Label {...this.props}/> //Think of this like it's grabbing the child component straight away, rather than having to pass through every component.
        );
    }
});

ReactDOM.render(
    <div>
        <Shirt color="Blue" num="12" size="medium"/> //Notice how "Display" Components are included here as well.
    </div>,
    document.querySelector("#container")
);
```
In a simple example like this, it might not seem like a big problem, but it _will_ lead to spaghetti code in the long term, even for simple projects.

## States
States are a way of storing data in a React component. Think of states like _member variables_; you have your React class, and the state of the React class consists of the variables relevant to that class. States are assigned differently from regular member variables. Instead of defining them in the constructor, we define them in a special function called `getInitialState`.
```javascript
var LightningCounter = React.createClass({
    getInitialState: function() { //This is a built-in function that is called right BEFORE the component is displayed on-screen.
        return {
            strikes: 0 //Think of this like kind of setting a member variable.
        };
    },
// ...
```
Because of JavaScript's nature, there is no way to distinguish overridden functions from user-defined ones, so you have to memorize them. Here is an example of a counter that updates every second and utilizes states:
```javascript
var LightningCounter = React.createClass({
    getInitialState: function() { //This is a built-in function that is called right BEFORE the component is displayed on-screen.
        return {
            strikes: 0 //Think of this like kind of setting a member variable.
        };
    },
    
    timerTick: function() { //This is our user-defined function.
        this.setState({ //setState is a built-in function that allows for changing the states that are initially set in getInitialState function.
            strikes: this.state.strikes + 100
        });
    },

    componentDidMount: function() { //This is a built-in function that is called right AFTER the component is displayed on-screen.
        setInterval(this.timerTick, 1000);
    },

    render: function() {
        var style = {
            color: "#66FFFF",
            fontSize: 50
        };

        return (
            <h1 style={style}>{this.state.strikes}</h1> //Notice we are not using "props", but "state" this time
        );
    }
});

var LightningCounterDisplay = React.createClass({
    render: function() {
        var commonStyle = {
            margin: 0,
            padding: 0
        };

        var divStyle = {
            width: 250,
            textAlign: "center",
            backgroundColor: "black",
            padding: 40,
            fontFamily: "sans-serif",
            color: "#999",
            borderRadius: 10
        };

        var textStyle = {
            emphasis: {
                fontSize: 38,
                ...commonStyle
            },
            smallEmphasis: {
                ...commonStyle
            },
            smal: {
                fontSize: 17,
                opacity: 0.5,
                ...commonStyle
            }
        };

        return (
            <div style={divStyle}>
                <LightningCounter/>
                <h2 style={textStyle.smallEmphasis}>LIGHTNING STRIKES</h2>
                <h2 style={textStyle.emphasis}>WORLDWIDE</h2>
                <p style={textStyle.small}>(since you loaded this example)</p>
            </div>
        );
    }
});

ReactDOM.render(
    <LightningCounterDisplay/>,
    document.querySelector("#container")
);
```

## Conclusion
This is all that I am going to throw at you this week since these concepts perplexed me more than anything else I have learned about React so far. I think it'd be a good idea to let this information marinate a little bit before we proceed further. Thank you, and good luck navigating the JavaScript world.

Check out the book that i am learning this from. It is full of more exercises and things that i don't cover here: [Learning React](https://www.amazon.co.uk/Learning-React-Hands-Building-Applications/dp/013484355X)

[<-- Previous Post](https://themindpalace.bearblog.dev/react-pt1-the-footwork/)
