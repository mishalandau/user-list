<template>
<transition name="fade">
    <v-touch v-if="!hideData"
        @panmove="panmoveHandler"
        @swipeleft="swipeHandler"
        @swiperight="swipeHandler"
        @panend="panendHandler"
        @pancancel="panendHandler"
        :pan-options="{ direction: 'horizontal' }"
        :swipe-options="{ direction: 'horizontal' }">
        <ul
            class="user-list"
            :class="{'user-list--animation': animationClass}"
            :style="{left: `${offsetLeft}px`}">
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
    </v-touch>
</transition>
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
    hideData = false;
    offsetUsers = 0;
    animationClass = false;
    maxCountUserRequest = 20;
    offsetLeft = 0;
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
        if (this.offsetUsers > this.userIds.length) {
            return new Promise((_, reject) => {
                reject(new Error('Users not found.'));
            });
        }

        try {
            const userIdsStorage = JSON.parse(localStorage.getItem('user_ids') as any) || [];
            if (!userIdsStorage.length) {
                throw new Error('user ids not found in localStorage');
            }

            this.userIds = userIdsStorage;
        } catch(_) {
            this.userIds = await axios.get('/user_ids').then(({ data }) => data);
            localStorage.setItem('user_ids', JSON.stringify(this.userIds));
        }


        const newUsers =  await this.fetchUserData(this.nextUsersIds);
        this.users.push(...newUsers);
        localStorage.setItem('users', JSON.stringify(this.users));
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
        const offsetBottom = height + scrollTop;
        const imperceptible = 600;
        return offsetBottom >= scrollHeight - imperceptible;
    }

    swipeHandler() {
        this.hideData = true;
        setTimeout(() => {
            this.clearData();
        }, 500);
    }

    panmoveHandler({ deltaX }: any) {
        this.offsetLeft = deltaX;
    }

    panendHandler() {
        this.offsetLeft = 0;
        this.animationClass = true;
        setTimeout(() => {
            this.animationClass = false;            
        }, 300);
    }

    clearData() {
        document.removeEventListener('scroll', this.handleScroll);
        localStorage.setItem('user_ids', '');
        localStorage.setItem('users', '');
        this.isPending = false;
        this.hideData = false;
        this.offsetUsers = 0;
        this.userIds = [];
        this.users = [];
    }

    created() {
        try {
            this.users = JSON.parse(localStorage.getItem('users') as any) || [];
        } catch(_) {
            this.addUsersToList();
        }
        document.addEventListener('scroll', this.handleScroll);
    }

    destroyed() {
        document.removeEventListener('scroll', this.handleScroll);
    }
}
</script>
<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s ease;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.user-list {
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    list-style: none;
    position: relative;

    &--animation {
        transition: all .5s ease;
    }

    &__item {
        margin: 20px;
    }
}
</style>
