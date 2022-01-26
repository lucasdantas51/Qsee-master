<template>
  <div class="modal-card" style="width: auto">
    <header class="modal-card-head" :class="{'has-background-danger' : settings.office == null}">
      <p v-if="settings.office == null" class="modal-card-title has-text-white">Configure cada guia antes de usar este aplicativo</p>
      <p v-else class="modal-card-title">Configuração</p>
    </header>
    <section class="modal-card-body">
      <b-tabs v-model="activeTab" type="is-boxed">
        <b-tab-item label="General" icon="settings-outline">
          <b-field label="Nome do Escritório">
            <b-input type="text" v-model="office" placeholder="Nome do escritório"></b-input>
          </b-field>
        </b-tab-item>

        <b-tab-item label="Kiosk" icon="ticket-confirmation">
          <b-field label="Saudações">
            <b-input type="text" v-model="kioskGreeting" placeholder="Texto para saudações"></b-input>
          </b-field>

          <b-field style="width:450px" label="Topics">
            <b-taginput
              type="is-primary"
              v-model="topics"
              ellipsis
              icon="label"
              placeholder="Adicionar tópico"
            ></b-taginput>
          </b-field>
        </b-tab-item>

        <b-tab-item label="Agentes" icon="comment-account-outline">
          <b-field label="Número de agentes">
            <b-input type="number" v-model="agents" placeholder="8" style="width:100px"></b-input>
          </b-field>
          <h2 class="title is-6">Defina como receber os chamados</h2>
          <div class="field">
            <b-radio v-model="takeCustomer" native-value="1">Próximo "Pagamento" ou próximo não pagamento</b-radio>
          </div>
          <div class="field">
            <b-radio v-model="takeCustomer" native-value="2">Pegue o próximo chamado (independentemente do tópico)</b-radio>
          </div>
          <div class="field">
            <b-radio v-model="takeCustomer" native-value="3">Escolha qualquer chamado em espera</b-radio>
          </div>
        </b-tab-item>

        <b-tab-item label="Display" icon="monitor">
          <!-- <h2 class="title is-6">Voice</h2> -->
          <h2 class="title is-6">Vozes</h2>
          <b-field>
            <b-select
              v-model="chosenVoice"
              placeholder="Escolha uma voz"
              icon="voice"
              name="voice"
              id="voice"
              expanded
            >
              <option
                v-for="(voiceSelect, idx) in voicesUK"
                :key="idx"
                :value="voiceSelect.name"
              >{{voiceSelect.name}} ({{voiceSelect.lang}})</option>
            </b-select>
          </b-field>
          <br>
          <nuxt-link v-if="settings.office != null" to="/slideshow">
            <button class="button is-primary">
              <span class="icon">
                <i class="mdi mdi-file-image mdi-24px"></i>
              </span>
              <span>Atualizar imagens</span>
            </button>
          </nuxt-link>
        </b-tab-item>
      </b-tabs>
    </section>
    <footer v-if="settings.office != null" class="modal-card-foot">
      <button class="button" type="button" @click="$parent.close()">Cancelar</button>
      <button @click.prevent="updateSettings('close')" class="button is-primary">
        <b-icon icon="check"></b-icon>
        <span>Salvar</span>
      </button>
    </footer>
    <footer v-else class="modal-card-foot">
      <button v-if="activeTab == 3" @click.prevent="updateSettings('no-close')" class="button">
        <b-icon icon="check"></b-icon>
        <span>Salvar e continuar</span>
      </button>
    </footer>
  </div>
</template>

<script>
import { fireDb } from '~/plugins/firebase.js'

export default {
  data() {
    return {
      activeTab: 0,
      settings: [],
      office: '',
      kioskGreeting: '',
      topics: [],
      agents: 0,
      takeCustomer: 1,
      synth: window.speechSynthesis,
      chosenVoice: '',
      voiceTypes: [],
      picURL: ''
    }
  },
  firestore() {
    return {
      settings: fireDb.collection('settings').doc('general')
    }
  },
  computed: {
    voicesUK() {
      return this.voiceTypes.filter(x => x.lang.indexOf('en') >= 0)
    }
  },
  created() {
    this.pics
    // Build list of available voices for display screen
    this.voiceTypes = this.synth.getVoices()
    this.synth.onvoiceschanged = () => {
      this.voiceTypes = this.synth.getVoices()
    }
    // Replace default values from settings in Firebase
    setTimeout(() => {
      this.office = this.settings.office
      this.takeCustomer = this.settings.takeCustomer
      this.agents = this.settings.agents
      this.kioskGreeting = this.settings.kioskGreeting
      this.topics = this.settings.topics
      this.chosenVoice = this.settings.chosenVoice
    }, 500)
  },
  methods: {
    updateSettings(closeDialogue) {
      const ref = fireDb.collection('settings').doc('general')
      const document = {
        office: this.office,
        kioskGreeting: this.kioskGreeting,
        topics: this.topics,
        agents: +this.agents,
        takeCustomer: +this.takeCustomer,
        chosenVoice: this.chosenVoice
      }
      ref
        .set(document)
        .then(() => {
          console.log(`Settings updated`)
        })
        .catch(error => {
          console.log('Error saving settings:', error)
          this.$dialog.alert({
            title: 'Error',
            message: error.code,
            type: 'is-danger',
            hasIcon: true,
            icon: 'alert-circle'
          })
        })
      if (closeDialogue == 'close') this.$parent.close()
    }
  }
}
</script>

<style scoped>
</style>
