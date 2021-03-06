<template>
    <article class="user-item" :class="{ editing: editing }">
        <div class="info" v-show="!editing">
            <img :src="user.avatar" width="128" height="128" alt="">

            <div class="right">
                <div>
                    <h1>{{ user.name }}
                        <i v-if="isCurrentUser" class="you fa fa-check-circle"></i>
                    </h1>
                    <p>{{ user.email }}</p>
                </div>

                <div class="buttons">
                    <button class="btn btn-blue" @click="edit" v-show="!confirmingDelete">
                        {{ user.id === state.current.id ? 'Update Profile' : 'Edit' }}
                    </button>
                    <button v-show="!isCurrentUser && !confirmingDelete"
                        class="btn btn-red" @click="confirmDelete">Delete</button>
                    <span v-show="confirmingDelete">
                        <button class="btn btn-red" @click="destroy">Confirm</button>
                        <button @click="cancelDelete">Cancel</button>
                    </span>
                </div>
            </div>
        </div>

        <form class="edit" @submit.prevent="update" v-else>
            <div class="input-row">
                <label>Name</label>
                <input type="text" v-model="user.name" required v-koel-focus="editing">
            </div>
            <div class="input-row">
                <label>Email</label>
                <input type="email" v-model="user.email" required>
            </div>
            <div class="input-row">
                <label>Password</label>
                <input type="password" v-model="user.password" placeholder="Leave blank for no changes">
            </div>
            <div class="input-row">
                <label></label>
                <button class="btn btn-green">Update</button>
                <button class="btn btn-red" @click.prevent="cancelEdit">Cancel</button>
            </div>
        </form>
    </article>
</template>

<script>
    import { clone, assign } from 'lodash';

    import userStore from '../../stores/user';

    export default {
        props: ['user'],
        replace: true,

        data() {
            return {
                state: userStore.state,
                editing: false,
                confirmingDelete: false,
                cached: {},
            };
        },

        computed: {
            /**
             * Determine if the current logged in user is the user bound to this component.
             *
             * @return {Boolean}
             */
            isCurrentUser() {
                return this.user.id === this.state.current.id;
            },
        },

        methods: {
            /**
             * Trigger editing a user.
             * If the user is the current logged-in user, redirect to the profile screen instead.
             */
            edit() {
                if (this.isCurrentUser) {
                    this.$root.loadMainView('profile');

                    return;
                }

                // Keep a cached version of the user for rolling back.
                this.cached = clone(this.user);
                this.editing = true;
            },

            /**
             * Cancel editing a user.
             */
            cancelEdit() {
                // Restore the original user's properties
                assign(this.user, this.cached);
                this.editing = false;
            },

            /**
             * Update the edited user.
             */
            update() {
                userStore.update(this.user, this.user.name, this.user.email, this.user.password, () => {
                    this.editing = false;
                });
            },

            /**
             * Confirm the delete action by triggering the confirming div's visibility.
             */
            confirmDelete() {
                this.confirmingDelete = true;
            },

            /**
             * Cancel the delete action.
             */
            cancelDelete() {
                this.confirmingDelete = false;
            },

            /**
             * Kill off the freaking user.
             */
            destroy() {
                userStore.destroy(this.user, () => {
                    this.$destroy(true);
                });
            },
        },
    };
</script>

<style lang="sass">
    @import "../../../sass/partials/_vars.scss";
    @import "../../../sass/partials/_mixins.scss";
</style>
