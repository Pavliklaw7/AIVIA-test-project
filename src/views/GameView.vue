<template>
  <v-container>
    <v-card>
      <v-card-title>Draw Squares</v-card-title>
      <v-card-text>
        <v-form ref="form" @submit.prevent="submit">
          <v-text-field
            v-model.number="x"
            label="X (number of squares horizontally)"
            :error="v$.x.$dirty && v$.x.$error"
            :error-messages="xErrors"
            required
            type="number"
          ></v-text-field>

          <v-text-field
            v-model.number="y"
            label="Y (number of squares vertically)"
            :error="v$.y.$dirty && v$.y.$error"
            :error-messages="yErrors"
            required
            type="number"
          ></v-text-field>

          <v-btn type="submit" color="primary">Draw</v-btn>
        </v-form>
        <v-alert v-if="formHasErrors" type="error" class="mt-4">Please correct the errors before submitting the form.</v-alert>
        <div v-if="drawSquares" class="squares-container">
          <div
            v-for="(row, index) in y"
            :key="'row-' + row"
            class="square-row"
            :id="'square-' + index"
          >
            <div
              v-for="col in x"
              :key="'col-' + col"
              class="square"
              :id="'square-' + ((row - 1) * x + col)"
              @mouseenter="handleMouseEnter(((row - 1) * x + col))"
            ></div>
          </div>
        </div>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import { ref, computed } from 'vue';
import useVuelidate from '@vuelidate/core';
import { required, minValue } from '@vuelidate/validators';
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const x = ref(0);
    const y = ref(0);
    const drawSquares = ref(false);

    const rules = {
      x: { required, minValue: minValue(1) },
      y: { required, minValue: minValue(1) }
    };

    const v$ = useVuelidate(rules, { x, y });

    const xErrors = computed(() => {
      const errors = [];
      if (v$.value.x.$dirty) {
        if (!v$.value.x.required) {
          errors.push('X is required');
        } else if (!v$.value.x.minValue) {
          errors.push('X must be at least 1');
        }
      }
      return errors;
    });

    const yErrors = computed(() => {
      const errors = [];
      if (v$.value.y.$dirty) {
        if (!v$.value.y.required) {
          errors.push('Y is required');
        } else if (!v$.value.y.minValue) {
          errors.push('Y must be at least 1');
        }
      }
      return errors;
    });

    const submit = () => {
      v$.value.$touch();
      if (!v$.value.$invalid) {
        drawSquares.value = true;
      }
    };

    const formHasErrors = computed(() => {
      return v$.value.$anyError;
    });

    const handleMouseEnter = (index) => {
      const square = document.querySelector(`#square-${index}`);
      if (!square.classList.contains('square-hovered')) {
        square.classList.add('square-hovered');
      } else {
        square.classList.remove('square-hovered');
      }
    };

    return {
      x,
      y,
      drawSquares,
      xErrors,
      yErrors,
      submit,
      formHasErrors,
      handleMouseEnter,
      v$
    };
  }
});
</script>

<style scoped>
.mt-4 {
  margin-top: 1rem;
}
.squares-container {
  display: flex;
  flex-direction: column;
}
.square-row {
  display: flex;
}
.square {
  width: 36px;
  height: 36px;
  background-color: lightblue;
  border: 1px solid #000;
  box-sizing: border-box;
}

.square-hovered {
  background-color: blue;
}
</style>
