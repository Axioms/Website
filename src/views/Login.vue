<template>
    <div class="login">
        <b-container>
            <b-card bg-variant="light" class="border">
                <h5 class="text-dark pad-down">Login</h5>
                <b-row>
                    <b-col cols="3"></b-col>
                    <b-col>
                        <b-form @submit.prevent="login()">
                            <b-form-group class="form" :state="$v.loginData.username.$dirty ? !$v.loginData.username.$error : null" invalid-feedback="username must be between 4 and 27 characters long">
                                <b-form-input id="input-1" :state="$v.loginData.username.$dirty ? !$v.loginData.username.$error : null" v-model="$v.loginData.username.$model" type="text" required placeholder="Username"></b-form-input>
                            </b-form-group>

                            <b-form-group class="form" :state="$v.loginData.password.$dirty ? !$v.loginData.password.$error : null" invalid-feedback="password must be between 8 and 27 characters long">
                                <b-form-input id="input-2" :state="$v.loginData.password.$dirty ? !$v.loginData.password.$error : null" v-model="$v.loginData.password.$model" type="password" required placeholder="Password"></b-form-input>
                            </b-form-group>

                            <b-form-group class="form" v-if="isSecondFactorRequied">
                                <b-form-input id="input-3" v-model="$v.loginData.otp.$model" type="text" required placeholder="One Time Password"></b-form-input>
                            </b-form-group>

                            <b-button type="submit" :disabled="$v.loginData.$invalid" variant="outline-primary">Submit</b-button>
                        </b-form>
                    </b-col>
                    <b-col cols="3"></b-col>
                </b-row>
                <b-row class="pt-4">
                    <b-col>
                        <router-link to="/passwordreset">Forgot Password</router-link>
                    </b-col>
                </b-row>
                <b-row>
                    <b-col></b-col>
                    <b-col cols="8">
                        <b-alert :show="dismissCountDown" fade dismissible variant="danger" @dismissed="dismissCountDown=0" @dismiss-count-down="countDownChanged" class="alert">
                            <p>{{ this.apiErr }}</p>
                            <b-progress variant="danger" :max="dismissSecs" :value="dismissCountDown" height="4px"></b-progress>
                        </b-alert>
                    </b-col>
                    <b-col></b-col>
                </b-row>
                <div class="login-logo">
                    <img src="@/assets/logo-dark.svg" id="navbar-icon"/>
                </div>
            </b-card>
        </b-container>
    </div>
</template>

<script>
    import { validationMixin } from 'vuelidate';
    import { required, minLength, maxLength } from 'vuelidate/lib/validators';
    export default {
        mixins: [validationMixin],
        data() {
            return {
                loginData: {
                    username: '',
                    password: '',
                    otp: ''
                },
                apiErr: null,
                dismissSecs: 10,
                dismissCountDown: 0,
                isSecondFactorRequied: false
            }
        },
        validations: {
                loginData: {
                    username: {
                        required,
                        minLenght: minLength(4),
                        maxLength: maxLength(27)
                    },
                    password: {
                        required,
                        minLength: minLength(8),
                        maxLength: maxLength(27)
                    },
                    otp: {
                        minLenght: minLength(0)
                    }
                }
            },
        methods: {
            async login() {
                let loginInfo = this.loginData.username + ":" + this.loginData.password;
                let config = {
                    headers: {
                        'X-Axiom-Identity-Token': btoa(loginInfo).replace(/=+$/,""),
                        'X-Axiom-Signed-Otp': btoa(this.loginData.otp).replace(/=+$/,""),
                    }
                };
                this.$axios.post(process.env.VUE_APP_API + '/auth/login', {}, config)
                .then((response) => {
                    this.$store.commit('setJWT', response.data.token);
                    if(this.$store.getters.jwtUuid) {
                        this.$store.commit("setUsername", this.loginData.username);

                        if(this.loginData.otp != '') {
                            this.$store.commit("setUsedOTP", true);
                        }

                        this.$router.push({ name: 'home', query: { redirect: '/' } });
                    } else {
                        this.$store.commit('setJWT', '');
                    }
                })
                .catch((error) => {

                    if(error.response.data.message != "Second Factor Auth Is Required To Login") {
                        this.apiErr = error.response.data.message;
                        if(this.apiErr == undefined) {
                            this.apiErr = "Could not communicate with the server.";
                        }
                        this.showAlert();
                    }
                    else {
                        this.isSecondFactorRequied = true;
                    }
                });
            },
            countDownChanged(dismissCountDown) {
                this.dismissCountDown = dismissCountDown;
            },
            showAlert() {
                this.dismissCountDown = this.dismissSecs;
            }
        }
    }
</script>

<style lang="scss" scoped>
    @media only screen and (max-width: 768px) {
        .container {
            padding: 0px !important;
        }
    }
    .test {
        padding: 30px;
    }
    .container {
        padding: 0rem 11rem;
    }
    .login-con {
        border: 1px solid #000; 
    }
    .login-logo {
        padding-top: 1rem;
    }
    .card {
        padding-bottom: 0% !important;
    }
    .pad-down {
        padding-bottom: 2rem;
    }
    .alert {
        margin: 1rem 0rem;
    }
    .form {
        text-align: left;
    }
</style>