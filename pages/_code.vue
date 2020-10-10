<template>
  <div>
    <b-spinner class="loading-spinner" style="position:fixed; left: 50%; top: 50%;"></b-spinner>
  </div>
</template>


<script>
export default {
  mounted(){
      this.$router.push('/')
  }
}
</script>
