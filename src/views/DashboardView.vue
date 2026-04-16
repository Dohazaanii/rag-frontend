<template>
  <div class="chat-app">

    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-header">
        <h2>My chats</h2>
        <button @click="createConversation" class="btn-new">+ New</button>
      </div>

      <div class="conversations-list">
        <div
          v-for="conv in conversations"
          :key="conv.id"
          class="conv-item"
          :class="{ active: currentConversation?.id === conv.id }"
          @click="selectConversation(conv)"
        >
          <span class="conv-title">{{ conv.title }}</span>
          <button class="btn-delete" @click.stop="deleteConversation(conv.id)">x</button>
        </div>

        <div v-if="conversations.length === 0" class="empty-conv">
          No conversations
        </div>
      </div>

      <div class="sidebar-footer">
        <button @click="logout" class="btn-logout">Logout</button>
      </div>
    </aside>

    <!-- Main Chat -->
    <main class="chat-main">

      <!-- No conversation selected -->
      <div v-if="!currentConversation" class="chat-welcome">
        <h1>How can I help you?</h1>
        <button @click="createConversation" class="btn-start">New Conversation</button>
      </div>

      <!-- Chat messages -->
      <div v-else class="chat-content">
        <div class="chat-header">
          <h3>{{ currentConversation.title }}</h3>
        </div>

        <div class="messages-container" ref="messagesContainer">
          <div
            v-for="msg in messages"
            :key="msg.id"
            class="message"
            :class="msg.role"
          >
            <div class="message-avatar">
              {{ msg.role === 'user' ? '' : '' }}
            </div>
            <div class="message-bubble">
              <div v-if="msg.role === 'assistant'" v-html="renderMarkdown(msg.content)"></div>
              <div v-else>{{ msg.content }}</div>
            </div>
          </div>

          <!-- Loading -->
          <div v-if="loading" class="message assistant">
            <div class="message-avatar">🤖</div>
            <div class="message-bubble typing">
              <span></span><span></span><span></span>
            </div>
          </div>
        </div>

        <!-- Input -->
        <div class="chat-input-area">
          <textarea
            v-model="newMessage"
            placeholder="Écrivez votre message..."
            @keydown.enter.exact.prevent="sendMessage"
            rows="1"
          ></textarea>
          <button @click="sendMessage" :disabled="loading || !newMessage.trim()">
            ➤
          </button>
        </div>
      </div>

    </main>
  </div>
</template>

<script>
import axios from 'axios'
import { marked } from 'marked'

const api = axios.create({
  baseURL: 'http://127.0.0.1:8000/api',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'Authorization': `Bearer ${localStorage.getItem('token')}` 
  }
})

export default {
  data() {
    return {
      conversations: [],
      currentConversation: null,
      messages: [],
      newMessage: '',
      loading: false,
    }
  },

  async mounted() {

    await this.loadConversations()
  },

  methods: {
    renderMarkdown(content) {
      return marked(content)
    },

    async loadConversations() {
      const res = await api.get('/conversations')
      this.conversations = res.data
    },

    async createConversation() {
      const res = await api.post('/conversations')
      this.conversations.unshift(res.data)
      await this.selectConversation(res.data)
    },

    async selectConversation(conv) {
      this.currentConversation = conv
      this.messages = []
      const res = await api.get(`/conversations/${conv.id}/messages`)
      this.messages = res.data
      this.scrollToBottom()
    },

    async sendMessage() {
      if (!this.newMessage.trim() || this.loading) return

      const content = this.newMessage
      this.newMessage = ''
      this.loading = true

      // Ajouter le message user localement
      this.messages.push({
        id: Date.now(),
        role: 'user',
        content: content,
      })
      this.scrollToBottom()

      try {
        const res = await api.post(`/conversations/${this.currentConversation.id}/messages`, {
          content: content,
        })

        // Ajouter la réponse de l'IA
        this.messages.push(res.data)

        // Mettre à jour le titre dans la sidebar
        await this.loadConversations()
        this.scrollToBottom()

      } catch (err) {
        this.messages.push({
          id: Date.now(),
          role: 'assistant',
          content: '❌ Erreur : impossible de contacter Ollama.',
        })
      } finally {
        this.loading = false
      }
    },

    async deleteConversation(id) {
      await api.delete(`/conversations/${id}`)
      this.conversations = this.conversations.filter(c => c.id !== id)
      if (this.currentConversation?.id === id) {
        this.currentConversation = null
        this.messages = []
      }
    },

    logout() {
      localStorage.removeItem('token')
      this.$router.push('/login')
    },

    scrollToBottom() {
      this.$nextTick(() => {
        const container = this.$refs.messagesContainer
        if (container) container.scrollTop = container.scrollHeight
      })
    }
  }
}
</script>

