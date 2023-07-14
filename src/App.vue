<template>
  <div>
    <b-card name="card">
      <div class="text-center">
        <img
          alt="Vue logo"
          class="text-center border border-success"
          src="./assets/logo.png"
        />
      </div>
      <div class="d-flex justify-content-between mb-4">
        <titulo-vue />
        <b-button @click="openModal()" variant="primary" class="btn btn-success"
          ><i class="fas fa-add fa-2x">ADD CLIENT</i></b-button
        >
      </div>
      <!--Declarar variable y emitir modal -->
      <ModalVue v-if="ModalVisible" @hidden="ModalVisible = false" @onSubmit="onSubmit" />
      <TablaComponente :key="tableController" />
    </b-card>
  </div>
</template>

<script>
import TablaComponente from "./components/TablaComponente.vue";
import "bootstrap/dist/css/bootstrap.css";
import TituloVue from "./components/TituloVue.vue";
import ModalVue from "./components/ModalVue.vue";

export default {
  name: "App",
  components: {
    TablaComponente,
    TituloVue,
    ModalVue,
  },
  methods: {
    openModal() {
      this.ModalVisible = true;
    },
    onSubmit(){
      this.tableController++
    }
  },
  data() {
    return {
      ModalVisible: false,
      price: "",
      money: {
        decimal: ",",
        thousands: ".",
        prefix: "$ ",
        suffix: "",
        precision: 2,
        masked: true,
      },
      tableController: 1
    };
  },
};
</script>