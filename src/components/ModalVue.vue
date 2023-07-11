<template>
  <b-modal
    id="modal-lg"
    size="lg"
    ref="ModalClient"
    @hidden="$emit('hidden')"
    hide-footer
    title="ADD CLIENT"
  >
    <!-- Contenido del modal -->
    <!-- Formulario -->
    <form @submit="handleSubmit">
      <h3>Personal information <i class="fas fa-user-plus"></i></h3>
      <input type="hidden" id="id" v-model="client.id" /><br />
      <ValidationProvider v-slot="v" rules="required|alpha" class="mb-4">
        <span class="text-danger mb-4">{{ v.errors[0] }}</span>
        <b-form-input
          type="text"
          id="name"
          v-model="client.name"
          placeholder="Name"
          :class="{ 'is-invalid': v.errors.length > 0 }"
          required
        /><br />
      </ValidationProvider>
      <ValidationProvider v-slot="v" rules="required" class="mb-4">
        <span class="text-danger mb-4">{{ v.errors[0] }}</span>
        <b-form-input
          type="date"
          id="dob"
          v-model="client.dob"
          :class="{ 'is-invalid': v.errors.length > 0 }"
          required
        /><br />
      </ValidationProvider>
      <ValidationProvider
        v-slot="v"
        rules="required|positive|length:9"
        class="mb-4"
      >
        <span class="text-danger mb-4">{{ v.errors[0] }}</span>
        <b-form-input
          type="number"
          id="phone"
          v-model="client.phone"
          placeholder="Phone"
          :class="{ 'is-invalid': v.errors.length > 0 }"
          required
        /><br />
      </ValidationProvider>
      <ValidationProvider v-slot="v" rules="required|email" class="mb-4">
        <span class="text-danger mb-4">{{ v.errors[0] }}</span>
        <b-form-input
          type="email"
          id="email"
          v-model="client.email"
          placeholder="Email"
          :class="{ 'is-invalid': v.errors.length > 0 }"
          required
        /><br />
      </ValidationProvider>
      <ValidationProvider v-slot="v" rules="required" class="mb-4">
        <span class="text-danger mb-4">{{ v.errors[0] }}</span>
        <b-form-input
          type="text"
          id="address"
          v-model="client.address"
          placeholder="Address"
          :class="{ 'is-invalid': v.errors.length > 0 }"
          required
        /><br />
      </ValidationProvider>
      <div class="d-flex justify-content-between">
        <h5>Payments <i class="fas fa-dollar-sign"></i></h5>
        <b-button @click="addNewPayment" variant="dark" :disabled="!areAllFieldsFilled"
          >ADD &nbsp;&nbsp;<i class="fas fa-circle-plus"></i
        ></b-button>
      </div>
      <div class="my-4" v-for="(item, index) in payments" :key="index">
        <input type="hidden" v-model="item.COD" /><br />
        <ValidationProvider v-slot="v" rules="required|positive" class="mb-4">
          <span class="text-danger mb-4">{{ v.errors[0] }}</span>
          <input
            class="form-control"
            type="number"
            v-model="item.id"
            placeholder="Transaction"
            :class="{ 'is-invalid': v.errors.length > 0 }"
            required
          /><br />
        </ValidationProvider>
        <ValidationProvider v-slot="v" rules="required|positive" class="mb-4">
          <span class="text-danger mb-4">{{ v.errors[0] }}</span>
          <input
            type="number"
            class="form-control"
            v-model="item.amount"
            placeholder="Amount"
            :class="{ 'is-invalid': v.errors.length > 0 }"
            required
          /><br />
        </ValidationProvider>
        <ValidationProvider v-slot="v" rules="required" class="mb-4">
          <span class="text-danger mb-4">{{ v.errors[0] }}</span>
          <b-form-input
            type="date"
            id="dob"
            v-model="item.date"
            :class="{ 'is-invalid': v.errors.length > 0 }"
            required
          /><br />
        </ValidationProvider>
        <b-button variant="dark" @click="deletePayment(index, item.COD)"
          >Eliminar <i class="fas fa-delete-left"></i
        ></b-button>
        <hr />
      </div>
      <div class="text-center">
        <b-button type="submit" variant="dark" style="width: 100%"
          >SAVE &nbsp;&nbsp;<i class="fas fa-arrow-right"></i
        ></b-button>
      </div>
    </form>
  </b-modal>
