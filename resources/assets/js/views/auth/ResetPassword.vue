<template>
  <form id="loginForm" @submit.prevent="validateBeforeSubmit">
    <div class="form-group">
      <sw-input
        v-model.trim="formData.email"
        :invalid="$v.formData.email.$error"
        :placeholder="$t('login.enter_email')"
        type="email"
        name="email"
        @input="$v.formData.email.$touch()"
      />
      <div v-if="$v.formData.email.$error">
        <span v-if="!$v.formData.email.required" class="help-block text-danger">
          {{ $t('validation.required') }}
        </span>
        <span v-if="!$v.formData.email.email" class="help-block text-danger">
          {{ $t('validation.email_incorrect') }}
        </span>
      </div>
    </div>
    <div class="form-group">
      <sw-input
        id="password"
        v-model.trim="formData.password"
        :invalid="$v.formData.password.$error"
        :placeholder="$t('login.enter_password')"
        type="password"
        name="password"
        @input="$v.formData.password.$touch()"
      />
      <div v-if="$v.formData.password.$error">
        <span
          v-if="!$v.formData.password.required"
          class="help-block text-danger"
        >
          {{ $t('validation.required') }}
        </span>
        <span
          v-if="!$v.formData.password.minLength"
          class="help-block text-danger"
        >
          {{
            $tc(
              'validation.password_length',
              $v.formData.password.minLength.min,
              { count: $v.formData.password.$params.minLength.min }
            )
          }}
        </span>
      </div>
    </div>
    <div class="form-group">
      <sw-input
        v-model.trim="formData.password_confirmation"
        :invalid="$v.formData.password_confirmation.$error"
        :placeholder="$t('login.retype_password')"
        type="password"
        name="password_confirmation"
        @input="$v.formData.password_confirmation.$touch()"
      />
      <div v-if="$v.formData.password_confirmation.$error">
        <span
          v-if="!$v.formData.password_confirmation.sameAsPassword"
          class="help-block text-danger"
        >
          {{ $t('validation.password_incorrect') }}
        </span>
      </div>
    </div>
    <sw-button type="submit" variant="primary">
      {{ $t('login.reset_password') }}
    </sw-button>
  </form>
</template>

<script type="text/babel">
const {
  required,
  email,
  sameAs,
  minLength,
} = require('vuelidate/lib/validators')

export default {
  data() {
    return {
      formData: {
        email: '',
        password: '',
        password_confirmation: '',
      },
      isLoading: false,
    }
  },
  validations: {
    formData: {
      email: {
        required,
        email,
      },
      password: {
        required,
        minLength: minLength(8),
      },
      password_confirmation: {
        sameAsPassword: sameAs('password'),
      },
    },
  },
  methods: {
    async validateBeforeSubmit(e) {
      this.$v.formData.$touch()

      if (!this.$v.formData.$invalid) {
        try {
          let data = {
            email: this.formData.email,
            password: this.formData.password,
            password_confirmation: this.formData.password_confirmation,
            token: this.$route.params.token,
          }
          this.isLoading = true
          let res = await axios.post('/api/v1/auth/reset/password', data)
          this.isLoading = false
          if (res.data) {
            toastr['success'](
              this.$t('login.password_reset_successfully'),
              'Success'
            )
            this.$router.push('/login')
          }
        } catch (err) {
          if (err.response && err.response.status === 403) {
            toastr['error'](
              err.response.data,
              this.$t('validation.email_incorrect')
            )
            this.isLoading = false
          }
        }
      }
    },
  },
}
</script>
