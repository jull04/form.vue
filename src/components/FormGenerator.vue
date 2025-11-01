<template>
  <form class="auto-form" @submit.prevent="handleSubmit">
    <div
      v-for="field in schema.fields"
      :key="field.model"
      class="form-field"
    >
      <!-- Текстовые поля -->
      <div v-if="['text', 'email', 'password'].includes(field.type)" class="input-group">
        <label :for="field.model" class="input__label">
          {{ field.label }}
          <span v-if="field.required" class="required">*</span>
        </label>
        <input
          :id="field.model"
          :type="field.type"
          :value="formValue[field.model]"
          @input="updateField(field.model, $event.target.value)"
          @blur="validateField(field)"
          :class="{ error: fieldErrors[field.model] }"
          :placeholder="field.placeholder || ''"
        />
        <div v-if="fieldErrors[field.model]" class="error-message">
          {{ fieldErrors[field.model] }}
        </div>
      </div>

      <!-- Выпадающий список -->
      <div v-else-if="field.type === 'select'" class="input-group">
        <label :for="field.model">
          {{ field.label }}
          <span v-if="field.required" class="required">*</span>
        </label>
        <select
          :id="field.model"
          :value="formValue[field.model]"
          @change="updateField(field.model, $event.target.value)"
          @blur="validateField(field)"
          :class="{ error: fieldErrors[field.model] }"
        >
          <option value="" disabled>Выберите опцию</option>
          <option
            v-for="option in field.options"
            :key="option"
            :value="option"
          >
            {{ option }}
          </option>
        </select>
        <div v-if="fieldErrors[field.model]" class="error-message">
          {{ fieldErrors[field.model] }}
        </div>
      </div>

      <!-- Чекбокс -->
      <div v-else-if="field.type === 'checkbox'" class="input-group checkbox-group">
        <label :for="field.model" class="checkbox-label">
          <input
            :id="field.model"
            type="checkbox"
            :checked="formValue[field.model]"
            @change="updateField(field.model, $event.target.checked)"
            @blur="validateField(field)"
            :class="{ error: fieldErrors[field.model] }"
          />
          {{ field.label }}
          <span v-if="field.required" class="required">*</span>
        </label>
        <div v-if="fieldErrors[field.model]" class="error-message">
          {{ fieldErrors[field.model] }}
        </div>
      </div>
    </div>

    <button type="submit" class="submit-button" :disabled="!isFormValid">
      Отправить
    </button>
  </form>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'FormGenerator',
  props: {
    schema: {
      type: Object,
      required: true,
      validator: (value) => {
        return value.fields && Array.isArray(value.fields)
      }
    },
    modelValue: {
      type: Object,
      default: () => ({})
    },
    clearAfterSubmit: {
      type: Boolean,
      default: true
    }
  },
  emits: ['update:modelValue', 'submit'],
  setup(props, { emit }) {
    const formValue = ref({ ...props.modelValue })
    const fieldErrors = ref({})
    const touchedFields = ref(new Set())

    // Инициализация значений по умолчанию
    const initializeForm = () => {
      props.schema.fields.forEach(field => {
        if (formValue.value[field.model] === undefined) {
          if (field.type === 'checkbox') {
            formValue.value[field.model] = false
          } else if (field.type === 'select') {
            formValue.value[field.model] = ''
          } else {
            formValue.value[field.model] = ''
          }
        }
      })
    }

    const clearForm = () => {
      props.schema.fields.forEach(field => {
        if (field.type === 'checkbox') {
          formValue.value[field.model] = false
        } else if (field.type === 'select') {
          formValue.value[field.model] = ''
        } else {
          formValue.value[field.model] = ''
        }
      })
      
      // Очищаем ошибки и затронутые поля
      fieldErrors.value = {}
      touchedFields.value = new Set()
      
      // Эмитим обновленные значения
      emit('update:modelValue', { ...formValue.value })
    }


    initializeForm()

    // Валидация поля
    const validateField = (field) => {
      touchedFields.value.add(field.model)
      
      const value = formValue.value[field.model]
      const errors = []

      // Проверка на обязательность
      if (field.required) {
        if (field.type === 'checkbox' && !value) {
          errors.push('Это поле обязательно для заполнения')
        } else if (field.type !== 'checkbox' && (!value || value.toString().trim() === '')) {
          errors.push('Это поле обязательно для заполнения')
        }
      }

      // Проверка минимальной длины
      if (field.minLength && value && value.length < field.minLength) {
        errors.push(`Минимальная длина: ${field.minLength} символов`)
      }

      // Проверка по регулярному выражению
      if (field.pattern && value) {
        const regex = new RegExp(field.pattern)
        if (!regex.test(value)) {
          errors.push(field.patternMessage || 'Неверный формат')
        }
      }

      // Email валидация
      if (field.type === 'email' && value) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailRegex.test(value)) {
          errors.push('Введите корректный email адрес')
        }
      }

      if (errors.length > 0) {
        fieldErrors.value[field.model] = errors.join(', ')
      } else {
        delete fieldErrors.value[field.model]
      }
    }

    // Валидация всей формы
    const validateForm = () => {
      props.schema.fields.forEach(field => {
        if (field.required) {
          validateField(field)
        }
      })
    }

    // Обновление значения поля
    const updateField = (fieldName, value) => {
      formValue.value[fieldName] = value
      emit('update:modelValue', { ...formValue.value })
      
      // Валидация при изменении, если поле уже было затронуто
      if (touchedFields.value.has(fieldName)) {
        const field = props.schema.fields.find(f => f.model === fieldName)
        if (field) validateField(field)
      }
    }

    // Обработка отправки формы
    const handleSubmit = () => {
      // Помечаем все поля как затронутые перед валидацией
      props.schema.fields.forEach(field => {
        touchedFields.value.add(field.model)
      })
      
      validateForm()
      
       if (isFormValid.value) {
        const submittedData = { ...formValue.value }
        emit('submit', submittedData)
        if (props.clearAfterSubmit) {
          clearForm()
        }
      }
    }

    // Вычисляемое свойство для проверки валидности формы
    const isFormValid = computed(() => {
      return Object.keys(fieldErrors.value).length === 0
    })

    // Следим за изменениями в схеме
    watch(() => props.schema, () => {
      initializeForm()
      fieldErrors.value = {}
      touchedFields.value = new Set()
    })

    // Следим за изменениями modelValue извне
    watch(() => props.modelValue, (newValue) => {
      formValue.value = { ...newValue }
    }, { deep: true })

    return {
      formValue,
      fieldErrors,
      updateField,
      validateField,
      handleSubmit,
      isFormValid,
      clearForm 
    }
  }
}
</script>

<style scoped>
.auto-form {
  width: 350px;
  margin: 0 auto;
}

.form-field {
  margin-bottom: 20px;
}

.input-group {
  display: flex;
  flex-direction: column;
}

label {
  margin-bottom: 5px;
  font-weight: 500;
  color: #afa5a5;
}

.required {
  color: #e74c3c;
}

input, select {
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s ease;
}

input:focus, select:focus {
  outline: none;
  border-color: #3498db;
}

input.error, select.error {
  border-color: #e74c3c;
}

.checkbox-group {
  flex-direction: row;
  align-items: center;
}

.checkbox-label {
  display: flex;
  align-items: center;
  margin-bottom: 0;
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  margin-right: 8px;
  width: auto;
}

.error-message {
  color: #e74c3c;
  font-size: 12px;
  margin-top: 5px;
}

.submit-button {
  width: 100%;
  padding: 12px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover:not(:disabled) {
  background-color: #2980b9;
}

.submit-button:disabled {
  background-color: #bdc3c7;
  cursor: not-allowed;
}
</style>