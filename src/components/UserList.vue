<template>
<ul class="user-list">
    <li
        class="user-list__item"
        :key="user.id"
        v-for="user in users">
        <User
            :first-name="user.firstName"
            :last-name="user.lastName"
            :avatar="user.avatar"
            :birthday="user.birthday" />
    </li>
</ul>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import User from '@/components/User.vue';
import axios from '@/config/axios';

interface IUser {
    firstName: string;
    lastName: string;
    avatar: string;
    birthday: number;
}

type TuserIds = number[];

interface IJson {
    [key: string]: any;
}

@Component({
    components: {
        User,
    },
})
export default class UserList extends Vue {
    isPending = false;
    offsetUsers = 0;
    maxCountUserRequest = 20;
    userIds: TuserIds = [];
    users: IUser[] = [];

    get nextUsersIds() {
        return this.userIds.splice(this.offsetUsers, this.maxCountUserRequest);
    }

    userToCamelCase(user: IJson): IUser {
        return {
            avatar: user.avatar,
            firstName: user.first_name,
            lastName: user.last_name,
            birthday: user.birthday,
        };
    }

    async fetchUserData(usersIds: TuserIds): Promise<IUser[]> {
        return axios.get('/users', {
            params: {
                id: usersIds,
            },
        })
            .then(({ data }) => data)
            .then((users) => {
                return users.map((user: IJson) => this.userToCamelCase(user));
            });
    }

    async addUsersToList() {
        this.userIds = await axios.get('/user_ids').then(({ data }) => data);
        const newUsers =  await this.fetchUserData(this.nextUsersIds);
        this.users.push(...newUsers);
        this.offsetUsers += this.maxCountUserRequest;

        return new Promise((resolve) => {
            resolve(newUsers);
        });
    }

    handleScroll(e: Event) {
        if (this.isPending) {
            return;
        }
        const $document = document as any;
        const height = $document.documentElement.clientHeight;
        const scrollHeight = $document.documentElement.scrollHeight;
        const scrollTop = window.pageYOffset;

        if (this.isScrollBottom(height, scrollHeight, scrollTop)) {
            this.isPending = true;

            this.addUsersToList().then(() => {
                this.isPending = false;
            });
        }
    }

    isScrollBottom(height: number, scrollHeight: number, scrollTop: number) {
        return height + scrollTop === scrollHeight;
    }

    created() {
        this.addUsersToList();
        document.addEventListener('scroll', this.handleScroll);
    }

    destroyed() {
        document.removeEventListener('scroll', this.handleScroll);
    }
}
</script>
<style lang="scss" scoped>
.user-list {
    margin: 0;
    padding: 0;
    display: flex;
    flex-wrap: wrap;
    list-style: none;
    grid-area: main;

    &__item {
        margin: 20px;
    }
}
</style>
