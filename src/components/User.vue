<template>
    <div class="user">
        <div class="user__avatar">
            <img :src="avatar" />
        </div>
        <div class="user__info">
            <div class="user__full-name">{{ fullName }}</div>
            <div class="user__birthday">{{ age }} лет</div>
        </div>
    </div>
</template>
<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class User extends Vue {
    @Prop({ required: true }) firstName!: string;
    @Prop({ required: true }) lastName!: string;
    @Prop({ required: true }) birthday!: number;
    @Prop({ required: true }) avatar!: string;

    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    get age() {
        const currentDate = new Date();
        const birthday = new Date(this.birthday);

        return currentDate.getFullYear() - birthday.getFullYear();
    }
}
</script>
<style lang="scss" scoped>
.user {
    display: flex;
    flex-direction: column;
    background: #000;

    &__info {
        color: #ffffff;
    }
}
</style>
