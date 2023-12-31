<template>
    <v-sheet class="about-content">
        <v-container>
            <v-row justify="center">
                <v-col class="col-md-6" justify="center">
                    <v-card>
                        <div id="formInfo">
                            <v-card-title  class="d-flex justify-space-between elevation-1">
                                <h5>Create new manager</h5>
                            </v-card-title>
                        </div>
                        <v-form ref="form" v-model="valid" lazy-validation>
                        <v-card-text class="pt-0">
                            <v-text-field
                                label="Full name"
                                v-model="formData.name"
                                :rules="rules.nameRules"
                                required
                            ></v-text-field>
                            <v-text-field
                                label="Login"
                                v-model="formData.login"
                                :rules="rules.loginRules"
                                required
                            ></v-text-field>
                            <v-text-field
                                label="Password"
                                type="password"
                                v-model="formData.password"
                                :rules="rules.passwordRules"
                            ></v-text-field>
                            <v-text-field
                                label="Confirm Password"
                                type="password"
                                v-model="formData.confirmPassword"
                                :rules="rules.confirmPasswordRules"
                                required
                            ></v-text-field>
                            <v-btn color="primary" large class="mt-3 mb-3" @click="submit">
                                Save
                            </v-btn>
                        </v-card-text>
                    </v-form>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
    </v-sheet>
</template>

<script>

import { mapGetters, mapActions } from "vuex"

export default {
    layout: "crm",
    middleware: ['admin'],
    data(){
        return {
        valid: true,
        roles: [
            'employee',
            'redactor',
            'root'
        ],
        formData: {
            name: '',
            login: '',
            email: '',
            phone: '',
            password: '',
            confirmPassword: '',
            role: 'employee'
        },
        rules: {
            loginRules: [
                v => !!v || 'Login is required',
                v => (v && v.length <= 10) || 'Login must be less than 10 characters',
            ],
            nameRules: [
                v => !!v || 'Name is required',
            ],
            passwordRules: [
                v => !!v || 'Password is required',
            ],
            confirmPasswordRules: [
                (value) => !!value || 'Type confirm password',
                (value) =>
                value === this.formData.password || 'The password confirmation does not match.',
            ],
            agreementRules: [
                (value) => !!value || 'At least one item should be selected',
            ]
        }
    }},
    mounted() {
        this.formData.role = this.$route.query.type
    },
    computed: mapGetters({
        employees: 'admin/getEmployees'
    }),
    methods: {
        ...mapActions({
            createEmployee: 'admin/createEmployee'
        }),
        submit(){
            this.formData.role = 'manager'
            if (this.$refs.form.validate()) {
                this.createEmployee(this.formData).then(response => {
                    const lastEmployee = this.employees[0]
                    this.$socket.emit('refreshUsersData')
                    this.$router.push(`/crm/employees`)
                })
            }
        }
    }
}
</script>

<style lang="scss" scoped>
.display-1 {
   color: $custom_blue !important;
   font-weight: 700 !important;
}

.about-content {
   p {
       color: $custom_red !important
   }
   .banner {
       height: 70vh;
       position: relative;
       &__content {
           position: absolute;
           top: 0;
           left: 0;
           width: 100%;
           height: 100%;
           z-index: 3;
           display: flex;
           justify-content: flex-start;
           flex-direction: column;
           padding: 0 30px;
           padding-top: 50px;
       }
       .v-card__title {
           text-align: left;
       }
       .v-card__text {
           text-align: left !important;
       }
       .v-btn {
           margin-left: 16px;
           max-width: 150px;
       }
       .v-image {
           width: 100%;
           height: 100%;
           object-fit: cover;
           object-position: center;
           position: absolute;
           z-index: 1;
           top: 0;
           left: 0;
       }
       &:after {
           content: "";
           position: absolute;
           left: 0;
           right: 0;
           width: 100%;
           height: 100%;
           background-color: rgba($color: $custom_blue, $alpha: 0.7);
           z-index: 2;
       }
   }
   .avatar-relative {
       position: relative;
       top: -80px;
       z-index: 4;
       max-width: 400px;
       min-width: 350px;
       margin-left: auto;
       margin-right: auto;
   }
}

.avatar-image {
   padding-top: 80px;
   .heading {
       font-size: 25px;
       color: $custom_blue;
       font-weight: bold;
   }
   .description button {
       text-decoration: underline;
   }
   .infoCust {
       .description {
           margin-top: 10px;
       }
       button {
           font-weight: normal;
           color: inherit;
       }
   }
}

.avatar-image .v-image {
   position: absolute;
   width: 160px;
   height: 160px;
   top: -79px;
   left: calc(50% - 80px);
   border-radius: 50% !important;
}

.avatar-image .edit {
   position: absolute;
   top: 30px;
   left: 200px
}

@media (min-width: 991px) {
   .grid-desk {
       display: flex;
   }
   .profile-edit {
       order: 1;
       margin-right: 20px;
       position: relative;
       z-index: 2;
   }
   .avatar-relative {
       order: 2;
   }
}
</style>
