<template>
    <div class="text-bg-dark" style="height: 100vh;">
    <div class="container-fluid" style="position: relative;">
      <div class="row">
        <div class="col-lg-2">
        <!--side nav-->
        <aside>
          <p> Admin Account </p>
          <router-link to="/admin/dashboard" class="activeBtn">
            <i class="bi bi-speedometer"></i>
              Dashboard
          </router-link>
          <button id="current" type="button" disabled>
            <i class="bi bi-clipboard-check-fill"></i>
              Registrations
          </button>
          <router-link to="/login" class="activeBtn">
            <i class="bi bi-telephone-plus-fill"></i>
            Manage Accounts
          </router-link>
          <router-link to="/login" class="activeBtn">
            <i class="bi bi-telephone-plus-fill"></i>
              Active Posts
          </router-link>
          <router-link to="/login" class="activeBtn">
            <i class="bi bi-telephone-plus-fill"></i>
              Reports
          </router-link>
        </aside>
        <!--end of side nav-->  
      </div>
      <div class="col-lg-10 pt-5">
        <h4>Incoming Tutor Center Registration Requests</h4>
        <table id="table" class="table table-responsive bg-light">
        <thead>
          <tr>
            <th>Tutor Center Name</th>
            <th>Address</th>
            <th>Contact Number</th>
            <th>Email</th>
            <th>Business Permit</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tc of tcList">
            <td>{{ tc.name }}</td>
            <td>{{ tc.address }}</td>
            <td>{{ tc.contactNumber }}</td>
            <td>{{ tc.email }}</td>
            <td>businesspermit.jpeg</td>
            <td>
              <i class="bi bi-check-circle-fill text-primary buttonIcon"
              @click="approveClicked(tc)"> Approve</i>
            </td>
            <td>
              <i class="bi bi-x-circle-fill text-danger buttonIcon"
              @click="denyClicked(tc)"> Deny</i>
            </td>
          </tr>
        </tbody>
      </table>
      </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { auth } from '../../firebase';
import { db } from '../../firebase';
import router from '../../router';
import { doc, setDoc, getDoc } from "firebase/firestore";
import { getAuth, createUserWithEmailAndPassword } from "firebase/auth";
import { collection, getDocs, updateDoc , deleteDoc} from "firebase/firestore"; 

  export default{
    data(){
      return {
        currentUser: '',
        tcList: [],
      }
    },
    // setup(){
    //   const user = ref(null);
    //   const db = getFirestore();
    //   const usersList = ref([]);

    //   onMounted(async () => {
    //     const itemsCollection = collection(db, 'all_users/admin');
    //     try {
    //       const querySnapshot = await getDoc(itemsCollection);
    //       usersList.value = querySnapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
    //     } catch (error) {
    //       console.error('Error fetching data from Firestore:', error.message);
    //     }
    //     user.value = auth.currentUser;
    //     console.log(user.value)
    //   })
    //   return user;
    // },
    methods:{
      async loadTable(){
        
        this.tcList = [];

        const querySnapshot = await getDocs(collection(db, "/pending_register/users/tutor_center"));
        querySnapshot.forEach((doc) => {
          this.tcList.push(doc.data());
          console.log(this.tcList)
        });
      },
      async approveClicked(tc){
        await setDoc(doc(db, "all_users", "tutor_center", "users", tc.email), {
          uuid: '',
          email: tc.email,
          name: tc.name,
          address: tc.address,
          contactNumber: tc.contactNumber,
          password: tc.password,
          businessPermitURL: '',
        });
        // this block creates an auth meaning it makes the actual account!!
        // this should only execute once the admin has approved the register
        const auth = getAuth();
        createUserWithEmailAndPassword(auth, tc.email, tc.password)

        .then(async (userCredential) => {
          // Signed up
          const user = userCredential.user;
          console.log(user);

          await deleteDoc(doc(db, "pending_register", "users", "tutor_center", tc.email));
        }).catch((error) => {
          const errorCode = error.code;
          const errorMessage = error.message;
          alert(error.message);
        });

        this.loadTable();
      },
      async denyClicked(tc){
        await deleteDoc(doc(db, "pending_register", "users", "tutor_center", tc.email));
        
        this.loadTable();
      },
    },
    async created(){
      const user = ref(null);
      user.value = auth.currentUser;
      this.currentUser = user.value;  

      const querySnapshot = await getDocs(collection(db, "/pending_register/users/tutor_center"));
      querySnapshot.forEach((doc) => {
        this.tcList.push(doc.data());
        console.log(this.tcList)
      });
    }
  };
</script>

<style scoped>
  @import "bootstrap/dist/css/bootstrap.css";
  .buttonIcon {
    cursor: pointer;
  }
  
  aside {
    color: #fff;
    width: 200px;
    padding-left: 20px;
    height: 100vh;
    background:  #006df3;
    border-top-right-radius: 40px;
  }

  aside a {
    font-size: 12px;
    color: #fff;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    -webkit-tap-highlight-color:transparent;
  }

  aside .activeBtn{
    font-size: 12px;
    color: #fff;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    -webkit-tap-highlight-color:transparent;
    border: 0;
    text-decoration: none;
    background:  #006df3;
    width: 100%;
    text-align: left;
    transition: 0.2s ease;
  }

  aside .activeBtn:hover {
    color: #3f5efb;
    background: #fff;
    outline: none;
    position: relative;
    background-color: #fff;
    border-top-left-radius: 20px;
    border-bottom-left-radius: 20px;
  }

  aside .activeBtn i, aside button i {
    margin-right: 5px;

  }

  aside .activeBtn:hover::after {
    content: "";
    position: absolute;
    background-color: transparent;
    bottom: 100%;
    right: 0;
    height: 35px;
    width: 35px;
    border-bottom-right-radius: 18px;
    box-shadow: 0 20px 0 0 #fff;
  }

  aside .activeBtn:hover::before {
    content: "";
    position: absolute;
    background-color: transparent;
    top: 38px;
    right: 0;
    height: 35px;
    width: 35px;
    border-top-right-radius: 18px;
    box-shadow: 0 -20px 0 0 #fff;
  }

  aside button[disabled] {
    font-size: 12px;
    color: #fff;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    border-top-left-radius: 20px;
    border-bottom-left-radius: 20px;
    -webkit-tap-highlight-color:transparent;
    border: 0;
    text-decoration: none;
    background:   #334dbe;
    width: 100%;
    text-align: left;
    transition: 0.2s ease;
  }

  aside p {
    margin: 0;
    padding: 40px 0;
  }
</style>