<template>
    <div class="pa-8">
        <div class="account-id">
            <div class="entry-text">Account ID</div>
            <div class="entry-value">{{getActiveAccount()}}</div>
        </div>

        <div class="account-id">
            <div class="entry-text">Dashboard Version</div>
            <div class="entry-value">{{this.dashboardVersion}}</div>
        </div>

        <div class="account-id">
            <div class="entry-text">Runtime Version</div>
            <div class="entry-value">{{this.apiRuntimeVersion}}</div>
        </div>

        <div class="account-id" v-if="this.lastLoginTs">
            <div class="entry-text">Last Login time</div>
            <div class="entry-value">{{epochToDateTime(this.lastLoginTs)}}</div>
        </div>

        <div class="toggle-redact-feature" v-if="localSetupType !== null">
            <div class="entry-text">Setup Type</div>
            <div class="entry-value">
                <div class="text-center">
                    <v-menu offset-y>
                        <template v-slot:activator="{ on, attrs }">
                            <span
                                v-bind="attrs"
                                v-on="on"
                                style="text-decoration: underline"
                            >
                                {{localSetupType}}
                            </span>
                        </template>
                        <v-list>
                            <v-list-item
                                v-for="(item, index) in setup_types"
                                :key="index"
                                @click="localSetupType = item"
                            >
                                <v-list-item-title>{{ item }}</v-list-item-title>
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </div>
            </div>
        </div>

        <div class="toggle-redact-feature" v-if="localRedactPayload !== null">
            <div class="entry-text">Redact sample data</div>
            <div class="entry-value">
                <v-switch
                    color="var(--themeColor)"
                    v-model="localRedactPayload"
                />
            </div>
        </div>

        <div class="toggle-redact-feature" v-if="!localMergeAsyncOutside">
            <div class="entry-text">Activate new merging</div>
            <div class="entry-value">
                <v-switch
                    color="var(--themeColor)"
                    v-model="localMergeAsyncOutside"
                />
            </div>
        </div>

        <div class="toggle-redact-feature">
            <div class="entry-text">Enable New Merging</div>
            <div class="entry-value">
                <v-switch
                    color="var(--themeColor)"
                    v-model="newMerging"
                />
            </div>
        </div>

        <div class="toggle-redact-feature" v-if="localTrafficAlertThresholdSeconds">
            <div class="entry-text">Traffic alert threshold</div>
            <div class="entry-value">
                <div class="text-center">
                    <v-menu offset-y>
                        <template v-slot:activator="{ on, attrs }">
                            <span
                                v-bind="attrs"
                                v-on="on"
                                style="text-decoration: underline"
                            >
                                {{convertSecondsToReadableTime(localTrafficAlertThresholdSeconds)}}
                            </span>
                        </template>
                        <v-list>
                            <v-list-item
                                v-for="(item, index) in traffic_alert_durations"
                                :key="index"
                                @click="localTrafficAlertThresholdSeconds = item.value"
                            >
                                <v-list-item-title>{{ item.text }}</v-list-item-title>
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
import func from "@/util/func";
import {mapState} from 'vuex'
    export default {
        name: "Account",
        components: { 
        },
        data () {
            return {
                setup_types: ["STAGING", "PROD", "QA", "DEV"],
                traffic_alert_durations: [
                    {text : "1 hour", value: 60*60*1},
                    {text : "4 hours", value: 60*60*4},
                    {text : "12 hours", value: 60*60*12},
                    {text : "1 Day", value: 60*60*24},
                    {text : "4 Days", value: 60*60*24*4},
                ]
            }
        },
        methods: {
            convertSecondsToReadableTime(val) {
                return func.convertSecondsToReadableTime(val)
            },
            getActiveAccount() {
                return this.$store.state.auth.activeAccount
            },
            epochToDateTime(timestamp) {
                return func.epochToDateTime(timestamp)
            }
        },
        mounted() {
            this.$store.dispatch('team/fetchAdminSettings')
            this.$store.dispatch('team/fetchUserLastLoginTs')
        },
        computed: {
            ...mapState('team', ['redactPayload', 'setupType', 'dashboardVersion', 'apiRuntimeVersion', 'mergeAsyncOutside', 'lastLoginTs', 'urlRegexMatchingEnabled', 'trafficAlertThresholdSeconds']),
            localRedactPayload: {
                get() {
                    return this.redactPayload
                },
                set(v) {
                    this.$store.dispatch('team/toggleRedactFeature', v)
                }
            },
            localSetupType: {
              get() {
                return this.setupType
              },
              set(v) {
                this.$store.dispatch('team/updateSetupType', v)
              }
            },
            localTrafficAlertThresholdSeconds: {
              get() {
                return this.trafficAlertThresholdSeconds
              },
              set(v) {
                this.$store.dispatch('team/updateTrafficAlertThresholdSeconds', v)
              }
            },
            localMergeAsyncOutside: {
                get() {
                    return this.mergeAsyncOutside
                },
                set(v) {
                    this.$store.dispatch('team/updateMergeAsyncOutside')
                }
            },
            newMerging: {
                get() {
                    return this.urlRegexMatchingEnabled
                },
                set(v) {
                    console.log(v)
                    this.$store.dispatch('team/updateEnableNewMerge', v)
                }
            }
        }
    }
</script>

<style lang="sass" scoped>
.entry-text
    font-weight: 500
    margin-right: 16px
    width: 200px


.account-id
    display: flex
    height: 50px
    vertical-align: middle
    line-height: 50px


.toggle-redact-feature
    display: flex
    height: 60px
    line-height: 60px
</style>

<style>
     .toggle-redact-feature .v-input--selection-controls {
        margin-top:20px;
        padding-top: 0px;
    }
</style>


