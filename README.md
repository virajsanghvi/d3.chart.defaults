# d3.chart.defaults

This library is make setting options, and reacting to them, simpler for Miso's [d3.chart](http://misoproject.com/d3-chart/). 

## Setup

- Include ```d3.js```
- Include ```d3.chart.js```
- Include ```d3.chart.defaults.js```

## API

d3.chart.initializeDefaults(chart, defaults, [events]);

- chart [d3.chart] an definition of d3.chart - what gets returned by d3.chart(...)
- defaults [Object] key value pairs of optionName to optionValue
- events [Object] optional key value pairs of event (or events separated by whitespace) to a prototype method name on your chart, prefixed by 'debounce:' if you want to collect any other potential changes before executing the callback 


## Guide

This plugin is fairly straightforward. Usually, creating a d3.chart looks like:

```javascript
  var MyChart = d3.chart('MyChart', {

    initialize: function (options) { ... },

    ...

  });
```

Some people may have a ```defaults``` object in their prototype, and extend it with ```options``` in ```initialize```. This seems reasonable, but then you're also also reaching into an instance ```options``` member, which doesn't seem very d3 like. Then, you like want to create the standard d3 getter/setter methods on the prototype, ```instance.myOption()``` to get and ```instance.myOption(newValue)``` to set. This leads to a bunch of annoying duplicate code, and many people appear to solve this in their own way or just live with the duplication. Further more, when options change, sometimes you want to react, and there's further duplicate code trying to react to that.

This library aims to simplify this process. After defining your chart, simply, define your defaults separately:

```javascript
  d3.chart.initializeDefaults(MyChart, {
    height: 400,
    width: 600,
    radiusValue: function (d) { return d.radius; },
    ...
  });
```

These defaults are automatically extended by the ```options``` passed to the constructor, and available on the instance at the option name prefixed with a '_' (to denote them as internal, and to not conflict with the public getter). For example:

```javascript
  var MyChart = d3.chart('MyChart', {

  `  initialize: function (options) {

      // get option or default
      console.log(this._height);
        
    },

    ...

  });

  d3.chart.initializeDefaults(MyChart, {
    height: 400,
    ...
  }); 

  // would log 400 (the default)
  var chartDefaults = d3.select(el)
    .append("svg")
      .chart("MyChart");

 
  // would log 1000 (the override)
  var chartOverrides = d3.select(el)
    .append("svg")
      .chart("MyChart", {
        height: 1000
      }); 
```

When setting values, because option ```height``` is available at ```this._height```, you can set new values directly, but if you want to progromatically react to changes, whether by you or by your consumer, you can use the public setter function created for your option, ```this.height([newValue])``` which will trigger the event ```change:height``` for you to handle.

```javascript
   ...

  `  initialize: function (options) {

      // register handler to do something on event changes
      this.on('change:height', function () {
        console.log('Height has changed: ', this._height);
      });

      this.height(1000); // logs "Height has changed: 1000"
    },

  ...

```

You can use this functionality to trigger redraws as you normally would.

Obviously there's still a bunch of duplicatation in setting up event handling, and you can more efficiently register callbacks by specifying the events parameter of ```initializeDefaults```. The following helps describe what this parameter does:
 
```javascript
  var MyChart = d3.chart('MyChart', {

    initialize: function (options) { ... },

    onHeightChange: function () {
      console.log('height change handler');
    },
    
    redraw: function () {
      console.log('redraw!');
    });

    ...

  });

  d3.chart.initializeDefaults(MyChart, {
    height 400,
    width: 600,
    boxSize: 10,
    ...
  }, {
    // set up my events
    'height': 'onHeightChange', // when height changes, call this.onHeightChange()
    'boxSize': 'redraw', // when boxSize changes, call this.redraw()
    'height width': 'debounce:redraw', // when height or width changes, debounce the even for 100ms (wait for any other changes before executing), then call this.redraw()- this is helpful when redraws are super expensive
  }); 

  // create a new chart
  var chart = d3.select(el)
    .append("svg")
      .chart("MyChart");

  chart.height(1000); // logs "height change handler"
  chart.width(500); // nothing happens
  chart.boxSize(20); // logs "redraw!"
  // after ~100ms logs "redraw!" due to height and width changes
```

Please note that in the above example, you'd likely put all redraws behind a debounce to prevent doing more work than necessary.
 
## Status

This is currently alpha quality and may change
