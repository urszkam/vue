<template>
<Alert ref="alert" />
  <div v-if="isLoggedIn">
    <section>
      <h1>Add new note</h1>
      <hr/><br/>

      <form @submit.prevent="submit">
        <div class="mb-3">
          <label for="title" class="form-label">Title:</label>
          <input type="text" name="title" v-model="form.title" class="form-control" />
        </div>
        <div class="mb-3">
          <label for="content" class="form-label">Content:</label>
          <textarea
            name="content"
            v-model="form.content"
            class="form-control"
          ></textarea>
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
      </form>
    </section>

    <br/><br/>

    <section>
      <h1>Notes</h1>
      <hr/><br/>

      <div v-if="notes.length">
        <div v-for="note in notes" :key="note.id" class="notes">
          <div class="card" style="width: 18rem;">
            <div class="card-body">
              <ul>
                <li><strong>Note Title:</strong> {{ note.title }}</li>
                <li><strong>Author:</strong> {{ note.author.username }}</li>
                <li><router-link :to="{name: 'Note', params:{id: note.id}}">View</router-link></li>
              </ul>
            </div>
          </div>
          <br/>
        </div>
      </div>

      <div v-else>
        <p>Nothing to see. Check back later.</p>
      </div>
    </section>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import { mapGetters, mapActions } from 'vuex';
import Alert from '@/components/AlertBox.vue';

export default defineComponent({
  name: 'DashboardView',
  data() {
    return {
      form: {
        title: '',
        content: '',
      },
    };
  },
  components: {Alert},
  created: async function() {
    try {
      await this.$store.dispatch('getNotes');
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
      ...mapGetters({notes: 'stateNotes'}),
  },
  methods: {
    ...mapActions(['createNote']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async submit() {
      try {
        await this.createNote(this.form);
        this.form.title = '';
        this.form.content = '';
      } catch (error) {
        if (error.response) {
          if (error.response === 401) {
            this.showAlertMsg('Log in to see the content');
          } else if (error.response.data && error.response.data.detail
              && error.response.data.detail[0].type === "value_error.any_str.max_length") {
            this.showAlertMsg('Too long ' + error.response.data.detail[0].loc[1] + '.');
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
