<template>
  <div class="main">
    <div class="row">
      <div class="col-md-2 left">
        <div class="row menu">
          <div class="col" style="margin:20px auto">
            <div class="row" style="text-align:center">
             <b>Welcome {{user.name}}</b>
            </div>
            <div class="row" style="margin-top:30px auto;">
            <div v-if="user.role == 'guest'" class="log-form col">

              <label for="exampleInputPassword1">Login Form</label>
              <p><label>Email address:</label></p>
              <input placeholder="Enter email" v-model="login.email" type="text">
              <label for="exampleInputPassword1">Password:</label>
              <input type="password" placeholder="Password" v-model="login.pass">
              
              <p><button @click="signIn" class="login">Log In</button></p>
              <p>{{logMsg}}</p>
              <p><router-link :to="'/register/'">Registration</router-link></p>
            </div>
          
            <div v-else class="col" style="text-align:center">
              <p><a href="#" @click="logOut()">Log Out</a></p>
              <p><a href="#" @click="showOrders()">My Orders</a></p>
              <div v-if="user.role == 'admin'" class="admin">
                  <p><router-link :to="'/admin/'" :role="user.role">Admin Panel</router-link></p>
              </div>
            </div>
            <div v-if="content !==''" class="home">
             <p><a href="/">Home</a></p> 
            </div>
            </div>
          </div>
        </div>

      <div class="row" style="margin: 20px">
        <p>Search By</p>
        Brand:
        <select v-model="filterBrand">
          <option value="">Select Brand</option>
          <option v-for="brand in dropdownBrand" :value="brand">{{brand}}</option>
        </select>
        Color:
        <select v-model="filterColor">
          <option value="">Select Color</option>
          <option v-for="color in dropdownColor" :value="color">{{color}}</option>
        </select>
        Speed:
        <select v-model="filterSpeed">
          <option value="">Select Speed</option>
          <option v-for="speed in dropdownSpeed" :value="speed">{{speed}}</option>
        </select>
        Year:
        <select v-model="filterYear">
          <option value="">Select Year</option>
          <option v-for="year in dropdownYear" :value="year">{{year}}</option>
        </select>
        <p><button @click="refresh()" class="btnn">Clear Filters</button></p>
        <p><button @click="search()" class="btnn">Find</button></p>
        <p>{{searchMsg}}</p>
      </div>
      </div>
      <div class="col-md-10 right">
          <div class="row">
        <div v-if="content == 'orders'">
          <order-section :user="user"></order-section>
        </div>

        <div v-if="content == 'car'">
          <car-section :car="selectedCar" :user="user"></car-section>        
        </div>

      <div v-if="content == ''" class="col">
        <span>{{errMsg}}</span>
          <table class="table" style="margin:20px; background:#fce3c7">
              <thead>
          <tr>
          <td class="col"><b>Model</b></td>
          <td class="col"><b>Brand</b></td></tr>
      </thead>
            <tbody>
          <tr  v-for = "car, key in cars">
          <td class="col"><a href="#" @click="showCar(car)">{{car.model}}</a></td>
          <td class="col">{{car.brand}}</td>
         </tr> 
      </tbody>
            </table>


      </div>
</div>

      </div>
    </div>

  </div>
</template>

<script>

