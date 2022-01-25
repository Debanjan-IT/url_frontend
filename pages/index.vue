<template>
<div class="container">
  <div v-if="security == 2">
    <b-card bg-variant="light" header="Invalid URL." class="text-center">
      <div>
        <b-btn variant="outline-success" @click.prevent="done()">Shorten an url</b-btn>
      </div>
    </b-card>
  </div>
  <div v-else-if="security == 0">
    <b-card bg-variant="light" text-variant="black" header="URL Shortener" class="text-center">
      <b-form-input type="text" v-model="url_text" placeholder="Enter the url" style="margin-top: 10px;"></b-form-input>
      <b-form-input type="text" v-model="s_url" style="margin-top: 10px;" readonly></b-form-input>
      <b-btn variant="outline-dark" style="margin-top: 10px;" @click.prevent="check()">Shorten Up</b-btn>
    </b-card>
  </div>
  <div v-else>
    <b-card bg-variant="light" header="Your URL may not be secure." class="text-center">
        <div>
          <b-btn variant="outline-danger" @click.prevent="cancel()">Cancel</b-btn>
          <b-btn variant="outline-success" @click.prevent="proceed()">Proceed</b-btn>
        </div>
      </b-card>
  </div>
</div>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      url_text: '',
      s_url: '',
      security: 0,
    }
  },
  async created() {
    this.url_text = "",
    this.s_url = "",
    this.security = 0
    let params= window.location.href.split('/')
    const result =  params.filter(e =>  e);
    if (result.length > 2) {
      let code = window.location.href.split("=").pop()
      const response = await this.$axios.post("http://localhost:4040/api/operations/get_link", {
        code: code
      })
      if (response.data.success) {
        window.location.assign(response.data.link)
      } else {
        this.security = 2
      }
    }
  },
  methods: {
    async check() {
      const response = await this.$axios.post("http://localhost:4040/api/operations/check", {
        url: this.url_text
      })
      if (response.data.security == 1){
        const resp = await this.$axios.post("http://localhost:4040/api/operations/shorten", {
          url: this.url_text
        })
        if (resp.data.status == "success") {
          this.s_url = `${window.location.href.split("?")[0]}?code=${resp.data.shortened_code}`
        }
      } else {
        this.security = 1
      }
    },
    async proceed() {
      const resp = await this.$axios.post("http://localhost:4040/api/operations/shorten", {
        url: this.url_text
      })
      if (resp.data.status == "success") {
        this.s_url = `${window.location.href.split("?")[0]}?code=${resp.data.shortened_code}`
        this.security = 0
      }
    },
    cancel() {
      this.url_text = "",
      this.s_url = "",
      this.security = 0
    },
    done() {
      this.url_text = "",
      this.s_url = "",
      this.security = 0
    }
  }
}
</script>

<style scoped>
.container{
  text-align: center;
}
.text-center{
  margin-top: 10%;
}
</style>