<template>
    <modal ref="modal" :title="$t('share.title')" @closed="clearTimer">
        <template v-slot:content>
            <p>Share the following campaign code with your party members to enable progress sync across all
                your devices!</p>
            <div v-if="code && progress" class="flex items-center flex-col my-6">
                <a v-clipboard:copy="code.code" class="cursor-pointer copied">
                    <countdown :label="code.code" :percentage="progress.percentage"></countdown>
                </a>
                <span>{{ progress.label }}</span>
                <span class="text-sm">(The code is only valid for one hour)</span>
            </div>
            <share-icons v-if="code && progress && url"
                         :url="url"
                         :networks="['whatsapp', 'email']"
                         :description="'Your campaign code: ' + code.code + ' (valid until: ' + progress.time+')'"
            />
        </template>
    </modal>
</template>

<script>
    import tippy from 'tippy.js';
    import StoryCodeRepository from "../../apiRepositories/StoryCodeRepository";

    export default {
        data() {
            return {
                url: '',
                story: null,
                code: null,
                progress: null,
                timer: null,
                copyTippy: null,
                codeRepository: new StoryCodeRepository
            }
        },
        mounted() {
            this.$bus.$on('open-share-campaign-code-modal', this.open);
        },
        destroyed() {
            this.clearTimer();
        },
        methods: {
            async open(story) {
                this.story = story;
                this.$refs['modal'].open();
                await this.fetchCode();
                this.setUrl();
                this.setTimer();
                this.addCopyTippy();
            },
            async fetchCode() {
                this.code = await this.codeRepository.find(this.story);
                this.updatePercentage();
            },
            updatePercentage() {
                if (this.code) {
                    this.progress = this.code.expirationProgress();
                    if (this.progress.percentage === 0) {
                        this.fetchCode();
                    }
                }
            },
            addCopyTippy() {
                if (this.copyTippy) {
                    return;
                }

                this.copyTippy = tippy('.copied', {
                    trigger: 'click',
                    content: this.$t('Copied'),
                    onShown(tippy) {
                        setTimeout(() => {
                            tippy.hide();
                        }, 1500);
                    }
                });
            },
            setTimer() {
                this.clearTimer();
                this.timer = setInterval(this.updatePercentage, 1000);
            },
            clearTimer() {
                if (this.timer) {
                    clearInterval(this.timer);
                }
            },
            setUrl() {
                this.url = location.href.replace('/#/', `/?code=${this.code.code}#/`);
            }
        }
    }
</script>
