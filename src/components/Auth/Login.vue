<template>
    <div>
        <div class="user-modal-container active" id="login-modal">
            <div class="user-modal">

                <ul class="form-switcher">
                    <li v-on:click="flip('register', $event)">
                        <a href="" id="register-form">Register</a>
                    </li>
                    <li v-on:click="flip('login', $event)">
                        <a class="active" href="" id="login-form">Login</a>
                    </li>
                </ul>

                <div class="form-register" id="form-register">
                    <div class="error-message" v-text="registerError"></div>
                    <input type="text" name="name" placeholder="First Name" v-model="register.firstName" v-on:keyup.enter="submit('register', $event)">
                    <input type="text" name="name" placeholder="Last Name" v-model="register.lastName" v-on:keyup.enter="submit('register', $event)">
                    <input type="email" name="email" placeholder="Email" v-model="register.email" v-on:keyup.enter="submit('register', $event)">
                    <div class="select-style">
                        <select id="accountType">
                            <option value="0" selected>Student</option>
                            <option value="1">Faculty</option>
                            <option value="2">Admin</option>
                        </select>
                    </div>
                    <input type="password" name="password" placeholder="Password" v-model="register.password" v-on:keyup.enter="submit('register', $event)">
                    <input type="submit" v-on:click="submit('register', $event)" v-model="registerSubmit" id="registerSubmit">
                    <div class="links"> 
                        <a href="" v-on:click="flip('login', $event)">Already have an account?</a>
                    </div>
                </div>

                <div class="form-login active" id="form-login">
                    <div class="error-message" v-text="loginError"></div>
                    <input type="text" name="user" placeholder="Email" v-model="login.email" v-on:keyup.enter="submit('login', $event)">
                    <input type="password" name="password" placeholder="Password" v-model="login.password" v-on:keyup.enter="submit('login', $event)">
                    <input type="submit" v-on:click="submit('login', $event)" v-model="loginSubmit" id="loginSubmit">
                    <div class="links"> 
                        <a href="" v-on:click="flip('password', $event)">Forgot your password?</a>
                    </div>
                </div>

                <div class="form-password" id="form-password">
                    <div class="error-message" v-text="passwordError"></div>
                    <input type="text" name="email" placeholder="Email" v-model="resetPassword.email" v-on:keyup.enter="submit('password', $event)">
                    <input type="submit" v-on:click="submit('password', $event)" v-model="passwordSubmit" id="passwordSubmit">
                </div>

            </div>
        </div>

    </div>
</template>

<script>
    
    const API_URL = process.env.API_URL

    export default {
        name: 'Login',
        data () {
            return {
                active: 'login',
                registerSubmit: 'Register',
                passwordSubmit: 'Reset Password',
                loginSubmit: 'Login',
                register: {
                    firstName: '',
                    lastName: '',
                    email: '',
                    password: '',
                    accountType: ''
                },
                login: {
                    email: '',
                    password: ''
                },
                resetPassword: {
                    email: ''
                },
                registerError: '',
                loginError: '',
                passwordError: '',
            }
        },
        methods: {
            flip: function (which, e) {
                e.preventDefault();
                if (which !== this.active) {
                    document.getElementById('form-' + this.active).classList.remove('active');
                    document.getElementById('form-' + which).classList.add('active');
                    var el0 = document.getElementById(which + '-form');
                    if (el0) {
                        el0.classList.add('active');
                    }
                    var el1 = document.getElementById(this.active + '-form');
                    if (el1) {
                        el1.classList.remove('active');
                    }
                    this.active = which;
                }
            },
            submit: function (which, e) {
                e.preventDefault();
                document.getElementById(which + 'Submit').classList.add('disabled');
                switch (which) {
                    case "register":
                        this.registerUser(which);
                    case 'login':
                        this.loginUser(which)
                        break;
                    case 'password':
                        this.resetPasswordUser(which)
                        break;
                }
            },
            loginUser (which) {
                var tok = this.login.email.toLowerCase() + ':' + this.login.password
                var hash = btoa(tok)
                this.$http.get(API_URL + '/v1/auth/user/token', {
                  headers: {
                    Authorization: 'Basic ' + hash
                  }
                }).then((response) => {
                    window.localStorage.setItem('user', JSON.stringify(response.data.data));
                    window.localStorage.setItem('token', response.data.data.token)
                    this.$router.push({name: 'admin'})
                }).catch((errors) => {
                    document.getElementById(which + 'Submit').classList.remove('disabled');
                    this.loginError = "You have entered an invalid email or password"
                })
            },
            registerUser (which) {
                this.register.email = this.register.email.toLowerCase()
                this.register.accountType = document.getElementById('accountType').selectedIndex;
                this.$http.post(API_URL + '/v1/auth/user', this.register).then((response) => {
                    window.localStorage.setItem('user', JSON.stringify(response.data.data));
                    window.localStorage.setItem('token', response.data.data.token)
                    this.$router.push({name: 'admin'})
                }).catch((errors) => {
                    document.getElementById(which + 'Submit').classList.remove('disabled');
                    this.registerError = "The email address you have entered is already registered"
                })
            },
            resetPasswordUser (which) {
                document.getElementById(which + 'Submit').classList.remove('disabled');
                this.passwordError = "This is not built yet."
            }
        }
    }
