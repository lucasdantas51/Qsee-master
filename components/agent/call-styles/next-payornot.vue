<template>
  <div class="box">
    <h2 class="title is-4">Próxima pessoa</h2>
    <b-tabs v-model="activeTab" type="is-boxed">
      <b-tab-item label="Non-payments" icon="ticket-confirmation">
        <span
          v-if="nextTicket.length"
          v-for="tkt in nextTicket"
          :key="tkt.id"
          class="has-text-centered"
        >
          <h2 class="title is-4">Chamado {{tkt.id}}</h2>
          <h3 class="title is-5">{{tkt.topic}}</h3>
          <p class="has-text-centered">{{time(tkt.printed)}}</p>
          <hr>
          <p class="has-text-centered">
            <button @click="takeCustomer(tkt.id, tkt.printed)" class="button is-primary">
              <span>Pegar essa pessoa</span>
            </button>
          </p>
        </span>
        <span v-if="!nextTicket.length" class="has-text centered">Nenhuma pessoa esperando</span>
      </b-tab-item>
      <b-tab-item label="Payments" icon="cash">
        <span
          v-if="nextPayment.length"
          v-for="pay in nextPayment"
          :key="pay.id"
          class="has-text-centered"
        >
          <h2 class="title is-4">Chamados {{pay.id}}</h2>
          <p class="has-text-centered">{{time(pay.printed)}}</p>
          <hr>
          <p class="has-text-centered">
            <button @click="takeCustomer(pay.id, pay.printed)" class="button is-primary">
              <span>Pegar essa pessoa</span>
            </button>
          </p>
        </span>
        <span v-if="!nextPayment.length" class="has-text centered">Nenhuma pessoa esperando</span>
      </b-tab-item>
    </b-tabs>
  </div>
</template>

<script>
import { fireDb } from '~/plugins/firebase.js'

export default {
  props: ['tickets', 'desk', 'today'],
  computed: {
    nextTicket() {
      const tkt = this.tickets
        .filter(
          x => x.topic != 'Payment' && x.state != 'seen' && x.state != 'called'
        )
        .slice(0, 1)
      return tkt
    },
    nextPayment() {
      const tkt = this.tickets
        .filter(
          x => x.topic == 'Payment' && x.state != 'seen' && x.state != 'called'
        )
        .slice(0, 1)
      return tkt
    }
  },
  methods: {
    takeCustomer(ticket, printed) {
      fireDb
        .collection(this.today)
        .doc(ticket)
        .set(
          {
            state: 'called',
            desk: this.desk,
            seen: new Date().getTime(),
            wait: new Date().getTime() - printed,
            timescalled: 1
          },
          { merge: true }
        )
      return
    },
    time(a) {
      return new Date(a).toLocaleTimeString('en-GB', {
        hour: 'numeric',
        minute: 'numeric'
      })
    }
  }
}
</script>

<style scoped>
</style>
