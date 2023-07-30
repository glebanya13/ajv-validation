<template>
  <div class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="onLogin">
      <div class="form-group">
        <label for="login">Email</label>
        <input
          v-model="formData.login"
          :class="{ 'is-invalid': formColor(errors.login) }"
          @blur="onBlur"
          type="text"
          id="login"
          placeholder="Enter your email"
        />
        <div class="error-wrapper">
          <div style="color: red; font-size: 12px">
            {{ errors.login }}
          </div>
        </div>
      </div>

      <div class="form-group">
        <label for="password">Password</label>
        <input
          v-model="formData.password"
          :class="{ 'is-invalid': formColor(errors.password) }"
          @blur="onBlur"
          type="password"
          id="password"
          placeholder="Enter your password"
        />

        <div class="error-wrapper">
          <div style="color: red; font-size: 12px">
            {{ errors.password }}
          </div>
        </div>
      </div>

      <button type="submit" :disabled="isFormInvalid">Login</button>
    </form>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from "vue";
import login from "@/schemas/login.json";
import Ajv from "ajv";
import ajvErrors from "ajv-errors";

export default defineComponent({
  setup() {
    const formData = ref({
      login: "",
      password: "",
    });

    const ajv = new Ajv({ allErrors: true });
    ajvErrors(ajv);

    let validator = ajv;
    validator.addSchema(login);

    interface FormErrors {
      [fieldName: string]: string[];
    }

    let errors = ref<FormErrors>({});
    const isFormInvalid = ref(false);

    watch(
      () => errors.value,
      (newErrors: Record<string, string[]>) => {
        isFormInvalid.value = Object.keys(newErrors).some(
          (field) => newErrors[field].length > 0
        );
      },
      { deep: true }
    );

    function onBlur(e: any) {
      const isValid = validator.validate(
        `/login.json#/properties/` + e.target.id,
        e.target.value
      );
      const validationErrors =
        validator.errors !== null ? validator.errors : {};
      if (!isValid) {
        if (validationErrors !== null && validationErrors !== undefined) {
          // eslint-disable-next-line @typescript-eslint/ban-ts-comment
          // @ts-ignore
          validationErrors.forEach((error: any) => {
            errors.value[e.target.id] = error.message;
          });
        } else {
          errors.value = {};
        }
      } else {
        errors.value = {};
      }
    }

    function onLogin() {
      const isValid = validator.validate("/login.json", formData.value);
      const validationErrors =
        validator.errors !== null ? validator.errors : {};

      if (!isValid) {
        if (validationErrors !== null && validationErrors !== undefined) {
          // eslint-disable-next-line @typescript-eslint/ban-ts-comment
          // @ts-ignore
          errors.value = validationErrors;
          // eslint-disable-next-line @typescript-eslint/ban-ts-comment
          // @ts-ignore
          for (const error of validationErrors) {
            const fieldName = error.instancePath.substring(1);

            errors.value[fieldName] = error.message;
          }
        } else {
          errors.value = {};
        }
      } else {
        errors.value = {};

        console.log("Login:", formData.value.login);
        console.log("Password:", formData.value.password);
      }
    }

    function formColor(error: any) {
      if (error) {
        if (error.length) {
          return true;
        } else {
          return false;
        }
      }
    }

    return {
      formData,
      errors,
      isFormInvalid,
      formColor,
      onBlur,
      onLogin,
    };
  },
});
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Open+Sans:wght@300;400;600;700&display=swap");

body {
  font-family: "Open Sans";
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
</style>
