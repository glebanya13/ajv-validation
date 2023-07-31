<template>
  <div class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="onLogin">
      <div class="form-group">
        <label for="login">Email</label>
        <input
          v-model="formData.login"
          :class="{ 'is-invalid': isInvalid(errors.has('login')) }"
          @blur="onBlur"
          type="text"
          id="login"
          placeholder="Enter your email"
        />
        <ul class="error-wrapper">
          <li v-for="errorMsg of errors.get('login')" :key="errorMsg">
            {{ errorMsg }}
          </li>
        </ul>
      </div>

      <div class="form-group">
        <label for="password">Password</label>
        <input
          v-model="formData.password"
          :class="{ 'is-invalid': isInvalid(errors.has('password')) }"
          @blur="onBlur"
          type="password"
          id="password"
          placeholder="Enter your password"
        />

        <ul class="error-wrapper">
          <li v-for="errorMsg of errors.get('password')" :key="errorMsg">
            {{ errorMsg }}
          </li>
        </ul>
      </div>

      <button type="submit" :disabled="errors.size > 0" @click="submit('ok')">
        Login
      </button>
    </form>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from "vue";
import login from "@/schemas/login.json";
import Ajv from "ajv";
import ajvFormats from "ajv-formats";
import ajvErrors from "ajv-errors";

export default defineComponent({
  setup() {
    const formData = ref({
      login: "",
      password: "",
    });

    const opts = { allErrors: true };
    const validator = ajvErrors(ajvFormats(new Ajv(opts)));

    let errors = ref<Map<string, string[]>>(new Map());
    watch(
      () => errors.value,
      () => void 0
    );

    function onBlur(e: any) {
      const fieldName = e.target.id;
      const fieldValue = e.target.value;
      if (
        validator.validate(`/login.json#/properties/${fieldName}`, fieldValue)
      ) {
        errors.value.delete(fieldName);
      } else if (validator.errors?.length) {
        errors.value.set(
          fieldName,
          validator.errors.map((e) => e.message) as string[]
        );
      }
    }

    function onLogin() {
      errors.value.clear();
      if (validator.validate("/login.json", formData.value)) {
        // valid, do nothing
      } else if (validator.errors?.length) {
        for (const [, e] of validator.errors.entries()) {
          if (!e.message) {
            continue;
          }
          const fieldName = e.instancePath.substring(1);
          const fieldErrors: string[] = errors.value.get(fieldName) || [];
          fieldErrors.push(e.message);
          errors.value.set(fieldName, fieldErrors);
        }
      }
    }

    function formColor(error: any) {
      return error?.length;
    }

    function submit(text: string) {
      alert(text);
    }

    return {
      formData,
      errors,
      isInvalid: formColor,
      onBlur,
      onLogin,
      submit,
    };
  },
});
</script>

<style>
body {
  font-family: sans-serif;
}

.login-form {
  max-width: 300px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.login-form h2 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
}

.is-invalid {
  border: 1px solid red;
}

.error-wrapper {
  color: red;
  font-size: 12px;
  margin: 0;
  padding: 0 20px;
}
</style>
