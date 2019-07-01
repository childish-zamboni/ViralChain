<template>
  <div class="dashboard">
    <div class="container">
      <div class="row justify-content-md-center">
        <div class="col-md-8 col-md-offset-2">
          <h1 class="page-header align-middle">ViralChain
            <img :src="user.avatarUrl() ? user.avatarUrl() : '/avatar-placeholder.png'" class="avatar">
            <small><span class="sign-out"><a href="#" @click.prevent="signOut">Sign Out</a></span></small>
          </h1>
          <h2 class="user-info">
            <small>
              {{ user.name() ? user.name() : 'Nameless Person'}}'s Profile
            </small>
            <small class="float-right">
            {{ user.username ? user.username : user.identityAddress }}
            </small>

          </h2>


          <transition name="fade">
          <form @submit.prevent="addTodo" :disabled="!todo || !title" v-show="writing">
              <input v-model="title" type="text" class="form-control" placeholder="Title" autofocus>
              <br>
              <textarea v-model="todo" type="text" style="width:100%;" rows="12" autofocus placeholder="Write your article here..."></textarea>
              <br>
              <br>
              <span class="input-group-btn">
                <button class="btn btn-default bg-light" type="submit" :disabled="!todo || !title">Publish</button>
                <a @click.prevent="writing=false;" href="#" class="float-right">Cancel</a>
              </span>

          </form>
          </transition>

          <a @click.prevent="writing=true;" href="#" v-if="!writing">Write a new article ✏️</a>

          <br><br><br>

          <ul class="list-group">
            <p><span style="text-decoration:underline">Your articles</span>&nbsp
              <a v-if="!articles" href="#" style="text-decoration:none!important" @click.prevent="articles=true;">➕</a>
              <a v-if="articles" href="#" style="text-decoration:none!important" @click.prevent="articles=false;">➖</a>
              <span class="float-right" v-show="articles">Delete 🗑</span>
            </p>
            <li v-for="todo in todos" v-if="todo.type==='news' && articles===true"
              class="list-group-item"
              :class="{completed: todo.completed}"
              :key="todo.id">
              <label @click.prevent="showNews(todo)" style="text-decoration:underline;">
                {{ todo.title }} ({{ todo.upvotes.length-todo.downvotes.length}} point<span v-show="todo.upvotes.length-todo.downvotes.length!=1">s</span>)
              </label>
              <a @click.prevent="todos.splice(todos.indexOf(todo), 1)"
                class="delete float-right"
                href="#">X</a>
            </li>
          </ul>

          <br><br>

          <ul class="list-group">
            <p><span style="text-decoration:underline">Your feed</span>&nbsp
              <a v-if="!feed" href="#" style="text-decoration:none!important" @click.prevent="feed=true;">➕</a>
              <a v-if="feed" href="#" style="text-decoration:none!important" @click.prevent="feed=false;">➖</a>
              <a class="float-right" href="#" style="text-decoration:none!important" @click.prevent="loadFeed">Refresh ⟲</a>
            </p>

            <li v-for="todo in todosFeed" v-if="todo.type==='news' && feed===true"
              class="list-group-item"
              :class="{completed: todo.completed}"
              :key="todo.id">
              <label @click.prevent="showNews(todo)" style="text-decoration:underline;">
                {{ todo.title }} ({{ todo.upvotes.length-todo.downvotes.length}} point<span v-show="todo.upvotes.length-todo.downvotes.length!=1">s</span>)
              </label>
            </li>

          </ul>

          <br><br>

          <form @submit.prevent="addSubscription" :disabled="! subUsername">
            <p style="font-style:italic">Subscribe to a user:</p>
            <div class="input-group">
            <input v-model="subUsername" type="text" class="form-control" placeholder="Username" autofocus>
              <span class="input-group-btn">
                <button class="btn btn-default bg-light" type="submit" :disabled="! subUsername">Add subscription</button>
              </span>
            </div>
          </form>

        </div>
      </div>
    </div>

    <transition name="fade">
    <modal v-if="showModal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <div class="modal-header">
            <p><span style="text-decoration:underline;">{{viewing.title}}</span> by <span style="font-weight: bold;">{{viewing.author}}</span></p>
            <button class="modal-default-button" @click="showModal = false">
              Close
            </button>
          </div>

          <div class="modal-body">
            {{viewing.text}}
          </div>

          <div class="modal-footer">
            <p style="width:100%">
              <span class="float-left">Points: {{viewing.upvotes.length - viewing.downvotes.length}}</span>
              <span class="float-right" v-if="!feedNews">
                <a v-if="!viewing.upvotes.includes(user.username)" href="#" class="arrow" @click.prevent="upvote">△</a>
                <a v-if="viewing.upvotes.includes(user.username)" href="#" class="arrow" @click.prevent="upvote">🔺</a>
                <a v-if="!viewing.downvotes.includes(user.username)" href="#" class="arrow" @click.prevent="downvote">▽</a>
                <a v-if="viewing.downvotes.includes(user.username)" href="#" class="arrow" @click.prevent="downvote">🔻</a>
              </span>
            </p>

          </div>

        </div>
      </div>
    </div>
    </modal>
  </transition>

  </div>
