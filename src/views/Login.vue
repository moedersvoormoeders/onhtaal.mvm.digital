<template id="login">
  <div style="background: royalblue">
  <div class="container vertical-center">
    <div class="container">
      <div class="row justify-content-center">
        <div class="col-md-5 col-10 login-box">
          <h2>mvm.digital Onthaal</h2>
          <form m v-on:submit.prevent="login">
            <div class="form-group">
              <label class="sr-only" for="username">Gebruikersnaam</label>
              <div class="input-group">
                <div class="input-group-prepend">
                  <div class="input-group-text"><i class="fas fa-user"></i></div>
                </div>
                <input type="text" class="form-control" name="username" id="username" v-model="username" placeholder="Gebruikersnaam">
              </div>
            </div>
            <div class="form-group">
              <label class="sr-only" for="password">Wachtwoord</label>
              <div class="input-group">
                <div class="input-group-prepend">
                  <div class="input-group-text"><i class="fas fa-key"></i></div>
                </div>
                <input type="password" class="form-control" name="password" id="password" v-model="password" placeholder="Wachtwoord">
              </div>
            </div>
            <button v-on:click="login()" class="btn btn-primary" ><i class="fas fa-sign-in-alt"></i> Login</button>
          </form>
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<script>

import { authService } from "../_services/auth.service"

export default {
  template: "#login",
  components: {},
  data: function () {
    return {
      username: "",
      password: "",
      submitted: false,
    };
  },

  computed: {
    loggingIn: function () {
      return this.$store.state.authentication.status.loggingIn;
    },
  },

  methods: {
    login: async function () {
      this.submitted = true;
      if (this.username && this.password) {
        try {
          await authService.login(this.username, this.password)
          this.$router.push("/search")
        } catch(e) {
          this.$Simplert.open({
            title: "Login error!",
            message: e,
            type : "error",
            customCloseBtnText: "Sluiten",
          });
        }
      }
    },
  },

  created: function () {},
};
</script>


<style scoped>
.container {
  background-color: royalblue;
}

.login-box {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
}
.vertical-center {
  min-height: 100vh;

  display: flex;
  align-items: center;
}
</style>