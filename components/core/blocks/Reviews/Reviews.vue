<template>
  <div class="border-top border-grey">
    <section class="container px-15 pb-10">
      <div class="md:flex">
        <div class="md:w-1/2 md:px-4 md:-ml-4 pt-10">
          <h2 class="text-h3 m-0 mb-10">
            {{ $t('Reviews') }}
          </h2>
          <reviews-list :per-page="4" :items="reviews ? reviews : []" />
        </div>
        <div class="md:w-1/2 md:px-4 md:-mr-4 pt-50">
          <h2 class="text-h3 m-0 mb-10 mt-10">
            {{ $t('Add review') }}
          </h2>
          <form action="#" @submit.prevent="outOfScope()">
            <div class="mb-3 pt50">
              <base-input
                type="text"
                :placeholder="$t('First name') + ' *'"
                v-model="formData.name"
                @blur="$v.formData.name.$touch()"
                :validations="[
                  {
                    condition: $v.formData.name.$error && !$v.formData.name.required,
                    text: $t('Field is required')
                  }
                ]"
              />
            </div>
            <div class="mb-3">
              <base-input
                type="email"
                :placeholder="$t('Email address') + ' *'"
                v-model="formData.email"
                @blur="$v.formData.email.$touch()"
                :validations="[
                  {
                    condition: $v.formData.email.$error && !$v.formData.email.required,
                    text: $t('Field is required')
                  },
                  {
                    condition: !$v.formData.email.email && $v.formData.email.$error,
                    text: $t('Please provide valid e-mail address.')
                  }
                ]"
              />
            </div>
            <div class="mb-3">
              <base-input
                type="text"
                :placeholder="$t('Summary') + ' *'"
                v-model="formData.summary"
                @blur="$v.formData.summary.$touch()"
                :validations="[
                  {
                    condition: $v.formData.summary.$error && !$v.formData.summary.required,
                    text: $t('Field is required')
                  }
                ]"
              />
            </div>
            <div class="mb-3">
              <base-textarea
                type="text"
                :placeholder="$t('Review') + ' *'"
                v-model="formData.review"
                @blur="$v.formData.review.$touch()"
                :validations="[
                  {
                    condition: $v.formData.review.$error && !$v.formData.review.required,
                    text: $t('Field is required')
                  }
                ]"
              />
            </div>
            <div class="m-0 buttons">
              <button-full
                @click.native="validate()"
                :class="{ 'w-auto': !currentUser }"
              >
                {{ $t('Add review') }}
              </button-full>
              <span
                class="block text-center py-4"
                v-if="!currentUser"
              >
                {{ $t('or') }} <a href="#" class="text-primary" @click.prevent="login()">{{ $t('login') }}</a> {{ $t('to account') }}
              </span>
            </div>
          </form>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { required, email } from 'vuelidate/lib/validators'

import BaseInput from 'theme/components/core/blocks/Form/BaseInput'
import BaseTextarea from 'theme/components/core/blocks/Form/BaseTextarea'
import ButtonFull from 'theme/components/theme/ButtonFull'
import ReviewsList from 'theme/components/theme/blocks/Reviews/ReviewsList'
import { Reviews } from '@vue-storefront/core/modules/review/components/Reviews'
import { AddReview } from '@vue-storefront/core/modules/review/components/AddReview'
export default {
  name: 'Reviews',
  data () {
    return {
      formData: {
        name: '',
        email: '',
        summary: '',
        review: ''
      }
    }
  },
  computed: {
    product () {
      return this.$store.state.product
    },
    currentUser () {
      return this.$store.state.user.current
    }
  },
  methods: {
    validate () {
      this.$v.$touch()
      if (!this.$v.$invalid) {
        this.submit()
      }
    },
    refreshList () {
      this.$store.dispatch('review/list', { productId: this.product.current.id })
    },
    submit () {
      this.addReview({
        'product_id': this.product.current.id,
        'title': this.formData.summary,
        'detail': this.formData.review,
        'nickname': this.formData.name,
        'review_entity': 'product',
        'review_status': 2,
        'customer_id': this.currentUser ? this.currentUser.id : null
      })
    },
    clearReviewForm () {
      this.formData.name = ''
      this.formData.email = ''
      this.formData.summary = ''
      this.formData.review = ''
      this.$v.$reset()
    },
    login () {
      this.$bus.$emit('modal-show', 'modal-signup')
    },
    fillInUserData () {
      if (this.currentUser) {
        this.formData.name = this.currentUser.firstname
        this.formData.email = this.currentUser.email
      }
    }
  },
  mounted () {
    this.$bus.$on('product-after-load', this.refreshList)
    this.$bus.$on('clear-add-review-form', this.clearReviewForm)
    this.$bus.$on('user-after-loggedin', this.fillInUserData)
  },
  destroyed () {
    this.$bus.$off('product-after-load', this.refreshList)
    this.$bus.$off('clear-add-review-form', this.clearReviewForm)
    this.$bus.$off('user-after-loggedin', this.fillInUserData)
  },
  beforeMount () {
    this.refreshList()
    this.fillInUserData()
  },
  mixins: [ Reviews, AddReview ],
  validations: {
    formData: {
      name: {
        required
      },
      email: {
        required,
        email
      },
      summary: {
        required
      },
      review: {
        required
      }
    }
  },
  components: {
    ButtonFull,
    BaseInput,
    BaseTextarea,
    ReviewsList
  }
}
</script>
