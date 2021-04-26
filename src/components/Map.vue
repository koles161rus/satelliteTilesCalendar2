<template>
  <div>
    <div class="map" ref="mapContainer" />
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  name: "Map",
  props: {
    date: String,
  },
  data() {
    return {
      map: null,
    };
  },
  watch: {
    date() {
      this.mapInit();
    },
  },
  methods: {
    mapInit() {
      if (this.map) {
        this.map.remove();
      }

      const tilePath =
        "wmts/epsg3857/best/" +
        "MODIS_Terra_CorrectedReflectance_TrueColor/default/" +
        `${this.date}/GoogleMapsCompatible_Level9/{z}/{y}/{x}.jpg`;

      mapboxgl.accessToken =
        "pk.eyJ1IjoiYnVyb3Z5YSIsImEiOiJjanVucnE3bHMweHRlM3pvNXAycXllaHl5In0.ytKUDnITJq8JScaXHW3qzQ";

      this.map = new mapboxgl.Map({
        container: this.$refs.mapContainer,
        style: {
          version: 8,
          sources: {
            gibs: {
              type: "raster",
              tiles: [
                "https://gibs-a.earthdata.nasa.gov/" + tilePath,
                "https://gibs-b.earthdata.nasa.gov/" + tilePath,
                "https://gibs-c.earthdata.nasa.gov/" + tilePath,
              ],
              tileSize: 256,
            },
          },
          layers: [
            {
              id: "gibs",
              type: "raster",
              source: "gibs",
            },
          ],
        },
        maxZoom: 8,
      });
    },
  },
  mounted() {
    this.mapInit();
  },
};
</script>

<style scoped lang="scss">
.map {
  position: absolute;
  width: 80%;
  height: 100%;
  right: 0;
}
</style>
