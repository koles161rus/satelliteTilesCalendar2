<template>
  <div class="calendar" @scroll="onScroll">
    <h3>Календарь снимков</h3>
    <div
      v-for="(date, index) in dates"
      :key="index"
      class="calendar__item"
      :ref="`item${index + 1}`"
      @click="handleClick(date, index)"
    >
      <div class="calendar__item-map" :ref="`mapContainer${index + 1}`" />
      <span class="calendar__item-date">{{ date.toLocaleDateString() }}</span>
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  name: "Calendar",
  data() {
    return {
      maps: [],
      dates: [],
      lastScrollTop: 0,
      lastTopEl: 0,
      lastBottomEl: 0,
    };
  },
  updated() {
    this.dates.forEach((date, i) => {
      if (i > this.dates.length - 8) {
        this.mapInit(date, i);
      }
    });
  },
  methods: {
    mapInit(date, i) {
      mapboxgl.accessToken =
        "pk.eyJ1IjoiYnVyb3Z5YSIsImEiOiJjanVucnE3bHMweHRlM3pvNXAycXllaHl5In0.ytKUDnITJq8JScaXHW3qzQ";

      const tilePath =
        "wmts/epsg3857/best/" +
        "MODIS_Terra_CorrectedReflectance_TrueColor/default/" +
        `${date
          .toLocaleDateString()
          .split(".")
          .reverse()
          .join("-")}/GoogleMapsCompatible_Level9/{z}/{y}/{x}.jpg`;

      const map = new mapboxgl.Map({
        container: this.$refs[`mapContainer${i + 1}`][0],
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
        zoom: 0,
        dragPan: false,
        scrollZoom: false,
        doubleClickZoom: false,
      });

      map.getCanvasContainer().style.cursor = "pointer";
      if (i < this.maps.length) {
        this.maps.splice(i, 1, map);
      } else {
        this.maps.push(map);
      }
    },
    onScroll({ target: { scrollTop, clientHeight, scrollHeight } }) {
      if (Math.ceil(scrollTop) + clientHeight >= scrollHeight) {
        this.getLastDates(this.dates[this.dates.length - 1]);

        this.maps.forEach((map, i) => {
          if (map && i < this.maps.length - 7) {
            map.remove();
            this.maps.splice(i, 1, null);
          }
        });
      }

      const itemScrollHeight = Math.ceil(scrollHeight / this.dates.length);
      if (scrollTop > this.lastScrollTop) {
        for (let i = 0; i < this.dates.length; i++) {
          if (Math.ceil(scrollTop) > itemScrollHeight * i) {
            this.lastBottomEl = i;
          }
        }

        this.maps.forEach((map, i) => {
          if (i === this.lastBottomEl - 10 && map) {
            map.remove();
            this.maps.splice(i, 1, null);
          }
        });

        this.maps.forEach((map, i) => {
          if (i === this.lastBottomEl + 4 && !map) {
            this.mapInit(this.dates[i], i);
          }
        });
      } else {
        for (let i = 13; i < this.dates.length; i++) {
          if (Math.ceil(scrollTop) < scrollHeight - itemScrollHeight * i) {
            this.lastTopEl = i;
          }
        }

        this.maps.forEach((map, i) => {
          if (i === this.dates.length - this.lastTopEl + 14 && map) {
            map.remove();
            this.maps.splice(i, 1, null);
          }
        });

        this.maps.forEach((map, i) => {
          if (i === this.dates.length - this.lastTopEl - 1 && !map) {
            this.mapInit(this.dates[i], i);
          }
        });
      }

      this.maps.forEach((map, i) => {
        if (map && !this.maps[i - 1] && !this.maps[i + 1]) {
          map.remove();
          this.maps.splice(i, 1, null);
        }
        if (!map && this.maps[i - 1] && this.maps[i + 1]) {
          this.mapInit(this.dates[i], i);
        }
      });

      this.lastScrollTop = scrollTop <= 0 ? 0 : scrollTop;
    },
    getLastDates(date) {
      for (let i = 1; i < 8; i++) {
        let newDate = new Date(date.getTime());
        newDate.setDate(date.getDate() - i);
        this.dates.push(newDate);
      }
    },
    handleClick(date, index) {
      this.$emit("handleAddDay", date);
      Object.values(this.$refs).forEach((container) => {
        container[0].style.border = "";
      });
      this.$refs[`item${index + 1}`][0].style.border = "2px solid #00f";
    },
  },
  mounted() {
    const currentDate = new Date();
    this.$emit("handleAddDay", currentDate);
    this.getLastDates(currentDate);
  },
};
</script>

<style scoped lang="scss">
.calendar {
  width: 20%;
  height: 100vh;
  background-color: #000;
  color: #fff;
  overflow-y: auto;
  overflow-x: hidden;
  scrollbar-width: thin;

  &__item {
    width: 280px;
    height: 200px;
    margin: 10px;
    position: relative;
    display: inline-block;
    cursor: pointer;

    &-map {
      width: 280px;
      height: 200px;
    }

    &-date {
      position: absolute;
      top: 0;
      left: 0;
      background: #000;
    }
  }

  &::-webkit-scrollbar {
    width: 8px;
  }

  &::-webkit-scrollbar-track {
    background: #000;
  }

  &::-webkit-scrollbar-thumb {
    background: #787878;
    border: 1px solid #000;
  }
}
</style>
