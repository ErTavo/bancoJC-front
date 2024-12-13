<template>
  <div>
    <h2>Transferir desde la cuenta: {{ accountNo }}</h2>

    <form @submit.prevent="submitTransfer">
      <div>
        <label for="toAccount">Cuenta destino (8 dígitos):</label>
        <input
          type="text"
          id="toAccount"
          v-model="toAccount"
          maxlength="8"
          placeholder="12345678"
          required
        />
      </div>

      <div>
        <label for="amount">Monto (decimal):</label>
        <input
          type="number"
          id="amount"
          v-model.number="amount"
          step="0.01"
          min="0.01"
          placeholder="100.00"
          required
        />
      </div>

      <button type="submit" :disabled="isLoading">
        {{ isLoading ? 'Cargando...' : 'Transferir' }}
      </button>
    </form>

    <button @click="goBack">Regresar a la cuenta</button>

    <h3>Historial de transferencias</h3>
    <div v-if="transferHistory.length > 0">
      <div v-for="transfer in transferHistory" :key="transfer.id" class="transfer-card">
        <p><strong>Fecha:</strong> {{ transfer.created_at }}</p>
        <p :class="{ sent: transfer.type === 'enviada', received: transfer.type === 'recibida' }">
          <strong>Monto:</strong> {{ transfer.amount }}
        </p>
        <p>
          <strong>
            {{ transfer.type === 'enviada' ? 'Enviada a:' : 'Recibida de:' }}
          </strong>
          {{ transfer.relatedAccount }}
        </p>
      </div>
    </div>
    <p v-else>Cargando historial...</p>
  </div>
</template>

<script>
import Swal from 'sweetalert2';

export default {
  props: {
    accountNo: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      toAccount: '',
      amount: null,
      transferHistory: [],
      isLoading: false,
    };
  },
  methods: {
    async submitTransfer() {
      if (!/^\d{8}$/.test(this.toAccount)) {
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: 'La cuenta destino debe tener exactamente 8 dígitos.',
        });
        return;
      }

      try {
        const username = localStorage.getItem('username');
        if (!username) {
          throw new Error('Usuario no encontrado en el almacenamiento local.');
        }

        this.isLoading = true;

        const otpResponse = await fetch('https://banco-jc-back.vercel.app/user/otp', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ username }),
        });

        if (!otpResponse.ok) {
          throw new Error('Error al solicitar el código OTP.');
        }

        const { value: otp } = await Swal.fire({
          title: 'Ingrese el código OTP',
          input: 'text',
          inputLabel: 'OTP',
          inputPlaceholder: 'Ingrese su código OTP',
          inputValidator: (value) => {
            if (!value) {
              return 'Debe ingresar el código OTP';
            }
          },
          showCancelButton: true,
        });

        if (!otp) {
          Swal.fire({
            icon: 'info',
            title: 'Cancelado',
            text: 'La transferencia ha sido cancelada.',
          });
          this.isLoading = false;
          return;
        }

        Swal.fire({
          title: 'Verificando OTP...',
          allowOutsideClick: false,
          didOpen: () => {
            Swal.showLoading();
          },
        });

        const validateOtpResponse = await fetch('https://banco-jc-back.vercel.app/user/validateOtp', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ username, otp }),
        });

        if (!validateOtpResponse.ok) {
          throw new Error('Código OTP inválido o expirado.');
        }

        Swal.fire({
          title: 'Procesando transferencia...',
          allowOutsideClick: false,
          didOpen: () => {
            Swal.showLoading();
          },
        });

        const transferResponse = await fetch('https://banco-jc-back.vercel.app/transfers/newTransfer', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            fromAccount: this.accountNo,
            toAccount: this.toAccount,
            amount: this.amount,
          }),
        });

        if (!transferResponse.ok) {
          throw new Error('Error al realizar la transferencia.');
        }

        await transferResponse.json();

        Swal.fire({
          icon: 'success',
          title: '¡Éxito!',
          text: 'Transferencia realizada con éxito.',
        });

        this.toAccount = '';
        this.amount = null;
        this.fetchTransferHistory();
      } catch (error) {
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: error.message,
        });
      } finally {
        this.isLoading = false;
      }
    },
    async fetchTransferHistory() {
      try {
        const response = await fetch('https://banco-jc-back.vercel.app/transfers/history', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ accountNo: this.accountNo }),
        });

        if (!response.ok) {
          throw new Error('Error al obtener el historial de transferencias.');
        }

        this.transferHistory = await response.json();
      } catch (error) {
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: error.message,
        });
      }
    },
    goBack() {
      this.$emit('back');
    },
  },
  mounted() {
    this.fetchTransferHistory();
  },
};
</script>

<style scoped>
form {
  margin-bottom: 20px;
}

div {
  margin-bottom: 10px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 10px 15px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.transfer-card {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 10px;
}

.sent {
  color: red;
}

.received {
  color: green;
}
</style>
