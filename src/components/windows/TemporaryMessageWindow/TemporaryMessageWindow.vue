<template>
    <div class="claim-rewards-info-window">
        <f-window
            ref="window"
            modal
            style="max-width: 700px;"
            title="! IMPORTANT WARNING !"
            animation-in="scale-center-enter-active"
            animation-out="scale-center-leave-active"
            class="tmp-info-window"
            :hide-on-escape-key="false"
            @window-hide="$emit('window-hide', $event)"
        >
            <p>
                Please note that in the following weeks, the URL
                <strong>fwallet.fantom.network</strong> will be upgraded to the new fWallet.
            </p>
            <p>
                If you're accessing this URL with a private key, mnemonic, or keystore file, make sure you have
                downloaded and saved the keystore file. <br />
                <strong>
                    You will not be able to access your wallet otherwise, and there's no way to retrieve it.
                </strong>
            </p>
            <p v-if="accounts.length > 0" class="align-center">
                <button class="btn secondary" @click="onDownloadBtnClick">Download keystore file(s)</button>
            </p>
            <p>
                You're already set in all other cases (Metamask, Ledger, and other browser wallets)!
            </p>
            <p class="align-center">
                <button class="btn large" @click="onOkBtnClick">Ok, understood</button>
            </p>
        </f-window>
    </div>
</template>

<script>
import FWindow from '@/components/core/FWindow/FWindow.vue';
import { mapGetters } from 'vuex';
import JSZip from 'jszip';
import fileDownload from 'js-file-download';

const FILE_NAME = 'fwallet_keystore_files';

export default {
    name: 'TemporaryMessageWindow',

    components: { FWindow },

    computed: {
        ...mapGetters(['accounts']),
    },

    mounted() {
        // if (!window.localStorage.getItem('tmp-msg')) {
        /*const url = new URL(window.location);
        if (url.host === HOST) {
            this.$refs.window.show();
        }*/
        // }
        this.$refs.window.show();
    },

    methods: {
        async downloadKeystoreFiles() {
            const keystoreFiles = this.getKeystoreFiles();

            if (keystoreFiles.length === 1) {
                fileDownload(keystoreFiles[0].file, keystoreFiles[0].name);
            } else if (keystoreFiles.length > 1) {
                fileDownload(await this.createZIP(keystoreFiles), `${FILE_NAME}.zip`);
            }
        },

        async createZIP(keystoreFiles) {
            const zip = new JSZip();
            const folder = zip.folder(FILE_NAME);

            keystoreFiles.forEach((keystorefile) => {
                folder.file(keystorefile.name, keystorefile.file);
            });

            return zip.generateAsync({ type: 'blob' });
        },

        getKeystoreFiles() {
            const keystoreFiles = [];

            this.accounts.forEach((account) => {
                if (account.keystore) {
                    keystoreFiles.push({
                        file: JSON.stringify(account.keystore),
                        name: `${this.$fWallet.getKeystoreFileName(account.keystore.address)}.json`,
                    });
                }
            });

            return keystoreFiles;
        },

        onDownloadBtnClick() {
            this.downloadKeystoreFiles();
        },

        onOkBtnClick() {
            this.$refs.window.hide();
            // window.localStorage.setItem('tmp-msg', true);
        },
    },
};
</script>

<style lang="scss">
@import 'style';
</style>
