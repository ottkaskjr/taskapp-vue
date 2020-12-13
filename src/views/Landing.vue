<template>
  <div>
    <Brand />
    <Tasks />
  </div>
</template>

<script>
// @ is an alias to /src
import Brand from '@/components/Brand.vue';
import Tasks from '@/components/Tasks.vue';

export default {
  name: 'Landing',
  components: {
    Brand,
    Tasks,
  },
};
</script>
<style>
body {
  background-color: #f7f9ff !important;
}
</style>
