<template>
  <Alert ref="alert" />
  <section v-if="isLoggedIn">
    <h1>Your Profile</h1>
    <hr/><br/>
    <div>
      <p><strong>Full Name:</strong> <span>{{ user.full_name }}</span></p>
      <p><strong>Username:</strong> <span>{{ user.username }}</span></p>
      <p><button @click="deleteAccount()" class="btn btn-primary">Delete Account</button></p>
    </div>
  </section>
</template>

<script>
import { defineComponent } from 'vue';
import { mapGetters, mapActions } from 'vuex';
import router from "@/router";
import Alert from "@/components/AlertBox.vue";

export default defineComponent({
  name: 'ProfileView',
  components: {Alert},
  beforeCreate() {
    if (!this.$store.getters.isAuthenticated) {
      return router.push('/login');
    }
  },
  created: async function() {
    try {
      await this.$store.dispatch('viewMe');
    } catch (error) {
      if (error.message && error.message === "Network Error") {
        this.showAlertMsg("Disconnected from server.");
      } else {
        this.showAlertMsg("An error occurred. Please try again later.");
      }
    }
  },
  computed: {
    isLoggedIn: function() {
      return this.$store.getters.isAuthenticated;
    },
    ...mapGetters({user: 'stateUser' }),
  },
  methods: {
    ...mapActions(['deleteUser']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async deleteAccount() {
      try {
        await this.deleteUser(this.user.id);
        await this.$store.dispatch('logOut');
        this.$router.push('/');
      } catch (error) {
        if (!error.response) {
          this.showAlertMsg("Disconnected from server.")
      } else {
          this.showAlertMsg("An error occurred. Please try again later.")
      }
    }
    }
  },
});
</script>
