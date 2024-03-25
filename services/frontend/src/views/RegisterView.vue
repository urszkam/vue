<template>
  <section>
  <Alert ref="alert" />
    <form @submit.prevent="submit">
      <div class="mb-3">
        <label for="username" class="form-label">Username:</label>
        <input type="text" name="username" v-model="user.username" class="form-control" />
      </div>
      <div class="mb-3">
        <label for="full_name" class="form-label">Full Name:</label>
        <input type="text" name="full_name" v-model="user.full_name" class="form-control" />
      </div>
      <div class="mb-3">
        <label for="password" class="form-label">Password:</label>
        <input type="password" name="password" v-model="user.password" class="form-control" />
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
  name: 'RegisterView',
  components: {Alert},
  data() {
    return {
      user: {
        username: '',
        full_name: '',
        password: '',
      },
    };
  },
  methods: {
    ...mapActions(['register']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async submit() {
      if (!this.user.username || !this.user.full_name || !this.user.password) {
        this.showAlertMsg('Please fill in all the fields.');
        return;
      }
      try {
        await this.register(this.user);
        await this.$router.push('/dashboard');
      } catch (error) {
        if (error.response) {
          if (error.response.status === 409) {
            this.showAlertMsg('Username already exists. Please try again or log in.');
          } else if (error.response.data && error.response.data.detail
              && error.response.data.detail[0].type === "value_error.any_str.max_length") {
            this.showAlertMsg('Too long '+ error.response.data.detail[0].loc[1] + '. Please try again.');
          } else {
            this.showAlertMsg('An error occurred. Please try again later.');
          }
        } else {
          this.showAlertMsg('Disconnected from server. Please try again later.');
        }
      }
    },
  },
});
</script>
