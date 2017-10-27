<template>
  <div class="hello">
          <table class="table" style="margin:20px; background:#fce3c7">
              <thead>
          <tr>
          <td class="col"><b>Model</b></td>
          <td class="col"><b>Brand</b></td>
          <td class="col"><b>Year</b></td>
          <td class="col"><b>Payment</b></td>
          <td class="col"><b>Status</b></td>
          </tr>
      </thead>
            <tbody>
          <tr  v-for = "order, key in orders">
          <td class="col">{{order.model}}</td>
          <td class="col">{{order.brand}}</td>
          <td class="col">{{order.year}}</td>
          <td class="col">{{order.payment}}</td>
          <td class="col">{{order.status}}</td>
         </tr> 
      </tbody>
            </table>
  </div>
</template>

<script>
export default {
  name: 'orders',
  props:['user'],
  data () {
    return {
      orders: []
    }
  },
  created(){
    this.getOrders()
  },
  methods:{
      getOrders: function(){
      var self = this
        var xhr = new XMLHttpRequest()
        xhr.open('GET', getUrl()+'orders/'+self.user.id, true)
        xhr.setRequestHeader("Authorization", "Basic " + btoa(self.user.id+":"+self.user.hash));        

          xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return
              if (xhr.status != 200) {
                    alert(xhr.status + ': ' + xhr.statusText)
              } else {
                var res = JSON.parse(xhr.responseText)
              if(res){
                self.orders = res
              }else{
                self.errMsg = 'No Orders'
              }
            }
        }
      xhr.send();
    },
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
  color: #42b983;
}
</style>
