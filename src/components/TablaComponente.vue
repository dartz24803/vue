<template>
  <div>
    <table class="table">
      <thead class="table-dark">
        <tr>
          <th scope="col">Nombre</th>
          <th scope="col">Birthday</th>
          <th scope="col">Phone</th>
          <th scope="col">Email</th>
          <th scope="col">Address</th>
          <th scope="col">Payments</th>
          <th scope="col">Total</th>
          <th scope="col">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="cliente in clientes" :key="cliente.id">
          <td>{{ cliente.name }}</td>
          <td>{{ cliente.dob }}</td>
          <td>{{ cliente.phone }}</td>
          <td>{{ cliente.email }}</td>
          <td>{{ cliente.address }}</td>
          <td>{{ cliente.npayments }}</td>
          <td>{{ cliente.total }}</td>
          <td>
            <b-button variant="success" @click="editarCliente(cliente.id)"
              >Editar <i class="fas fa-edit"></i></b-button
            >&nbsp;&nbsp;
            <b-button variant="danger" @click="eliminarCliente(cliente.id)"
              >Eliminar <i class="fas fa-trash"></i></b-button
            >
          </td>
        </tr>
      </tbody>
    </table>
    <ModalVue
      v-if="ModalVisible"
      @hidden="ModalVisible = false"
      :paymentsData="modalData"
      :clientSelect="clientSelect"
      :paymentsSelect="paymentsSelect"
    />
  </div>
</template>

<script>
import axios from "axios";
import ModalVue from "./ModalVue.vue";

export default {
  components: {
    ModalVue,
  },
  data() {
    return {
      clientes: [],
      payments: [],
      borrados: [],
      ModalVisible: false,
      modalData: null,
    };
  },
  mounted() {
    this.fetchClientes();
  },
  methods: {
    fetchClientes() {
      axios
        .get("http://127.0.0.1:8000/api/clientes")
        .then((response) => {
          this.clientes = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    editarCliente(clienteId) {
     // this.$emit("hidden");
      axios
        .get(`http://127.0.0.1:8000/api/buscar/${clienteId}`)
        .then((response) => {
          this.clientSelect = response.data[0];
          this.paymentsSelect = this.clientSelect.payments; 
          console.log(this.payments);
          this.ModalVisible = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    eliminarCliente(clienteId) {
      axios
        .post(`http://127.0.0.1:8000/api/eliminar/${clienteId}`)
        .then((response) => {
          this.clientes = response.data;
          window.location.reload();
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>
