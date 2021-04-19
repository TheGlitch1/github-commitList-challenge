<template>
  <div class="hello">
    <div>
      <b-button v-b-modal.modal-1 variant="primary">Authorize github</b-button>

      <b-modal id="modal-1" title="BootstrapVue">
        <div class="row justify-content-center">
          <div class="col-8">
            <input class="col-12 m-2" type="text" v-model="username" />
            <button
              class="btn btn-primary"
              @click="$router.push('commitList/' + username)"
            >
              Enter
            </button>
          </div>
        </div>
      </b-modal>
    </div>

    <div class="row"></div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      username: "roji",
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
