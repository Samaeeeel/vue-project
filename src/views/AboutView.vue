<script setup lang="ts"> 
    import { ref, onMounted, watch } from 'vue'
    import { brotliDecompressSync } from 'zlib'

    interface Producto{
        idProducto?:number
        prodNombre: string
        prodCategoria: string
        prodPrecio: Number
        prodStock: Number
        prodImg: string[]
    }

    const productos = ref<Producto[]>([])
    const mostrarForm = ref(false)
    const mensaje = ref('')

    const form = ref<Producto>({
        prodNombre: '',
        prodCategoria: '',
        prodPrecio: 0,
        prodStock: 1,
        prodImg: []
    })

    const imagenesInput = ref('')

    const fetchProductos = async () => {
        try{
            const res = await fetch('https://backendpawstails.runasp.net/api/gestion/productos')
            productos.value = await res.json()
        } catch {
            alert('Error al cargar productos')
        }
    }

    const abrirCrear = () => {
        form.value = { prodNombre: '', prodCategoria: '', prodPrecio: 0, prodStock: 1, prodImg: [] }
        imagenesInput.value = ''
        mostrarForm.value = true
    }

    const editar = (p: Producto) => {
        form.value = { ...p, prodImg:[...(p.prodImg || [])]}
        imagenesInput.value = form.value.prodImg.join('\n')
        mostrarForm.value = true
    }

    const eliminar = async (id:number) => {
        if(confirm('Eliminar Producto?')) return
        await fetch(`https://backendpawstails.runasp.net/api/gestion/productos/${id}`, {method: 'DELETE'})
        fetchProductos()
    }

    const guardar = async () => {
        form.value.prodImg = imagenesInput.value.split('\n').map((i) => i.trim()).filter(Boolean)

        const url = form.value.idProducto
            ? `https://backendpawstails.runasp.net/api/gestion/productos/$(form.value.idProducto)`
            : `https://backendpawstails.runasp.net/api/gestion/productos`
        
        const method = form.value.idProducto ? 'PUT' : 'POST'

        try {
            await fetch (url, {
                method,
                headers: {'Content Type':'application/json'},
                body: JSON.stringify(form.value),
            })
            mensaje.value = 'Guardado exitosamente'
            fetchProductos()
            cerrarForm()
        } catch {
            mensaje.value = 'Error al guardar producto'
        }
    }
    const cerrarForm = () => {
        mostrarForm.value = false
    }
    onMounted(fetchProductos)

</script>
<template>

    <div class="app">
        <h1 class="header"> Gestion Productos </h1>
        <button @click="abrirCrear"> Crear Productos </button>
        <!--Tabla para mostrar productos-->
        <table>
            <thead>
                <tr>
                    <th>ID Producto</th>
                    <th>Nombres</th>
                    <th>Categorias</th>
                    <th>Precio</th>
                    <th>Stock</th>
                    <th>Imagenes</th>
                    <th>Editar</th>
                    <th>Eliminar</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="p in productos" :key="p.idProducto">
                    <td>{{ p.idProducto }}</td>
                    <td>{{ p.prodNombre }}</td>
                    <td>{{ p.prodCategoria }}</td>
                    <td>{{ p.prodPrecio }}</td>
                    <td>{{ p.prodStock }}</td>
                    <td>
                        <img v-for="(img, i) in p.prodImg" :key ="i" :src ="img" class="img-mini"/>
                    </td>
                    <td><button @click="editar(p)">Editar</button></td>
                    <td><button @click="eliminar(p.idProducto)">Eliminar</button></td>
                </tr>
            </tbody>
        </table>

        <!--Formulario para crear o editar productos-->
        <div v-if ="mostrarForm" class="modal">
            <h2>{{form.idProducto ? 'EditarProducto' : 'Crear Producto' }}</h2>
            <input v-model="form.prodNombre" placeholder="Nombre" />
            <input v-model="form.prodCategoria" placeholder="Categoria" />
            <input type="number" v-model.number ="form.prodPrecio" placeholder="Precio" />
            <input type="number" v-model.number ="form.prodStock" placeholder="Stock" />
            <textarea v-model="imagenesInput" placeholder="URL de Imagen, una por linea"></textarea>

            <div class="preview">
                <img v-for="(img, i) in form.prodImg" :key ="i" :src ="img" class="img-mini"/>
            </div>

            <button @click="guardar">Guardar</button>
            <button @click="cerrarForm">Cancelar</button>
            <p>{{ mensaje }}</p>
        </div>
    </div>
</template>


<style scooped></style>