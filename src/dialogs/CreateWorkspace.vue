<template>
    <DialogForm
        v-slot="{ submit }"
        :dialog="dialog"
        title="Create Workspace"
        submit-label="Create"
        @completed="createWorkspace"
    >
        <v-select
            v-if="$auth.mode === Mode.Solid"
            v-model="storage"
            :items="$auth.user.storages"
            :rules="rules.storage"
            label="Storage"
        />
        <v-text-field
            ref="name"
            v-model="name"
            :rules="rules.name"
            label="Name"
            validate-on-blur
            autofocus
            @keydown.enter="submit"
        />
    </DialogForm>
</template>

<script lang="ts">
import Vue from 'vue';

import { ValidationRule } from 'vuetify';

import User from '@/models/users/User';
import SolidUser from '@/models/users/SolidUser';

import { Dialog } from '@/services/UI';

import DialogForm from '@/dialogs/DialogForm.vue';

import Validations from '@/utils/Validations';
import { Mode } from '@/services/Auth';

interface Data {
    storage: string;
    name: string;
}

export default Vue.extend({
    components: {
        DialogForm,
    },
    props: {
        dialog: {
            type: Object as () => Dialog,
            required: true,
        },
    },
    data(): Data {
        return {
            storage: '',
            name: '',
        };
    },
    computed: {
        rules(): { [field: string]: ValidationRule[] } {
            return {
                storage: [
                    Validations.required,
                ],
                name: [
                    Validations.required,
                    Validations.maxLength(255),
                ],
            };
        },
        Mode: () => Mode,
    },
    created() {
        if (this.$auth.mode === Mode.Solid) {
            this.$auth.withUser((user: User) => {
                this.storage = (user as SolidUser).storages[0];
            });
        }
    },
    mounted() {
        // Autofocus does not seem to work as expected inside dialogs
        // see: https://github.com/vuetifyjs/vuetify/search?q=autofocus+dialog&type=Issues
        this.$nextTick((this.$refs.name as any).focus);
    },
    methods: {
        async createWorkspace() {
            let args: any[] = [];
            switch (this.$auth.mode) {
                case Mode.Offline:
                    args = [this.name];
                    break;
                case Mode.Solid:
                    args = [this.storage, this.name];
                    break;
            }

            this.$ui.completeDialog(this.dialog.id);
            this.$ui.wrapAsyncOperation(
                this.$workspaces.createWorkspace(...args),
                `Creating ${this.name} workspace...`
            );
        },
    },
});
</script>
