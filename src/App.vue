<template>
  <v-app>
    <v-main>
      <Form />
    </v-main>
  </v-app>
</template>

<script>
import Form from "./components/Form";
export default {
  name: "App",
  components: {
    Form,
  },
  data: () => ({
    //
  }),
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  background-color: #fafafa;
  padding: 24px;
  box-sizing: border-box;
}

html,
body,
#app {
  height: 100%;
}

* {
  box-sizing: border-box;
}
</style>
