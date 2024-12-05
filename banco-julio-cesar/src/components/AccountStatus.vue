<template>
  <div>
    <h2>Estado de la Cuenta</h2>
    <p v-if="loading">Cargando datos...</p>
    <p v-else-if="errorMessage">{{ errorMessage }}</p>
    <p v-else>Saldo: {{ balance }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      balance: null,
      loading: true,
      errorMessage: null,
    };
  },
  async mounted() {
    const hash = localStorage.getItem('authHash');

    if (!hash) {
      this.errorMessage = 'Sesión expiró. Por favor, inicie sesión de nuevo.';
      this.loading = false;
      return;
    }

    try {
      // Verificar la validez del hash
      const username = localStorage.getItem('username');
      const validationResponse = await fetch('https://banco-jc-back.vercel.app/user/validateUserHash', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, hash }),
      });

      if (validationResponse.ok) {
        const validationData = await validationResponse.json();
        if (validationData.success) {
          const response = await fetch('https://tu-backend.com/api/account-status', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ hash }),
          });

          if (response.ok) {
            const data = await response.json();
            this.balance = data.balance;
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
  },
};
</script>
