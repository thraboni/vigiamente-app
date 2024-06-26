<template>
  <div
    class="rows"
    role="form"
    aria-label="Formulário de cadastro de usuário"
    v-if="isAdmin"
  >
    <form @submit.prevent="cadastrar">
      <div class="row">
        <h2>Cadastrar Usuário</h2>
      </div>
      <div class="row">
        <label for="nomeUsuario">Nome</label>
        <input
          type="text"
          class="input"
          v-model="nomeUsuario"
          id="nomeUsuario"
          name="nomeUsuario"
          placeholder="Nome"
          required
        />
      </div>
      <div class="row">
        <label for="perfilUsuario">Perfil</label>
        <select
          class="input"
          v-model="tipoPerfil"
          id="perfilUsuario"
          name="perfilUsuario"
          required
        >
          <option value="" disabled>Selecione o perfil</option>
          <option value="administrador">Administrador</option>
          <option value="usuario">Usuário</option>
        </select>
      </div>
      <div class="row">
        <label for="loginUsuario">Login</label>
        <input
          type="text"
          class="input"
          v-model="loginUsuario"
          id="loginUsuario"
          name="loginUsuario"
          placeholder="Login"
          required
        />
      </div>
      <div class="row">
        <label for="emailUsuario">E-mail</label>
        <input
          type="text"
          class="input"
          v-model="emailUsuario"
          id="emailUsuario"
          name="emailUsuario"
          placeholder="E-mail"
          required
        />
      </div>
      <div class="row">
        <button>Cadastrar Usuário</button>
      </div>
    </form>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { useStore } from "@/store";
import { TipoNotificacao } from "@/interfaces/INotificacao";
import useNotificador from "@/hooks/notificador";
import { criaUsuario } from "@/service/userService";
import IUsuario from "@/interfaces/IUsuario";
import { pegaUsuario } from "@/service/userService";

export default defineComponent({
  name: "CadastroUsuario",
  data() {
    return {
      nomeUsuario: "",
      tipoPerfil: "",
      loginUsuario: "",
      emailUsuario: "",
      perfilUsuario: false,
      isAdminState: false,
    };
  },
  computed: {
    isAdmin(): boolean {
      return this.isAdminState;
    },
  },
  methods: {
    async cadastrar() {
      if (this.tipoPerfil === "administrador") {
        this.perfilUsuario = true;
      } else {
        this.perfilUsuario = false;
      }
      const novoUsuario: IUsuario = {
        nome: this.nomeUsuario,
        admin: this.perfilUsuario,
        usuario: this.loginUsuario,
        email: this.emailUsuario,
        senha: "",
      };
      try {
        await criaUsuario(novoUsuario);
        this.notificar(
          TipoNotificacao.SUCESSO,
          "Cadastro efetuado",
          "O usuário foi cadastro com sucesso!"
        );
        this.nomeUsuario = "";
        this.tipoPerfil = "";
        this.loginUsuario = "";
        this.emailUsuario = "";
        this.perfilUsuario = false;
      } catch (error) {
        this.notificar(
          TipoNotificacao.FALHA,
          "Falha no Cadastro",
          "Erro ao cadastrar usuário!"
        );
      }
    },
    async verificarAdmin() {
      const token = localStorage.getItem("token");
      if (token) {
        try {
          const usuario = await pegaUsuario();
          if (usuario.admin) {
            this.isAdminState = true;
          }
        } catch (error) {
          this.isAdminState = false;
        }
      } else {
        this.isAdminState = false;
      }
    },
  },
  setup() {
    const store = useStore();
    const { notificar } = useNotificador();
    return {
      store,
      notificar,
    };
  },
  mounted() {
    this.verificarAdmin(); // Verifica se o usuário é admin ao montar o componente
  },
});
</script>

<style scoped>
div.rows {
  padding: 2rem;
  font-size: 1rem;
  font-weight: 600;
}

div.row {
  padding: 0 0 2rem 0;
}

h2 {
  font-size: 2rem;
  font-weight: bold;
}

input {
  margin-top: 0.5rem;
  background-color: rgba(181, 202, 141, 0.3);
  border: none;
  color: #b5ca8d;
}

input::placeholder {
  color: #b5ca8d;
  font-style: italic;
}

select {
  margin-top: 0.5rem;
  background-color: rgba(181, 202, 141, 0.3);
  border: none;
  color: #b5ca8d;
}

button {
  background-color: #426b69;
  color: white;
  border-radius: 0.25rem;
  width: 12rem;
  height: 2.5rem;
}

button:hover {
  font-weight: 600;
  box-shadow: 0px 0px 5px black;
}
</style>
