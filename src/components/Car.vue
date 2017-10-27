<template>
  <div class="book row" style="margin:20px">
    <div class="col" style="text-align:center">
      <h3>Book Details</h3>
      <div class="info">
        <h4>{{car.model}}</h4>
        <p>{{car.brand}}</p>
        <p>Motor: {{car.motor}} </span></p>
        <p>Color: {{car.color}} </span></p>
        <p>Price: {{car.price}} $</p>

        <div v-if="user.role != 'guest'" class="add">

          <div class="form">
            <select v-model="payment">
              <option value="">Select Pay method</option>
              <option v-for="opt in dropDown" :value="opt.value">{{opt.title}}</option>
            </select>
          <button @click="makeOrder()" class="add-to-cart btnn">Make pre-order</button>
          <p>{{msg}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Book',
  props: ['car', 'user'],
  data () {
    return {
      msg: '',
      dropDown:[
         {
          value: "cash",
          title: "Cash"
        },
        {
          value: "btc",
          title: "Bit Coin"
        },
        {
          value: "webmoney",
          title: "Web Money"
        },
      ],
      payment: '',
      // isFormActive: false
    }
  },
  created(){
  },
  methods:{
    makeOrder: function(){
      var self = this
      if(self.payment){
      var xhr = new XMLHttpRequest();
      var json = JSON.stringify({
         id_car: self.car.id,
         payment: self.payment,
         id_user: self.user.id
      });

          xhr.open("POST", getUrl()+'orders/', true)
          xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');
            xhr.onreadystatechange = function() {
                if (xhr.readyState != 4) return
                  if (xhr.status != 200) {
                        alert(xhr.status + ': ' + xhr.statusText)
                  } else {
                    var res = JSON.parse(xhr.responseText)
                    if(!res){
                      self.msg = 'Something wrong'
                    }
                    else{
                      self.msg = 'Thank You! Order added.'
                      self.payment = ''
                    }
                  }
            }
          xhr.send(json)
       }else{
         self.msg = 'Choose pay method please.'
       }
    }

  }



}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

  .book{
  background-color: #fce3c7;

  }

td{
  padding:20px;
}
th{
  padding:20px;
}

table{
  margin-top: 40px;
}

.info{
  width: 40%;
  margin: auto;
}

a {
  color: #42b983;
}

.btnn{
  height: 30px;
  background:#FFDCA8;
  color:black;
}
</style>
