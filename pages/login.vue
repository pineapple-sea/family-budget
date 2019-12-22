<template>
  <v-card
    class="mx-auto"
    max-width="344"
    outlined
  >
    <v-list-item three-line>
      <v-list-item-content>

        <div
          v-if="state === 'signin'"
          class="overline mb-4"
          @click="changeState('signup')"
        >
          Sign-up
        </div>

        <div
          v-if="state === 'signup'"
          class="overline mb-4"
          @click="changeState('signin')"
        >
          Sign-in
        </div>

        <v-list-item-title class="headline mb-1">
          {{ state === 'signin' ? 'Sign-in' : 'Sign-up' }}
        </v-list-item-title>

        <v-form
          v-if="state === 'signin'"
          ref="form"
          v-model="valid"
          lazy-validation
        >
          <v-text-field
            v-model="email"
            :rules="emailRules"
            label="E-mail"
            required
          ></v-text-field>

          <v-text-field
            v-model="password"
            :rules="passwordRules"
            label="Password"
            required
          ></v-text-field>
        </v-form>

        <v-form
          v-if="state === 'signup'"
          ref="form"
          v-model="valid"
          lazy-validation
        >
          <v-text-field
            v-model="email"
            :rules="emailRules"
            label="E-mail"
            required
          ></v-text-field>

          <v-text-field
            v-model="password"
            :rules="passwordRules"
            label="Password"
            required
          ></v-text-field>
          <v-text-field
            v-model="password"
            :rules="passwordRules"
            label="Confirm password"
            required
          ></v-text-field>
        </v-form>
      </v-list-item-content>
    </v-list-item>

    <v-card-actions>
      <v-btn
        v-if="state === 'signin'" 
        :disabled="!valid"
        color="success"
        class="mr-4"
        @click="login"
      >
        Sign-in
      </v-btn>

      <v-btn
        v-if="state === 'signup'"
        :disabled="!valid"
        color="success"
        class="mr-4"
        @click="register"
      >
        Sign-up
      </v-btn>

      <v-btn
        v-if="state === 'signin'"
        color="success"
        class="mr-4"
        @click="forgotPassword" 
      >
        Forgot your password?
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>

export default {
  layout: 'login',
  data: () => ({
    email: '',
    emailRules: [
      v => !!v || 'E-mail is required',
      v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
    ],
    valid: true,
    password: '',
    passwordRules: [
      v => !!v || 'Password is required',
      v => (v && v.length >= 6) || 'Password must be at least 6 characters long',
    ],
    state: 'signin',
  }),

  methods: {
    login () {
      debugger;
      if (this.$refs.form.validate()) {
        this.snackbar = true
        this.loginRegular(this.email, this.password)
      }
    },
    register () {

    },
    forgotPassword () {

    },
    changeState (state) {
      this.state = state
    },
    reset () {
      this.$refs.form.reset()
    },
    resetValidation () {
      this.$refs.form.resetValidation()
    },
  },
}
</script>
