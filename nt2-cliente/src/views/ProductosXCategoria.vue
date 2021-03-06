<template>
  <div class="container">
    <div class="py-5 text-center noImprimir">
      <h2>Realizar Compra</h2>
      <p class="lead noImprimir">
        Por favor agregá tus productos al carrito y realizá la compra
      </p>
    </div>

    <div class="col-12 imprimir">
      <div class="alert alert-success" role="alert">
        Gracias por realizar tu compra! El producto lo retiras en la sucursal
        {{ sucursal.name }} - Direccion: {{ sucursal.address }}
      </div>
    </div>

    <div class="row g-5">
      <div class="col-md-5 col-lg-4 order-md-last">
        <h4 class="d-flex justify-content-between align-items-center mb-3">
          <span class="text">Productos seleccionados:</span>
          <span class="badge bg-dark rounded-pill text-white">{{
            cantidad
          }}</span>
        </h4>
        <ul class="list-group mb-3">
          <div
            v-if="productosAgregados.length == 0"
            class="list-group-item d-flex justify-content-between lh-sm"
          >
            <h6 class="my-0">No hay productos seleccionados</h6>
          </div>
          <div v-else>
            <li
              class="list-group-item d-flex justify-content-between lh-sm"
              v-for="item in productosAgregados"
              v-bind:key="item._id"
            >
              <div>
                <h6 class="my-0">{{ item.name }} ({{ item.cantSelec }})</h6>
                <small class="text-muted">${{ item.precio }} x unidad</small>
              </div>
              <div class="noImprimir">
                <button
                  class="btn"
                  type="button"
                  v-on:click="this.quitarProducto(item)"
                >
                  &#x1F5D1;
                </button>
              </div>
            </li>
            <li class="list-group-item d-flex justify-content-between">
              <span>Total</span>
              <strong>${{ total }}</strong>
            </li>
          </div>
        </ul>
        <div v-if="productosAgregados.length != 0" class="noImprimir">
          <button
            type="button"
            class="btn btn-secondary"
            v-on:click="this.vaciarCarrito()"
          >
            Vaciar carrito &#x1F5D1;
          </button>
        </div>
      </div>

      <div class="col-md-7 col-lg-8 noImprimir">
        <h4 class="mb-3">Agregar productos</h4>
        <div class="row g-3">
          <div class="col-sm-12">
            <label class="form-label">Buscar categoria</label>
            <select
              class="form-control"
              @change="cambiarCategoria"
              v-model="categoriaId"
            >
              <option value="">Elegí una categoría</option>
              <option
                v-for="categ in categorias"
                v-bind:key="categ._id"
                v-bind:value="categ._id"
              >
                {{ categ.name }}
              </option>
            </select>
          </div>

          <div class="col-12">
            <div class="form-group">
              <br />
              <label class="form-label" v-if="productos.length > 0"
                >Elegí un producto</label
              >
              <ul class="list-group mb-12">
                <li
                  class="list-group-item justify-content-between lh-sm"
                  v-for="(prod, index) in productos"
                  v-bind:producto="prod"
                  v-bind:key="index"
                >
                  <div class="row">
                    <div class="col-md-6">
                      <h6 class="my-0">{{ prod.name }}</h6>
                      <small class="text-muted">${{ prod.precio }}</small>
                    </div>
                    <div class="col-md-3">
                      <input
                        v-model="prod.cantSelec"
                        type="number"
                        name="cantidad"
                        class="form-control"
                        minlength="1"
                        maxlength="4"
                        min=0
                        max=500
                        v-on:change="agregarAlCarrito(prod)"
                      />
                    </div>
                  </div>
                </li>
              </ul>
            </div>
          </div>

          <div class="col-12">
            <div class="form-group">
              <label class="form-label">Elegir sucursal</label>
              <select
                class="form-control"
                v-model="sucursalId"
                @change="cambiarSucursal"
              >
                <option value="">Elegí una sucursal</option>
                <option
                  v-for="sucursal in sucursales"
                  v-bind:key="sucursal._id"
                  v-bind:value="sucursal._id"
                >
                  {{ sucursal.name }} - {{ sucursal.address }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <button
          class="w-100 btn btn-success btn-lg"
          type="button"
          v-if="getCarritos.length > 0"
          v-on:click="imprimirTicket()"
        >
          Imprimir ticket
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import srvProducto from "../services/ProductoService.js";
import srvCategoria from "../services/CategoriaProdService.js";
import srvSucursal from "../services/SucursalService.js";

export default {
  name: "ProductosXCategoria",
  data() {
    return {
      cantidad: 0,
      productos: [],
      categoriaId: "",
      sucursalId: "",
      sucursal: { name: "", address: "" },
      categorias: [],
      productosAgregados: [],
      sucursales: [],
      total: 0,
      cantSelec: 1,
    };
  },
  created: async function () {
    try {
      const categ = await srvCategoria.getCategorias();
      this.categorias = categ.data;
      const suc = await srvSucursal.getSucursales();
      this.sucursales = suc.data;
      this.setInfoProductos();
    } catch (err) {
      console.log("No se pudo cargar las categorias " + err.message);
    }
  },
  computed: {
    ...mapGetters(["getCarritos"]),
    ...mapActions(["limpiarCarrito"]),
  },
  methods: {
    async cambiarCategoria() {
      try {
        const prods = await srvProducto.getProductosPorCategoria(
          this.categoriaId
        );
        this.productos = prods.data;
      } catch (err) {
        this.productos = [];
        console.log("No se pudo cambiar la categoría " + err.message);
      }
    },
    setInfoProductos() {
      this.productosAgregados = this.getCarritos;
      this.calcularTotal();
    },
    agregarAlCarrito(producto) {
      if(producto.cantSelec>0){
        this.$store.dispatch("agregarAlCarrito", [producto]).then(() => {
          this.calcularTotal();
        });
      }else{
        this.quitarProducto(producto);
      }
    },
    calcularTotal() {
      this.cantidad = this.productosAgregados.reduce(function (acum, elemento) {
        return (acum += parseInt(elemento.cantSelec));
      }, 0);
      this.total = this.productosAgregados.reduce(function (acum, elemento) {
        return (acum += elemento.precio * parseInt(elemento.cantSelec));
      }, 0);
    },
    quitarProducto(producto) {
      this.$store.dispatch("quitarProducto", [producto]).then(() => {
        this.cantidad -= 1;
        this.total -= producto.precio;
      });
    },
    imprimirTicket() {
      if (this.sucursal.name != "" && this.sucursal.name != null) {
        this.$store.dispatch("limpiarCarrito").then(() => {
          window.print();
          this.setInfoProductos();
        });
      } else {
        alert("Debe elegir una sucursal para finalizar la compra");
      }
    },
    vaciarCarrito() {
      this.$store.dispatch("limpiarCarrito").then(() => {
        this.setInfoProductos();
      });
    },
    async cambiarSucursal() {
      try {
        const suc = await srvSucursal.getSucursalesPor(this.sucursalId);
        this.sucursal = suc.data;
      } catch (err) {
        this.sucursal = "";
        console.log("No se pudo cambiar la sucursal " + err.message);
      }
    },
  },
};
</script>

<style>
body {
  background-color: #edeff1 !important;
}
@media print {
  @page {
    size: landscape;
  }
  div {
    break-inside: avoid;
  }
  .noImprimir {
    display: none;
  }
  .imprimir {
    display: block !important;
  }
}
.imprimir {
  display: none;
}
</style>