
<template>
  <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 p-4">
    <div v-for="producto in carrito" :key="producto.id" class="card rounded-lg card-compact w-full bg-white shadow">
      <figure>
        <div
          v-bind:style="{ 'background-image': 'url(https://http2.mlstatic.com/D_' + producto.thumbnail_id + '-O.webp)' }"
          class="border-b-1 h-56 w-full bg-contain bg-center bg-no-repeat"></div>

      </figure>
      <div class="card-body">
        <NuxtLink :to="`/producto/${producto.id}`">
          <h4 class="line-clamp-2 group-hover:line-clamp-4  hover:underline">{{ producto.title }}</h4>
          <h1>U$D {{ producto.price }}</h1>
        </NuxtLink>
        <div class="card-actions justify-end">
          <button
            @click="addToCart(store.agregarProducto({ id: producto.id, title: producto.title, price: producto.price, thumbnail: producto.thumbnail, quantity: 1, subTotal: producto.price }))"
            class="btn btn-xs ">Añadir a Carrito</button>
        </div>
      </div>

    </div>
  </div>
  <div v-if="pendiente" class="flex justify-center p-8">
        <progress class="progress w-56 "></progress>
    </div>
</template>

<script setup>
import { useCarrito } from '~/stores/carrito';
const store = useCarrito();

import { ref, onMounted } from 'vue';

const offset = ref(0);
const totalResults = ref(0);
const carrito = ref([]);
const pendiente = ref(false);

onMounted(() => {
  cargarProductos();
  store.cargarProductosMl();
  store.cargarCategorias();
  window.addEventListener('scroll', cargarMas);

});

const cargarMas = () => {
  let scrollPosition = window.innerHeight + window.scrollY;
  let pageHeight = document.documentElement.scrollHeight;
  let bottomOfPage = scrollPosition >= pageHeight;

  if (bottomOfPage && !pendiente.value && offset.value + 50 < totalResults.value && offset.value < totalResults.value) {
    offset.value = offset.value + 50;

    cargarProductos();
  }
};

const cargarProductos= () => {
  pendiente.value = true;

  fetch(`https://api.mercadolibre.com/sites/MLV/search?seller_id=96773693&offset=${offset.value}&limit=48`)
    .then((response) => response.json())
    .then((data) => {
      agregarCarrito(data.results);
      pendiente.value = false;
      totalResults.value = data.paging.total;

    });
};

const agregarCarrito = (nuevoscarrito) => {
  nuevoscarrito.forEach((producto) => {
    carrito.value.push(producto);
  });
};
</script>
