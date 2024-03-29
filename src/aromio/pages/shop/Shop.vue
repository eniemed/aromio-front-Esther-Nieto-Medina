<template>
    <main :class="['fondo', { 'background-dark': comprobarDarkMode() }]">
        <DarkModeBtn />
        <section>
            <article :class="['contenedor-buscador', { 'background-dark': comprobarDarkMode() }]">
                <label for="buscador">Search</label>
                <input v-model="busqueda" id="buscador" type="search" @keyup.enter="search(busqueda)" placeholder="Search...">
                <img class="search-icon" @click="search(busqueda)" src="../../../assets/search-icon.svg" alt="Click here to search">
            </article>
            <article :class="['contenedor-cards', { 'background-dark': comprobarDarkMode() }]">
                <div :class="['card', { 'card-oscuro': comprobarDarkMode() }]" @click="navigationProduct(product.id)"
                    v-for="product in products" :key="product.id">
                    <img class="imagen" :src="product.image" :alt="product.name">
                    <div :class="['circulo', 'circulo-' + product.id]"></div>
                    <p class="nombre">{{ product.flavor_name }}</p>
                    <h3 class="titulo">{{ product.name }}</h3>
                    <span class="precio">${{ product.price }}</span>
                    <div class="contenedor-btn">
                        <button @click.stop="addToCart(product)" class="añadir-carrito">Add to cart ⟶</button>
                        <button @click.stop="addToWishlist(product.id)" class="añadir-favoritos"><img class="favoritos"
                                src="../../../assets/corazon.svg" alt="Add to your wishlist"></button>
                    </div>
                </div>
            </article>
        </section>
    </main>
</template>

<script>
import router from '@/aromio/router/router'
import DarkModeBtn from '@/aromio/shared/DarkModeBtn.vue'
import { darkMode } from '@/aromio/stores/darkMode'
import '@/assets/css/base.css'
import '@/assets/css/darkMode.css'
import { userStore } from '@/aromio/stores/userStore'

export default {
    data() {
        return {
            products: [],
            busqueda: "",
            wishlistNumbers: [],
            carrito: userStore().cart,
            username: userStore().username
        };
    },

    //cuando el componente se monta se recogen los productos y se almacenan en una variable
    mounted() {
        this.getProducts()
    },
    methods: {

        //devuelve true o false para así poder determinar si el user activa o no el modo oscuro
        comprobarDarkMode() {
            const darkModeBtn = darkMode()
            return darkModeBtn.darkMode
        },

        async getProducts() {
            try {
                const res = await fetch("http://localhost:8000/api/products");
                const data = await res.json();
                this.products = data;
            }
            catch (error) {
                console.error(error);
            }
        },

        //lleva a la ruta del producto concreto
        navigationProduct(id) {
            router.push(`/public/${id}`);
        },

        //se asegura de que el input no sea un espacio ni esté en blanco y luego hace la búsqueda
        async search(busqueda) {
            try {
                if (busqueda.trim() === "") {
                    return this.getProducts();
                }
                const res = await fetch(`http://localhost:8000/api/products/search?dato=${busqueda}`);
                const data = await res.json();
                this.products = data;
            }
            catch (error) {
                console.error(error);
            }
        },

        //busca en el campo de wishlist del usuario en concreto e introduce los números en un array para su facil manejo (ya que la respuesta es una cadena)
        async addToWishlist(productId) {
            try {
                const response = await fetch(`http://localhost:8000/api/wishlist/${this.username}/add/${productId}`, {
                    method: 'PUT',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                });
                if (!response.ok) {
                    console.error('Error al agregar el producto a la wishlist:', response.statusText);
                    return;
                }
                this.wishlistNumbers.push(productId);
            }
            catch (error) {
                console.error('Error al agregar el producto a la wishlist:', error);
            }
        },

        //función añadir al carrito
        addToCart(product) {

            //comprueba si el producto está o no en el carrito comparando sus ids
            const index = this.carrito.findIndex((item) => item.id === product.id);

            //si está (es decir, findIndex NO devuelve un -1), se aumenta la cantidad del mismo en vez de añadirlo de nuevo
            if (index !== -1) {
                this.carrito[index].cantidad += 1;
            }

            //si NO está, lo mete en el carrito con sus datos
            else {
                this.carrito.push({
                    id: product.id,
                    name: product.name,
                    cantidad: 1,
                    image: product.image,
                });
            }
            const user = userStore()
            user.setCart(this.carrito)
        },
    },
    components: { DarkModeBtn }
}
</script>

