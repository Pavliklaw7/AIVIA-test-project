<template>
  <v-container>
    <v-card>
      <v-card-title>Login</v-card-title>
      <v-card-text>
        <v-form ref="form" @submit.prevent="submit">
          <v-text-field
            v-model="email"
            label="Email"
            :error="v$.email.$dirty && v$.email.$error"
            :error-messages="emailErrors"
            required
          ></v-text-field>

          <v-text-field
            v-model="password"
            type="password"
            label="Password"
            :error="v$.password.$dirty && v$.password.$error"
            :error-messages="passwordErrors"
            required
          ></v-text-field>

          <v-btn type="submit" color="primary">Submit</v-btn>
        </v-form>
        <v-alert v-if="formHasErrors" type="error">Please correct the errors before submitting the form.</v-alert>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import { ref, computed } from 'vue';
import useVuelidate from '@vuelidate/core';
import { required, email, minLength } from '@vuelidate/validators';
import { defineComponent } from 'vue';
import router from '@/router';

export default defineComponent({
  setup() {
    const password = ref('');
    const email = ref('');

    const mustBeEmail = (email) => {
      return String(email)
      .toLowerCase()
      .match(
        /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|.(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      );
    };


    const rules = computed(() => {
      return {
        email: { mustBeEmail, required, $autoDirty: true },
        password: { required, minLength: minLength(6), $autoDirty: true }
      }
})

    const v$ = useVuelidate(rules, { email, password });

    const emailErrors = computed(() => {
      const errors = [];
      if (v$.value.email.$dirty) {
        if (!v$.value.email.mustBeEmail) {
          errors.push('Email is required');
        } else if (!v$.value.email.email) {
          errors.push('Email must be valid');
        }
      }
      return errors;
    });

    const passwordErrors = computed(() => {
      const errors = [];
      if (v$.value.password.$dirty) {
        if (!v$.value.password.required) {
          errors.push('Password is required');
        } else if (!v$.value.password.minLength) {
          errors.push('Password must be at least 6 characters long');
        }
      }
      return errors;
    });

    const submit = async () => {
      const result = await v$.value.$validate()

      console.log(result)
      v$.value.$touch();
      if (!v$.value.$invalid) {
        router.push('/game')
      }
    };

    const formHasErrors = computed(() => {
      return v$.value.$anyError;
    });


    const shouldDisplayError = (fieldName) => {
      return v$?.[fieldName]?.$dirty && !v$?.[fieldName]?.$pending && !v$?.[fieldName]?.$model;
    };


    return {
      email,
      password,
      emailErrors,
      passwordErrors,
      submit,
      shouldDisplayError,
      formHasErrors,
      v$
    };
  }
});
</script>