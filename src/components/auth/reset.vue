<template>
  <div>
             <v-scale-transition>
           <v-btn v-show="btn"
          fixed @click="$router.go(-1)"
          bottom
          right fab
          color="white" style="z-index:10" class="mb-12">
          <v-icon>mdi-arrow-left</v-icon>
        </v-btn>
    </v-scale-transition>
    <v-card-text class="">
      <v-form
          onSubmit="return false;" ref="form">
      <v-container class="px-0 pb-0">
      <v-row>
        <v-col cols="12" class="pb-1">
        <v-text-field
          color="orange darken-3"
          label="Phone number"
          type="text"
          placeholder="08012111234"
          :rules="numberRules" style="width:320px" class="ma-auto px-8"
           validate-on-blur
          @keyup.enter.native="reset"
          :error-messages='error.phone'
           v-model="phone"
          v-if="!choice"
          hint="A verification code will be sent to this number."
          required
        ></v-text-field>
        <v-text-field
          color="orange darken-3"
          label="Email"
          type="email" style="width:320px" class="ma-auto px-8"
          v-if="choice"
          
          :rules="[rules.required, rules.email]"
          validate-on-blur
          @keyup.enter.native="reset"
          placeholder="eg. foodrepublic@gmail.com"
           v-model="email"
          :error-messages='error.email'
          hint="A verification code will be sent to this email."
          
        ></v-text-field>
        
        </v-col>
      </v-row>
          </v-container>
            <v-card color="transparent" flat tile class="ma-auto text-center" width="290px">
          <!-- <v-switch class="px-5 grey--text mt-0 mx-auto" v-model="choice" color="grey" >
<template v-slot:label>
  <span class="caption" v-if="!choice">Use <strong>email</strong> for verification</span>
  <span class="caption" v-if="choice">Use <strong>phone</strong> for verification</span>
        
      </template>

          </v-switch> -->

        <v-btn
          :loading="loading"
          @click.prevent="reset()"
          dark
          rounded
          color="white"
        class="caption font-weight-black text--darken-4 mx-2 orange--text"
        >next</v-btn>
            </v-card>
      </v-form>
    </v-card-text>
  </div>
</template>
<style>
</style>
<script>
import axios from 'axios'
import wrapper from 'axios-cache-plugin'

let http = wrapper(axios, {
  maxCacheSize: 15, // cached items amounts. if the number of cached items exceeds, the earliest cached item will be deleted. default number is 15.
  ttl: 60000, // time to live. if you set this option the cached item will be auto deleted after ttl(ms).
  excludeHeaders: true // should headers be ignored in cache key, helpful for ignoring tracking headers
})

export default {
  data() {
    return {
      name: "",
      email: "",
      loading: false,
      btn: false,
      phone: '',
      choice: false,
      error: {
        email: '',
        phone: ''
      },
      rules: {
        required: value => !!value || "Required.",
        phone: value => value.length >= 8 || "Min 8 characters",
        c_password: value =>
          this.password === this.c_password || "does not match passoword",
        email: value => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
          return pattern.test(value) || "Invalid e-mail.";
        }
      },
      numberRules: [
        (v) => v.length > 10 || "11 numbers expected",
        (v) => !!v || 'Phone number is required',
        (v) => v[0] === '0' || 'Phone number must start with "0"',
        (v) => /^[0-9]*$/.test(v) || 'Number must be valid'
      ]
    } //08164691026
  },
    mounted(){
    setTimeout(() => {
      this.btn=true
    }, 500);
  },
    beforeRouteLeave (to, from, next) {
  this.btn = false
     setTimeout(() => {
      next()
    }, 50);
},
  methods: {
  reset() {
    if (this.$refs.form.validate()) {
    var sn = this
    this.loading = true
      const url = '/reset'
      if (!sn.choice) {
        var d = sn.phone
        var t = 'phone'
      } else {
        var d = sn.email
        var t = 'email'
      }
      http({
        url: url,
        method: 'post',
        params: {
          choice: sn.choice,
          data: d,
          type: t,
          phone: sn.phone,
          email: sn.email
        }
      })
        .then((response) => {
          sn.loading = false
          const d = response.data.success.user
          var type = ''
          if (d.email) {
           var type = d.email
          } else {
           var type = d.phone
          }
          sn.$store.dispatch('setUser', d)
          sn.$store.dispatch('snack', {
            color: 'green',
            text: 'A 5 digit passcode has been sent to '+type  
          })
          sn.$router.push('/auth/resetpasscode')
        }).catch(function (error) {
          sn.loading = false
          sn.$store.dispatch('snack', {
            color: 'red',
            text: 'An error occured'  
          })
        })
      } else {
          sn.loading = false
      }
    }
  }
}
</script>
