<template>
  <div>
    <h1 class="text-3xl mb-6 font-bold text-center">chat con la api yesno</h1>

    <div class="mx-auto relative max-w-md [&>*:nth-child(even)]:ml-auto">
      <!-- Mostrar mensajes del chat -->
      <div
        v-for="(message, index) in messages"
        :key="index"
        class="w-max max-w-sm m-4 px-4 py-2"
        :class="{
          'bg-gray-200 rounded-full rounded-bl-none': !message.sentByUser,
          'bg-pink-500 text-white rounded-2xl rounded-br-none': message.sentByUser
        }"
      >
        <p>{{ message.text }}</p>
        <img
          v-if="message.image"
          :src="message.image"
          alt="response image"
          class="mt-2 rounded-md"
        />
      </div>
    </div>

    <div class="bg-gray-300 p-4">
      <!-- Input para enviar mensaje -->
      <input
        v-model="newMessage"
        @keydown.enter="sendMessage"
        class="flex items-center h-10 w-full rounded px-3 text-sm"
        type="text"
        placeholder="Escribe tu mensaje…"
      />
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const newMessage = ref('') // Variable para el mensaje nuevo
    const messages = ref([
      {
        text: 'Hola, puedo ayudarte respondiendo solo si y no',
        time: new Date(),
        sentByUser: false
      }
    ]) // Mensaje inicial del chat

    // Obtener la hora actual
    const getCurrentTime = () => {
      return new Date()
    }

    // Calcular el tiempo transcurrido desde el envío del mensaje
    const getRelativeTime = (time) => {
      const now = new Date()
      const diffMs = now - new Date(time)
      const diffMins = Math.floor(diffMs / (1000 * 60))

      if (diffMins < 1) {
        return 'Hace un momento'
      } else if (diffMins === 1) {
        return 'Hace 1 minuto'
      } else if (diffMins < 60) {
        return `Hace ${diffMins} minutos`
      } else {
        const hours = Math.floor(diffMins / 60)
        return hours === 1 ? 'Hace 1 hora' : `Hace ${hours} horas`
      }
    }

    // Función para enviar mensajes
    const sendMessage = async () => {
      if (newMessage.value.trim() !== '') {
        // Agregar el mensaje del usuario al chat
        messages.value.push({
          text: newMessage.value,
          time: getCurrentTime(),
          sentByUser: true
        })

        // Guardar el mensaje temporalmente y limpiar el campo de texto
        const userMessage = newMessage.value
        newMessage.value = ''

        // Hacer la solicitud a la API de yesno.wtf
        try {
          const response = await fetch('https://yesno.wtf/api')
          const data = await response.json()

          // Agregar la respuesta del bot al chat
          messages.value.push({
            text: data.answer,
            image: data.image,
            time: getCurrentTime(),
            sentByUser: false
          })
        } catch (error) {
          // Manejo de errores si la petición falla
          console.error('Error fetching from API:', error)
          messages.value.push({
            text: 'Lo siento, hubo un error al procesar tu solicitud.',
            time: getCurrentTime(),
            sentByUser: false
          })
        }
      }
    }

    return {
      newMessage,
      messages,
      sendMessage,
      getRelativeTime
    }
  }
}
</script>
