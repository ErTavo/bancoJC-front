<template>
  <div>
    <h2>Transferencias</h2>
    <form @submit.prevent="handleTransfer">
      <label for="recipient">Destinatario:</label>
      <input v-model="recipient" id="recipient" type="text" />
      
      <label for="amount">Monto:</label>
      <input v-model="amount" id="amount" type="number" />
      
      <button type="submit">Enviar</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      recipient: '',
      amount: 0,
    };
  },
  methods: {
    async handleTransfer() {
      console.log(`Enviando $${this.amount} a ${this.recipient}`);
    },
  },
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-top: 20px;
}

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 300px;
  margin: 0 auto;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
}
</style>
