<template>
  <div class="login-page">
    <div class="login-card">

      <div class="login-header">
        <div class="logo"></div>
        <h1>Log In</h1>
      </div>

      <div v-if="error" class="alert-error">
        {{ error }}
      </div>

      <div class="form-group">
        <label>Email</label>
        <div class="input-wrapper">
          <span class="input-icon"></span>
          <input
            v-model="email"
            type="email"
            placeholder=""
          />
        </div>
      </div>

      <div class="form-group">
        <label>Password</label>
        <div class="input-wrapper">
          <span class="input-icon"></span>
          <input
            v-model="password"
            type="password"
            placeholder=""
          />
        </div>
      </div>

      <button @click="login" :disabled="loading" class="btn-login">
        <span v-if="loading" class="spinner"></span>
        {{ loading ? 'Connexion...' : 'Login' }}
      </button>

    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      email: '',
      password: '',
      error: '',
      loading: false,
    }
  },
  methods: {
    async login() {
      this.error = ''
      this.loading = true

      try {
        const response = await axios.post('http://127.0.0.1:8000/api/login', {
          email: this.email,
          password: this.password,
        })

        localStorage.setItem('token', response.data.access_token)
        this.$router.push('/dashboard')

      } catch (err) {
        this.error = err.response?.data?.message || 'Email ou mot de passe incorrect'
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.login-page {
  min-height: 100vh;
  background: #f0f4f8;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Segoe UI', sans-serif;
}

.login-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 3rem 2.5rem;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
  border: 1px solid #e8edf2;
}

.login-header {
  text-align: center;
  margin-bottom: 2rem;
}

.logo {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.login-header h1 {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1a202c;
  margin-bottom: 0.5rem;
}

.login-header p {
  color: #718096;
  font-size: 0.95rem;
}

.form-group {
  margin-bottom: 1.25rem;
}

label {
  display: block;
  font-size: 0.875rem;
  font-weight: 600;
  color: #2d3748;
  margin-bottom: 0.5rem;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 0.875rem;
  font-size: 1rem;
  pointer-events: none;
}

input {
  width: 100%;
  padding: 0.75rem 0.875rem 0.75rem 2.75rem;
  border: 1.5px solid #e2e8f0;
  border-radius: 8px;
  font-size: 0.95rem;
  color: #2d3748;
  background: #f7fafc;
  transition: border-color 0.2s, box-shadow 0.2s;
  outline: none;
}

input:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
  background: #fff;
}

input::placeholder {
  color: #a0aec0;
}

.btn-login {
  width: 100%;
  padding: 0.875rem;
  background: #4f46e5;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 0.75rem;
  transition: background 0.2s, transform 0.1s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.btn-login:hover:not(:disabled) {
  background: #4338ca;
  transform: translateY(-1px);
}

.btn-login:disabled {
  opacity: 0.65;
  cursor: not-allowed;
  transform: none;
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255,255,255,0.4);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.alert-error {
  background: #fff5f5;
  border: 1px solid #fed7d7;
  color: #c53030;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  font-size: 0.875rem;
  margin-bottom: 1.25rem;
}
</style>