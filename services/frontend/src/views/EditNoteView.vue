<template>
  <Alert ref="alert" />
  <section v-if="isLoggedIn">
    <h1>Edit note</h1>
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
</template>

<script>
import { defineComponent } from 'vue';
import { mapGetters, mapActions } from 'vuex';
import router from "@/router";
import Alert from "@/components/AlertBox.vue";

export default defineComponent({
  name: 'EditNote',
  components: {Alert},
  props: ['id'],
  data() {
    return {
      form: {
        title: '',
        content: '',
      },
    };
  },
  beforeCreate() {
    if (!this.$store.getters.isAuthenticated) {
      router.push('/login');
    }
  },
  created: function() {
    this.GetNote();
  },
  computed: {
    isLoggedIn: function() {
      return this.$store.getters.isAuthenticated;
    },
    ...mapGetters({ note: 'stateNote' }),
  },
  methods: {
    ...mapActions(['updateNote', 'viewNote']),
    showAlertMsg(msg) {
      this.$refs.alert.showAlert(msg);
    },
    async submit() {
    try {
      let note = {
        id: this.id,
        form: this.form,
      };
      await this.updateNote(note);
      this.$router.push({name: 'Note', params:{id: this.note.id}});
    } catch (error) {
      if (error.response) {
        if (error.response.data && error.response.data.detail
            && error.response.data.detail[0].type === "value_error.any_str.max_length") {
          this.showAlertMsg('Too long ' + error.response.data.detail[0].loc[1] + '.');
        }
      } else {
          this.showAlertMsg('Disconnected from server. Please try again later.');
      }
    }
    },
    async GetNote() {
      try {
        await this.viewNote(this.id);
        this.form.title = this.note.title;
        this.form.content = this.note.content;
      } catch (error) {
        if (error.response) {
          if (error.response.data && error.response.data.detail
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
