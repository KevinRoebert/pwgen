<template>
  <div class="center">
    <div class="grid">
      <vs-row>
        <vs-col w="12">
          <div class="pw-box">
            <div class="pw">
              {{ password }}
            </div>
            <vs-button
              v-clipboard="() => password"
              v-clipboard:success="clipboardSuccessHandler"
              icon
              dark
              class="copy"
            >
              <i class="bx bx-clipboard" />
              Copy
            </vs-button>
          </div>
        </vs-col>
      </vs-row>
      <vs-row>
        <vs-col offset="4" w="4">
          <div class="config">
            <div class="con-switch">
              <vs-button
                icon
                flat
                color="rgb(59,222,200)"
                class="generate"
                @click="generate()"
              >
                <i class="bx bx-refresh" />
                Generate
              </vs-button>
              <vs-input
                id="length"
                v-model="config.length"
                type="number"
                class="switch"
                label="Length"
              />
              <vs-switch v-model="config.lowercase" class="switch" color="rgb(0, 196, 141)">
                Lowercase
              </vs-switch>
              <vs-switch v-model="config.uppercase" class="switch" color="rgb(0, 196, 141)">
                Uppercase
              </vs-switch>
              <vs-switch v-model="config.number" class="switch" color="rgb(0, 196, 141)">
                Numbers
              </vs-switch>
              <vs-switch v-model="config.ambiguous" class="switch" color="rgb(0, 196, 141)">
                Ambiguous Characters (e.g. I/l, O/0)
              </vs-switch>
            </div>
          </div>
        </vs-col>
      </vs-row>
    </div>
  </div>
</template>

<script>

export default {
  data: () => ({
    password: '',
    config: {
      length: 59,
      lowercase: true,
      uppercase: true,
      number: true,
      ambiguous: false
    }
  }),
  watch: {
    password (val) {
      const minLen = 29

      if (val.length < minLen) {
        this.openNotification('top-right', 'danger', '<i class="bx bx-error-circle" />', 'Insecure Password', 'Your chosen password is insecure. Consider to increase the length of your password.')
      } else {
        const bitLen = Math.floor(new Blob([this.password]).size / 6 * 40)// * 8 * 5 / 6 => size without separators
        this.openNotification('top-right', 'success', '<i class="bx bx-rocket" />', 'Secure Password', 'Congratulations, your chosen password is secure and ' + bitLen + 'bit strong.')
      }
    }
  },
  beforeMount () {
    this.generate()
  },
  methods: {
    generate () {
      const alphabetLowercase = 'abcdefghjkmnpqrstuvwxyz'
      const alphabetUppercase = 'ABCDEFGHJKMNPQRSTUVWXYZ'
      const lowercaseAmbiguous = 'ilo'
      const uppercaseAmbiguous = 'ILO'
      const numberAmbiguous = '01'
      const number = '23456789'

      this.password = '' // reset old password
      const len = this.config.length % 6 === 0 ? this.config.length + 1 : this.config.length
      const array = new Uint32Array(Math.ceil(5 * len / 6))
      window.crypto.getRandomValues(array)

      let pool = ''

      if (this.config.lowercase) {
        pool += alphabetLowercase

        if (this.config.ambiguous) {
          pool += lowercaseAmbiguous
        }
      }

      if (this.config.uppercase) {
        pool += alphabetUppercase

        if (this.config.ambiguous) {
          pool += uppercaseAmbiguous
        }
      }

      if (this.config.number) {
        pool += number

        if (this.config.ambiguous) {
          pool += numberAmbiguous
        }
      }

      let pw = ''
      for (let i = 0; i < array.length; i++) {
        if (i > 0 && i % 5 === 0) {
          pw += '-'
        }

        pw += pool.charAt(array[i] % pool.length)
      }

      this.password = pw.substring(0, this.config.length)
    },
    openNotification (position = null, color, icon, title, text) {
      this.$vs.notification({
        progress: 'auto',
        duration: 10000,
        flat: true,
        width: 'auto',
        icon,
        color,
        position,
        title,
        text
      })
    },
    clipboardSuccessHandler ({
      value,
      event
    }) {
      this.openNotification('top-right', 'success', '<i class="bx bx-clipboard" />', 'Password Copied', 'Your password was copied to the clipboard.')
    }
  }
}
</script>

<style>
[id*="--length"] {
  width: 100%;
  min-width: 48px;
  padding: 5px 5px 5px 10px;
  justify-content: center;
  border-radius: 20px;
  border: 0;
  position: relative;
  margin-top: 4px;
}
</style>

<style scoped>

.pw-box {
  width: 100%;
  margin: 0 auto;
  text-align: center;
}

.pw {
  background-color: white;
  border-radius: 5px;
  padding: 10px;
  float: left;
  font-weight: bold;
  font-size: xx-large;
}

.copy {
  float: left;
  margin-left: 15px;
  padding: 10px;
  font-weight: bold;
  font-size: larger;
  background-color: rgb(46, 73, 94);
}

.generate {
  font-weight: bold;
  font-size: larger;
  width: 100%;
  margin-bottom: 50px;
}

.config {
  align-items: center;
  justify-content: center;
  padding: 20px;
  width: max-content;
  margin-left: 25%;
}

.con-switch {
  align-items: center;
  justify-content: center;
}

.switch {
  margin-bottom: 10px;
}

.center {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  font-family: Helvetica, sans-serif;
  background-color: rgb(46, 73, 94);
  color: rgb(0, 196, 141);
}
</style>
