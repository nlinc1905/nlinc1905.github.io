---
layout: post
title:  "How to Build a Network Graph with Sigma.js"
date:   2017-03-24 23:19:22 -0500
categories: JavaScript, HTML, D3, Sigma, Data Visualization, Network Science
---
[This link](https://github.com/nlinc1905/JavaScript-Network-Graphs) leads to an example of a network link chart built with [sigma.js](http://sigmajs.org/).  Sigma is a great JavaScript library for building custom link charts and it has a lot of functionality.  In fact, [Linkurious](http://linkurio.us/) is a company that builds web based link charts that use Sigma as the backbone.  Linkurious has a lot of custom built features that expand upon Sigma that can only be accessed by purchasing the product, but for the common developer, knowledge of Sigma is all that is needed to build link charts with a wide range of functionality.  In this article, I will explain some of the features of my example Sigma network graph, piece by piece.

Referring to this file: <https://github.com/nlinc1905/JavaScript-Network-Graphs/blob/master/sigma_network_example.html>, the code for the network begins at the `<script>` tag, although the dependency libraries are imported first:  

{% highlight html %}
<script src="assets/js/sigma.min.js"></script>
<script src="assets/js/sigma.parsers.json.min.js"></script>
<script src="assets/js/sigma.forceAtlas2.worker.js"></script>
<script src="assets/js/sigma.forceAtlas2.supervisor.js"></script>
<script src="assets/js/sigma.pathfinding.astar.js"></script>
<script src="http://code.jquery.com/jquery-1.9.1.js"></script>
{% endhighlight %}

The graph is inserted into the div with id=network_graph, and it is styled like so:

{% highlight css %}
#network-graph {
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	position: absolute;
}
{% endhighlight %}

Now that that’s out of the way, the first block of code disables the default behavior of right clicking so that right clicks can be used to reset the graph.

{% highlight javascript %}
document.getElementById('network-graph').oncontextmenu = function (e) {
    e.preventDefault();
};
{% endhighlight %}

The next function finds and returns all neighbors to any node.  The first part of the function declares empty variables k and a neighbors object.  The second part iterates through the indices of each node and gets the indices of all neighboring nodes.  

{% highlight javascript %}
sigma.classes.graph.addMethod('neighbors', function(nodeId) {
	var k,
		neighbors = {},
		index = this.allNeighborsIndex[nodeId] || {};

	for (k in index)
		neighbors[k] = this.nodesIndex[k];

	return neighbors;
});
{% endhighlight %}

The next block first declares jnet and s as empty variables, grabs the JSON network data, and then parses it.  The jnet variable is set equal to the JSON string response, and s becomes a new sigma object (a network graph object).  The sigma object has properties graph and container, which represent the data for the network graph and the HTML element to put it in, respectively.  The `buildNetwork()` function is called within this function, because the data has to be loaded before the network can be built.  Any other functions that require the data to be loaded first can be put here too.

{% highlight javascript %}
var jnet, s;
$.getJSON("assets/data/user_network.json", function(response){
		jnet = response;
		s = new sigma({
			graph: jnet,
			container: 'network-graph'
	   });
	   buildNetwork();
});
{% endhighlight %}

The next section defines the sigma network object.  The first step is to optimize the network layout.  This is required so that the nodes have coordinates that make sense, otherwise they would all be jumbled together you couldn’t easily see anything except a tangled mess of nodes.  The layout is optimized using the force atlas algorithm, which places node locations at the maximum distance from one another, which keeping more similar nodes closer.  So 2 nodes with many of the same connections will be closer together than nodes with different connections, but the distance between them will still be maximized within the constraints of the network’s boundaries so that the network is easy to look at and understand.  The algorithm requires time to converge onto a solution (it’s solving a complex differential equation subject to a constraint), so the function gives it a few seconds and then stops the algorithm from continuing.  If let go, it would never stop optimizing, but the result would be pointless because the nodes do not visibly change location much (I tried it to see what would happen, and 5 seconds is plenty of time for a good enough solution).  

{% highlight javascript %}
function layoutOptimize(s) {
	
	//Start the ForceAtlas2 algorithm to optimize network layout
	s.startForceAtlas2({worker: false, barnesHutOptimize: false, slowDown: 1000, iterationsPerRender: 1000});
	
	//Set time interval to allow layout algorithm to converge on a good state
	var t = 0;
	var interval = setInterval(function() {
		t += 1;
		if (t >= 5) {
			clearInterval(interval);
			s.stopForceAtlas2();
		}
	}, 100);
}
{% endhighlight %}

