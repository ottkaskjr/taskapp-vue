<template>
  <div>
    <Brand />
    <AddTask />
    <Tasks />
  </div>
</template>

<script>
// @ is an alias to /src
import Brand from '@/components/Brand.vue';
import Tasks from '@/components/Tasks.vue';
import AddTask from '@/components/AddTask.vue';

export default {
  name: 'Landing',
  components: {
    Brand,
    Tasks,
    AddTask,
  },
};
</script>
<style>
body {
  background-color: #f7f9ff !important;
}
</style>