<style scoped>

label {
    color: transparent;
    user-select: none;
}
.contenedor-buscador {
    display: flex;
    position: relative;
    justify-content: center;
    padding-top: 10vh;
    align-items: center;
}

.añadir-favoritos {
    border: 0;
    background-color: transparent;
    cursor: pointer;
}

.search-icon {
    height: 2.5vh;
    position: absolute;
    background-color: white;
    margin-left: 29vh;
    cursor: pointer;
}

.contenedor-buscador input {
    height: 4.8vh;
    width: 30vh;
    border-radius: 30px;
    border: 0;
    background-color: white;
    filter: drop-shadow(-4px 5px 7px rgba(0, 0, 0, 0.25));
    text-indent: 16px;
}

.contenedor-buscador input::placeholder {
    font-family: 'Alata', sans-serif;
    font-size: 1.5vh;
    text-indent: 16px;
    color: rgb(181, 181, 181);
    letter-spacing: 0.5px;
}

.contenedor-buscador input:focus {
    outline: 0;
}

.contenedor-buscador input:focus::placeholder {
    color: transparent;
}

.fondo {
    display: flex;
    flex-direction: column;
}

.favoritos {
    margin-right: 8px;
    transition: 0.3s;
    z-index: 3;
}

.favoritos:hover {
    transform: scale(1.3);
}

.añadir-carrito:hover {
    transform: scale(1.1);
    background-color: transparent;
}

.circulo {
    position: absolute;
    width: 150px;
    height: 150px;
    border-radius: 50%;
    z-index: -1;
    top: 0;
    margin-left: 58px;
    transition: all 0.3s;
}

.card:hover .circulo {
    transform: translateY(15px);
}

.card {
    position: relative;
    background-color: white;
    width: 290px;
    height: 350px;
    margin-bottom: 200px;
    text-align: center;
    border-radius: 30px;
    padding: 10px;
    filter: drop-shadow(-4px 10px 10px rgba(72, 72, 72, 0.5));
    cursor: pointer;
    transition: all 0.3s;
}

.card:hover {
    filter: drop-shadow(-10px 12px 10px rgba(72, 72, 72, 0.7));
    transform: scale(1.1);
}

.card:hover .imagen {
    transform: translateY(-15px);
}

.contenedor-cards {
    padding: 30vh;
    padding-top: 20vh;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-evenly;
}

.nombre {
    color: rgb(136, 136, 136);
}

.titulo {
    color: rgb(73, 30, 18);
}

.imagen {
    background-color: transparent;
    width: 100%;
    height: 35vh;
    object-fit: cover;
    border-radius: 8px;
    display: block;
    margin: -150px 0 auto;
    filter: drop-shadow(-4px 10px 10px rgba(72, 72, 72, 0.5));
    transition: all 0.3s;
}

.nombre,
.titulo,
.precio {
    font-family: 'Alata', sans-serif;
    margin: 10px 0;
    background-color: transparent;
}

.precio {
    font-family: 'Alata', sans-serif;
    margin: 10px 0;
    background-color: white;
    margin-top: 20px;
    color: rgb(109, 109, 109);
}

.añadir-carrito,
.favoritos {
    padding: 8px;
    cursor: pointer;
    border: none;
    background-color: white;
    border-radius: 4px;
    margin-top: 8px;
}

.contenedor-btn {
    background-color: white;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-top: 10px;
}

.añadir-carrito {
    font-family: 'Alata', sans-serif;
    padding: 8px;
    cursor: pointer;
    border: none;
    background-color: white;
    border-radius: 4px;
    color: rgb(131, 73, 49);
    transition: .3s ease all;
    margin-left: 10px;
}

@media screen and (max-width: 800px) {

    .card {
        background-color: white;
        width: 290px;
        height: 350px;
        margin-bottom: 150px;
        position: relative;
        z-index: 1;
    }

    .contenedor-cards {
        padding: 0px;
        padding-top: 200px;
    }

    .nombre,
    .precio,
    .titulo,
    .favoritos,
    .añadir-carrito,
    .contenedor-btn {
        background-color: transparent;
        position: relative;
        z-index: 2;
    }
}
</style>
