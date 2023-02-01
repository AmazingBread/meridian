<template>
	<div id="map"></div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import "leaflet-polylinedecorator";
import "leaflet-timedimension";
import "leaflet-timedimension/src/leaflet.timedimension.control.css";
import featureCollection from "@/assets/featureCollection.json";

import { Icon }  from 'leaflet'
delete Icon.Default.prototype._getIconUrl;

Icon.Default.mergeOptions({
    iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
    iconUrl: require('leaflet/dist/images/marker-icon.png'),
    shadowUrl: require('leaflet/dist/images/marker-shadow.png')
});

export default {
	name: "TimeDimension",
	props: {
		msg: String,
	},
	data() {
		return {
			map: null,
		};
	},
	mounted() {
		this.init();
		// this.drawPolyLine();
		
	},
	methods: {
		init() {
			this.map = new L.map("map", {
				timeDimensionControl: true, //replay
				timeDimensionControlOptions: {
					//replay
					timeSliderDragUpdate: true,
					loopButton: false,
					autoPlay: true,
					playerOptions: {
						transitionTime: 100,
						loop: false,
						startOver: true,
					},
				},
				timeDimension: true, //replay
			}).setView([51.505, -0.09], 13);
			L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
				maxZoom: 19,
				attribution:
					'&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
			}).addTo(this.map);
			this.timeDimension(featureCollection);
		},
		timeDimension(featureCollection) {
			let startDate = new Date();
			startDate.setUTCHours(0, 0, 0, 0);

			let geoJSONLayer = L.geoJSON(featureCollection, {
				onEachFeature: function (feature, layer) {
					if (layer instanceof L.Polyline) {
						layer.setStyle({
							color: feature.properties.color,
							weight: "4",
							opacity: 0.8,
							zIndexOffset: 900,
						});
					}
					layer.bindTooltip(feature.properties.name);
				},
			});

			let geoJSONTDLayer = L.timeDimension.layer.geoJson(geoJSONLayer, {
				period: "PT2M",
				updateTimeDimensionMode: "replace",
				updateTimeDimension: true,
				addlastPoint: true,
				// waitForReady: true,
			});

			geoJSONTDLayer.addTo(this.map);
			this.map.fitBounds(geoJSONLayer.getBounds());
		},
	},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
body {
	margin: 0;
}
html {
	margin: 0;
	/* min-width: 320px; */
}
#map {
	height: 100vh;
	z-index: 99;
}
</style>
