---
layout: post
title: [Tutorial] How to Build real-time data visualization with D3, Crossfilter, and Websockets in Python by example
---

## What is Real Time Data Visualization
The vast majority of data visualization consists of a static set of data that is pulled upon user request. So, the data only gets updated when the user wants it. It is a request-response pattern. User requests info, server responds with data & visualizations are populated.

Real-time data visualization is applicable when you have data that is rapidly updating in real time and your application needs to keep a ‘pulse’ on and monitor data passively. This means we have charts that update automatically while you keep your browser open.
Just some examples where you might want real-time data viz include but aren’t limited to:

- Stock/Financial instrument analysis
- Embedded systems
- Web traffic or server data
- security systems
- geospacial / gps monitoring (think uber
- business intelligence
- marketing intelligence
- industrial or manufacturing
- IoT devices
- Anything else that requires real-time monitoring
 
 ### End product:
 <iframe width="560" height="315" src="https://www.youtube.com/embed/VI3TnZugspc" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>



### What we are using
#### Languages: 
- Python
- Javascript
#### JS Libraries:
- d3.js
- crossfilter.js
- dc.js : makes d3 and crossfilter work swimmingly
- python’s tornado library for websockets, ioloop, and web

We are going to use these tools to build a websocket server that publishes some mock data every second. We will then build some static interactive charts with d3,crossfilter, and dc.js .

Finally, we will have our d3 chart connect to our websocket server, and updates to the chart will happen in real time. It’ll be pretty sweet. 

## Before we start & setup
I’m using python 2.7 for this. 

#### Download Python [here](https://www.python.org/download/releases/2.7/). Install it then open a cmd or terminal and type “python” to ensure proper setup - if you get a command prompt everything should be good. 

If you have issues or errors  installing python there are plenty of resources on stackoverflow to help better than I can. 

- Install pip 

- Get Sublime Text for text editing. Or use text editor of your choice. 

- Create a new folder called ‘rt-data-viz’  

## Building A Simple Websocket Server in Python
First we’ll build our data source. For this tutorial we’re building a simple websocket server that periodically sends out new data.

In your ‘rt-data-viz’ folder, create a new file & save it with the name “websocket_server.py”

We need to install the `tornado` package to run our websockets and ioloop . On your terminal/console run `pip install tornado`

In websocket_server.py, we can start coding now. 

First, import all required packages:

```import time
import random
import json
import datetime
from tornado import websocket, web, ioloop
from datetime import timedelta
from random import randint ```

 Using tornado’s websocket, we need to build a handler class. I’m adding some empty functions that we’ll fill out next:
 
 <script src="https://gist.github.com/benjaminmbrown/54c81b13e737feb9852d.js"></script>

 Now to fill out our web socket handler. The most important function for us is `send_data()`. It it where we’re building a json object of random data, sending it via `self.write_message()`. 

After we send the message we use ioLoop to create a timeout that will send data periodically. Finally, we create the websocket web app instance, set it to listen on port 8001, and start our ioloop instance. The completed websocket_server.py:

<script src="https://gist.github.com/benjaminmbrown/46c9f9fce74f137b0e3e.js"></script>

We can start this server by going to our command prompt in the rt-data-viz folder and typing `python websocket_server.py`. You’ll notice nothing will happen. This is because in order for the socket to become active, we need to have our client-side code open the connection on that port. We’ll be able to see this after we create & run our client-side code next.

## Building our Charts with D3 and Crossfilter

We’re going to use `d3.js` and `crossfilter.js` to create two charts that share the same data. 

`crossfilter` helps us explore multivariate data sets with functions that can create dimensions based on the data and group variants.The `dc.js ` library combines both together so that we can use the actual charts themselves to filter the data when on user interaction. 

Their websites explain in more detail: [d3.js](https://d3js.org/) , [crossfilter.js](http://square.github.io/crossfilter/), [dc.js](https://dc-js.github.io/dc.js/) 

We are going to create two charts:

 - 1 pie chart that displays money spent by year
 - 1 bar chart that shows money spent by person. IT will look like this:
 ![enter image description here](https://web.archive.org/web/20160713141252im_/http://www.benjaminmbrown.com/wp-content/uploads/2016/02/static-charts.jpg)


If a user  clicks a part of the chart, it will filter both the chart you click and the other chart will also reflect the data change.. Here I clicked the year 2014 on the chart.

![enter image description here](https://web.archive.org/web/20160713141252im_/http://www.benjaminmbrown.com/wp-content/uploads/2016/02/static-chart-2014-selected.jpg)

Create and `index.html file` in your `rt-data-viz` folder. Here’s a basic template with the required libraries:

<script src="https://gist.github.com/benjaminmbrown/3ff10264b5bd1d83fcc6.js"></script>

Inside our body, we need to create two divs to hold our chart data:

```<div id=”chart-ring-year”></div>
<div id=”chart-row-spenders”></div>```

Then we are going to start with our d3/crossfilter work. Create a static array of json objects:

```
 var data1 = [ 
 {Name: ‘Ben’, Spent: 330, Year: 2014, ‘total':1},
  {Name: ‘Aziz’, Spent: 1350, Year: 2012, ‘total':2}, 
  {Name: ‘Vijay’, Spent: 440, Year: 2014, ‘total':2}, 
  {Name: ‘Jarrod’, Spent: 555, Year: 2015, ‘total':1},];
```

Then we create a variable to hold our ‘crossfiltered’ data:
`var xfilter = crossfilter(data1);`

With that data we can now create dimensions using crossfilter’s `dimension()` function. We have 3 dimensions we’re going to use for this example: Name, Year, and Spent:

```
var yearDim = xfilter.dimension(function(d) {return +d.Year;});
var spendDim = xfilter.dimension(function(d) {return Math.floor(d.Spent/10);});
var nameDim = xfilter.dimension(function(d) {return d.Name;});
```

With those dimensions set, we can now group them. To simplify it a bit, the groups are basically the end result that populates each chart. So in the images above, the pie chart shows each year, and the size of each piece of the pie represents the amount spent that year. So, we would call our group spendPerYear and use that in our chart. That looks like:

`var spendPerYear = yearDim.group().reduceSum(function(d) {return +d.Spent;});`

And our other chart shows the amount spent per person (or “Name” in our json object):

`var spendPerName = nameDim.group().reduceSum(function(d) {return +d.Spent;});`

Our code looks like this so far:
<script src="https://gist.github.com/benjaminmbrown/35192e8b406d1bd0c6fa.js"></script>

So we now have our data, dimensions, and groups setup. We need to render the charts. Let’s create a rendering function. We’ll call it `render_plots()`. It will render charts with dc.js’s `renderAll()` function: 

```function render_plots(){ #chart plots go here soon #render all the charts dc.renderAll() }```

 Inside `render_plots()` We’re going to create a pie chart, which is going to render to our `yearRingChart` div we created previously. In the pie chart we are using the year dimension and the spendPerYear grouping. We also set the widgth, height, and innerRadius attributes which is specific to pie charts:

`yearRingChart  .width(200).height(200)  .dimension(yearDim)  .group(spendPerYear)  .innerRadius(50);`

For the rowChart/ bar chart we set it up similarly:

`spenderRowChart  .width(250).height(200)  .dimension(nameDim)  .group(spendPerName);`

We are going to be adding a lot more data to this bar chart, so we’ll need to resize it dynamically to ‘fit’ larger data sets. We do that by setting elasticX to true:

`spenderRowChart  .width(250).height(200)  .dimension(nameDim)  .group(spendPerName)  .elasticX(true);`

<script src="https://gist.github.com/benjaminmbrown/716b187c9ca49abccd1d.js"></script>

And we can now see our functioning charts code…here’s our final d3.js + crossfilter.js + dc.js which calls the render_plot() function :

<script src="https://gist.github.com/benjaminmbrown/e26cd1dac859de8bb3c6.js"></script>

This is cool, you’ll want to see it in action. You’ll need to run a local webserver and point it to your file. Open up your folder rt-data-viz in cmd prompt/terminal and type:

`python -m SimpleHTTPServer 3000 `

Keep your terminal open and open a tab on your web browser of choice. Go to the address http://localhost:3000/ to see it live.

## Updating D3 Charts with Real-Time Updates from Websocket Server

So, now we have a websocket server that is posting new data every second or so, and and we have some static charts that expose d3 and crossfilter functionality. We gotta make them talk now. We now need to modify our chart code to connect to the websocket , handle data from the websocket, and correctly update the charting solution in real-time. In the javascript of our index file, we need to create a new websocket connection that connects to our websocket_server.py on port 8001. We do this by creating a new websocket like so:    

`var connection = new WebSocket(‘ws://localhost:8001/websocket’); `

Then we need a function that will update our charts any time the websocket publishes 
and update. We do that with Websocket’s `onmessage()` function:
```
connection.onmessage = function(event){  

//get data & parse  
var newData = JSON.parse(event.data);  

#### put data into an array of json objects  
var updateObject = [{“Name”: newData.Name,“Year” : newData.Year,“Spent”: newData.Spent,“payType: newData.payType }]  

####add this new array into our data
xfilter.add(updateObject);

 ####redraw our charts with new data
dc.redrawAll(); 

}   
```
Here’s the function in code:

<script src="https://gist.github.com/benjaminmbrown/f90ef182b3fa87f24603.js"></script>

And here’s the entire index.html which will work with your websocket server to retrieve data as it is published:

<script src="https://gist.github.com/benjaminmbrown/8307fa544dddfa635558.js"></script>

So how do you make this work? Well, we really have to spin up two servers to run the websocket_server and the client side code, respectively.

If you havn’t figured it out already, you start your server first , then your client.
Open a command prompt in the rt-data-viz folder and start up the websocket:

`python websocket_server.py`

Then open another prompt and run the client:

`python -m SimpleHTTPServer 3000`

Open your localhost:3000 in your browser and after a few seconds you should see the charts updating!  Not only are they updating with new data from the websocket server, but you can click on the charts and they will crossfilter each other. Really flippin cool.

## Conclusion

We’ve covered websockets, d3, crossfilter, and dcjs. Hopefully you have taken a lot of value out of my efforts here! Let me know what you think. Tweet me [@benjaminmbrown](http://twitter.com/benjaminmbrown) for fastest responses.

All the code is available on my github tutorial repo: 

[https://github.com/benjaminmbrown/real-time-data-viz-d3-crossfilter-websocket-tutorial](https://github.com/benjaminmbrown/real-time-data-viz-d3-crossfilter-websocket-tutorial)

### Contact
Linkedin: [benjaminmichaelbrown](https://www.linkedin.com/in/benjaminmichaelbrown/)
Twitter: [benjaminmbrown](http://twitter.com/benjaminmbrown)
Github: [benjaminmbrown](http://github.com/benjaminmbrown)
Site: [benjaminmbrown.github.io](benjaminmbrown.github.io)