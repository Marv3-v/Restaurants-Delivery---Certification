<template>
<div>
  <h1 class='title-shop' style='margin-top: 100px; margin-bottom: -50px'>Ordenar</h1>
  <main class="main art-grid">
        <div class="contenedorFlex">
            <article style="cursor: default;" v-for="(prod, index) in carrito" :key="index"  class='cardCart'>
                <div class='card__img'>
                <img :src="require(`../assets/img/americana.png`)">
                </div>
                <div class='card__precis'>
                            <div>
                                <span class="card__preci card__preci--before">{{ prod.nombre }}</span>
                                <span class="card__preci card__preci--now" style="color:  #F7567C">Q.{{prod.precio}}</span>
                            </div>
                {{ prod.cantidad }}
                </div>     
            </article>
        </div>
        <div class="article" v-if="carrito.length <= 0">
          <article style="cursor: default; background:  #E6E7E9; color: #5D576B" class='cardTotal'>
            <div class='card__precis'>
                          <div>
                              <span class="card__preci card__preci--before">PAGO</span>
                              <span class="card__preci card__preci--now" style="color:  #F7567C">No disponible</span>
                          </div>
            <div class="comprar" @click="goTo">
           <span>Restaurantes</span>
            </div>
            </div>     
          </article>
        </div>
        <div v-else class="checkout" >
            <div class="cardCheckout" v-if="!submitted">
                <div class="">
                  <p style="font-size: 13px; opacity: .5">CLIENTE:</p>
                  <p style="font-size: 12px">{{ currentUser.usuario }} - {{ direccion.clientes.nombre }}</p>
                  <p style="font-size: 12px"><b>NIT: </b>{{direccion.clientes.nit }}</p>

                  <hr class="hreClass">
                </div>
                  <p style="font-size: 13px;  opacity: .5">DIRECCIÓN DE ENVÍO:</p>
                <div style="font-size: 12px">
                  <p>{{ direccion.nombre }} - {{ direccion.calle }} - {{ direccion.descripcion }}</p>
                </div>
                  <hr class="hreClass">
                <p style="font-size: 13px; opacity: .5">MÉTODO DE PAGO ÚNICO:</p>
                <div style="font-size: 12px">
                  <p>Pago Contra Entrega</p>
                </div>
                  <hr class="hreClass">
                <p style="font-size: 13px;  opacity: .5">RESUMEN:</p>
                <div style="font-size: 12px">
                  <p>Subtotal: Q.{{ subtotal.toFixed(2) }} </p>
                  <p>Envio: Q.8.00</p>
                  <p>Total: Q.{{ total.toFixed(2) }}</p>
                </div>
          <span class="comprar" style="text-align: center" @click="makeDelivery">COMPRAR</span>
          </div>
          <div v-else>
            <div class="cardCheckout"> 
                <div style="font-size: 12px">
                  <p>Orden Realizada con Éxito:)</p>
            </div>
          </div>
          </div>
        </div>
        
  </main>
<br>
</div>
</template>
<script>
import PedidosDataService from "../services/PedidosDataService";
import LocalidadesDataService from "../services/LocalidadesDataService";

export default {
name: 'Checkout',
 data() {
   return {
     carrito: [],
     currentItem: null,
     total: null,
     subtotal: null,
     direccion: null,
     detalles: { 
     },
     submitted: false
   }
 },
 methods: {
    newTutorial() {
      this.submitted = false;
      this.tutorial = {};
  },
   getCart() {
     this.carrito = this.$store.getters.getCartProducts;
   },
   getLocalidad() {
     LocalidadesDataService.get(this.currentUser.id)
        .then(response => {
          this.direccion = response.data;
          // console.log(response.data);
        })
        .catch(e => {
          console.log(e);
        });  
   },
  addOneMore(producto) {
      this.currentItem = { id: producto.id, nombre: producto.nombre, precio: producto.precio, image: producto.imagen_path};
      // console.log(this.currentProd);
      //Llamamos a la mutacion que incrementara el dato del carrito
      this.$store.dispatch("addToCart", this.currentItem);
      this.getTotal();
    },
  removeByOne(producto) {
      this.currentItem = { id: producto.id};
      this.$store.dispatch("deleteFromCart", this.currentItem);
      this.getTotal();
  },
  removeCompletely(producto) {
    this.currentItem = { id: producto.id}
    this.$store.dispatch("deleteCompletely", this.currentItem);
    this.getTotal();
  },
  goTo() {
    this.$router.push("/restaurantes");
  },
  getTotal() {
    let localT = 0;
      this.$store.getters.getCartProducts.map(el => {
        localT += parseFloat(el["cantidad"]) * parseFloat(el["precio"]);
      }
      );
      this.total = parseFloat(8) +localT;
      // console.log(this.total);
  },
  getSubtotal() {
    this.subtotal = this.total - 8;
  },
  makeDelivery() {

    // console.log(this.myOrder);
// 
    var data = { 
        "total": this.total, 
        "clienteId": this.currentUser.id, 
        "estadoId": 1, 
        "metodoId": 1, 
        "tarifa": 8, 
        "detalles": this.$store.state.cartProducts
    }

    PedidosDataService.create(data)
        .then(response => {
          console.log(response.data);
          this.submitted = true;
          this.$store.state.count = 0;
          this.$store.state.cartProducts = []
        })
        .catch(e => {
          console.log(e);
        });
// 
  }
 }, 
 mounted() {
   this.getCart();
   this.getTotal();
   this.getSubtotal();
   if (!this.currentUser) {
      this.$router.push('/login');
   }
   this.getLocalidad();
 },
 computed: {
    currentUser() {
      return this.$store.state.auth.user;
    },
  },
}
</script>
