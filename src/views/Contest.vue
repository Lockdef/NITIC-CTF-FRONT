<template>
  <div class="contest">
    <div v-if="isLogin">
      <v-container>
        <div v-for="type in problemTypes" :key="type">
          <h1 class="mt-4 display-2">{{ type }}</h1>
          <v-row>
            <v-col
              cols="12"
              sm="6"
              md="4"
              lg="3"
              v-for="problem in problemsList[type]"
              :key="problem.id"
            >
              <v-btn
                @click="showModal(problem)"
                color="light-green darken-3"
                width="100%"
              >
                <span class="white--text headline">
                  {{ problem.problemTitle }}
                </span>
              </v-btn>
            </v-col>
          </v-row>
        </div>
        <v-card class="mt-5">
          <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">ランク</th>
                  <th class="text-left">名前</th>
                  <th class="text-left">スコア</th>
                </tr>
              </thead>
              <tbody>
                <tr class="user-rank">
                  <td>{{ userRank.rank }}</td>
                  <td>{{ userRank.username }}</td>
                  <td>{{ userRank.score }}</td>
                </tr>
                <tr v-for="user in ranking.filter( user => user.username !== userRank.username )" :key="user.rank">
                  <td>{{ user.rank }}</td>
                  <td>{{ user.username }}</td>
                  <td>{{ user.score }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-card>
        <v-dialog v-model="dialog">
          <v-card v-if="dialog">
            <v-list-item-content class="mx-6">
              <v-list-item-title>
                <h1 class="mt-2 mb-4">
                  {{ currentProblem.problemTitle }}
                  <span
                    v-if="result != null"
                    :class="result ? 'ac-color' : 'wa-color'"
                  >
                    {{ result ? "AC" : "WA" }}
                  </span>
                </h1>
              </v-list-item-title>
                <div class="grey--text">作問者: {{ currentProblem.writer }}</div>
                <div class="grey--text">難易度: {{ currentProblem.difficulty }}</div>
                <div class="mb-5 grey--text">点数: {{ currentProblem.score }}</div>
              <div v-html="mdIt.render(currentProblem.problemText)" />
              <v-text-field v-model="flag" placeholder="flag" />
              <v-btn color="primary" @click="problemSubmit(currentProblem.id)">
                Submit
              </v-btn>
              <v-card-actions>
                <v-btn @click="dialog = false">Close</v-btn>
              </v-card-actions>
            </v-list-item-content>
          </v-card>
        </v-dialog>
      </v-container>
    </div>
    <div v-if="isLogin === false">
       <v-container>
         <h1 class="align-center">ログインしてください</h1>
         <p class="align-center">ログインにはGoogleアカウントが必要になります。<br />
         ログイン後、ご自身の名前をクリックすることでユーザーネームを変更可能です。</p>
       </v-container>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import markdownIt from 'markdown-it'
import firebase from 'firebase'

export default {
  name: 'Contest',
  data () {
    return {
      dialog: false,
      flag: null,
      currentProblem: null,
      problemsList: null,
      problemTypes: null,
      result: null,
      ranking: null,
      uid: null,
      userRank: null,
      isLogin: null
    }
  },
  computed: {
    mdIt () {
      return new markdownIt({ // eslint-disable-line new-cap
        html: true,
        linkify: true,
        breaks: true,
        typographer: true
      })
    }
  },
  mounted () {
    this.isLogined()
    this.axios
      .get('https://api.waku-waku-club.com/api/problem/list/' + this.$route.params.id)
      .then(response => {
        this.problemsList = response.data
        this.problemTypes = Object.keys(this.problemsList).reverse()
      })
      .catch(response => console.log(response))
    this.axios
      .get('https://api.waku-waku-club.com/api/standings/' + this.$route.params.id)
      .then(response => {
        this.ranking = response.data
      })
      .catch(response => console.log(response))
  },
  methods: {
    showModal (problem) {
      this.currentProblem = problem
      this.dialog = true
      this.result = null
      this.flag = null
    },
    isLogined () {
      firebase.auth().onAuthStateChanged(
        function (user) {
          if (user) {
            this.uid = user.uid
            this.isLogin = true
            this.axios
              .get(
                'https://api.waku-waku-club.com/api/score/contest_1/' + this.uid
              )
              .then(response => {
                this.userRank = response.data
              })
          } else {
            this.isLogin = false
          }
        }.bind(this)
      )
    },
    problemSubmit (problemId) {
      this.axios
        .post('https://api.waku-waku-club.com/api/problem/submit', {
          contestName: this.$route.params.id,
          problemId: problemId,
          flag: this.flag,
          uid: this.uid
        })
        .then(response => {
          this.result = response.data.AC
        })
        .catch(response => console.log())
    }
  }
}
</script>

<style scoped>
.wa-color {
  color: #fff;
  background-color: #b71c1c;
  padding: 1px 5px;
}
.ac-color {
  color: #fff;
  background-color: #00c853;
  padding: 1px 5px;
}
.user-rank {
  background-color: #eee;
}
</style>
