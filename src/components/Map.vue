<template>
  <div id="map" style="height: 100vh"></div>
</template>

<script lang="ts" setup>
import { onMounted } from "vue";
import "leaflet.locatecontrol";
import * as L from "leaflet";
import markerIconUrl from "leaflet/dist/images/marker-icon.png";
import markerIconRetinaUrl from "leaflet/dist/images/marker-icon-2x.png";
import markerShadowUrl from "leaflet/dist/images/marker-shadow.png";

import places from "../assets/places.json"

let map: L.Map;

// https://cescobaz.com/2023/06/14/setup-leaflet-with-svelte-and-vite/
L.Icon.Default.prototype.options.iconUrl = markerIconUrl;
L.Icon.Default.prototype.options.iconRetinaUrl = markerIconRetinaUrl;
L.Icon.Default.prototype.options.shadowUrl = markerShadowUrl;
L.Icon.Default.imagePath = "";

function getgeo() {
  // 現在地を表示するコントロールを追加
  //@ts-ignore
  const lc = L.control.locate({
    position: 'topleft',
    setView: 'once',
    keepCurrentZoomLevel: true,
    markerStyle: {
      weight: 2,
      opacity: 0.8,
      fillOpacity: 0.8
    },
    circleStyle: {
      weight: 1,
      clickable: false
    },
    icon: 'leaflet-location-icon',
    metric: false,
    strings: {
      title: "Show me where I am",
      popup: "You are within {distance} {unit} from this point",
      outsideMapBoundsMsg: "You seem to be outside the boundaries of the map"
    },
    locateOptions: {
      maxZoom: 10,
      enableHighAccuracy: true
    },
    //@ts-ignore
    onLocationError: (err) => {
      console.warn(err);
    }
  }).addTo(map);

  lc.start();
}

//@ts-ignore
function make_popup_content(feature): string{
  let str = `<h3>${feature.properties.name}</h3>`
  if (feature.properties.notes){
    str += `<p>${feature.properties.notes}</p>`
  }

  if (feature.properties.start && feature.properties.end){
    str += `<h4>開催日</h4><p>${feature.properties.start} - ${feature.properties.end}`
  }

  if (feature.properties.links && feature.properties.links.length > 0){
    str += "<h4>リンク</h4><ul>"
    //@ts-ignore
    feature.properties.links.forEach(element => {
      str += `<li><a href="${element}" target="_blank" rel="noopener noreferrer">${element}</li>`
    });
    str += "</ul>"
  }

  return str;
}

onMounted(() => {
  map = L.map("map").setView([35, 135], 5);
  getgeo();

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  }).addTo(map);
  
  //@ts-ignore
  L.geoJson(places.features, {
    onEachFeature: function (feature, layer) {
      if (feature.properties && feature.properties.name) {
        layer.bindPopup(make_popup_content(feature), {autoClose: true, closeButton: true});
        //@ts-ignore
        layer.on("mouseover", function (e) {
          //@ts-ignore
          this.openPopup();
        });
        //@ts-ignore
        layer.on("mouseout", function (e) {
          //@ts-ignore
          //this.closePopup();
        });
        //@ts-ignore
        layer.on("click", function (e) {
          //alert("クリックされました");
        });
      }
    },
  }).addTo(map);
});
</script>

<style scoped>
#map {
  height: 100vh;
  width: 100%;
}
</style>
