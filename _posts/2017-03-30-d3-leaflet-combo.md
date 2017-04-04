---
layout: post
cover: 'assets/images/cover7.jpg'
title: An Attempt to Integrate d3 and Leaflet
date:   2017-03-04 10:18:00
tags: maps
subclass: 'post tag-test tag-content'
categories: 'kyle'
navigation: True
logo: 'assets/images/ghost.png'
published: false
---
<script type="text/javascript" src="my_layer.json"></script>
<script type="text/javascript" src="http://d3js.org/d3.v3.min.js"></script>
<script type="text/javascript" src="http://d3js.org/queue.v1.min.js"></script>
<script type="text/javascript" src="http://d3js.org/topojson.v0.min.js"></script>

<style>

svg {
  position: relative;
}


.map {
  width: 960px;
  height: 500px;
}
.map {
  position: relative;
  overflow: hidden;
}
.layer {
  position: absolute;
}
.tile {
  pointer-events: none;
  position: absolute;
  width: 256px;
  height: 256px;
}

	.info {
		padding: 6px 8px;
		font: 14px/16px Arial, Helvetica, sans-serif;
		background: white;
		background: rgba(255,255,255,0.8);
		box-shadow: 0 0 15px rgba(0,0,0,0.2);
		border-radius: 5px;
	}
	.info h4 {
		margin: 0 0 5px;
		color: #777;
	}
.legend {
	text-align: left;
	line-height: 18px;
	color: #555;
}
.legend i {
	width: 18px;
	height: 18px;
	float: left;
	margin-right: 8px;
	opacity: 0.7;
}
div.tooltip {
  position: absolute;
  text-align: center;
  width: 150px;
  height: 25px;
  padding: 2px;
  font-size: 10px;
  background: #FFFFE0;
  border: 1px;
  border-radius: 8px;
  pointer-events: none;
}
</style>

<div id="example"></div>
<select id="groupA"></select>
<div id="map">
</div>

<script src="//d3js.org/d3.v3.min.js"></script>
<script src="../d3.geo.tile.min.js"></script>
<script>
labels = ["A", "B", "C", "D"];
options = [0, 1, 2, 3];

// Build the dropdown menu
d3.select("#groupA").on("change", onChange)
    .selectAll("option")
	  .data(labels).enter()
	  .append("option")
		.text(function (d) { return d; });

d3.selection.prototype.moveToFront = function() {  
  return this.each(function(){
    this.parentNode.appendChild(this);
  });
};
var map = L.map('map').setView([47.6062, -122.3321], 11);
L.tileLayer('http://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}', {
	maxZoom: 18,
	attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
}).addTo(map);

var svg = d3.select(map.getPanes().overlayPane).append("svg"),
    g = svg.append("g").attr("class", "leaflet-zoom-hide");

d3.json("d3_layer.json", function(error, collection) {
  if (error) throw error;

  var transform = d3.geo.transform({point: projectPoint}),
      path = d3.geo.path().projection(transform);

  var feature = g.selectAll("path")
      .data(collection.features)
    .enter().append("path").attr("d", path)
    			 .style("fill",function (d,i) { return getColor(d.properties.gap) })
    			 .style("stroke",function (d,i) { return getBorderColor(d.properties.missing) })
    			 .style("stroke-width", 2)
    			 .style("stroke-dasharray", 3)
    			 .attr("fill-opacity",0.7)
           .style("stroke-opacity",1)
           .on("mouseover",highlightFeature)
           .on("click", highlightFeature)
           .on("mouseout", resetHighlightMouse)
           .on("dblclick", zoomToFeature);

  map.on("viewreset", reset);
  reset();

  // Reposition the SVG to cover the features.
  function reset() {
    var bounds = path.bounds(collection),
        topLeft = bounds[0],
        bottomRight = bounds[1];

    svg .attr("width", bottomRight[0] - topLeft[0])
        .attr("height", bottomRight[1] - topLeft[1])
        .style("left", topLeft[0] + "px")
        .style("top", topLeft[1] + "px");

    g.attr("transform", "translate(" + -topLeft[0] + "," + -topLeft[1] + ")");

    feature.attr("d", path);
  }

  // Use Leaflet to implement a D3 geometric transformation.
  function projectPoint(x, y) {
    var point = map.latLngToLayerPoint(new L.LatLng(y, x));
    this.stream.point(point.x, point.y);
  }
});

var legend = L.control({position: 'bottomright'});

legend.onAdd = function (map) {

  var div = L.DomUtil.create('div', 'info legend'),
    grades = [0, 5, 10, 20, 30, 40, 50, 60],
    labels = [],
    from, to;

  for (var i = 0; i < grades.length; i++) {
    from = grades[i];
    to = grades[i + 1];

    labels.push(
      '<i style="background:' + getColor(from + 1) + '"></i> ' +
      from + (to ? '&ndash;' + to : '+'));
  }

  div.innerHTML = labels.join('<br>');
  return div;
};

legend.addTo(map);
// control that shows state info on hover
var info = L.control();

info.onAdd = function (map) {
  this._div = L.DomUtil.create('div', 'info');
  this.update();
  return this._div;
};

info.update = function (props) {
  this._div.innerHTML = '<h4>Achievement Gap</h4>' +  (props ?
    '<b>' + props.ES_ZONE + '</b><br />' + props.gap + '% more White students meet standards than their Black peers'
    : 'Hover over a school');
};
var lastClicked;
function highlightFeature(e) {

  var feature = d3.select(this)
  feature.style("stroke-width", 5)
  feature.moveToFront()
  feature.style("stroke-dasharray", 0 )
  feature.attr("fill-opacity",0.7)
  feature.style("stroke-opacity",1)
  var layer = e.target;


  info.update(e.properties);
  if (lastClicked && (lastClicked.id != feature.id)) {
    resetHighlightClick(lastClicked);
  }

  lastClicked = feature;
}

var geojson;

function resetHighlightMouse(feature) {
  var feature = d3.select(this)
  resetStyle(feature)
  info.update();
}
function resetHighlightClick(feature) {

  resetStyle(feature)
}

function resetStyle(feature) {

  feature.style("stroke-width", 2)
  feature.style("stroke-dasharray", 3)
  feature.attr("fill-opacity",0.7)
  feature.style("stroke-opacity",1.0)
}

function zoomToFeature(e) {
  map.fitBounds(e.target.getBounds());
}

function onEachFeature(feature, layer) {
  layer.on({
    mouseover: highlightFeature,
    click: highlightFeature,
    mouseout: resetHighlightMouse,
    dblclick: zoomToFeature
  });
}


info.addTo(map);


// get color depending on population density value
function getColor(d) {
 return d > 60 ? '#800026' :
        d > 50  ? '#BD0026' :
        d > 40  ? '#E31A1C' :
        d > 30  ? '#FC4E2A' :
        d > 20   ? '#FD8D3C' :
        d > 10   ? '#FEB24C' :
        d > 5   ? '#FED976' :
                   '#FFEDA0';
}

// get border based on which is misisng
function getBorderColor(d) {
 return d == 'white' ? 'white':
        d == 'black' ? 'black':
        d == 'none'  ? 'gray':
                        'gray';
}

</script>
