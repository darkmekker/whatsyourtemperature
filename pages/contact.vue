<template>
  <main>
    <header class="w-full max-w-3xl mx-auto relative mb-4 px-24 px-md-0">
      <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" class="absolute -left-0" />
      <h1 class="max-w-2xl">{{ content.title }}</h1>
      <nuxt-content :document="content" />
    </header>
    <section v-show="content" class="w-full max-w-3xl mx-auto px-24 px-md-0">
      <form @submit.prevent="submitForm" ref="form">
        <div class="flex w-full mb-16">
          <div class="w-1/2 px-10">
            <h2 class="text-lg text-right mb-5">MY PROCESS</h2>
          </div>
          <div class="w-1/2 px-10">
            <h2 class="text-lg mb-5">MY TEMPERATURE</h2>
          </div>
        </div>
        <div class="w-full max-w-2xl">
          <p v-show="showFormError" class="text-red-500 text-center mb-5">Please fill out all fields</p>

          <label class="sr-only" for="company-name">Company Name</label>
          <input
            class="bg-grey-500"
            type="text"
            id="company-name"
            name="company-name"
            v-model="formData.companyName"
            placeholder="Company Name*"
            required
          />

          <label class="sr-only" for="name">Contact Name</label>
          <input
            type="text"
            id="name"
            name="name"
            v-model="formData.contactName"
            placeholder="Contact Name*"
            required
          />

          <label class="sr-only" for="email">Email</label>
          <input type="email" id="email" name="email" v-model="formData.email" placeholder="Email Address*" required />

          <label class="sr-only" for="message">Message</label>
          <textarea
            id="message"
            name="message"
            v-model="formData.message"
            placeholder="Annotations … "
            style="height: 200px"
            required
          ></textarea>

          <input type="hidden" name="selected-process" v-model="formData.selectedProcess" />
          <input type="hidden" name="selected-temp-min" v-model="formData.selectedSliderValMin" />
          <input type="hidden" name="selected-temp-max" v-model="formData.selectedSliderValMax" />

          <div class="flex items-center">
            <input class="btn w-auto m-auto" type="submit" value="Submit" />
          </div>
        </div>
      </form>
    </section>
  </main>
</template>

<script>
import VerticalRangeSlider from '@/components/contact/VerticalRangeSlider'
import ProcessList from '@/components/contact/ProcessList.vue'

export default {
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('contact/_index').fetch()
    } catch (e) {
      error({ message: 'Contact data not found' })
    }
    return { content }
  },
  components: {
    VerticalRangeSlider,
    ProcessList,
  },
  data() {
    return {
      formData: {
        companyName: '',
        contactName: '',
        email: '',
        message: '',
        selectedProcess: '',
        selectedTempMin: null,
        selectedTempMax: null,
      },
      showFormError: false,
    }
  },
  methods: {
    submitForm() {
      console.log('Submit form')
      // Perform additional validation or submit logic here
      if (this.$refs.form.checkValidity()) {
        // Copy form data to clipboard
        this.copyFormDataToClipboard()

        // Form is valid, submit the data
        this.$refs.form.submit()
      } else {
        // Form is invalid, show error messages or take appropriate action
        console.log('Form is not valid! Please fill out all fields and try again.')
        this.showFormError = true
      }
    },
    copyFormDataToClipboard() {
      const formDataText = JSON.stringify(this.formData, null, 2)
      navigator.clipboard
        .writeText(formDataText)
        .then(() => {
          console.log('Form data copied to clipboard')
          // Optionally, you can show a success message to the user
        })
        .catch((error) => {
          console.error('Failed to copy form data to clipboard:', error)
          // Optionally, you can show an error message to the user
        })
    },
  },
}
</script>
