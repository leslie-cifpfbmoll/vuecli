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
          <button v-on:click="createComanda()">Crear comanda</button>
        </div>
      </div>
      <div class="col-md-5 col-12">
         <div>
    {{ today }}
  </div>
        <h2>Comandas</h2>
  
        <div id="response">{{mostrarComanda()}}</div>.
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
      today:moment().format("DD/MM/YYY HH:mm")
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
          date:this.today

        }),
        headers: {
          "Content-type": "application/json; charset=UTF-8"
        }
      })
        .then(response => response.json())
        .then(json => console.log(json));
        this.mostrarComanda();
    },
    mostrarComanda: function() {
      var text = "";
      var total=0;
      fetch("http://localhost:3000/comandas")
        .then(response => response.json())
        .then(function(json) {
          text += "<div>";
         
          for (var i = 0; i < json.length; i++) {
            text += "<p>Comanda " + json[i].id+" - "+ json[i].date + " <button>Borrar</button><button>Editar</button></p><ul>";
             total=0;
            for (var x = 0; x < json[i].productos.length; x++) {
              text += "<li> Producto: " + json[i].productos[x].nombre;
              text += " Precio:  " + json[i].productos[x].precio;
              text += "x :  " + json[i].productos[x].unidad + " = ";
              text +=
                json[i].productos[x].precio * json[i].productos[x].unidad +
                "</li>";
                total+=  json[i].productos[x].precio * json[i].productos[x].unidad
            }
            text += "</ul><p>Total de la comanda: "+total+"</p>";
          }

          text += "</div>";
       
           document.getElementById("response").innerHTML = text;
        
        });
     
    }
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
.list {
  border: solid 1px #2c3e50;
  padding: 15px;
}
</style>