</script>

<style>
    .user-modal-container * {
        box-sizing: border-box;
    }
    .user-modal-container {
        position: fixed;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        opacity: 0;
        visibility: hidden;
        overflow-y: auto;
        z-index: 3;
        font-family:'Lato', 'Helvetica Neue', 'Helvetica', 'Arial', 'sans-serif';
        font-size: 14px;
        background-color: rgba(17, 17, 17, .9);
        -webkit-transition: all 0.25s linear;
        -moz-transition: all 0.25s linear;
        -o-transition: all 0.25s linear;
        -ms-transition: all 0.25s linear;
        transition: all 0.25s linear;
    }
    .user-modal-container.active {
        opacity: 1;
        visibility: visible;
    }
    .user-modal-container .user-modal {
        position: relative;
        margin: 50px auto;
        width: 90%;
        max-width: 500px;
        background-color: #f6f6f6;
        cursor: initial;
    }
    .user-modal-container .form-login, .user-modal-container .form-register, .user-modal-container .form-password {
        padding: 75px 25px 25px;
        display: none;
    }
    .user-modal-container .form-login.active, .user-modal-container .form-register.active, .user-modal-container .form-password.active {
        display: block;
    }
    .user-modal-container ul.form-switcher {
        margin: 0;
        padding: 0;
    }
    .user-modal-container ul.form-switcher li {
        list-style: none;
        display: inline-block;
        width: 50%;
        float: left;
        margin: 0;
    }
    .user-modal-container ul.form-switcher li a {
        width: 100%;
        display: block;
        height: 50px;
        line-height: 50px;
        color: #666666;
        background-color: #dddddd;
        text-align: center;
    }
    .user-modal-container ul.form-switcher li a.active {
        color: #000000;
        background-color: #f6f6f6;
    }
    .user-modal-container input {
        width: 100%;
        padding: 10px;
        margin-bottom: 10px;
        border: 1px solid #eeeeee;
    }
    .user-modal-container input[type="submit"] {
        color: #f6f6f6;
        border: 0;
        margin-bottom: 0;
        background-color: #3fb67b;
        cursor: pointer;
    }
    .user-modal-container input[type="submit"]:hover {
        background-color: #3aa771;
    }
    .user-modal-container input[type="submit"]:active {
        background-color: #379d6b;
    }
    .user-modal-container .links {
        text-align: center;
        padding-top: 25px;
    }
    .user-modal-container .links a {
        color: #3fb67b;
    }
    .user-modal-container input[type="submit"].disabled {
        background-color: #98d6b7;
    }
    .select-style {
        border: 1px solid #eeeeee;
        width: 100%;
        background: #fafafa url("data:image/png;base64,R0lGODlhDwAUAIABAAAAAP///yH5BAEAAAEALAAAAAAPABQAAAIXjI+py+0Po5wH2HsXzmw//lHiSJZmUAAAOw==") no-repeat 97% 50%;
        margin-bottom: 10px;
        background-color: white;
        padding: 5px;
    }
    .select-style select {
        color: #666666;
        padding: 5px 8px;
        width: 100%;
        border: none;
        box-shadow: none;
        background: transparent;
        background-image: none;
        -webkit-appearance: none;
    }
    .select-style select:focus {
        outline: none;
    }
</style>
