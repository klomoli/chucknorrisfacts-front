<template>
  <div>
    <v-layout :wrap="true">
      <v-flex xs12>
        <form @submit.prevent="submit">
            <v-select
              :items="filters"
              label="Select filter"
              data-vv-name="select"
              @change="selectFilter"
            ></v-select>
            <v-text-field
              v-if="searchFilterSelected"
              v-model="term"
              :counter="30"
              label="Search chuck norris facts"
            ></v-text-field>
            <v-select
              v-model="selectCategory"
              v-if="randomCategoryFilterSelected"
              :items="categories"
              label="Select category"
              data-vv-name="select"
            ></v-select>
          <v-btn
            class="mr-4"
            type="submit"
          >
            find
          </v-btn>
          <v-btn @click="clear">
            clear
          </v-btn>
        </form>
      </v-flex>
      <v-spacer></v-spacer>
      <v-flex xs12>
        <h2>
          RESULTS
        </h2>
      </v-flex>
      <v-flex xs12>
        <v-data-table
          :items="facts"
          :headers="headers"
          class="elevation-1"
          show-expand>
          <template #expanded-item="{headers,item}">
            <td :colspan="headers.length">
              <v-row>
                <v-col>
                  <p>{{ item.id }}</p>
                </v-col>
                <v-col>
                  <v-img :src=item.icon_url></v-img>
                </v-col>
                <v-col>
                  <p>{{ item.url }}</p>
                </v-col>
                 <v-col>
                  <p>{{ item.value }}</p>
                </v-col>
              </v-row>  
            </td>
          </template>
        ></v-data-table>
      </v-flex>
      <v-spacer></v-spacer>
      <v-flex xs12>
        <h2>
          LAST Searches
        </h2>
      </v-flex>
      <form @submit.prevent="sendByEmail">
        <v-text-field
          v-model="email"
          :rules="emailRules"
          label="Your email"
        ></v-text-field>
        <v-text-field
          v-model="lastIdSearch"
          label="Enter de #ID to send"
        ></v-text-field>
      <v-btn
        class="mr-6"
        type="submit"
      >
        Send search by email
      </v-btn>
      <v-alert
        v-if= "emailSend"
        border="right"
        color="blue-grey"
        dark
      >
        {{ emailText }}
      </v-alert>
      </form>
      <v-spacer></v-spacer>
      <v-flex xs12>
        <v-data-table
          :items="searches"
          :headers="headers"
          class="elevation-1"
          show-expand>
          <template #expanded-item="{headers,item}">
            <td :colspan="headers.length">
              <v-row v-for="fact in item.response.result" v-bind:key="fact.id">
                <v-col>
                  <p>{{ fact.id }}</p>
                </v-col>
                <v-col>
                  <v-img :src=fact.icon_url></v-img>
                </v-col>
                <v-col>
                  <p>{{ fact.url }}</p>
                </v-col>
                <v-col>
                  <p>{{ fact.value }}</p>
                </v-col>
              </v-row>  
            </td>
          </template>
        ></v-data-table>
      </v-flex>
    </v-layout>
  </div> 
</template>

<script>
  export default {
    data(){
      return{
        searchFilterSelected: false,
        randomCategoryFilterSelected: false,
        randomFilterSelected: false,
        email: '',
        emailRules: [
          v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
        ],
        lastIdSearch: 89,
        emailSend: false,
        emailText: '',
        term: '',
        select: null,
        categories: [],
        filters: [
          {text: "Random", value: "randomness"},
          {text: "Random with category", value: "random_category"},
          {text: "Search", value: "search"}
        ],
        headersExpaded:[
          {text: "#ID", value: "id"},
          {text: "#Icon", value: "icon_url"},
          {text: "#URL", value: "url"},
          {text: "#Value", value: "value"}
        ],
        facts: [],
        searches: []
      }
    },
    methods: {
      getFactsFromDB(){
        this.axios.get('/searches/200/0')
          .then(res => {
            console.log(res.data)
             this.searches = res.data;
             this.lastIdSearch = this.searches.last
          })
          .catch(e => {
            console.log(e.response);
          })
      },
      getFactsBySearch(search){
        this.axios.get('/facts?search='+search)
          .then(res => {
            this.facts = this.getResponses(res);
            this.getFactsFromDB();
          })
          .catch(e => {
            console.log(e.response);
          })
      },
      getFactsByrandomness(){
        this.axios.get('/facts?randomness=1')
          .then(res => {
            this.facts.push(this.getResponse(res));
            this.getFactsFromDB();
          })
          .catch(e => {
            console.log(e.response);
          })
      },
      getFactsByrandomCategory(selectedCategory){
        this.axios.get('/facts?random_category='+selectedCategory)
          .then(res => {
            this.facts.push(this.getResponse(res));
            this.getFactsFromDB();
          })
          .catch(e => {
            console.log(e.response);
          })
      },
      submit () {
        if(this.searchFilterSelected)
          this.getFactsBySearch(this.term);
        if(this.randomFilterSelected)
          this.getFactsByrandomness();
        if(this.randomCategoryFilterSelected)
          this.getFactsByrandomCategory(this.selectCategory);
      },
      clear () {
        this.term = ''
        this.searchFilterSelected = false
        this.randomFilterSelected = false
        this.randomCategoryFilterSelected = false
      },
      selectFilter(item){
        this.searchFilterSelected = false
        this.randomFilterSelected = false
        this.randomCategoryFilterSelected = false
        if(item=="search"){
          this.searchFilterSelected = true
        }else if(item=="random_category"){
          this.randomCategoryFilterSelected = true
        }else if(item=="randomness"){
          this.randomFilterSelected = true
        }
      },
      getCategories(){
        this.axios.get('https://api.chucknorris.io/jokes/categories')
          .then(res => {
            this.categories = res.data;
          })
          .catch(e => {
            console.log(e.response);
          })
      },
      getResponses(res){
        return res.data.table.data.result;
      },
      getResponse(res){
        return res.data.table.data;
      },
      sendByEmail(){
        this.axios.post('/search/'+this.lastIdSearch+'/email', {
          email: this.email
        })
        .then(res => {
          this.emailSend = true;
          this.emailText = res.data.data;
          this.email = '';
          this.lastIdSearch = '';
        })
        .catch(e => {
          console.log(e.response);
        })
      }
    },
    computed: {
      headers(){
        return [
          {text: "#ID", value: "id"},
          {text: "Created At", value: "created_at"},
          {text: "Term", value: "term"},
          {text: "Filter", value: "filter"},
        ]
      },
      headersLastSearch(){
        return [
          {text: "#ID", value: "id"},
          {text: "Created At", value: "created_at"}
        ]
      }
    },
    created() {
      this.getFactsFromDB();
      this.getCategories();
    }
  }
</script>
