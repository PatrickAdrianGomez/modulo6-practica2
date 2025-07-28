<template>
    <div class="container mt-4">
        <!-- Encabezado con buscador -->
        <div class="d-flex justify-content-between align-items-center mb-4 flex-wrap">
            <h1 class="mb-3 mb-md-0">
                <i class="bi bi-person-lines-fill me-2"></i>{{ title }}
            </h1>

            <div class="d-flex flex-column flex-md-row gap-3 w-100 w-md-auto">
                <!-- Buscador -->
                <div class="search-box position-relative flex-grow-1">
                    <i class="bi bi-search position-absolute"></i>
                    <input type="text" class="form-control ps-4" v-model="filtroNombre"
                        placeholder="Buscar por nombre..." @input="aplicarFiltro">
                    <button v-if="filtroNombre" class="btn btn-sm btn-link position-absolute end-0 top-0 bottom-0"
                        @click="limpiarFiltro">
                        <i class="bi bi-x"></i>
                    </button>
                </div>

                <!-- Filtro por categorías -->
                <div class="dropdown" ref="categoriaDropdown">
                    <button class="btn btn-outline-secondary dropdown-toggle" type="button"
                        @click="toggleCategoriaDropdown" aria-expanded="false">
                        <i class="bi bi-funnel me-2"></i>
                        {{ categoriaFiltro ? obtenerCategoria(categoriaFiltro)?.nombre : 'Todas las categorías' }}
                    </button>
                    <ul class="dropdown-menu" :class="{ show: mostrarDropdownCategorias }">
                        <li>
                            <a class="dropdown-item" href="#" @click.prevent="cambiarCategoriaFiltro(null)">
                                <i class="bi bi-x-circle me-2"></i> Todas las categorías
                            </a>
                        </li>
                        <li>
                            <hr class="dropdown-divider">
                        </li>
                        <li v-for="categoria in categorias" :key="categoria.id">
                            <a class="dropdown-item d-flex align-items-center" href="#"
                                @click.prevent="cambiarCategoriaFiltro(categoria.id)">
                                <span class="color-badge me-2" :style="{ backgroundColor: categoria.color }"></span>
                                {{ categoria.nombre }}
                            </a>
                        </li>
                    </ul>
                </div>

                <!-- Botón Nuevo Contacto -->
                <button class="btn btn-primary" @click="abrirModalParaCrear()">
                    <i class="bi bi-plus-circle me-2"></i> Nuevo
                </button>
            </div>
        </div>

        <!-- Mensaje cuando no hay resultados -->
        <div v-if="contactosFiltrados.length === 0" class="alert alert-info">
            <i class="bi bi-info-circle me-2"></i>
            {{ filtroNombre ? 'No se encontraron contactos con ese nombre' : 'No hay contactos registrados' }}
        </div>

        <!-- Tabla de contactos -->
        <div v-else class="table-responsive">
            <table class="table table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Apellido</th>
                        <th>Teléfono</th>
                        <th>Correo</th>
                        <th>Categorías</th>
                        <th>País</th>
                        <th>Ciudad</th>
                        <th>Fecha de Nacimiento</th>
                        <th>Favorito</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(contacto, index) in contactosFiltrados" :key="contacto.id">
                        <td>{{ contacto.id }}</td>
                        <td>{{ contacto.nombre }}</td>
                        <td>{{ contacto.apellido }}</td>
                        <td>{{ contacto.telefono }}</td>
                        <td>{{ contacto.correo }}</td>
                        <td>
                            <span v-for="catId in contacto.categorias" :key="catId" class="badge me-1" :style="{
                                backgroundColor: obtenerCategoria(catId)?.color + '20',
                                color: obtenerCategoria(catId)?.color,
                                border: '1px solid ' + obtenerCategoria(catId)?.color
                            }">
                                {{ obtenerCategoria(catId)?.nombre }}
                            </span>
                        </td>
                        <td>{{ obtenerNombrePais(contacto.paisId) }}</td>
                        <td>{{ obtenerNombreCiudad(contacto.ciudadId) }}</td>
                        <td>{{ contacto.fechaNacimiento }}</td>
                        <td>
                            <i class="bi" :class="contacto.favorito ? 'bi-star-fill text-warning' : 'bi-star'"></i>
                        </td>
                        <td>
                            <button class="btn btn-sm btn-outline-warning me-2" @click="abrirModalParaEditar(index)">
                                <i class="bi bi-pencil"></i>
                            </button>
                            <button class="btn btn-sm btn-outline-danger" @click="eliminarContacto(index)">
                                <i class="bi bi-trash"></i>
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="modalContacto" tabindex="-1" ref="modalRef">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">
                            {{ modalMode === 'crear' ? 'Nuevo Contacto' : 'Editar Contacto' }}
                        </h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body">
                        <contacto-add v-if="modalMode === 'crear'" :paises="paises" :ciudades="ciudadesFiltradas"
                            @created="agregarContacto" @cancelar="cerrarModal" :categorias="categorias"
                            @pais-seleccionado="actualizarCiudadesFiltradas" />
                        <contacto-edit v-if="modalMode === 'editar' && contactoSeleccionado"
                            :key="contactoSeleccionado.id" :contacto="contactoSeleccionado" :paises="paises"
                            :ciudades="ciudadesFiltradas" :categorias="categorias" @updated="actualizarContacto"
                            @cancelar="cerrarModal" @pais-seleccionado="actualizarCiudadesFiltradas" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import ContactoAdd from '@/components/contacto-add.vue'
