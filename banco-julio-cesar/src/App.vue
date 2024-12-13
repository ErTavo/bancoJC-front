<template>
  <div id="app">
    <h1>Banco Julio Cesar</h1>
    <div>
      <LoginComponent 
        v-if="currentView === 'login'" 
        @loggedIn="handleLoginSuccess" 
      />
      <AccountStatus 
        v-if="currentView === 'account'" 
        :hash="userHash" 
        :username="username"
        @goToTransfer="goToTransfer" 
      />
      <TransferComponent 
        v-if="currentView === 'transfer'" 
        :accountNo="accountNo"
        @back="setView('account')" 
      />
    </div>
  </div>
</template>

<script>
import LoginComponent from './components/LoginComponent.vue';
import AccountStatus from './components/AccountStatus.vue';
import TransferComponent from './components/TransferComponent.vue';

export default {
  components: {
    LoginComponent,
    AccountStatus,
    TransferComponent,
  },
  data() {
    return {
      currentView: 'login',
      userHash: null,
      username: null,
      accountNo: null, 
    };
  },
  created() {
    this.checkSession();
  },
  methods: {
    async checkSession() {
      const hash = localStorage.getItem('authHash');
      const username = localStorage.getItem('username');

      if (hash && username) {
        try {
          const baseUrl = "https://banco-jc-back.vercel.app";
          const response = await fetch(`${baseUrl}/user/validateUserHash`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ username, hash }),
          });

          if (response.ok) {
            this.userHash = hash;
            this.username = username;
            this.setView('account');
          } else {
            localStorage.removeItem('authHash');
            localStorage.removeItem('username');
          }
        } catch (error) {
          console.error("Error al validar la sesión:", error);
          localStorage.removeItem('authHash');
          localStorage.removeItem('username');
        }
      }
    },
    handleLoginSuccess(hash) {
      this.userHash = hash; 
      this.username = localStorage.getItem('username');
      this.setView('account');
    },
    setView(view) {
      this.currentView = view;
    },
    goToTransfer(accountNo) {
      this.accountNo = accountNo; 
      this.setView('transfer');    
    }
  },
};
</script>



<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-top: 20px;
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 10px;
  max-width: 600px;
  margin: auto;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #007bff;
  margin-bottom: 20px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 5px;
  font-size: 1rem;
}

button:hover {
  background-color: #0056b3;
}

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 300px;
  margin: 0 auto;
}

input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}
</style>
