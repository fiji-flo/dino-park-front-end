<template>
  <div class="input">
    <input
      :class="{ ['input--w-error']: Boolean(errorMsg) }"
      :type="type"
      :min="min"
      :id="id"
      :placeholder="placeholder"
      :maxlength="maxlength"
      :pattern="pattern"
      :value="value"
      :required="required"
      @input="$emit('input', $event.target.value)"
      @focus="handleInputFocus"
    />
    <span v-if="maxlength">{{ value.length }}/{{ maxlength }}</span>
    <span v-if="errorMsg" class="input__error-msg">{{ errorMsg }}</span>
  </div>
</template>

<script>
export default {
  name: 'TextInput',
  props: {
    type: {
      type: String,
      default: 'text',
    },
    placeholder: String,
    id: String,
    maxlength: Number,
    value: {
      type: [String, Number],
      default: '',
    },
    min: {
      type: Number,
      default: null,
    },
    errorMsg: {
      type: String,
      default: '',
    },
    pattern: {
      type: String,
      default: '.*',
    },
    required: Boolean,
  },
  methods: {
    handleInputFocus() {
      this.$emit('focus');
    },
  },
};
</script>

<style>
.input {
  position: relative;
}
.input > input {
  border: 1px solid transparent;
  background-color: var(--gray-20);
  border-radius: var(--formElementRadius);
  color: var(--black);
  margin: 0;
  width: 100%;
  font-size: 1em;
  padding: 0.5em 0.9em;
}

.input > input:hover {
  border-color: var(--blue-60);
}

.input input[maxlength] {
  padding: 0.5em 4.5em 0.5em 0.9em;
}

.input input[maxlength] + span {
  position: absolute;
  right: 1em;
  top: 0.5em;
  color: var(--gray-50);
  letter-spacing: 0.1em;
}

input.input--w-error ~ .input__error-msg {
  color: var(--neon-red);
  padding: 0em 1em;
  font-size: small;
  display: inline-block;
}
input:focus:invalid.input--w-error ~ .input__error-msg,
input:valid.input--w-error ~ .input__error-msg {
  display: none;
}
input:invalid:not(focus).input--w-error ~ .input__error-msg {
  display: inline-block;
}
input:invalid:not(focus).input--w-error {
  border-color: var(--neon-red);
}
</style>