</template>

<script>
import { userSession } from '../userSession'
import { lookupProfile } from 'blockstack';

var STORAGE_FILE = 'todos.json'

export default {
  name: 'dashboard',
  props: ['user'],
  data () {
    return {
      todos: [],
      todo: '',
      title: '',
      uidCount: 0,
      writing: false,
      showModal: false,
      articles: true,
      feed: true,
      subUsername: '',
      todosFeed: [],
      feedNews: false,
    }
  },
  watch: {
    todos: {
      handler: function (todos) {
        // encryption is now enabled by default
        return userSession.putFile(STORAGE_FILE, JSON.stringify(todos), { encrypt: false })
      },
      deep: true
    }
  },
  mounted () {
    this.fetchData();
  },
  methods: {

    fetchData () {
      var temp = this;

      userSession.getFile(STORAGE_FILE, { decrypt: false }) // decryption is enabled by default
        .then((todosText) => {
          var todos = JSON.parse(todosText || '[]')
          todos.forEach(function (todo, index) {
            todo.id = index
            if (todo.type === 'subscription') {
              lookupProfile(todo.subscription).then(function(profile){
                var url = profile.apps[window.location.href.substring(0,window.location.href.length-1)] + 'todos.json';
                axios.get(url).then(function(res){
                  //console.log(res.data);
                  temp.todosFeed = temp.todosFeed.concat(res.data);
                });
              })
              .catch(function(e){

              });
            }
          })
          this.uidCount = todos.length
          this.todos = todos
        })
    },

    loadFeed () {
      var temp = this;
      this.todosFeed = [];
      for (var i=0; i<this.todos.length; i++) {
        if (this.todos[i].type === 'subscription') {
          lookupProfile(this.todos[i].subscription).then(function(profile){
            var url = profile.apps[window.location.href.substring(0,window.location.href.length-1)] + 'todos.json';
            axios.get(url).then(function(res){
              //console.log(res.data);
              temp.todosFeed = temp.todosFeed.concat(res.data);
            });
          });
        }
      }
    },

    addTodo () {
      if (!this.todo.trim() || !this.title.trim()) {
        return;
      }
      this.todos.unshift({
        id: this.uidCount++,
        text: this.todo,
        completed: false,
        title: this.title,
        author: this.user.name(),
        upvotes: [this.user.username],
        downvotes: [],
        type: 'news',
        authorID: this.user.username
      })
      this.todo = ''
      this.title = ''
      this.writing = false;
    },

    addSubscription () {
      if (!this.subUsername.trim()) {
        return;
      }
      for (var i=0; i<this.todos.length; i++) {
        if (this.todos[i].type === 'subscription' && this.todos[i].subscription === this.subUsername ) {
          return;
        }
      }
      this.todos.unshift({
        id: this.uidCount++,
        subscription: this.subUsername,
        type: 'subscription'
      });
      this.loadFeed();
      this.subUsername = '';
    },

    signOut () {
      userSession.signUserOut(window.location.href)
    },

    showNews (news) {
      if(news.authorID === this.user.username) {
        this.feedNews = false;
      } else {
        this.feedNews = true;
      }
      this.viewing = news;
      news.completed = true;
      this.showModal = true;
    },

    upvote () {
      var index = this.viewing.upvotes.indexOf(this.user.username);
      var index2 = this.viewing.downvotes.indexOf(this.user.username);
      if (index === -1) {
        this.viewing.upvotes.push(this.user.username);
        if (index2 != -1) {
          this.viewing.downvotes.splice(index2, 1);
        }
      } else {
        this.viewing.upvotes.splice(index, 1);
      }
    },

    downvote () {
      var index = this.viewing.downvotes.indexOf(this.user.username);
      var index2 = this.viewing.upvotes.indexOf(this.user.username);
      if (index === -1) {
        this.viewing.downvotes.push(this.user.username);
        if (index2 != -1) {
          this.viewing.upvotes.splice(index2, 1);
        }
      } else {
        this.viewing.downvotes.splice(index, 1);
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.dashboard {
  padding-top: 2rem;
}

input::placeholder {
  color: grey;
}

label {
  margin-bottom: 0;
  // width: 100%;
  cursor: pointer;
  input[type="checkbox"] {
    margin-right: 5px;
  }
}
.list-group-item {
  &.completed label {
    color: grey;
    text-decoration: none!important;
    }
  .delete {
    color: black!important;
  }
}

.fade-enter-active {
  transition: opacity .5s;
}
.fade-leave-active {
  transition: opacity .25s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
  color: black!important;
}

.modal-container {
  width: 50%;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-default-button {
  float: right;
}

.arrow {
  color:black!important;
  text-decoration:none!important;
}

</style>
