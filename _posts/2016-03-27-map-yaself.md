---
layout: post
cover: 'assets/images/cover7.jpg'
title: Putting a map in jekyll
date:   2014-08-12 10:18:00
tags: fables fiction maps
subclass: 'post tag-test tag-content'
categories: 'kyle'
navigation: True
logo: 'assets/images/ghost.png'
---


<div id="map">

</div>

<script>

var mapboxAccessToken = 'pk.eyJ1Ijoia2hldXRvbiIsImEiOiJjaW1janY0MWUwMDBvdWZsdnY4YXdubjRrIn0._VsfvE51YTv8S4SeKoIRlA';
var map = L.map('map').setView([37.8, -96], 4);

L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=' + mapboxAccessToken, {
    id: 'kheuton.phg9o4a0',
    attribution: 'None'
}).addTo(map);

</script>
