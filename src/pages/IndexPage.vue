<template>
  <div ref="mapDiv" style="width: 100%; height: 80vh"></div>
</template>

<script>
import { ref, onMounted } from "vue";
import { Loader } from "@googlemaps/js-api-loader";
import { dataFetch } from "../json/jsonFetch";
import { ParseDMS } from "../dataParse";

const myJson = dataFetch();
const G_API_KEY = "AIzaSyCSTKU-GNt-E7mBt196FvKNZSky5aOpA_A";

console.log(myJson[0].lat_geodesica);
console.log(myJson[0].lat_geodesica.split(/[^\d\w]+/));

export default {
  name: "App",
  setup() {
    const loader = new Loader({ apiKey: G_API_KEY, libraries: ["places","visualization"] });
    const mapDiv = ref(null);
    let map = ref(null);

    const myWayPoints = [];
    const myHeatMapPoints = [];

    function convertCodigo(cod) {
      var num = cod.substring(0, 3);
      var estado = cod.substring(4, 6);

      return estado + " - " + num;
    }

    myJson.map((element) => {
      if (element.classe != 0) {
        myWayPoints.push({
          location: {
            lat: ParseDMS(element.lat_geodesica),
            lng: ParseDMS(element.long_geodesica),
            weight: element.classe,
          },
          stopover: false,
          classe: element.classe,
        });
      }
    });

    myJson.forEach((element) => {
        myHeatMapPoints.push({
          location: {
            lat: ParseDMS(element.lat_geodesica),
            lng: ParseDMS(element.long_geodesica),
            weight: element.classe,
          },
          stopover: false,
          classe: element.classe,
        });
    });

    onMounted(async () => {
      await loader.load().then((google) => {
        map.value = new google.maps.Map(mapDiv.value, {
        center: {
          lat: ParseDMS(myJson[myJson.length - 1].lat_geodesica),
          lng: ParseDMS(myJson[myJson.length - 1].long_geodesica),
        },
        zoom: 7,
      });

      const gradient0 = [
          "rgba(0, 255, 0, 0)",
          "rgba(0, 255, 0, 1)",
        ];

      const gradient1 = [
        "rgba(255, 255, 0, 0)",
        "rgba(255, 255, 0, 1)",
      ];

      const gradient2 = [
        "rgba(255, 100, 0, 0)",
        "rgba(255, 100, 0, 1)",
      ];

      const gradient3 = [
        "rgba(255, 0, 0, 0)",
        "rgba(255, 0, 0, 1)",
      ];

      let heatmap0 = new google.maps.visualization.HeatmapLayer({
        data: myHeatMapPoints.filter(e => e.classe === 0).map(e =>
           {return {location: new google.maps.LatLng(e.location.lat, e.location.lng), weight: 2} }
        ),
        map: map.value,
        gradient: gradient0,
        dissipating: true
      });

      let heatmap1 = new google.maps.visualization.HeatmapLayer({
        data: myHeatMapPoints.filter(e => e.classe === 1).map(e =>
        {return {location: new google.maps.LatLng(e.location.lat, e.location.lng), weight: 2} }
        ),
        map: map.value,
        gradient: gradient1,
        dissipating: true
      });

      let heatmap2 = new google.maps.visualization.HeatmapLayer({
        data: myHeatMapPoints.filter(e => e.classe === 2).map(e =>
          {return {location: new google.maps.LatLng(e.location.lat, e.location.lng), weight: 2} }
        ),
        map: map.value,
        gradient: gradient2,
        dissipating: true
      });

      let heatmap3 = new google.maps.visualization.HeatmapLayer({
        data: myHeatMapPoints.filter(e => e.classe === 3).map(e =>
          {return {location: new google.maps.LatLng(e.location.lat, e.location.lng), weight: 2} }
        ),
        map: map.value,
        gradient: gradient3,
        dissipating: true,
        weight: 10
      });

      heatmap0.set('radius', 10)
      heatmap1.set('radius', 10)
      heatmap2.set('radius', 10)
      heatmap3.set('radius', 10)
      heatmap0.set('opacity', 1)
      heatmap1.set('opacity', 1)
      heatmap2.set('opacity', 1)
      heatmap3.set('opacity', 1)

      const directionRenderer = new google.maps.DirectionsRenderer({
        draggable: false,
        suppressMarkers: true,
      });

      const directionService = new google.maps.DirectionsService();

      directionRenderer.setMap(map.value);
      directionService
        .route({
          origin: {
            lat: ParseDMS(myJson[myJson.length - 1].lat_geodesica),
            lng: ParseDMS(myJson[myJson.length - 1].long_geodesica),
          },
          destination: {
            lat: ParseDMS(myJson[0].lat_geodesica),
            lng: ParseDMS(myJson[0].long_geodesica),
          },
          travelMode: google.maps.TravelMode.DRIVING,
        })
        .then((response) => {
          directionRenderer.setDirections(response);
        })
        .catch((err) => {
          console.log(err);
        });

      var pinColorY = "#FFFF00";
      var pinColorB = "#0000FF";

      // Pick your pin (hole or no hole)
      var pinSVGHole =
        "M12,11.5A2.5,2.5 0 0,1 9.5,9A2.5,2.5 0 0,1 12,6.5A2.5,2.5 0 0,1 14.5,9A2.5,2.5 0 0,1 12,11.5M12,2A7,7 0 0,0 5,9C5,14.25 12,22 12,22C12,22 19,14.25 19,9A7,7 0 0,0 12,2Z";
      var labelOriginHole = new google.maps.Point(12, 15);
      var pinSVGFilled =
        "M 12,2 C 8.1340068,2 5,5.1340068 5,9 c 0,5.25 7,13 7,13 0,0 7,-7.75 7,-13 0,-3.8659932 -3.134007,-7 -7,-7 z";
      var labelOriginFilled = new google.maps.Point(12, 9);

      var markerImageYellow = {
        // https://developers.google.com/maps/documentation/javascript/reference/marker#MarkerLabel
        path: pinSVGHole,
        anchor: new google.maps.Point(12, 17),
        fillOpacity: 2,
        fillColor: pinColorY,
        strokeWeight: 2,
        strokeColor: "black",
        scale: 1.5,
        labelOrigin: labelOriginFilled,
      };

      var markerImageBlue = {
        // https://developers.google.com/maps/documentation/javascript/reference/marker#MarkerLabel
        path: pinSVGHole,
        anchor: new google.maps.Point(12, 17),
        fillOpacity: 2,
        fillColor: pinColorB,
        strokeWeight: 2,
        strokeColor: "black",
        scale: 1.5,
        labelOrigin: labelOriginFilled,
      };

      myWayPoints.forEach((element) => {
        if (element.classe === 2) {
          let marker = new google.maps.Marker({
            position: element.location,
            map: map.value,
            draggable: false,
            icon: markerImageYellow,
          });
        } else {
          let marker = new google.maps.Marker({
            position: element.location,
            map: map.value,
            draggable: false,
            icon: markerImageBlue,
          });
        }
      });
      });

    });

    return { mapDiv };
  },
};
</script>
