<template>
<span v-tooltip="data.name">
  <div class="image">
    <img :src="image" :alt="data.personUrl"/>
  </div>
</span>
</template>
 
<script>
 
export default {
  data () {
    return {
      data: {}
    }
  },
  computed: {
    image: function() {
      return 'src/image/' + this.data.personUrl;
    }
  }
}
</script>

<style>
.image img {
        height: 25px;
        width: 25px;
        border-radius: 50px;
        box-shadow: inset 0 0 1px #e0e0e0, inset 0 0 14px rgba(0,0,0,0.2);
    }
</style>