The next function, `buildNetwork`, handles the lion’s share of the work.  Rather than go through it all at once, let’s look at it piece by piece.  First the function sets the initial node and edge colors.

{% highlight javascript %}
//Create the function to build the network graph
function buildNetwork() {
	//Save the initial colors of the nodes and edges
	s.graph.nodes().forEach(function(n) {
		n.originalColor = n.color;
	});
	s.graph.edges().forEach(function(e) {
		e.originalColor = e.color;
	});
	
	//Override initial edge colors
	s.settings({
		edgeColor: 'default',
		defaultEdgeColor: '#999'
	});
	
	// Refresh the graph to see the changes:
	s.refresh();
{% endhighlight %}

The next function finds all neighbors for any node that is hovered over with the mouse.  Neighbors are stored in an object called toKeep, and the node colors, as well as the edge colors for any links touching neighboring nodes, are set to blue.  

{% highlight javascript %}
	s.bind('overNode', function(e) {
		var nodeId = e.data.node.id,
			toKeep = s.graph.neighbors(nodeId);
			toKeep[nodeId] = e.data.node;
			
		s.graph.nodes().forEach(function(n) {
			if (toKeep[n.id])
				n.color = '#36648B';
			else
				n.color = n.originalColor;
		});

		s.graph.edges().forEach(function(e) {
			if (toKeep[e.source] && toKeep[e.target])
				e.color = '#0099CC';
			else
				e.color = e.originalColor;
		});
		
		//Refresh graph to update colors
		s.refresh();
	});
{% endhighlight %}

When the mouse moves off of a node, the hovering stops, so the next function defines what happens.  When a node is no longer being hovered over, its color and the color of all connecting edges should return to the default values.

{% highlight javascript %}
	s.bind('outNode', function(e) {
	
		s.graph.nodes().forEach(function(n) {
			n.color = n.originalColor;
		});
		
		s.graph.edges().forEach(function(e) {
			e.color = e.originalColor;
		});
		
		//Refresh graph to update colors
		s.refresh();
	});
{% endhighlight %}

The next function defines what happens when a node is clicked.  First, all neighbors of that node are found.  Any nodes that are not neighbors are colored light gray so that they appear faded.  The same applies to their connections.  So when a node is clicked, only its neighbors and edges will be visible.  The node that was clicked will have its color set to green.  If many nodes are clicked in succession, the network that is visible will be the nodes that are shared between clicked nodes.

{% highlight javascript %}
	s.bind('clickNode', function(e) {
		var nodeId = e.data.node.id,
			toKeep = s.graph.neighbors(nodeId);
			toKeep[nodeId] = e.data.node;

		s.graph.nodes().forEach(function(n) {
			if (toKeep[n.id] == toKeep[nodeId])
				n.originalColor = '#008000';
			else if (toKeep[n.id])
				n.color = n.originalColor;
			else
				n.color = '#eee',
				n.hidden = true;
		});

		s.graph.edges().forEach(function(e) {
			if (toKeep[e.source] && toKeep[e.target])
				e.color = e.originalColor;
			else
				e.color = '#eee',
				e.hidden = true;
		});

		//Refresh graph to update colors
		s.refresh();
	});
{% endhighlight %}

The next function returns everything to default, essentially resetting the network, when a user right clicks on it.

{% highlight javascript %}
	s.bind('rightClickStage', function(e) {
		s.graph.nodes().forEach(function(n) {
			n.originalColor = "#2d2d3c",
			n.color = n.originalColor,
			n.hidden = false;
		});

		s.graph.edges().forEach(function(e) {
			e.color = e.originalColor,
			e.hidden = false;
		});

		//Refresh graph to update colors
		s.refresh();
	});
{% endhighlight %}
	
Finally, the layout optimize function is called to start the force atlas algorithm.

{% highlight javascript %}
	layoutOptimize(s);
}  //This bracket closes the buildNetwork() function
{% endhighlight %}

That’s it!  Sigma is not so bad when it’s broken down.  I built this network piece by piece, starting with a static network and slowly adding functions one by one.  I would recommend doing it the same way, otherwise your code will be messy and a pain to debug.  