<template>
  <div>
    <h2>Lottery contract</h2>
    <p>
      This contract is managed by {{ manager }}. <br> There are currently {{ players.length }} people entered, competing to win {{ balance | wei }} ether!
    </p>

    <hr>

    <form @submit.prevent="submited">
      <h4>Want to try your luck?</h4>
      <div>
        <label for="Amount">Amount of ether to enter</label>
        <input type="text" v-model="value">
      </div>
      <button type="submit">Enter</button>
    </form>

    <hr>

    <h4>Read to pick a winner?</h4>
    <button @click="winnerHandle">Pick a winner</button>

    <hr>

    <h1>{{ message }}</h1>
  </div>
</template>

<script>
import web3 from '@/libraries/web3'
import lottery from '@/libraries/lottery'

export default {
  name: 'HelloWorld',
  async mounted() {
    const manager = await lottery.methods.manager().call()
    const players = await lottery.methods.getPlayers().call()
    const balance = await web3.eth.getBalance(lottery.options.address)

    this.manager = manager
    this.players = players
    this.balance = balance
  },
  data() {
    return {
      manager: '',
      players: [],
      balance: '',
      value: '',
      message: ''
    }
  },
  filters: {
    wei(balance) {
      return web3.utils.fromWei(balance, 'ether')
    }
  },
  methods: {
    async submited() {
      const accounts = await web3.eth.getAccounts()

      this.message = 'Waiting on transaction success...'

      await lottery.methods.enter().send({
        from: accounts[0],
        value: web3.utils.toWei(this.value, 'ether')
      })

      this.message = 'You have been entered!'
    },
    async winnerHandle() {
      const accounts = await web3.eth.getAccounts()

      this.message = 'Waiting on transaction success...'

      await lottery.methods.pickWinner().send({
        from: accounts[0]
      })

      this.message = 'A winner has been picked!'
    }
  }
}
</script>