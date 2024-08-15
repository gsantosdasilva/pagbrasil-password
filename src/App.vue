<template>
  <div>
    <h1>PagBrasil Password</h1>
    <input type="text" v-model="username" placeholder="Username">
    <input type="password" v-model="password" placeholder="Password">
    <button @click="verify">Verificar</button>
    <table v-if="showTable">
      <thead>
        <tr>
          <th>Username</th>
          <th>Senha Criptografada</th>
          <th>Senha Desencriptada</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.username">
          <td>{{ user.username }}</td>
          <td>{{ user.password }}</td>
          <td>{{ decrypt(user.password) }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>{{ errorMessage }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      users: [],
      showTable: false,
      errorMessage: ''
    }
  },
  methods: {
    async verify() {
      try {
        const response = await fetch('/usuarios.json');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }

        this.users = await response.json();
        console.log("Usuários carregados:", this.users);

        const encryptedPassword = this.encrypt(this.password);
        console.log("Senha digitada:", this.password);
        console.log("Senha criptografada esperada:", encryptedPassword);

        const user = this.users.find(user => user.username === this.username && user.password === encryptedPassword);
        console.log("Usuário encontrado:", user);

        if (user) {
          this.showTable = true;
          this.errorMessage = '';
        } else {
          this.errorMessage = 'Usuário ou senha inválidos';
          this.showTable = false;
        }
      } catch (error) {
        console.error('Erro ao carregar os dados:', error);
        this.errorMessage = 'Erro ao carregar os dados: ' + error.message;
        this.showTable = false;
      }
    },
    encrypt(password) {
      // Converte cada caractere para código ASCII, multiplica por 57 e concatena com '|'
      const encryptedPassword = password.split('')
        .map(char => (char.charCodeAt(0) * 57))
        .join('|') + '|';
      // Codifica em Base64
      const base64Password = btoa(encryptedPassword);
      console.log("Senha criptografada:", base64Password);
      return base64Password;
    },
    decrypt(encryptedPassword) {
      // Decodifica Base64
      const decodedPassword = atob(encryptedPassword);
      // Divide a string por '|'
      return decodedPassword.split('|')
        .filter(part => part) // Remove partes vazias
        .map(code => String.fromCharCode(parseInt(code) / 57))
        .join('');
    }
  }
}
</script>

<style scoped>
/* Adicione estilos personalizados, se necessário */
</style>
