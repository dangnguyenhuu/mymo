<template>
  <div class="entrance p-form">
    <div class="entrance_item entrance_logo">
      <p><img src="../assets/img/logo.png" alt="MyMo" class="logo_img"></p>
      <h1 class="c-ttl-primary"><span class="ttl_break">PC、スマホでいつでもメモ！</span><span class="ttl_break">クラウドメモアプリ</span></h1>
    </div><!-- /.entrance_item entrance_logo -->
    <div class="entrance_item entrance_signin">
      <h2 class="c-ttl-secondary">ログイン</h2>
      <label for="c_email" class="p-form_input_wrap">
        <input
          v-model="email"
          id="c_email"
          type="email"
          class="p-form_input"
          placeholder=" ">
        <span class="p-form_input-label">メールアドレス</span>
        <span class="p-form_input-border"></span>
      </label>
      <label for="c_pass" class="p-form_input_wrap">
        <input
          v-model="password"
          id="c_pass"
          type="password"
          class="p-form_input"
          placeholder=" ">
        <span class="p-form_input-label">パスワード</span>
        <span class="p-form_input-border"></span>
      </label>
      <p>
        <button @click="signIn" class="c-btn c-btn-secondary">ログイン</button>
      </p>
      <p>
        <button @click="googleLogin" class="c-btn c-btn-secondary">Googleアカウントでログイン</button>
      </p>
      <p>
        <router-link to="/reset_pass">パスワードをお忘れですか？</router-link>
      </p>
    </div><!-- /.entrance_item entrance_signin -->
    <div class="entrance_item entrance_signup">
      <h2 class="c-ttl-secondary">登録</h2>
      <label for="r_email" class="p-form_input_wrap">
        <input
          v-model="newemail"
          id="r_email"
          type="email"
          class="p-form_input"
          placeholder=" ">
        <span class="p-form_input-label">メールアドレス</span>
        <span class="p-form_input-border"></span>
      </label>
      <label for="r_pass" class="p-form_input_wrap">
        <input
          v-model="newpassword"
          id="r_pass"
          type="password"
          class="p-form_input"
          placeholder=" ">
        <span class="p-form_input-label">パスワード</span>
        <span class="p-form_input-border"></span>
      </label>
      <p>
        <button @click="signUp" class="c-btn c-btn-caution">登録</button>
      </p>
    </div><!-- /.entrance_item entrance_signup -->
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  name: 'Signin',
  data () {
    return {
      email: '',
      password: '',
      newemail: '',
      newpassword: ''
    }
  },
  mounted () {
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        this.$router.push('/')
      }
    })
  },
  methods: {
    signUp () {
      this.$parent.isLoginState = false
      const protocol = location.protocol + '//'
      const redirect = {
        url: protocol + window.location.host + '/#/signin'
      }
      firebase.auth().createUserWithEmailAndPassword(this.newemail, this.newpassword)
        .then(user => {
          const cUser = firebase.auth().currentUser
          // Emailの確認もつけておく
          cUser
            .sendEmailVerification(redirect)
            .then(() => {
              alert(cUser.email + 'に登録確認メールを送信しました。')
            })
            .catch(err => {
              alert(err.code, err.message)
            })
          this.$parent.isLoginState = true
        })
        .catch(err => {
          if (err.code === 'auth/invalid-email') {
            alert('不正なメールアドレスです。')
          } else if (err.code === 'auth/weak-password') {
            alert('パスワードは6文字以上入力してください。')
          } else if (err.code === 'auth/email-already-in-use') {
            alert('入力したメールアドレスは既に登録されています。')
          }
          this.$parent.isLoginState = true
          // alert(err.code + err.message)
        })
    },
    signOut () {
      firebase.auth().signOut().then(() => {
        this.$parent.isLogin = false
        this.$parent.userData = null
        this.$router.push('/signin')
      })
    },
    signIn () {
      firebase.auth().signInWithEmailAndPassword(this.email, this.password)
        .then(user => {
          this.$router.push('/')
        })
        .catch(err => {
          if (err.code === 'auth/invalid-email') {
            alert('不正なメールアドレスです。')
          } else if (err.code === 'auth/wrong-password') {
            alert('パスワードが違います。')
          } else if (err.code === 'auth/user-not-found') {
            alert('登録されていないユーザーです。')
          }
        })
    },
    googleLogin () {
      firebase.auth().signInWithRedirect(new firebase.auth.GoogleAuthProvider())
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
