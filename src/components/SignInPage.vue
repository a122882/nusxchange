<template>
  <div class="home">
    <img
      src="https://content.presspage.com/uploads/2580/1920_utown.jpg?10000"
      class="image"
    />
    <div class="login">
      <img style="width: 80px; height: 80px" src="../assets/nusxchange.png" />
      <h2>Login</h2>
      <div style="width: 70%">
        <v-text-field v-model="email" label="Email" type="email" />
        <v-text-field v-model="password" label="Password" type="password" />
        <v-btn class="button" block @click="Login">Login</v-btn>
        <p v-if="invalidSignin" style="color: red; text-align:center">Invalid Email/Password</p>
        <p v-if='emailNotVerify' style="color: red; text-align:center">Please verify your email</p>
      </div>
      <p style="margin-top:5% ; text-align:center">or</p>
      <div style="width: 100%;" id="firebaseui-auth-container"/>
      
      <div class="error-handling">
        <router-link class="links" to="/password-reset"
          >Forgot your password?</router-link
        >
        <div
          style="
            display: flex;
            align-content: center;
            justify-content: space-between;
          "
        >
          <h3>Don't have an account?</h3>
          <router-link class="links" to="/register">Register Here</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { signInWithEmailAndPassword } from "firebase/auth";
import { auth } from "@/firebase";

import firebase from '@/uifire.js'
import 'firebase/compat/auth'
import * as firebaseui from 'firebaseui'
import 'firebaseui/dist/firebaseui.css'
import {getFirestore} from 'firebase/firestore';
import {firebaseApp} from '../firebase.js';
import { doc, getDoc} from 'firebase/firestore';
const db = getFirestore(firebaseApp);
export default {
  data() {
    return {
      email: "",
      password: "",
      invalidSignin: false,
      emailNotVerify: false,
    };
  },
   mounted() {
        var ui = firebaseui.auth.AuthUI.getInstance()
        if (!ui) {
            ui = new firebaseui.auth.AuthUI(firebase.auth())
        }
   this.googleSignin().then(page => {
    var uiConfig = {
      signInSuccessUrl: page,
      signInOptions: [
        firebase.auth.GoogleAuthProvider.PROVIDER_ID
      ]
    };
    ui.start("#firebaseui-auth-container", uiConfig)
  }).catch(error => {
    console.log("Error checking user: ", error);
  });
  const style = document.createElement('style');
  document.head.appendChild(style);


    },
  methods: {
  googleSignin() {
    return new Promise((resolve, reject) => {
      firebase.auth().onAuthStateChanged(user => {
        if (user) {
          const usersRef = doc(db, "Account", user.uid);
          getDoc(usersRef).then(querySnapshot => {
            if (querySnapshot.exists()) {
              const userDoc = querySnapshot.data();
              console.log("User exists: ", userDoc);
              resolve("/signin/account-management-page");
            } else {
              console.log("User does not exist");
              resolve("/register-google");
            }
          }).catch(error => {
            console.log("Error getting documents: ", error);
            reject(error);
          });
        } else {
          resolve("/signin");
        }
      });
    });
  },
  
    async Login() {
    try {
      const userCredential = await signInWithEmailAndPassword(
        auth,
        this.email,
        this.password
      );
      const user = userCredential.user;
      if (user.emailVerified) {
        console.log("User logged in:", user);
      this.$router.push("/signin/account-management-page");
      } else {
        console.error("User email not verified")
        this.emailNotVerify = true
        this.invalidSignin = false 
      }
       // Redirect to the desired page after login
    } catch (error) {
      console.error("Login error:", error);
     this.emailNotVerify = false  
      this.invalidSignin = true

      // Handle errors here, such as displaying an error message to the user
    }

    }
  },
};
</script>

<style scoped>
.home {
  display: flex;
  flex-flow: row nowrap;
}
.home .image {
  width: 70%;
  height: 100%;
  border-radius: 10px;
}
.home .login {
  width: 100%;
  display: flex;
  flex-flow: column nowrap;
  justify-content: space-around;
  align-items: center;
  margin-top: 3%;
  margin-bottom: 10%;
}
.home .login .button {
  background-color: var(--primary);
  color: white;
}
.home .error-handling {
  justify-content: center;
  font-size: 14px;
  width: 60%;
  margin-top: 5%;
  width: 70%;
}
.home .error-handling .links {
  color: var(--secondary);
  align-self: center;
  font-weight: bold;
}
</style>
