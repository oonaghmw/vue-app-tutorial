<template>
  <div id="employee-form">
    <form @submit.prevent="handleSubmit">
      <label>Employee name</label>
      <input
        ref="first"
        type="text"
        :class="{ 'has-error': submitting && invalidName }"
        v-model="employee.name"
        @focus="clearStatus"
        @keypress="clearStatus"
      />
      <label>Employee Email</label>
      <input
        type="text"
        :class="{ 'has-error': submitting && invalidEmail }"
        v-model="employee.email"
        @focus="clearStatus"
      />
      <p v-if="error && submitting" class="error-message">
        !Please fill out all required fields
      </p>
      <p v-if="success" class="success-message">Employee successfully added</p>
      <button>Add Employee</button>
    </form>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "employee-form",

  data() {
    return {
      submitting: false,
      error: false,
      success: false,
      employee: {
        name: "",
        email: "",
      },
      invalidEmail: false,
    };
  },
  computed: {
    invalidName() {
      return this.employee.name === "";
    },

    /*invalidEmail() {

				return this.employee.email === ''
			},*/
  },
  methods: {
    async validateEmail(email) {
      /* Using API from 
      https://rapidapi.com/Top-Rated/api/e-mail-check-invalid-or-disposable-domain/*/
      const response = await axios(
        `https://mailcheck.p.rapidapi.com/?domain=${email}`,
        {
          method: "GET",
          headers: {
            "x-rapidapi-key":
              "PRIVATEKEY",
            "x-rapidapi-host": "mailcheck.p.rapidapi.com",
          },
        }
      );
      console.log(response.data.valid);
      return response.data.valid;
    },

    async handleSubmit() {
      this.clearStatus();
      this.submitting = true;
      const isValidEmail = await this.validateEmail(this.employee.email);
      console.log({ isValidEmail });
      this.invalidEmail = isValidEmail;
      
      if (this.invalidName || this.invalidEmail) {
        this.error = true;
        return;
      }

      this.$emit("add:employee", this.employee);
      this.$refs.first.focus();
      this.employee = {
        name: "",
        email: "",
      };

      this.error = false;
      this.success = true;
      this.submitting = false;
    },

    clearStatus() {
      this.success = false;
      this.error = false;
    },
  },
};
</script>

<style scoped>
form {
  margin-bottom: 2rem;
}

[class*="-message"] {
  font-weight: 500;
}

.error-message {
  color: #d33c40;
}

.success-message {
  color: #32a95d;
}
</style>

