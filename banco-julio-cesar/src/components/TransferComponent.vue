<template>
  <div>
    <h2>Transferir desde la cuenta: {{ accountNo }}</h2>
    <!-- Aquí va el formulario de transferencia -->

    <!-- Botón de regresar -->
    <button @click="goBack">Regresar a la cuenta</button>
  </div>
</template>

<script>
export default {
  props: {
    accountNo: {
      type: String,
      required: true,
    },
  },
  methods: {
    // Emitir el evento para volver a la vista de 'account'
    goBack() {
      this.$emit('back'); // Emitir evento para regresar
    }
  },
};
</script>