</template>
<script>
import axios from "axios";
import { ValidationProvider } from "vee-validate";
import { extend } from "vee-validate";
import { required, alpha, email, length } from "vee-validate/dist/rules";

// Override the default message.
extend("length", {
  ...length,
  message: "This field requires 9 numbers",
});

// Override the default message.
extend("email", {
  ...email,
  message: "This field is email valid",
});

// Override the default message.
extend("alpha", {
  ...alpha,
  message: "This field is only letters",
});

// Override the default message.
extend("required", {
  ...required,
  message: "This field is required",
});
extend("positive", {
  validate: (value) => {
    return value >= 0;
  },
  message: "This field accept positive numbers only",
});

export default {
  name: "ModalVue",
  components: {
    ValidationProvider,
  },
  computed: {
    areAllFieldsFilled() {
      for (const item of this.payments) {
        if (!item.id || !item.amount || !item.date) {
          return false;
        }
      }
      return true;
    },
  },
  props: ["clientSelect", "paymentsSelect"],
  data() {
    return {
      modalVisible: false,
      payments: [{ cod: 0, id: null, amount: null, date: null }],
      client: {
        id: null,
        address: null,
        name: null,
        phone: null,
        email: null,
      },
    };
  },
  methods: {
    // Función para abrir el modal
    openModal() {
      this.modalVisible = true;
      const paymentsData = {
        payments: this.getPaymentsLength(),
        total: this.getTotal(),
      };
      this.$emit("open-modal", paymentsData);
    },
    handleSubmit(event) {
      event.preventDefault();
      // Convertir los valores de id a enteros en los pagos
      const parsedPayments = this.payments.map((payment) => ({
        cod: parseInt(payment.COD),
        id: parseInt(payment.id),
        amount: parseInt(payment.amount),
        date: new Date(payment.date).toISOString().slice(0, 10), // Corregir el formato de fecha
      }));
      // Crear un objeto con los datos del formulario
      const formData = {
        id: this.client.id,
        name: this.client.name,
        dob: this.client.dob,
        phone: this.client.phone,
        email: this.client.email,
        address: this.client.address,
        payments: parsedPayments,
      };
      var elementos = document.querySelectorAll(".text-danger.mb-4");
      var valores = [];

      for (let i = 0; i < elementos.length; i++) {
        valores.push(elementos[i].innerHTML.trim());
      }

      if (valores.every((valor) => valor === "")) {
        axios
          .post("http://127.0.0.1:8000/api/crearcliente", formData)
          .then((response) => {
            const client = response.data;
            console.log(client);
            this.modalVisible = false;
            this.$emit("hidden");

            window.location.reload();
          })
          .catch((error) => {
            // Manejar el error si ocurre
            console.error(error);
          });
      } else {
        alert("Revisar formulario");
      }
    },

    addNewPayment() {
      this.payments.push({ cod: null, id: null, amount: null, date: null });
    },

    getPaymentsLength() {
      return this.payments.length;
    },
    obtenerTotal(total) {
      console.log(total);
    },

    deletePayment(index, COD) {
      this.payments.splice(index, 1);
      if (COD > 0) {
        axios
          .post(`http://127.0.0.1:8000/api/eliminarpago/${COD}`)
          .then((response) => {
            const client = response.data;
            console.log(client);
          })
          .catch((error) => {
            // Manejar el error si ocurre
            console.error(error);
          });
      }
    },
  },
  // Mostrar el modal al montar el componente
  mounted() {
    this.$nextTick(() => {
      this.$refs.ModalClient.show();
    });

    if (this.clientSelect) {
      this.client = this.clientSelect;
    }
    if (this.paymentsSelect) {
      this.payments = this.paymentsSelect;
    }
  },
};
</script>
<style>
.is-invalid {
  border-color: red;
}
</style>
