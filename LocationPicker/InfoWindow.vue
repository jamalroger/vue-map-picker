<template>
  <div>
    <strong>{{ title }}</strong>
    <p v-for="(item, index) in body" :key="index">{{ item }}</p>
  </div>
</template>


<script>
export default {
  data() {
    return {
      title: "Note",
      body: ["Click and drag this marker to pick your location."],
    };
  },
  methods: {
    showAddress(place) {
      this.title = "";
      this.body = place.formatted_address.split(",").map((item) => item.trim());
    },
    showError() {
      this.title = "Oups";
      this.body = [
        "Google Maps could not determine the approximate postal address of this location.",
      ];
    },
  },
};
</script>


<style scoped>
div {
  font-family: monospace;
  color: #333;
}
p {
  margin-bottom: 0;
}
p:not(:first-of-type) {
  margin-top: 6px;
}
</style>