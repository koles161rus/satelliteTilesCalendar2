<template>
  <div id="app">
    <Calendar @handleAddDay="handleAddDay" />
    <Map :date="date" />
  </div>
</template>

<script>
import Calendar from "./components/Calendar.vue";
import Map from "./components/Map.vue";

export default {
  name: "App",
  components: {
    Calendar,
    Map,
  },
  data() {
    return {
      date: "",
    };
  },
  methods: {
    handleAddDay(date) {
      this.date = date.toLocaleDateString().split(".").reverse().join("-");
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
}

body {
  margin: 0;
  overflow: hidden;
}
</style>
