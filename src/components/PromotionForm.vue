<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

interface FormData {
  link: string
  title: string
  previousPrice: number
  currentPrice: number
  platform: string
}

interface ValidationError {
  link?: string
  title?: string
  previousPrice?: string
  currentPrice?: string
  platform?: string
}

const formData = ref<FormData>({
  link: '',
  title: '',
  previousPrice: 0,
  currentPrice: 0,
  platform: ''
})

const errors = ref<ValidationError>({})
const apiError = ref<string>('')
const successMessage = ref<string>('')
const isSubmitting = ref<boolean>(false)

const validateForm = (): boolean => {
  errors.value = {}
  let isValid = true

  if (!formData.value.link.trim()) {
    errors.value.link = 'Link is required'
    isValid = false
  }

  if (!formData.value.title.trim()) {
    errors.value.title = 'Title is required'
    isValid = false
  }

  if (formData.value.previousPrice <= 0) {
    errors.value.previousPrice = 'Previous price must be a positive number'
    isValid = false
  }

  if (formData.value.currentPrice <= 0) {
    errors.value.currentPrice = 'Current price must be a positive number'
    isValid = false
  }

  if (!formData.value.platform.trim()) {
    errors.value.platform = 'Platform is required'
    isValid = false
  }

  return isValid
}

const handleSubmit = async () => {
  apiError.value = ''
  successMessage.value = ''

  if (!validateForm()) {
    return
  }

  isSubmitting.value = true

  try {
    const response = await axios.post('http://localhost:3000/api/generate-promotion', {
      link: formData.value.link,
      title: formData.value.title,
      previousPrice: formData.value.previousPrice,
      currentPrice: formData.value.currentPrice,
      platform: formData.value.platform
    })

    successMessage.value = 'Promotion created successfully!'
    
    // Reset form
    formData.value = {
      link: '',
      title: '',
      previousPrice: 0,
      currentPrice: 0,
      platform: ''
    }
  } catch (error: any) {
    if (error.response) {
      apiError.value = `Error: ${error.response.data.message || error.response.statusText}`
    } else if (error.request) {
      apiError.value = 'Error: No response from server. Please check if the backend is running.'
    } else {
      apiError.value = `Error: ${error.message}`
    }
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <div class="promotion-form">
    <h1>Create Promotion</h1>
    
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label for="link">Link *</label>
        <input
          id="link"
          v-model="formData.link"
          type="url"
          placeholder="https://example.com/product"
          :class="{ error: errors.link }"
        />
        <span v-if="errors.link" class="error-message">{{ errors.link }}</span>
      </div>

      <div class="form-group">
        <label for="title">Title *</label>
        <input
          id="title"
          v-model="formData.title"
          type="text"
          placeholder="Product title"
          :class="{ error: errors.title }"
        />
        <span v-if="errors.title" class="error-message">{{ errors.title }}</span>
      </div>

      <div class="form-group">
        <label for="previousPrice">Previous Price *</label>
        <input
          id="previousPrice"
          v-model.number="formData.previousPrice"
          type="number"
          step="0.01"
          min="0"
          placeholder="0.00"
          :class="{ error: errors.previousPrice }"
        />
        <span v-if="errors.previousPrice" class="error-message">{{ errors.previousPrice }}</span>
      </div>

      <div class="form-group">
        <label for="currentPrice">Current Price *</label>
        <input
          id="currentPrice"
          v-model.number="formData.currentPrice"
          type="number"
          step="0.01"
          min="0"
          placeholder="0.00"
          :class="{ error: errors.currentPrice }"
        />
        <span v-if="errors.currentPrice" class="error-message">{{ errors.currentPrice }}</span>
      </div>

      <div class="form-group">
        <label for="platform">Platform *</label>
        <input
          id="platform"
          v-model="formData.platform"
          type="text"
          placeholder="Amazon, eBay, etc."
          :class="{ error: errors.platform }"
        />
        <span v-if="errors.platform" class="error-message">{{ errors.platform }}</span>
      </div>

      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? 'Submitting...' : 'Create Promotion' }}
      </button>

      <div v-if="apiError" class="api-error">
        {{ apiError }}
      </div>

      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>
    </form>
  </div>
</template>

<style scoped>
.promotion-form {
  max-width: 500px;
  margin: 50px auto;
  padding: 30px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
}

input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
}

input:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

input.error {
  border-color: #f44336;
}

.error-message {
  display: block;
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
}

button {
  width: 100%;
  padding: 14px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover:not(:disabled) {
  background-color: #45a049;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.api-error {
  margin-top: 20px;
  padding: 12px;
  background-color: #ffebee;
  color: #c62828;
  border-radius: 4px;
  text-align: center;
}

.success-message {
  margin-top: 20px;
  padding: 12px;
  background-color: #e8f5e9;
  color: #2e7d32;
  border-radius: 4px;
  text-align: center;
}
</style>
