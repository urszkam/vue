<template>
  <div v-if="show" class="alert alert-warning alert-dismissible fade show" role="alert">
    {{ message }}
    <button type="button" class="btn-close" @click="dismissAlert" aria-label="Close"></button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: false,
      message: ''
    };
  },
  methods: {
    showAlert(message) {
      this.message = message;
      this.show = true;
    },
    dismissAlert() {
      this.show = false;
    }
  }
};
</script>