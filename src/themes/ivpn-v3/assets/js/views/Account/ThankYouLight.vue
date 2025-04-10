<template>
    <div>
        <div class="payment-received" v-if="isLoaded">
            <h2>Your IVPN Light access is ready</h2>
            <p>We have received your payment.</p>
            <hr />
            <h4>Your account is live until {{ $filters.formatActiveUntil(account.active_until) }}.</h4>
            <hr />
            <div class="steps">
            <p>Next steps:</p>
                <ol>
                    <li>Paste the private key saved in the previous step to the field below.</li>
                    <li>Save the generated QR code or config file to avoid losing access.</li>
                    <li>Download and open the <a href="https://www.wireguard.com/" target="_blank" rel="noreferrer">WireGuard</a> app.</li>
                    <li>Scan the QR code, or add the configuration provided.</li>
                    <li>Connect to the IVPN service</li>
                </ol>
            </div>
            <input type ="text" v-model="privateKey" placeholder="Paste your private key here">

            <button
                :disabled="!isValidPrivateKey"
                @click.prevent="handleDownload()"
                class="btn btn-solid"
                style="margin-bottom: 1em"
                target="_blank"
            >
                <down-icon
                    style="width: 16px; height: 16px; fill: #449cf8"
                />Download configuration
            </button>

            <div v-if="isValidPrivateKey && qrCodes.length > 0" v-for="qr in qrCodes">
                <p>Access to:</p>
                <div v-if="this.multihop">
                    <p><country-flag :country="qr.entryCountryCode" size='normal'/> {{ qr.entryCity }}</p>
                    <p><country-flag :country="qr.countryCode" size='normal'/> {{ qr.city }}</p>
                </div>
                <div v-else>
                <p><country-flag :country="qr.countryCode" size='normal'/> {{ qr.city }}</p>
                </div>
                <div class="code" v-html="qr.qrCode"></div>
            </div>
            <textarea disabled v-if="isValidPrivateKey && (qrCodes.length == 1)" v-model="wireguardConfig" cols="50" rows="50">
            </textarea>

            <div class="steps">
                <h5>For further access beyond {{ $filters.formatActiveUntil(account.active_until) }} pay for a <a target="_blank" rel="noreferrer" href="https://www.ivpn.net/light">separate IVPN Light access</a>, or <a target="_blank" rel="noreferrer" href="https://www.ivpn.net/en/pricing">generate</a> an IVPN Standard or Pro account.</h5>
            </div>
        </div>
    </div>
</template>

<script>

import SuccessIcon from "@/components/icons/success.vue";
import DownIcon from "@/components/icons/btn/Download.vue";
import { mapState } from "vuex";
import CountryFlag from 'vue-country-flag-next'
import Api from "@/api/api";
import qrcode from "qrcode-generator";
import JSZip from "jszip";
import FileSaver from "file-saver";
import wireguard from '@/wireguard';

