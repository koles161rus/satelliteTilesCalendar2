<template>
  <div class="calendar" @scroll="onScroll">
    <h3>Календарь снимков</h3>
    <div
      v-for="(item, index) in dates"
      :key="index"
      class="calendar__item"
      :ref="`item${index + 1}`"
      @click="handleClick(item.date, index)"
    >
      <img :src="item.src" class="calendar__item-img" />
      <span class="calendar__item-date">{{
        item.date.toLocaleDateString()
      }}</span>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-debugger */
/* eslint-disable no-unused-vars */
export default {
  name: "Calendar",
  data() {
    return {
      dates: [],
    };
  },
  methods: {
    onScroll({ target: { scrollTop, clientHeight, scrollHeight } }) {
      if (Math.ceil(scrollTop) + clientHeight >= scrollHeight) {
        this.getLastDates(this.dates[this.dates.length - 1].date);
      }
    },
    getLastDates(date) {
      for (let i = 1; i < 8; i++) {
        let newDate = new Date(date.getTime());
        newDate.setDate(date.getDate() - i);
        const src =
          "https://gibs-b.earthdata.nasa.gov/wmts/epsg3857/best/MODIS_Terra_CorrectedReflectance_TrueColor/default/" +
          newDate.toLocaleDateString().split(".").reverse().join("-") +
          "/GoogleMapsCompatible_Level9/0/0/0.jpg";

        this.dates.push({ date: newDate, src });
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

    &-img {
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
