<template>
  <div id="body" class="col-md-12">
    <div class="row">
      <div class="col-md-5 col-12">
        <div class="crear">
          <h1>Crear comanda</h1>
          <p>
            Producto:
            <input v-model="nombre" />
          </p>
          <p>
            Precio:
            <input v-model="precio" type="number" />
            <button v-on:click="agregar()">Agregar</button>
          </p>
        </div>
        <div v-if="productos.length>0" class="list">
          <ul v-bind:class="item">
            <li v-for="(producto, index) in productos" :key="producto.id">
              <span>{{ producto.nombre }}</span>
              <span>{{ producto.precio }}€ x</span>
              <input v-model="producto.unidad" type="number" />
              <span>uds. = {{ totalProducto(index) }}</span>
              <button v-on:click="borrar(index)">borrar</button>
            </li>
          </ul>
          <h4>Total: {{ totalProductos }} €</h4>
          <button v-if="update === ''" v-on:click="createComanda()">Crear comanda</button>
          <button v-else v-on:click="updateComanda(idcomanda)">Actualizar comanda</button>
        </div>
      </div>
      <div class="col-md-5 col-12">
        <div></div>
        <h2>Comandas</h2>

        <div class="comanda" v-for="comanda in comandas" :key="comanda.id">
          <span>Comanda: {{ comanda.id }} - {{ comanda.date }}</span>
          <button v-on:click="deleteComanda(comanda.id)">Borrar</button>
          <button v-on:click=" idComanda(comanda.id)">Actualizar</button>
          <ul>
            <li v-for="producto in comanda.productos" :key="producto.id">
              <span>{{ producto.nombre }}</span>
              <span>{{ producto.precio }}€ x</span>
              <span>{{ producto.unidad }}</span>
              <span>uds. = {{ producto.unidad * producto.precio }}</span>
            </li>
          </ul>
          <h4>Total: {{ comanda.total }}</h4>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as moment from "moment/moment";
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      item: "item",
      productos: [],
      nombre: "",
      precio: "",
      today: moment().format("DD/MM/YYY HH:mm"),
      comandas: [],
      total: 0,
      update: "",
      idcomanda: 0
    };
  },
  methods: {
    agregar: function() {
      this.productos.push({
        nombre: this.nombre,
        precio: this.precio,
        unidad: 1
      });
     
     
    },
    borrar: function(index) {
      this.productos.splice(index, 1);
    },
    totalProducto: function(index) {
      return this.productos[index].precio * this.productos[index].unidad;
    },

    deleteComanda: function(index) {
      var myurl = "http://localhost:3000/comandas";
      fetch(myurl + "/" + index, {
        method: "DELETE"
      }).then(this.mostrarComanda);
    },
    createComanda: function() {
      var total = [];
      for (var i = 0; i < this.productos.length; i++) {
        total.push({
          nombre: this.productos[i].nombre,
          precio: this.productos[i].precio,
          unidad: this.productos[i].unidad
        });
      }

      fetch("http://localhost:3000/comandas", {
        method: "POST",
        body: JSON.stringify({
          productos: total,
          date: this.today,
          total: this.totalProductos
        }),
        headers: {
          "Content-type": "application/json; charset=UTF-8"
        }
      })
        .then(response => response.json())
        .then(json => console.log(json))
        .then(this.mostrarComanda);

      this.productos = [];
    },
    mostrarComanda: function() {
      let vm = this;

      fetch("http://localhost:3000/comandas")
        .then(response => response.json())
        .then(function(json) {
          vm.comandas = json;
        });
    },
    updateComanda: function(index) {
      var total = [];
      for (var i = 0; i < this.productos.length; i++) {
        total.push({
          nombre: this.productos[i].nombre,
          precio: this.productos[i].precio,
          unidad: this.productos[i].unidad
        });
      }
      fetch("http://localhost:3000/comandas/" + index, {
        method: "PUT",
        body: JSON.stringify({
          productos: total,
          date: this.today,
          total: this.totalProductos
        }),
        headers: {
          "Content-type": "application/json; charset=UTF-8"
        }
      })
        .then(response => response.json())
        .then(json => console.log(json))
        .then((this.productos = [])).then(this.mostrarComanda).then(this.update="");
    },
    idComanda: function(index) {
      let vm = this;
      vm.update = "si";
      fetch("http://localhost:3000/comandas/" + index)
        .then(response => response.json())
        .then(function(json) {
          vm.productos = json.productos;
          vm.idcomanda = json.id;
        });
    }
  },
  mounted() {
    this.mostrarComanda();
  },
  computed: {
    totalProductos: function() {
      var total = 0;
      for (var i = 0; i < this.productos.length; i++) {
        total += this.totalProducto(i);
      }
      return total;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}

a {
  color: #42b983;
}
.item li input {
  width: 35px;
}
.item li {
  padding-bottom: 10px;
}
span {
  margin: 5px;
}

.comanda {
  border: solid 1px #2c3e50;
  padding: 15px;
}
</style>
