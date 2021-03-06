<template>
  <form @submit.prevent="addTask">
    <div v-if="errors" class="bg-red-lightest border border-red-light text-red-dark px-4 py-3 rounded relative mb-3" role="alert">
      <li v-for="error in errors" :key="error[0]">{{ error[0] }}</li>
    </div>

    <div class="p-3 mb-4 appearance-none bg-white border-none rounded-full flex shadow-md items-center">
      <div class="border-r w-1/3 pr-1 flex items-center">
        <datetime type="datetime" v-model="form.due_at" placeholder="Due at" :minute-step="5" class="w-full" input-class="w-full"></datetime>
        <span v-if="form.due_at" @click="clearDueAt" class="flex-none rounded-full bg-grey hover:bg-red h-6 w-6 cursor-pointer flex items-center justify-center shadow">
          <i class="fa fa-times text-white"></i>
        </span>
      </div>

      <input v-focus v-model="form.title" class="w-full no-outline px-3" placeholder="What needs to be done?" ref="task" />
    </div>

    <div class="flex justify-end">
      <button type="submit" :class="[isDisabled ? 'opacity-50 cursor-not-allowed' : 'hover:text-indigo hover:bg-white']" class="flex text-white border-2 border-indigo rounded-full bg-indigo uppercase px-3 py-2 text-xs font-bold no-outline align-middle">
        <i v-if="isLoading" class="fa fa-spinner fa-spin mr-1" aria-hidden="true"></i>
        <i v-else class="fa fa-plus mr-1" aria-hidden="true"></i> Add
      </button>
    </div>
  </form>
</template>

<script>
import Form from '@/utils/Form'
import moment from 'moment'

export default {
  data () {
    return {
      endpoint: '/tasks/',
      isLoading: false,
      errors: null,
      form: new Form({
        title: '',
        due_at: null
      })
    }
  },

  computed: {
    isDisabled () {
      return this.form.title === '' || this.isLoading
    }
  },

  methods: {
    addTask () {
      if (this.isDisabled) {
        return
      }
      this.isLoading = true
      this.errors = null

      if (this.form.due_at) {
        this.form.due_at = moment(this.form.due_at).format('YYYY-MM-DD HH:mm:ss')
      }

      this.form.post('tasks')
        .then(({ data }) => {
          this.$emit('created', data)

          this.form.reset()
          this.$refs.task.focus()
          this.isLoading = false
        })
        .catch(({ errors }) => {
          this.isLoading = false
          this.errors = errors
        })
    },

    clearDueAt () {
      this.form.due_at = null
    }
  }
}
</script>
