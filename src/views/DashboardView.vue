<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const logout = () => {
  localStorage.removeItem('token')
  router.push('/login')
}
</script>

<template>
  <div style="padding: 2rem; text-align: center;">
    <h1>🎉 Bienvenue sur le Dashboard !</h1>
    <p>Tu es bien connecté.</p>
    <button @click="logout">Se déconnecter</button>
  </div>
</template>