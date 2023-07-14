<template>
  <div>
    <b-table
      striped
      hover
      :items="clientes"
      :fields="fields"
      tbody-class="table-info"
      thead-class="table-dark"
      bordered
      :key="tableController"
    >
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
          Editar <i class="fas fa-edit"></i> </b-button
        >
        <b-button variant="danger" @click="mostrarConfirmacion(row.item.id)">
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
      @onSubmit="onSubmit"
    />
  </div>
</template>

<script>
import axios from "axios";
import ModalVue from "./ModalVue.vue";
import Swal from "sweetalert2/dist/sweetalert2.js";
import "sweetalert2/dist/sweetalert2.css";

export default {
  components: {
    ModalVue,
  },
  data() {
    return {
      tableController: 1,
      fields: [
        { key: "nombre", label: "Nombre" },
        { key: "birthday", label: "Birthday" },
        { key: "phone", label: "Phone" },
        { key: "email", label: "Email" },
        { key: "address", label: "Address" },
        { key: "payments", label: "Payments" },
        { key: "total", label: "Total" },
        { key: "acciones", label: "Acciones" },
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
    mostrarConfirmacion(clienteId) {
      Swal.fire({
        title: "¿Estás seguro?",
        text: "Esta acción no se puede deshacer",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Sí, eliminar",
        cancelButtonText: "Cancelar",
      }).then((result) => {
        if (result.isConfirmed) {
          // Acción si el usuario confirma
          this.eliminarCliente(clienteId);
          Swal.fire({
            position: "center",
            icon: "success",
            title: "Logrado!",
            showConfirmButton: false,
            timer: 1500,
          });
        }
      });
    },
    async onSubmit() {
      await this.fetchClientes();
    },

    async fetchClientes() {
      try {
        const { data } = await axios.get("http://127.0.0.1:8000/api/clientes");
        this.clientes = data;
      } catch (error) {
        console.log(error);
      }
    },
    async editarCliente(clienteId) {
      try {
        const { data } = await axios.get(
          `http://127.0.0.1:8000/api/buscar/${clienteId}`
        );
        this.clientSelect = data[0];
        this.paymentsSelect = this.clientSelect.payments;
        this.ModalVisible = true;
      } catch (error) {
        console.log(error);
      }
    },
    async eliminarCliente(clienteId) {
      try {
        const { data } = await axios.post(
          `http://127.0.0.1:8000/api/eliminar/${clienteId}`
        );
        this.clientes = data;
        this.fetchClientes();
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
