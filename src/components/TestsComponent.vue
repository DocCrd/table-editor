<template>
  <div>hi people!</div>
</template>
<script>
export default {
  props: ['names'],
  data() {
    return {
      cheer: 'cheers',
    };
  },
  methods: {
    log(e) {
      console.log('cheers', e.target.value);
    },
  },
};
</script>
<style></style>
