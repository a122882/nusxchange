<template>
<v-dialog v-model="showErrorModal" max-width="700" style= "  margin: auto; display: flex; flex-direction: column;justify-content: center;height: 100%;">
  <v-card>
    <v-card-title>Error</v-card-title>
    <v-card-text>
      <div v-if="!(this.userName && this.major)">Ensure that all required fields are filed.</div>

    </v-card-text> 
    <v-card-actions >
      <v-btn color="primary" @click="showErrorModal = false, errorMessage = ''" >OK</v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
  <div class="register">
    <img id='image' src="../assets/register.png" alt="">
  <div class="" >
    <h2 style="transform: translateX(65%)">Register</h2>
    <form id="form">
    <v-text-field v-model="userName" label="Username *" type="text" required/>
    <v-autocomplete v-model="major" label="Select Your Major *" :items="majorOption"> 

      <v-list>
        <v-list-item v-for="(option, index) in majorOption" :key="index" @click="major = option">
          <v-list-item-title>{{ option }}</v-list-item-title>
        </v-list-item>
      </v-list>

    </v-autocomplete>

    <v-text-field v-model="tele" label="Telegram Handle" type="text" />
    <v-autocomplete v-model="exchangeUni" label="Exchange University" :items="uniOption"> 

      <v-list>
        <v-list-item v-for="(option, index) in uniOption" :key="index" @click="exchangeUni = option">
          <v-list-item-title>{{ option }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-autocomplete>
    <v-autocomplete v-model="semester" label="Semester" :items="semesterOption"> 

      <v-list>
        <v-list-item v-for="(option, index) in semesterOption" :key="index" @click="semester = option">
          <v-list-item-title>{{ option }}</v-list-item-title>
        </v-list-item>
      </v-list>

    </v-autocomplete>

    <v-btn class="button" block @click="register" style="transform: translateX(60%)">Register</v-btn>
    </form>
  </div>
</div>
</template>

<script>
import '@mdi/font/css/materialdesignicons.min.css';
import {auth, firebaseApp} from '../firebase.js';
import {getFirestore} from 'firebase/firestore'
import {doc, setDoc, getDocs, collection} from 'firebase/firestore';
import { onAuthStateChanged } from "firebase/auth";

const db = getFirestore(firebaseApp);


export default {
    name: "RegisterPage",

    async mounted() {
        const universityList = await getDocs(collection(db, 'ListOfUniversities'))
        universityList.forEach((docs) => {
          this.uniOption.push(docs.id)
        })
        const year = new Date().getFullYear() % 100
        const year2 = year + 1
        const year3 = year + 2
        this.semesterOption.push("Semester 1"+ ", "+year+"/"+ year2)
        this.semesterOption.push("Semester 2"+ ", " +year+"/"+ year2)
        this.semesterOption.push("Semester 1"+ ", " +year2+"/"+ year3)
        this.semesterOption.push("Semester 2"+ ", " +year2+"/"+ year3  )
        onAuthStateChanged(auth, (user) => {
            if (user) {
                this.user = user
            }
        })
    },
    data() {
      return {
        majorOption: ['english', 'math'],
        userName: "",
        name: "",
        major: "",
        tele: "",
        exchangeUni: "",
        semester: "",
        uniOption: ['None'],
        semesterOption: ["None"],
        errorMessage: "",
        showErrorModal: false,
        user: null,
      }
    }, 


    methods: {

    async register() {
        if (this.formIsValid) {


            // Navigate to the account management page
            await this.addAccount(this.user)
            this.$router.push("/signin/account-management-page");
            document.getElementById('form').reset()
        } 
        else {
          this.showErrorModal = true;
        }
    },

      async addAccount() {
      try {
        const docRef = await setDoc(doc(db, "Account", this.user.uid), {
          username: this.userName,
          email: null,
          major: this.major,
          password: false,
          telegram: this.tele ? this.tele : "",
          exchangeUniversity: this.exchangeUni ? this.exchangeUni : "None",
          favouriteUniversity: [],
          semester: this.semester ? this.semester : "None",
          photo: false,
          })
          console.log(docRef)
      }
      catch(error) {
        console.error("Error adding document", error)
        this.showErrorModal = true
      }
      },
    },

    computed: {
      formIsValid() {
        return (
          this.userName &&
          this.majorOption.includes(this.major) &&
          (!this.exchangeUni || this.uniOption.includes(this.exchangeUni)) &&
          (!this.semester || this.semesterOption.includes(this.semester))
        );
      },
    },
    }

</script>

<style scoped>

.register {
  display: flex;
  flex-flow: row nowrap;
}
#image {
  width: 100%;
  height: 100%;
  margin-right: 10%;
}

.button {
  background-color: var(--primary);
  color: white;
}
.register {
  justify-content: center;
  font-size: 14px;
  width: 80%;
}


.v-text-field {
  color: var(--secondary);
  width: 250%;
  height: 8%;
  margin-bottom: -20%;
}

/*#inner-v {
  position: absolute;
  top: 52%;
  margin-left: 16%;
  border: none;
  cursor: pointer;
}*/
.register {
  align-self: center;
  font-weight: bold;
}
.register h3, .register .button {
  text-align: center;
  width: 100%;
}

.input-container {
  display: flex;
  align-items: center;
}

#inner-v {
  margin-left: 5px;
}

.small {
  width: 10px;
  height: 10px; 
}

</style>