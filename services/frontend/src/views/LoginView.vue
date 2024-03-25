<template>
  <section>
  <Alert ref="alert" />
    <form @submit.prevent="submit">
      <div class="mb-3">
        <label for="username" class="form-label">Username:</label>
        <input type="text" name="username" v-model="form.username" class="form-control" />
      </div>
      <div class="mb-3">
        <label for="password" class="form-label">Password:</label>
        <input type="password" name="password" v-model="form.password" class="form-control" />
      </div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
  </section>
</template>

<script>
import { defineComponent } from 'vue';
import { mapActions } from 'vuex';
import Alert from "@/components/AlertBox.vue";

export default defineComponent({
  name: 'LoginView',
  components: {Alert},
  data() {
    return {
      form: {
        username: '',
        password:'',
      },
    };
  },
  methods: {
    ...mapActions(['logIn']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async submit() {
      if (!this.form.username || !this.form.password) {
        this.showAlertMsg('Please fill in all the fields');
        return ;
      }
      try {
        const User = new FormData();
        User.append('username', this.form.username);
        User.append('password', this.form.password);
        await this.logIn(User);
        await this.$router.push('/dashboard');
      }
      catch (error) {
        if (error.response) {
          if (error.response.status === 401) {
            this.showAlertMsg('Incorrect username or password');
          }
          else {
            this.showAlertMsg('An error occurred. Please try again later.');
          }
        } else {
            this.showAlertMsg('Disconnected from server. Please try again later.');
        }
      }
    },
  }
});
</script>
