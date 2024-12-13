<template>
  <div>
    <h2>Estado de la Cuenta</h2>
    <div class="actions">
      <button @click="logout" class="logout-btn">Cerrar Sesión</button>
      <button @click="openCreateAccountForm" class="create-account-btn">Crear Cuenta</button>
    </div>
    <p v-if="loading">Cargando datos...</p>
    <p v-else-if="errorMessage">{{ errorMessage }}</p>
    <div v-else>
      <div v-for="(account, index) in accounts" :key="index" class="account-card">
        <div class="account-details">
          <div class="account-type">{{ account.type }}</div>
          <div class="account-no">{{ account.accountNo }}</div>
        </div>
        <div class="balance-container">
          <div class="currency">{{ account.currency }}</div>
          <div class="balance">{{ account.balance }}</div>
        </div>

        <div class="buttons-container">
          <button @click="openDepositForm(account)" class="deposit-btn" title="Depositar">&#8593;</button>
          <button @click="goToTransfer(account)" class="transfer-btn" title="Transferir">&#8594;</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2';
export default {
  data() {
    return {
      accounts: [],
      loading: true,
      errorMessage: null,
      username: localStorage.getItem('username'), 
    };
  },
  methods: {
    logout() {
      localStorage.clear(); // Limpia todo el localStorage
      window.location.href = '/login'; // Redirige al componente de login
    },
    goToTransfer(account) {
      this.$emit('goToTransfer', account.accountNo);  
    },
    openCreateAccountForm() {
      Swal.fire({
        title: 'Crear nueva cuenta',
        html: `
          <form id="create-account-form">
            <div class="form-group">
              <label for="balance">Balance</label>
              <input type="number" id="balance" class="swal2-input" placeholder="Ej. 5000.00" required>
            </div>
            <div class="form-group">
              <label for="type">Tipo de cuenta</label>
              <select id="type" class="swal2-input">
                <option value="Ahorro">Ahorro</option>
                <option value="Monetaria">Monetaria</option>
              </select>
            </div>
            <div class="form-group">
              <label for="currency">Moneda</label>
              <select id="currency" class="swal2-input">
                <option value="Q">Q</option>
                <option value="$">$</option>
              </select>
            </div>
          </form>
        `,
        focusConfirm: false,
        preConfirm: () => {
          const balance = document.getElementById('balance').value;
          const type = document.getElementById('type').value;
          const currency = document.getElementById('currency').value;
          
          if (!balance || !type || !currency) {
            Swal.showValidationMessage('Por favor, complete todos los campos.');
            return false;
          }

          return { balance, type, currency };
        }
      }).then((result) => {
        if (result.isConfirmed) {
          this.createAccount(result.value);
        }
      });
    },
    
    async createAccount({ balance, type, currency }) {
      try {
        const response = await fetch('https://banco-jc-back.vercel.app/account/createAccount', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            username: this.username, 
            balance: parseFloat(balance), 
            type,
            currency,
          }),
        });

        if (response.ok) {
          const newAccount = await response.json();
          this.accounts.push(newAccount); 
          Swal.fire('Éxito', 'La cuenta se ha creado correctamente.', 'success');
        } else {
          Swal.fire('Error', 'No se pudo crear la cuenta. Intente nuevamente.', 'error');
        }
      } catch (error) {
        Swal.fire('Error', 'Hubo un problema al comunicarse con el servidor.', 'error');
      }
    },
    openDepositForm(account) {
      Swal.fire({
        title: 'Depositar en la cuenta',
        html: `
          <div class="form-group">
            <label for="abono">Monto a depositar</label>
            <input type="number" id="abono" class="swal2-input" placeholder="Ej. 1000.00" required>
          </div>
        `,
        focusConfirm: false,
        preConfirm: () => {
          const abono = document.getElementById('abono').value;
          if (!abono) {
            Swal.showValidationMessage('Por favor, ingrese un monto válido.');
            return false;
          }
          return { abono: parseFloat(abono) };
        }
      }).then((result) => {
        if (result.isConfirmed) {
          this.depositToAccount(account.accountNo, result.value.abono);
        }
      });
    },
    async depositToAccount(accountNo, abono) {
      try {
        const response = await fetch('https://banco-jc-back.vercel.app/account/deposito', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ accountNo, abono }),
        });

        if (response.ok) {
          Swal.fire('Éxito', 'El depósito se realizó correctamente.', 'success');
          const updatedAccount = await response.json();
          const accountIndex = this.accounts.findIndex(acc => acc.accountNo === accountNo);
          if (accountIndex !== -1) {
            this.accounts[accountIndex].balance = updatedAccount.balance;
          }
        } else {
          Swal.fire('Error', 'No se pudo realizar el depósito. Intente nuevamente.', 'error');
        }
      } catch (error) {
        Swal.fire('Error', 'Hubo un problema al comunicarse con el servidor.', 'error');
      }
    }
  },
  async mounted() {
    const hash = localStorage.getItem('authHash');

    if (!hash) {
      this.errorMessage = 'Sesión expiró. Por favor, inicie sesión de nuevo.';
      this.loading = false;
      return;
    }

    try {
      const validationResponse = await fetch('https://banco-jc-back.vercel.app/user/validateUserHash', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username: this.username, hash }),
      });

      if (validationResponse.ok) {
        const validationData = await validationResponse.json();
        if (validationData.success) {
          const response = await fetch('https://banco-jc-back.vercel.app/account/accountBalance', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username: this.username }),
          });

          if (response.ok) {
            const data = await response.json();
            if (Array.isArray(data)) {
              this.accounts = data;  
            } else {
              this.errorMessage = 'La respuesta no es válida';
            }
          } else {
            this.errorMessage = 'Error al obtener el balance';
          }
        } else {
          this.errorMessage = 'El hash ha expirado. Redirigiendo a la página de inicio de sesión.';
          setTimeout(() => {
            window.location.href = '/login';
          }, 3000);
        }
      } else {
        this.errorMessage = 'Error al verificar el hash';
      }
    } catch (error) {
      this.errorMessage = 'Error al conectar con el servidor';
    } finally {
      this.loading = false;
    }
  }
};
</script>





<style scoped>
.actions {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.logout-btn {
  background-color: #f71f1f;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 20px;
}

.logout-btn:hover {
  background-color: #d32f2f;
}

.create-account-btn:hover {
  background-color: #45a049;
}
.create-account-btn {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 20px;
}

form {
  display: flex;
  flex-direction: column;
}

.form-group {
  margin-bottom: 15px;
}

.swal2-input {
  padding: 10px;
  font-size: 14px;
  border-radius: 5px;
  border: 1px solid #ccc;
}
.account-card {
  background-color: #dfdee0;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.account-details {
  width: 100%;
  display: flex;
  flex-direction: column; 
  align-items: flex-start;
  margin-bottom: 10px;
}

.account-type {
  font-size: 16px;
  color: #0225c0;
  font-weight: bold;
  text-align: center;
  margin-bottom: 5px; 
}

.account-no {
  font-size: 18px;
  color: #555;
}

.balance-container {
  display: flex;
  justify-content: flex-end; 
  width: 100%;
  align-items: center;
}

.currency {
  font-size: 20px;
  font-weight: bold;
  color: #333;
}

.balance {
  font-size: 20px;
  font-weight: bold;
  color: #000000;
  margin-left: 5px;
}
.account-card {
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.transfer-btn {
  background: none;
  border: none;
  font-size: 30px;
  color: gray;
  cursor: pointer;
  align-self: center; 
  margin-top: 10px;
}

.transfer-btn:hover {
  color: #0225c0; 
}
</style>
