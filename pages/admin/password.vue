<template>
    <v-row class="d-flex justify-center">
        <v-col cols="6">
            <v-card elevation="4" class="mx-auto">
                <v-toolbar flat>
                    <v-toolbar-title class="">
                        Ganti Password
                    </v-toolbar-title>
                </v-toolbar>
                <v-card-text>
                    <v-col cols="12">

                        <form @submit.prevent="submit">
                            <v-row class="d-flex align-center" no-gutters dense>
                                <v-col cols="2">Password Lama</v-col>
                                <v-col cols="1">:</v-col>
                                <v-col cols="9">
                                    <v-text-field v-model="values.password_lama" :error-messages="errors.password_lama"
                                        @keypress="validate('password_lama')" @blur="validate('password_lama')"
                                        type="password" label="Password Lama" solo dense>
                                    </v-text-field>
                                </v-col>
                            </v-row>
                            <v-row class="d-flex align-center" no-gutters dense>
                                <v-col cols="2">Password Baru</v-col>
                                <v-col cols="1">:</v-col>
                                <v-col cols="9">
                                    <v-text-field v-model="values.password_baru" :error-messages="errors.password_baru"
                                        @keypress="validate('password_baru')" @blur="validate('password_baru')"
                                        type="password" label="Password Baru" solo dense>
                                    </v-text-field>
                                </v-col>
                            </v-row>
                            <v-row class="d-flex align-center" no-gutters dense>
                                <v-col cols="2">Konfirmasi Password Baru</v-col>
                                <v-col cols="1">:</v-col>
                                <v-col cols="9">
                                    <v-text-field v-model="values.password_confirmation"
                                        :error-messages="errors.password_confirmation"
                                        @keypress="validate('password_confirmation')" type="password"
                                        @blur="validate('password_confirmation')" label="Konfirmasi Password Baru" solo
                                        dense>
                                    </v-text-field>
                                </v-col>
                            </v-row>
                            <v-row class="d-flex align-center" no-gutters dense>
                                <v-col cols="2"></v-col>
                                <v-col cols="1"></v-col>
                                <v-col cols="9">
                                    <v-btn color="primary" type="submit">
                                        SIMPAN
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </form>
                    </v-col>
                </v-card-text>
            </v-card>
        </v-col>
    </v-row>
</template>
<script>
import { object, string, date, mixed, ref } from 'yup'

const loginSchema = object({
    password_lama: string().required('Password Lama harus diisi'),
    password_baru: string().required('Password Baru harus diisi'),
    password_confirmation: string().oneOf([ref('password_baru'), null], 'Password harus sama'),
})
export default {
    layout: 'admin',
    middleware: 'auth',
    data() {
        return {
            values: {
                password_lama: '',
                password_baru: '',
                password_confirmation: '',
            },
            errors: {
                password_lama: '',
                password_baru: '',
                password_confirmation: '',
            },
            menu: false,
            loading: false,
        }
    },
    methods: {
        async submit() {
            loginSchema
                .validate(this.values, { abortEarly: false })
                .then(() => {
                    const fd = new FormData()
                    fd.append('password_lama', this.values.password_lama)
                    fd.append('password_baru', this.values.password_baru)
                    fd.append('password_confirmation', this.values.password_confirmation)

                    this.$axios.$post('/users/password', fd)
                        .then(() => {
                            const Toast = this.$swal.mixin({
                                toast: true,
                                position: 'top-end',
                                showConfirmButton: false,
                                showCloseButton: true,
                                background: '#7C9885',
                                color: 'white',
                                timer: 3000,
                                timerProgressBar: true,
                                didOpen: (toast) => {
                                    toast.addEventListener('mouseenter', this.$swal.stopTimer)
                                    toast.addEventListener('mouseleave', this.$swal.resumeTimer)
                                }
                            })

                            Toast.fire({
                                icon: 'success',
                                title: 'Sukses Ganti Password'
                            })
                            // this.$router.push('/admin/dashboard')
                        })
                        .catch(({ response }) => {
                            this.errors = {};
                            this.$toast.error(response.data.message, {

                            });
                        })
                })
                .catch(err => {
                    err.inner.forEach(error => {
                        this.errors[error.path] = error.message;
                    });
                });
        },
        isNik(evt) {
            evt = evt ? evt : window.event;
            var charCode = evt.which ? evt.which : evt.keyCode;
            var length = evt.target.value.length + 1;
            if (
                charCode > 31 &&
                (charCode < 48 || charCode > 57) &&
                charCode !== 46
            ) {
                if (charCode == 13) {
                    return true
                } else {
                    evt.preventDefault();
                }
            } else {
                if (length == 17) {
                    evt.preventDefault();
                } else {
                    return true;
                }
            }
        },
        isNumber(evt) {
            evt = evt ? evt : window.event;
            var charCode = evt.which ? evt.which : evt.keyCode;
            if (
                charCode > 31 &&
                (charCode < 48 || charCode > 57) &&
                charCode !== 46
            ) {
                evt.preventDefault();
            } else {
                return true;
            }
        },
        validate(field) {
            loginSchema
                .validateAt(field, this.values)
                .then(() => {
                    this.errors[field] = '';
                })
                .catch(err => {
                    this.errors[field] = err.message
                })
        }
    }
}
</script>