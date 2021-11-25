<template>
  <div class="Home">
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item">
            <a class="nav-link" href="javascript:;" @click="byCategory(null)">Home</a>
          </li>
          <template v-for="(item_parent, i) in category_parent">
            <li class="nav-item" :key="i" v-if="getChild(item_parent.id).length == 0">
              <a class="nav-link" href="javascript:;" @click="byCategory(item_parent.slug)">{{item_parent.category_name}}</a>
            </li>
            <li v-else class="nav-item dropdown" :key="i">
              <a class="nav-link dropdown-toggle" href="javascript:;" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                {{item_parent.category_name}}
              </a>
                <div class="dropdown-menu" aria-labelledby="navbarDropdown">
                  <a v-for="(item_child, i) in getChild(item_parent.id)" :key="i" class="dropdown-item" href="javascript:;" @click="byCategory(item_child.slug)">{{item_child.category_name}}</a>
                </div>
            </li>
          </template>
        </ul>
      </div>
    </nav>
    <div class="container-fluid p-5">
        <div class="row">
          <div v-if="loading" class="col-lg-12 text-center">
            <h4>Loading ...</h4>
          </div>
          <div class="col-lg-12">
            <div class="row">
              <div class="col-lg-3 mb-4" v-for="item in items" :key="item.id">
                <div class="card">
                  <img class="img-fluid img-product" :src="item.images[0]" alt="Card image cap">
                  <div class="card-body border-top">
                    <h5 class="card-title">{{item.product_name}}</h5>
                    <div class="row">
                      <div class="col-lg-6 text-danger">
                        <p v-if="item.discount == 0" class="text-success">Rp.{{item.price}}</p>
                        <del v-else>Rp.{{item.price}}</del>
                      </div>
                      <div class="col-lg-6 text-success"><p v-if="item.discount != 0">Rp.{{item.price - item.discount}}</p></div>
                      <div class="col-lg-6"><p>{{item.seller_name}}</p></div>
                      <div class="col-lg-6 text-right"><p>{{item.city_name}}</p></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div v-if="errorBarang" class="col-lg-12 text-center">
            <h4>Barang Tidak Tersedia</h4>
          </div>
          <div class="col-lg-12 text-right mt-4">
            <button @click="prev" :disabled="page == 1" class="btn border mr-3">Previous</button>
            <button @click="next" :disabled="total_page == page" class="btn border">Next</button>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
const axios = require('axios');
export default {
  name: 'Home',
  data(){
    return{
      category_parent: [],
      category_child: [],
      per_page: 8,
      page: 1,
      category: null,
      items: [],
      loading: false,
      prevDisable: true,
      nexDisable: false,
      total_page: null,
      errorBarang: false,
    }
  },
  mounted(){
    this.get_category()
    this.get_product()
  },
  methods:{
    get_category: function(){
      axios.get('http://localhost:8080/znwasn28')
      .then(response => {
        for(let i in response.data.data){
          if(response.data.data[i].status != 0){
             if(response.data.data[i].parent_id == 0){
               this.category_parent.push(response.data.data[i])
             } else {
               this.category_child.push(response.data.data[i])
             }
          }
        }
      })
      .catch(error => {
        console.log(error);
      })
    },
    get_product: function(){
      this.loading = true
      this.errorBarang = false

      axios.get('http://localhost:8080/search-product', {
        params: {
          per_page : this.per_page,
          page: this.page,
          category_name: this.category
        }
      })
      .then(response => {
        this.loading = false
        if(response.data.data.length != 0){
            this.items = response.data.data
            this.total_page = Math.ceil(response.data.total / this.per_page)
        } else {
            this.errorBarang = true
        }
      })
      .catch(error => {
        console.log(error);
      })
    },
    next: function(){
        this.page = this.page + 1;
        this.items = []
        document.body.scrollTop = 0;
        document.documentElement.scrollTop = 0;
        this.get_product();
    },
    prev: function(){
        this.page = this.page - 1;
        this.items = []
        document.body.scrollTop = 0;
        document.documentElement.scrollTop = 0;
        this.get_product();
    },
    getChild:function(data){
      let datas = []
      
      for(let i in this.category_child){
        if(this.category_child[i].parent_id == data){
          datas.push(this.category_child[i]) 
        }
      }
      return datas
    },
    byCategory: function(data){
      this.category = data
      this.items = []
      console.log(this.category)
      this.get_product();
    }
  }
}
</script>
<style scoped>
.img-product{
  height: 300px;
}
.btn{
  cursor: pointer;
}
.btn:disabled {
  background: #dddddd;
}
.navbar-light .navbar-nav .nav-link{
  font-size: 10px;
}
</style>
