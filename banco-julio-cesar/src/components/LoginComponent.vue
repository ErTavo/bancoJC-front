<template>
  <div>
    <h2>Iniciar Sesión</h2>
    <form @submit.prevent="handleLogin">
      <label for="username">Usuario:</label>
      <input v-model="username" id="username" type="text" />

      <label for="password">Contraseña:</label>
      <input v-model="password" id="password" type="password" />

      <button type="submit">Entrar</button>
      <span class="register-link" @click="openRegistrationModal">Registrarse</span>
    </form>
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
  </div>
</template>

<script>
import Swal from "sweetalert2";

export default {
  data() {
    return {
      username: "",
      password: "",
      errorMessage: null,
    };
  },
  methods: {
    async promptForOtp() {
    const { value: otp } = await Swal.fire({
      title: 'Ingrese el código OTP',
      input: 'text',
      inputLabel: 'Codigo',
      inputPlaceholder: 'Ingrese el código',
      showCancelButton: true,
      confirmButtonText: 'Verificar',
      preConfirm: (otp) => {
        if (!otp) {
          Swal.showValidationMessage("Por favor ingrese el codigo de 4 digitos que recibio en su correo");
        }
        return otp;
      },
    });
    return otp;
  },
    async handleLogin() {
      try {
        const baseUrl = "https://banco-jc-back.vercel.app";
        const response = await fetch(`${baseUrl}/user/authenticate`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            username: this.username,
            password: this.password,
          }),
        });

        if (response.ok) {
          const data = await response.json();
          await fetch(`${baseUrl}/user/otp`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ username: this.username }),
          });
          const otpCode = await this.promptForOtp();
          if (otpCode) {
            await this.verifyCode(otpCode, data.hash);
          }
        } else {
          this.errorMessage = "Usuario o contraseña incorrectos";
        }
      } catch (error) {
        this.errorMessage = "Error al conectar con el servidor";
      }
    },


    async verifyCode(otp, hash) {
    try {
      const baseUrl = "https://banco-jc-back.vercel.app";
      const response = await fetch(`${baseUrl}/user/validateOtp`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username: this.username, otp: otp }),
      });

      if (response.ok) {
        const data = await response.json();
        if (data.success) {
          localStorage.setItem('authHash', hash)
          localStorage.setItem('username', this.username)
          localStorage.setItem('logged', true)
          this.$emit("loggedIn", hash);
        } else {
          Swal.fire("Error", "Código incorrecto. Intente nuevamente.", "error");
        }
      } else {
        Swal.fire("Error", "Error al verificar el código.", "error");
      }
    } catch (error) {
      Swal.fire("Error", "Error al verificar el código.", "error");
    }
  },


    openRegistrationModal() {
      Swal.fire({
        title: "Registrar Cuenta",
        html: `
          <form id="registration-form">
            <input type="text" id="reg-username" class="swal2-input" placeholder="Usuario" required>
            <input type="email" id="reg-email" class="swal2-input" placeholder="Correo" required>
            <input type="password" id="reg-password" class="swal2-input" placeholder="Contraseña" required>
          </form>
        `,
        preConfirm: () => {
          const username = document.getElementById("reg-username").value;
          const email = document.getElementById("reg-email").value;
          const password = document.getElementById("reg-password").value;

          if (!username || !email || !password) {
            Swal.showValidationMessage("Todos los campos son obligatorios.");
            return null;
          }

          return { username, email, password };
        },
        confirmButtonText: "Crear Usuario",
        showCancelButton: true,
        cancelButtonText: "Cancelar",
      }).then(async (result) => {
        if (result.isConfirmed) {
          const { username, email, password } = result.value;
          try {
            const baseUrl = "https://banco-jc-back.vercel.app";
            const response = await fetch(`${baseUrl}/user/users`, {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ username, email, password }),
            });

            if (response.ok) {
              Swal.fire("Éxito", "Usuario registrado exitosamente.", "success");
            } else {
              Swal.fire("Error", "No se pudo registrar el usuario.", "error");
            }
          } catch (error) {
            Swal.fire("Error", "Error al conectar con el servidor", "error");
          }
        }
      });
    },
  },
};
</script>

<style>
.error {
  color: red;
  margin-top: 10px;
}
.register-link {
  color: #007bff;
  margin-left: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  text-decoration: underline;
}
.register-link:hover {
  color: #0056b3;
}
</style>
