<template>
  <div>
    <div class="loader" >
        <a class="is-loading">
            <i class="fas fa-cog"></i>
        </a>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isLoading: Boolean
  },
  mounted() {
      console.log(this.isLoading);
  }
};
</script>