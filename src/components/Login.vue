<template>
    <div id="login">
        <table>
            <tbody>
            <tr>
                <td class="half-width">
                    <button @click="switchForm()">
                        {{buttonReverseName()}}
                    </button>
                </td>
                <td>
                    <button @click="loginAsGuest()">
                        {{$t('continueAsGuest')}}
                    </button>
                </td>
            </tr>
            <tr style="height: 200px">
                <td></td>
                <td></td>
            </tr>
            <tr>
                <td>
                    {{"E-mail"}}
                </td>
                <td>
                    <input type="text" name="username" v-model="username"/>
                </td>
            </tr>
            <tr>
                <td>
                    {{$t('password')}}
                </td>
                <td>
                    <input type="password" name="password" v-model="password"/>
                </td>
            </tr>
            <tr v-if="!isLogin">
                <td>
                    {{$t("repeatPassword")}}
                </td>
                <td>
                    <input v-model="repeatedPassword"/>
                </td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <button v-on:click="performLoginPageAction()">{{buttonName()}}</button>
                </td>
            </tr>
            <tr v-if="incorrectCredentials" class="warning-message">
                <td colspan="2">
                    {{getIncorrectLoginOrPasswordMessage()}}
                </td>
            </tr>
            <tr v-for="message in validationMessages" v-bind:value="message" class="warning-message">
                <td colspan="2">
                    {{message}}
                </td>
            </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    import axios from 'axios';
    import {mapState} from 'vuex';

    export default {
        data() {
            return {
                isLogin: true,
                username: "pazuk1985@gmail.com",
                password: "admin",
                repeatedPassword: "",
                validationMessages: []
            };
        },

        computed: {
            ...mapState({
                basicUrl: state => state.dictionary.basicUrl,
                incorrectCredentials: state => state.dictionary.incorrectCredentials,
                motorcycleCatalogueId: state => state.dictionary.motorcycleCatalogueId,
                appLanguage: state => state.dictionary.appLanguage
            })
        },

        created() {
            this.onUrlChange();
        },

        watch: {
            '$route': 'onUrlChange'
        },

        methods: {
            onUrlChange() {
                this.$i18n.locale = this.appLanguage;
            },

            performLoginPageAction() {
                if (this.isLogin) {
                    this.login();
                } else {
                    this.signUp();
                }
            },

            loginIfValid(validationMessages, newUserName) {
                this.validationMessages = validationMessages;
                if (this.validationMessages.length === 0) {
                    console.log("a new user created: " + newUserName);
                    this.login();
                }
            },

            loginAsGuest() {
                this.username = "guest";
                this.password = "guest";
                this.login();
            },

            login() {
                let credentialsUrl ="username=" + this.username + "&" + "password=" + this.password;
                axios
                    .post(this.basicUrl + "/login", credentialsUrl)
                    .then(response => {
                        if (response.status === 200) {
                            this.$store.dispatch("setLoadingState", true);
                            this.setIncorrectCredentials(false);
                            let authorization = response.data.Authorization;
                            this.$store.dispatch("setAuthorization", authorization);
                            this.$store.dispatch("setUserName", this.username);
                            this.pushToHome();
                            console.log("logged in as " + this.username);
                        }
                    })
                    .catch(error => {
                        this.setIncorrectCredentials(true);
                        console.log("login failed: " + this.getIncorrectLoginOrPasswordMessage());
                    });
            },

            pushToHome() {
                this.$router.push({ path: `/item/id/${this.motorcycleCatalogueId}/${this.appLanguage}` });
            },

            signUp() {
                let newUser = {
                    name: this.username,
                    password: this.password,
                    repeatedPassword: this.repeatedPassword
                };
                axios
                    .post(this.basicUrl + "/user/create", newUser)
                    .then(response => {this.loginIfValid(response.data, newUser.name)});
            },

            switchForm() {
                this.isLogin = !this.isLogin;
                this.resetData();
            },

            buttonName() {
                return this.isLogin ? this.$t("loginButton") : this.$t("signUp");
            },

            buttonReverseName() {
                return this.isLogin ? this.$t("signUp") : this.$t("loginButton");
            },

            resetData() {
                this.resetUserData();
                this.validationMessages = [];
                this.$store.dispatch("setIncorrectCredentials", false);
            },

            resetUserData() {
                this.username = "";
                this.password = "";
                this.repeatedPassword = "";
            },

            setIncorrectCredentials(incorrectCredentials) {
                this.$store.dispatch("setIncorrectCredentials", incorrectCredentials);
            },

            getIncorrectLoginOrPasswordMessage() {
                return this.$t("incorrectLoginOrPassword");
            }
        }
    }
</script>

<style scoped>
    table {
        text-align: left;
    }

    .half-width {
        width: 50%;
    }

    .warning-message {
        text-align: center;
        color: red;
    }
</style>