<template>
  <div class="home">
    <UserList/>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import UserList from '@/components/UserList.vue';

@Component({
  components: {
    UserList,
  },
})
export default class Home extends Vue {}
</script>
