<template>
    <div>
        <h2>{{ $t('account.services.mail.betaTitle') }}</h2>

        <div v-if="!subIdDeletedAt && loaded">
            <p>{{ $t('account.services.mail.betaDescription1') }}</p>
            <p>{{ $t('account.services.mail.betaDescription2') }}</p>
            <p>{{ $t('account.services.mail.betaDescription3') }}</p>
            <p>{{ $t('account.services.mail.betaDescription4') }}</p>
            <p>{{ $t('account.services.mail.betaDescription5') }}</p>
            <p v-if="!generated">
                <p>
                {{ $t('account.services.mail.follow') }}
                </p>
                <a class="btn btn-solid btn-big" @click="add">{{ $t('account.services.mail.generate') }}</a>
            </p>
            <p v-else>
                <p>
                {{ $t('account.services.mail.follow') }}<br>
                </p>
                <p>
                    {{ $t('account.services.mail.signup') }} <a target="_blank" :href="'https://mailx.net/signup/' + subId">https://mailx.net/signup/{{ subId }}</a><br>
                    {{ $t('account.services.mail.expireNote') }}
                </p>
            </p>
            <p>{{ $t('account.services.mail.feedback') }} <a href="mailto:mailx@ivpn.net">mailx@ivpn.net</a>.</p>
        </div>

        <div v-if="subIdDeletedAt && loaded">
            <p>{{ $t('account.services.mail.signed') }}</p>
            <p>{{ $t('account.services.mail.access') }} <a target="_blank" href="https://mailx.net">{{ $t('account.services.mail.here') }}</a>.</p>
            <p>{{ $t('account.services.mail.betaDescription5') }}</p>
            <p>
                {{ $t('account.services.mail.submit') }}<br>
                GitHub - <a target="_blank" href="https://github.com/ivpn/mailx">https://github.com/ivpn/mailx</a><br>
                Email - <a href="mailto:mailx@ivpn.net">mailx@ivpn.net</a>
            </p>
        </div>

    </div>
</template>

<script>
import progressSpinner from "@/components/ProgressSpinner.vue";
import { mapState } from "vuex";
import { useI18n } from "vue-i18n";

export default {
    components: {
        progressSpinner
    },
    data() {
        return {
            loaded: false,
            language: "en",
            subId: "",
            subIdDeletedAt: "",
            store: false,
            success: "",
            generated: false,
        };
    },
    computed: {
        ...mapState({
            account: (state) => state.auth.account,
            error: (state) => state.account.error,
            inProgress: (state) => state.account.inProgress,
        }),
    },
    async beforeMount() {
        if (!this.account.product.capabilities.has_mail) {
            window.location.href = "/" + this.language + "/account/";
            return;
        }

        await this.$store.dispatch("auth/reload");

        this.subId = this.account["email_service_id"];
        this.subIdDeletedAt = this.account["email_service_deleted_at"];
        this.loaded = true;
    },
    mounted() {
        useI18n().locale.value = window.location.href.split("/")[3];
        this.language = window.location.href.split("/")[3];
    },
    methods: {
        async add() {
            let res = await this.$store.dispatch("account/addEmailSubscription");
            if (res) {
                this.subId = res.id;
                this.generated = true;
            }
        },
    },
};
</script>

<style lang="scss">
@import "@/styles/_vars.scss";
@import "@/styles/base.scss";

.status {
    margin-top: 8px;
    text-transform: uppercase;
    display: inline-block;
    padding: 0px 16px 0px 16px;
    line-height: 28px;
    font-size: 12px;
    border-radius: 4px;
    letter-spacing: 1px;

    &.active {
        background: #64ad07;
        color: $white;
        
    }

    &.inactive {
        @include light-theme((
            background: #29292E80,
            color: $white
        ));

        @include dark-theme((
            background: rgba(255, 255, 255, 0.5),
            color: #29292E
        ));
    }
}

.table-kw {
    
    .row {
        padding: 10px 0!important;
        margin-bottom: 24px;
    }
}
</style>