export default {
    components: { SuccessIcon, DownIcon,CountryFlag },
    data() {
        return {
            isLoaded: false,
            privateKey: "",
            isValidPrivateKey: false,
            wireguardConfig: "",
            qrCodes: [],
            wireguardConfigs: [],
            multihop: false,
        };
    },
    async created() {
        document.getElementById("my-account").remove();
        await this.$store.dispatch("auth/reload");
        await this.$store.dispatch("wireguard/load");
    },
    methods: {
        async handleDownload() {
            this.downloadArchive();
        },
        downloadArchive() {
            
            let self = this;
            let zip = new JSZip();
            let basename = this.multihop ? (this.multihop_basename + ".conf") : null
            this.wireguardConfigs.forEach(function (config) {
                zip.file(config.basename, self.configString(config));
            });
            zip.generateAsync({ type: "blob" }).then(function(content) {
                FileSaver.saveAs(content, "ivpn-wireguard-config.zip");
            });
        },
        generateQRCode(res) {
            if (!res) {
                return;
            }
            let configString = this.configString(res);
            this.wireguardConfig = configString;
            let qr = qrcode(0, "M");
            qr.addData(configString);
            qr.make();
            let location = {};
            location.qrCode = qr.createSvgTag(4)
            location.countryCode = res.country_code;
            location.city = res.city;
            if( res.multihop ){
                this.multihop = true;
                location.countryCode = res.exit_country_code;
                location.city = res.exit_city;
                location.entryCity = res.city;
                location.entryCountryCode = res.country_code;
            }else{
                location.countryCode = res.country_code;
                location.city = res.city;
            }
            this.qrCodes.push(location);
        },
        configString(config) {
            return "[Interface]" +
            "\nPrivateKey = " + this.privateKey +
            "\nAddress = " + config.interface.address +
            "\nDNS = " + config.interface.dns +
            "\n\n[Peer]" +
            "\nPublicKey = " + config.peer.public_key +
            "\nAllowedIPs = " + config.peer.allowed_ips +
            "\nEndpoint = " + config.peer.endpoint;
        },
    },
    computed: {
        ...mapState({
            account: (state) => state.auth.account,
            keys: (state) => state.wireguard.keys,
            configs: (state) => state.wireguard.configs,
        }),
    },
    watch: {
        configs: {
            handler: function (after, before) {
                after.then((res) => {
                    
                    res.forEach((cfg) => {
                        this.wireguardConfigs.push(cfg);
                        this.generateQRCode(cfg);
                    });
                    this.qrCodes = this.qrCodes.filter((v,i,a)=>a.findIndex(v2=>(JSON.stringify(v2) === JSON.stringify(v)))===i)
                });
            },
            deep: true
        },
        account: {
            handler: function (after, before) {
                this.isLoaded = true;
            },
            deep: true
        },
        privateKey: {
            handler: function (after, before) {
                this.isValidPrivateKey = wireguard.isValidKey(after);
                if(this.isValidPrivateKey){
                    this.qrCodes = [];
                    this.wireguardConfigs = [];
                    this.$store.dispatch("wireguard/loadConfigs");
                }
            },
            deep: true
        }
    },
    beforeMount(){
        document.getElementById("my-account").remove();
    },
    mounted(){
        document.getElementById("my-account").remove();
    }
};
</script>

<style lang="scss" scoped>
@import "../../styles/_vars.scss";
@import "../../styles/base.scss";

.payment-received {
    display: flex;
    flex-direction: column;
    align-items: center;
    max-width: 700px;
    text-align: center;

    h2 {
        margin-bottom: 1em;
    }

    h4{
        color: #FF3344;
        font-style: normal;
        font-weight: 400;
    }

    textarea{
        @include light-theme((
            background:  #F0F0F0,
            color: rgba(41, 41, 46, 0.5)
        ));

        @include dark-theme((
            background:  #3D3D42,
            color: white,
        ));
        border:0;
        margin:20px;
        min-height:200px;

    }

    h5{
        font-style: normal;
        font-weight: 400;
        text-align: center;
        line-height: 30px;
        font-size: 16px;
        margin: 10px;
    }

    p {
        max-width: 550px;
        margin-bottom:5px;
    }

    .steps{
        text-align: left;
    }

    ol{
        text-align: left;
        margin-bottom:0px;
        letter-spacing: -0.4px;
    }

    hr{
        width:100%;
        background:rgba(61, 61, 66, 0.5);
        margin:30px;
    }

    .promo-block {
        &:first-of-type {
            margin-top: 72px;
        }

        display: flex;
        flex-direction: column;
        align-items: center;
        max-width: 700px;
        text-align: center;

        border-top: 1px solid #99999940;
        margin: 24px 0px 0px 0;
        padding: 8px 0px 4px 0px;
        width: 80%;

        p {
            margin-bottom: 8px;
            margin-top: 16px;
            max-width: 650px;
        }

        ul.links {
            display: flex;
            list-style: none;
            margin: 0;
            padding: 0;

            li {
                &:before {
                    display: none;
                }

                flex-grow: 1;
                &.social {
                    width: 135px;
                }

                a {
                    padding: 0px 8px;
                }

                padding: 0px 8px;
                margin: 0px;
            }

            .social {
                margin: 0px 10px;
            }
        }
    }

    .btn-solid{
        width: 100%;
        margin: 1em;
    }
}
</style>