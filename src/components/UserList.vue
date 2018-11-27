<template>
    <span>{{ userIds }}</span>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import axios from 'axios';

interface IUser {
    firstName: string;
    lastName: string;
    avatar: string;
    birthday: number;
}

@Component
export default class UserList extends Vue {
    userIds: number[] = [];
    users: IUser[] = [];

    async created() {
        this.userIds = await axios.get('http://localhost:3000/user_ids').then(({ data }) => data);
    }
}
</script>
