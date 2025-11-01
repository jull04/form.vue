<template>
  <div class="app">
    <h1 class="app__title">Автоформа</h1>
    
    <FormGenerator 
      :schema="formSchema" 
      v-model="formData"
      :clear-after-submit="true" 
      @submit="handleSubmit"
    />

    <div class="form-data">
      <h3 class="title">Данные формы:</h3>
      <pre>{{ JSON.stringify(formData, null, 2) }}</pre>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import FormGenerator from './components/FormGenerator.vue'

export default {
  name: 'App',
  components: {
    FormGenerator
  },
  setup() {
    const formData = ref({})
    
    const formSchema = ref({
      fields: [
        { 
          type: "text", 
          label: "Имя", 
          model: "name", 
          required: true,
          placeholder: "Введите ваше имя"
        },
        { 
          type: "email", 
          label: "Email", 
          model: "email", 
          required: true,
          placeholder: "Введите ваш email"
        },
        { 
          type: "password", 
          label: "Пароль", 
          model: "password", 
          required: true, 
          minLength: 6,
          placeholder: "Введите пароль"
        },
        { 
          type: "select", 
          label: "Роль", 
          model: "role", 
          options: ["Админ", "Пользователь", "Модератор"], 
          required: true 
        },
        { 
          type: "checkbox", 
          label: "Согласен с условиями", 
          model: "terms", 
          required: true 
        }
      ]
    })

    const handleSubmit = (data) => {
      console.log('Форма отправлена:', data)
      alert('Форма успешно отправлена!')
    }

    return {
      formSchema,
      formData,
      handleSubmit
    }
  }
}
</script>

<style>
.app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.app__title {
  margin: 0 auto 20px;
  color: white;
}

.title {
  color: black;
  margin-bottom: 10px;
  font-weight: 600;
}

.form-data {
  margin-top: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 4px;
}

pre {
  background-color: #2c3e50;
  color: #ecf0f1;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
}
</style>
