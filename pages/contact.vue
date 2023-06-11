<template>
  <main class="pt-6 md:pt-14">
    <header class="w-full max-w-3xl mx-auto mb-12 pt-20 md:pt-0 md:px-24 relative">
      <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" class="absolute top-0 -left-0" />
      <h1 class="max-w-2xl mb-8">{{ content.title }}</h1>
      <nuxt-content :document="content" />
    </header>

    <section class="w-full max-w-3xl mx-auto md:px-24">
      <form @submit.prevent="submitForm" netlify ref="form" name="Contact Form">
        <input type="hidden" name="form-name" value="Contact Form" />
        <div class="flex w-full mb-16">
          <div class="w-1/2 pr-2 md:pr-10">
            <h2 class="text-lg text-center md:text-right mb-5">MY PROCESS</h2>

            <ProcessList :containerHeight="400" @processListChange="formData.selectedProcess = $event" />
          </div>
          <div class="w-1/2 md:pl-10">
            <h2 class="text-lg mb-5 text-center md:text-left">MY TEMPERATURE</h2>
            <div class="pl-10">
              <VerticalRangeSlider
                :min="100"
                :max="1000"
                :defaultValues="[100, 450]"
                :containerHeight="400"
                @sliderValueChange="
                  {
                    ;(formData.selectedTempMin = $event[0].value), (formData.selectedTempMax = $event[1].value)
                  }
                "
              />
            </div>
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
        submissionDateTime: null,
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
      // Form is valid, set submission date/time
      this.formData.submissionDateTime = new Date().toISOString()

      //const formDataText = JSON.stringify(this.formData, null, 2)
      // formDataText should show each form field on a new line
      // with the field name and value separated by a colon

      // Copy form data to clipboard
      const formDataText = Object.entries(this.formData)
        .map(([key, value]) => `${key}: ${value}`)
        .join('\n')

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