import Order from './Order'
import Car from './Car'
export default {
  name: 'Home',
  data () {
    return {
      content: '',
      selectedCar:'',
      user:{},
      logMsg: '',
      searchMsg:'',
      login:{
        email:'',
        pass:''
      },
      cars: [],
      errMsg: '',
      dropdownYear:[],
      dropdownBrand:[],
      dropdownColor:[],
      dropdownSpeed:[],
      filterBrand:'',
      filterColor:'',
      filterSpeed:'',
      filterYear:'',
    }
  },
  created(){
    this.getUserData()
    this.getCars()
   
  },
    methods:{
      signIn: function(){
      var self = this
      self.errMsg = ''
      self.logMsg = ''
      var xhr = new XMLHttpRequest();
      var json = JSON.stringify({
         login: self.login.email,
         pass: self.login.pass
      });
          xhr.open("PUT", getUrl()+'auth/', true)
          xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');
            xhr.onreadystatechange = function() {
                if (xhr.readyState != 4) return
                  if (xhr.status != 200) {
                        alert(xhr.status + ': ' + xhr.statusText)
                  } else {
                    var res = JSON.parse(xhr.responseText)
                    if(!res)
                    {
                      self.logMsg = 'Wrong login or password'
                    }
                    else
                    {
                      self.user = res[0]
                      localStorage['user'] = JSON.stringify({id: self.user.id, hash: self.user.hash})
                      self.clearInputs()
                    }
                  }
            }
          xhr.send(json)
      },
      // showBookInfo: function(book){
      //   var self = this
      //   self.selectedBook = book
      //   self.content = 'book'
      // },
    getCars: function(){
      var self = this
        var xhr = new XMLHttpRequest()
        xhr.open('GET', getUrl()+'cars/', true)
          xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return
              if (xhr.status != 200) {
                    alert(xhr.status + ': ' + xhr.statusText)
              } else {
                var res = JSON.parse(xhr.responseText)
              if(res){
                self.cars = res
                self.makeDropDowns()
              }else{
                self.errMsg = 'Cars not found'
              }
              }
        }
        xhr.send();
    },
        logOut: function(){
        var self = this
        self.content = ''
        self.errMsg = ''
        self.user = {}
        delete localStorage['user']
        self.getUserData()
      },
      getUserData: function(){
        var self = this
        if(localStorage['user'])
        {
         self.user = JSON.parse(localStorage['user'])
        }
        else
        {
          self.user.name = ''
          self.user.role = 'guest'
          self.user.id = '0'
          self.user.hash = '0'

        }
        self.checkAuth()
      },
      checkAuth: function(){
      var self = this
      var xhr = new XMLHttpRequest();
          xhr.open("GET", getUrl()+'auth/', true)
          xhr.setRequestHeader("Authorization", "Basic " + btoa(self.user.id+":"+self.user.hash));
          xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');
            xhr.onreadystatechange = function() {
                if (xhr.readyState != 4) return
                  if (xhr.status != 200) {
                  } else{
                   var res = JSON.parse(xhr.responseText)
                   if(!res){
                    self.user.name = ''
                    self.user.role = 'guest'
                    self.user.id = '0'
                    self.user.hash = '0'
                   }
                   else{
                      self.getUserInfo()
                   }
                  }
            }
          xhr.send()        
      },
      getUserInfo: function(){
      var self = this
      var xhr = new XMLHttpRequest();
          xhr.open("GET", getUrl()+'users/'+self.user.id, true)
          xhr.setRequestHeader("Authorization", "Basic " + btoa(self.user.id+":"+self.user.hash));
          xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');
            xhr.onreadystatechange = function() {
                if (xhr.readyState != 4) return
                  if (xhr.status != 200) {
                  } else{
                   var res = JSON.parse(xhr.responseText)
                   if(!res){
                    self.user.name = ''
                    self.user.role = 'guest'
                    self.user.id = '0'
                    self.user.hash = '0'
                   }
                   else{
                    self.user = res[0]
                   }
                  }
            }
          xhr.send()  
      },
    clearInputs: function(){
      var self = this
      self.login.email = ''
      self.login.pass = ''
    },

    showOrders: function(){
      var self = this
      self.errMsg = ''
      self.content = 'orders'
    },
    showCar: function(car){
      var self = this
      self.errMsg = ''
      self.selectedCar = car
      self.content = 'car'
    },
    makeDropDowns: function(){
      var self = this

      self.cars.forEach(function(car){
        self.dropdownBrand.push(car.brand)
        self.dropdownYear.push(car.year)
        self.dropdownColor.push(car.color)
        self.dropdownSpeed.push(car.speed)
      })
      self.dropdownBrand = self.dropdownBrand.filter( onlyUnique )
      self.dropdownYear = self.dropdownYear.filter( onlyUnique )
      self.dropdownColor = self.dropdownColor.filter( onlyUnique )
      self.dropdownSpeed = self.dropdownSpeed.filter( onlyUnique )
    },
    search: function(){
      var self = this
      self.searchMsg = ''
      self.errMsg = ''
      self.content = ''
      if(self.filterYear){
        self.cars = {}
        var searchUrl = getUrl()+'cars/year/'+self.filterYear+'/'
        if(self.filterBrand){
          searchUrl +='brand/'+self.filterBrand+'/'
        }
        if(self.filterColor){
          searchUrl +='color/'+self.filterColor+'/'
        }
        if(self.filterSpeed){
          searchUrl +='speed/'+self.filterSpeed+'/'
        }
         var xhr = new XMLHttpRequest()
        xhr.open('GET', searchUrl, true)
          xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return
              if (xhr.status != 200) {
                    alert(xhr.status + ': ' + xhr.statusText)
              } else {
                var res = JSON.parse(xhr.responseText)
              if(res !==false){
                self.cars = res
              }else{
                self.errMsg = 'Cars with these parametres not found'
              }
            }
        }
        xhr.send();

      }else{
        self.searchMsg = 'Year is required parameter to search.'
      }
    },
    refresh:function (){
      var self = this
      self.errMsg = ''
      self.filterSpeed = ''
      self.filterColor = ''
      self.filterBrand = ''
      self.filterYear = ''
      self.getCars()
    }
  },

  computed:{

  },
  components:{
    'order-section': Order,
    'car-section': Car
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}


ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #83512F;
  font-weight: bold;
}


.logged{
  height: 200px;
}

table{
  margin-top: 40px;
}

.left{
  background-color: #fce3c7;
}
.login{
  height: 30px;
  background:#DFCA95;
  color:black;
}

.btnn{
  height: 30px;
  background:#FFDCA8;
  color:black;
}

</style>