<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.chat-app {
  display: flex;
  height: 100vh;
  font-family: 'Segoe UI', sans-serif;
  background: #f7f8fc;
}

/* Sidebar */
.sidebar {
  width: 280px;
  background: #1e1e2e;
  color: white;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
}

.sidebar-header {
  padding: 1.25rem;
  border-bottom: 1px solid rgba(255,255,255,0.08);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.sidebar-header h2 {
  font-size: 1rem;
  font-weight: 600;
}

.btn-new {
  background: #4f46e5;
  color: white;
  border: none;
  padding: 0.4rem 0.75rem;
  border-radius: 6px;
  font-size: 0.8rem;
  cursor: pointer;
}

.btn-new:hover { background: #4338ca; }

.conversations-list {
  flex: 1;
  overflow-y: auto;
  padding: 0.5rem;
}

.conv-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.15s;
  margin-bottom: 2px;
}

.conv-item:hover { background: rgba(255,255,255,0.07); }
.conv-item.active { background: rgba(79,70,229,0.3); }

.conv-title {
  font-size: 0.875rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
}

.btn-delete {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  opacity: 0;
  transition: opacity 0.15s;
}

.conv-item:hover .btn-delete { opacity: 1; }

.empty-conv {
  text-align: center;
  color: rgba(255,255,255,0.3);
  font-size: 0.85rem;
  padding: 2rem;
}

.sidebar-footer {
  padding: 1rem;
  border-top: 1px solid rgba(255,255,255,0.08);
}

.btn-logout {
  width: 100%;
  background: none;
  border: 1px solid rgba(255,255,255,0.15);
  color: rgba(255,255,255,0.6);
  padding: 0.6rem;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.15s;
}

.btn-logout:hover {
  background: rgba(255,255,255,0.05);
  color: white;
}

/* Main */
.chat-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.chat-welcome {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  color: #4a5568;
}

.welcome-icon { font-size: 4rem; }

.chat-welcome h1 {
  font-size: 1.75rem;
  color: #1a202c;
}

.btn-start {
  margin-top: 1rem;
  padding: 0.75rem 2rem;
  background: #4f46e5;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
}

.btn-start:hover { background: #4338ca; }

.chat-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.chat-header {
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #e2e8f0;
  background: white;
}

.chat-header h3 {
  font-size: 1rem;
  color: #2d3748;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.message {
  display: flex;
  gap: 0.75rem;
  align-items: flex-start;
}

.message.user { flex-direction: row-reverse; }

.message-avatar {
  font-size: 1.5rem;
  flex-shrink: 0;
}

.message-bubble {
  max-width: 70%;
  padding: 0.875rem 1.125rem;
  border-radius: 12px;
  font-size: 0.95rem;
  line-height: 1.6;
}

.message.user .message-bubble {
  background: #4f46e5;
  color: white;
  border-radius: 12px 12px 0 12px;
}

.message.assistant .message-bubble {
  background: white;
  color: #2d3748;
  border: 1px solid #e2e8f0;
  border-radius: 12px 12px 12px 0;
}

/* Typing animation */
.typing {
  display: flex;
  gap: 4px;
  align-items: center;
  padding: 1rem 1.25rem;
}

.typing span {
  width: 8px;
  height: 8px;
  background: #a0aec0;
  border-radius: 50%;
  animation: bounce 1.2s infinite;
}

.typing span:nth-child(2) { animation-delay: 0.2s; }
.typing span:nth-child(3) { animation-delay: 0.4s; }

@keyframes bounce {
  0%, 60%, 100% { transform: translateY(0); }
  30% { transform: translateY(-6px); }
}

/* Input */
.chat-input-area {
  padding: 1rem 1.5rem;
  background: white;
  border-top: 1px solid #e2e8f0;
  display: flex;
  gap: 0.75rem;
  align-items: flex-end;
}

textarea {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 1.5px solid #e2e8f0;
  border-radius: 10px;
  font-size: 0.95rem;
  font-family: inherit;
  resize: none;
  outline: none;
  max-height: 150px;
  overflow-y: auto;
  transition: border-color 0.2s;
}

textarea:focus { border-color: #4f46e5; }

.chat-input-area button {
  width: 44px;
  height: 44px;
  background: #4f46e5;
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background 0.2s;
  flex-shrink: 0;
}

.chat-input-area button:hover:not(:disabled) { background: #4338ca; }
.chat-input-area button:disabled { opacity: 0.5; cursor: not-allowed; }
</style>