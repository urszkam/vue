<template>
  <Alert ref="alert" />
  <div v-if="note && isLoggedIn">
    <p><strong>Title:</strong> {{ note.title }}</p>
    <p><strong>Content:</strong> {{ note.content }}</p>
    <p><strong>Author:</strong> {{ note.author.username }}</p>

    <div v-if="user.id === note.author.id">
      <p><router-link :to="{name: 'EditNote', params:{id: note.id}}" class="btn btn-primary">Edit</router-link></p>
      <p><button @click="removeNote()" class="btn btn-secondary">Delete</button></p>
    </div>
  </div>
</template>


<script>
import { defineComponent } from 'vue';
import { mapGetters, mapActions } from 'vuex';
import router from "@/router";
import Alert from "@/components/AlertBox.vue";

export default defineComponent({
  name: 'NoteView',
  components: {Alert},
  props: ['id'],
  beforeCreate() {
    if (!this.$store.getters.isAuthenticated) {
      router.push('/login');
    }
  },
  async created() {
    try {
      await this.viewNote(this.id);
    } catch (error) {
      if (!error.response && error.message === "Network Error") {
      this.showAlertMsg("Disconnected from server.")
    } else {
        this.showAlertMsg("An error occurred. Please try again later.")
      }
    }
  },
  computed: {
    isLoggedIn: function() {
      return this.$store.getters.isAuthenticated;
    },
    ...mapGetters({ note: 'stateNote', user: 'stateUser'}),
  },
  methods: {
    ...mapActions(['viewNote', 'deleteNote']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async removeNote() {
      try {
        await this.deleteNote(this.id);
        this.$router.push('/dashboard');
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
