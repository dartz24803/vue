<template>
  <div>
    <b-table striped hover :items="clientes" :fields="fields" tbody-class="table-info" thead-class="table-dark" bordered>
      <template #cell(nombre)="row">
        {{ row.item.name }}
      </template>
      <template #cell(birthday)="row">
        {{ row.item.dob }}
      </template>
      <template #cell(phone)="row">
        {{ row.item.phone }}
      </template>
      <template #cell(email)="row">
        {{ row.item.email }}
      </template>
      <template #cell(address)="row">
        {{ row.item.address }}
      </template>
      <template #cell(payments)="row">
        {{ row.item.npayments }}
      </template>
      <template #cell(total)="row">
        {{ row.item.total }}
      </template>
      <template #cell(acciones)="row">
        <b-button variant="success" @click="editarCliente(row.item.id)">
          Editar <i class="fas fa-edit"></i>
        </b-button>
        <b-button variant="danger" @click="eliminarCliente(row.item.id)">
          Eliminar <i class="fas fa-trash"></i>
        </b-button>
      </template>
    </b-table>
    
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
      fields: [
        { key: 'nombre', label: 'Nombre' },
        { key: 'birthday', label: 'Birthday' },
        { key: 'phone', label: 'Phone' },
        { key: 'email', label: 'Email' },
        { key: 'address', label: 'Address' },
        { key: 'payments', label: 'Payments' },
        { key: 'total', label: 'Total' },
        { key: 'acciones', label: 'Acciones' }
      ],
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
          console.log(this.clientes)
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
          this.fetchClientes();
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>