import ContactoEdit from '@/components/contacto-edit.vue'
import paisesData from '@/data/paises.js'
import ciudadesData from '@/data/ciudades.js'
import categoriasData from '@/data/categorias.js'
import contactosCategoriasData from '@/data/contacto-categoria.js'

export default {
    name: 'ContactoList',
    components: { ContactoAdd, ContactoEdit },
    data() {
        return {
            contactos: [],
            categorias: [],
            contactosCategorias: [],
            paises: [],
            title: 'Lista de Contactos',
            ciudades: [],
            ciudadesFiltradas: [],
            modalBootstrapInstance: null,
            contactoSeleccionado: null,
            indiceSeleccionado: null,
            categoriaFiltro: null,
            modalMode: 'crear',
            filtroNombre: '',
            contactosFiltrados: [],
            mostrarDropdownCategorias: false,
            dropdownInstance: null
        }
    },
    mounted() {
        //this.modalBootstrapInstance = new bootstrap.Modal(this.$refs.modalRef)
        this.obtenerListaContactos()
        this.obtenerListaPaises()
        this.obtenerListaCiudades()
        this.initModal()
    },
    created() {
        // Cargar datos
        this.categorias = categoriasData
        this.contactosCategorias = contactosCategoriasData
        this.contactosFiltrados = [...this.contactos]

        // Mapear categorías a contactos
        this.contactos.forEach(contacto => {
            contacto.categorias = this.contactosCategorias
                .filter(cc => cc.contactoId === contacto.id)
                .map(cc => cc.categoriaId)
        })
    },
    methods: {
        obtenerListaContactos() {
            this.axios.get(process.env.VUE_APP_API_URL+'/contactos')
                .then(response => {
                    this.contactos = response.data
                    this.contactosFiltrados = [...this.contactos]
                })
                .catch(error => {
                    console.error('Error al obtener contactos:', error)
                })
        },
        obtenerListaPaises() {
            this.axios.get(process.env.VUE_APP_API_URL+'/paises')
                .then(response => {
                    this.paises = response.data
                })
                .catch(error => {
                    console.error('Error al obtener países:', error)
                })
        },
        obtenerListaCiudades() {
            this.axios.get(process.env.VUE_APP_API_URL+'/ciudades')
                .then(response => {
                    this.ciudades = response.data
                })
                .catch(error => {
                    console.error('Error al obtener ciudades:', error)
                })
        },
        initModal() {
            if (this.$refs.modalRef) {
                this.modalBootstrapInstance = new bootstrap.Modal(this.$refs.modalRef);
            } else {
                console.error('Elemento modal no encontrado')
            }
            // Inicializar el dropdown si usas Bootstrap JS
            if (typeof bootstrap !== 'undefined' && this.$refs.categoriaDropdown) {
                this.dropdownInstance = new bootstrap.Dropdown(
                    this.$refs.categoriaDropdown.querySelector('.dropdown-toggle')
                )
            }

            // Cerrar el dropdown al hacer clic fuera
            document.addEventListener('click', (e) => {
                if (!this.$refs.categoriaDropdown?.contains(e.target)) {
                    this.mostrarDropdownCategorias = false
                }
            })
        },
        toggleCategoriaDropdown() {
            this.mostrarDropdownCategorias = !this.mostrarDropdownCategorias

            // Si estás usando Bootstrap JS
            if (!this.dropdownInstance && this.$refs.categoriaDropdown) {
                this.dropdownInstance = new bootstrap.Dropdown(this.$refs.categoriaDropdown.querySelector('.dropdown-toggle'))
            }

            if (this.mostrarDropdownCategorias) {
                this.dropdownInstance?.show()
            } else {
                this.dropdownInstance?.hide()
            }
        },
        obtenerCategoria(id) {
            console.log('Obteniendo categoría con ID:', id);
            return this.categorias.find(c => c.id === id)
        },
        cambiarCategoriaFiltro(categoriaId) {
            this.categoriaFiltro = categoriaId
            this.mostrarDropdownCategorias = false
            this.aplicarFiltro()

            // Opcional: cerrar el dropdown después de seleccionar
            const dropdownElement = this.$el.querySelector('#dropdownCategorias')
            if (dropdownElement) {
                const dropdown = bootstrap.Dropdown.getInstance(dropdownElement) ||
                    new bootstrap.Dropdown(dropdownElement)
                dropdown.hide()
                this.dropdownInstance?.hide()
            }
        },
        aplicarFiltro() {
            // Si no hay filtros, mostrar todos
            if (!this.filtroNombre && !this.categoriaFiltro) {
                this.contactosFiltrados = [...this.contactos]
                return
            }

            this.contactosFiltrados = this.contactos.filter(contacto => {
                // Verificar filtro por nombre
                const coincideNombre = !this.filtroNombre ||
                    contacto.nombre.toLowerCase().includes(this.filtroNombre.toLowerCase()) ||
                    contacto.apellido.toLowerCase().includes(this.filtroNombre.toLowerCase())

                // Verificar filtro por categoría
                let coincideCategoria = true
                if (this.categoriaFiltro) {
                    const categoriasContacto = this.contactosCategorias
                        .filter(cc => cc.contactoId === contacto.id)
                        .map(cc => cc.categoriaId)
                    coincideCategoria = categoriasContacto.includes(this.categoriaFiltro)
                }

                return coincideNombre && coincideCategoria
            })
        },
        obtenerContactosPorCategoria(categoriaId) {
            return this.contactos.filter(contacto => {
                return this.contactosCategorias.some(
                    cc => cc.contactoId === contacto.id && cc.categoriaId === categoriaId
                )
            })
        },

        obtenerCategoriasPorContacto(contactoId) {
            return this.contactosCategorias
                .filter(cc => cc.contactoId === contactoId)
                .map(cc => this.categorias.find(c => c.id === cc.categoriaId))
                .filter(Boolean)
        },
        limpiarFiltro() {
            this.filtroNombre = ''
            this.contactosFiltrados = [...this.contactos]
        },
        obtenerNombrePais(paisId) {
            const pais = this.paises.find(p => p.id === paisId)
            return pais ? pais.nombre : 'Desconocido'
        },
        obtenerNombreCiudad(ciudadId) {
            const ciudad = this.ciudades.find(c => c.id === ciudadId)
            return ciudad ? ciudad.nombre : 'Desconocida'
        },
        actualizarCiudadesFiltradas(paisId) {
            this.ciudadesFiltradas = this.ciudades.filter(c => c.paisId === paisId)
        },
        abrirModalParaCrear() {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }
            this.modalMode = 'crear'
            this.contactoSeleccionado = null
            this.ciudadesFiltradas = []
            this.modalBootstrapInstance.show()
        },
        abrirModalParaEditar(index) {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }
            const contactoFiltrado = this.contactosFiltrados[index]
            const indiceSeleccionado = this.contactos.findIndex(c => c.id === contactoFiltrado.id)
            // Verificar que se encontró el contacto
            if (indiceSeleccionado === -1) {
                console.error('Contacto no encontrado en el array principal');
                return;
            }
            this.modalMode = 'editar'
            this.indiceSeleccionado = index
            this.contactoSeleccionado = { ...this.contactos[indiceSeleccionado] }
            this.actualizarCiudadesFiltradas(this.contactoSeleccionado.paisId)
            this.modalBootstrapInstance.show()
        },
        agregarContacto(nuevoContacto) {
            this.axios.post(process.env.VUE_APP_API_URL+'/contactos', nuevoContacto)
                .then(response => {
                    this.contactos.push(response.data)
                })
                .catch(error => {
                    console.error('Error al agregar contacto:', error)
                })
            //const maxId = Math.max(...this.contactos.map(c => c.id), 0)
            //nuevoContacto.id = maxId + 1
            //this.contactos.push(nuevoContacto)
            this.aplicarFiltro() // Actualizar la lista filtrada
            this.cerrarModal()
        },
        actualizarContacto(contactoActualizado) {
            //this.contactos.splice(this.indiceSeleccionado, 1, contactoActualizado)
            this.axios.patch(process.env.VUE_APP_API_URL`/contactos/${contactoActualizado.id}`, contactoActualizado)
                .then(response => {
                    const index = this.contactos.findIndex(c => c.id === response.data.id)
                    if (index !== -1) {
                        this.$set(this.contactos, index, response.data)
                    }
                })
                .catch(error => {
                    console.error('Error al actualizar contacto:', error)
                })
            this.aplicarFiltro() // Actualizar la lista filtrada
            this.cerrarModal()
        },
        eliminarContacto(index) {
            const contactoFiltrado = this.contactosFiltrados[index]
            const indiceReal = this.contactos.findIndex(c => c.id === contactoFiltrado.id)

            if (confirm('¿Eliminar este contacto?')) {
                //this.contactos.splice(indiceReal, 1)
                this.axios.delete(process.env.VUE_APP_API_URL`/contactos/${contactoFiltrado.id}`)
                    .then(() => {
                        this.contactos.splice(indiceReal, 1)
                    })
                    .catch(error => {
                        console.error('Error al eliminar contacto:', error)
                    })
                this.aplicarFiltro() // Reaplicar filtro después de eliminar
            }
        },
        cerrarModal() {
            this.modalBootstrapInstance.hide()
        }
    }
}
</script>

<style scoped>
.dropdown {
    position: relative;
    display: inline-block;
}

.dropdown-menu {
    position: absolute;
    z-index: 1000;
    /* Asegura que esté por encima de otros elementos */
    display: none;
    min-width: 10rem;
    padding: 0.5rem 0;
    margin: 0;
    font-size: 1rem;
    color: #212529;
    text-align: left;
    list-style: none;
    background-color: #fff;
    background-clip: padding-box;
    border: 1px solid rgba(0, 0, 0, 0.15);
    border-radius: 0.25rem;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.175);
}

.dropdown-menu.show {
    display: block;
}

.color-badge {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    margin-right: 8px;
}

.dropdown-item {
    display: block;
    width: 100%;
    padding: 0.25rem 1rem;
    clear: both;
    font-weight: 400;
    color: #212529;
    text-align: inherit;
    text-decoration: none;
    white-space: nowrap;
    background-color: transparent;
    border: 0;
    cursor: pointer;
}

.dropdown-item:hover {
    background-color: #f8f9fa;
}

.dropdown-divider {
    height: 0;
    margin: 0.5rem 0;
    overflow: hidden;
    border-top: 1px solid #e9ecef;
}
</style>