<template>
    <div class="container mt-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h1 class="m-0">
                <i class="bi bi-pencil-square me-2"></i> {{ title }}
            </h1>
            <button class="btn btn-primary" @click="abrirModalParaCrear()">
                <i class="bi bi-plus-circle"></i> Nueva Categoría
            </button>
        </div>

        <div v-if="!categorias || categorias.length === 0" class="alert alert-warning">
            <i class="bi bi-exclamation-triangle me-2"></i>
            No hay categorías disponibles. Debe agregar categorías primero.
        </div>

        <div v-else class="table-responsive">
            <table class="table table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Descripción</th>
                        <th>Color</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(categoria, index) in categorias" :key="categoria.id">
                        <td>{{ categoria.id }}</td>
                        <td>{{ categoria.nombre }}</td>
                        <td>{{ categoria.descripcion }}</td>
                        <td>{{ categoria.color }}</td>
                        <td class="text-nowrap">
                            <button class="btn btn-sm btn-outline-warning me-2" @click="abrirModalParaEditar(index)">
                                <i class="bi bi-pencil"></i> Editar
                            </button>
                            <button class="btn btn-sm btn-outline-danger" @click="eliminarCategoria(index)">
                                <i class="bi bi-trash"></i> Eliminar
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div class="modal fade" id="modalCategoria" tabindex="-1" ref="modalRef">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">
                            {{ modalMode === 'crear' ? 'Nueva Categoría' : 'Editar Categoría' }}
                        </h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body">
                        <categoria-add 
                            v-if="modalMode === 'crear'" 
                            @created="categoriaCreada" 
                            @cancelar="cerrarModal" />
                        <categoria-edit 
                            v-if="modalMode === 'editar' && categoriaSeleccionada" 
                            :categoria="categoriaSeleccionada"
                            :categorias="categorias"
                            @updated="categoriaActualizada" 
                            @cancelar="cerrarModal" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import CategoriaAdd from '@/components/categoria-add.vue';
import CategoriaEdit from '@/components/categoria-edit.vue';
import categoriasList from '@/data/categorias.js'; // Asegúrate de tener un archivo JSON con las categorías

export default {
    name: 'CategoriaList',
    components: {
        CategoriaAdd,
        CategoriaEdit
    },
    props: {
        categorias: {
            type: Array,
            required: true,
            default: () => []
        }
    },
    data() {
        return {
            title: 'Lista de Categorías',
            categorias: categoriasList,
            modalMode: 'crear',
            categoriaSeleccionada: null,
            indiceSeleccionado: null,
            modalBootstrapInstance: null
        };
    },
    mounted() {
        this.obtenerListaCategorias()
        this.initialModal()
    },
    methods: {
        obtenerListaCategorias() {
            this.axios.get(process.env.VUE_APP_API_URL+'/categorias')
                .then(response => {
                    this.categorias = response.data
                })
                .catch(error => {
                    console.error('Error al obtener categorías:', error)
                })
        },
        initialModal() {
            if (this.$refs.modalRef) {
                this.modalBootstrapInstance = new bootstrap.Modal(this.$refs.modalRef);
            } else {
                console.error('Referencia al modal no encontrada');
            }
        },
        abrirModalParaCrear() {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }
            this.modalMode = 'crear';
            this.categoriaSeleccionada = null;
            this.modalBootstrapInstance?.show();
        },
        abrirModalParaEditar(index) {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }
            this.modalMode = 'editar';
            this.categoriaSeleccionada = { ...this.categorias[index] };
            this.modalBootstrapInstance?.show();
        },
        cerrarModal() {
            this.modalBootstrapInstance?.hide();
        },
        categoriaCreada(nuevaCategoria) {
            this.axios.post(process.env.VUE_APP_API_URL+'/categorias', nuevaCategoria)
                .then(response => {
                    this.categorias.push(response.data);
                })
                .catch(error => {
                    console.error('Error al agregar categoría:', error);
                });
            //const maxId = Math.max(...this.categorias.map(c => c.id), 0);
            //nuevaCategoria.id = maxId + 1;
            //this.categorias.push(nuevaCategoria);
            this.cerrarModal();
        },
        categoriaActualizada(categoriaActualizada) {
            //const index = this.categorias.findIndex(c => c.id === categoriaActualizada.id);
            /*if (index !== -1) {
                this.$set(this.categorias, index, categoriaActualizada);
            }*/
            //this.categorias.splice(this.indiceSeleccionado, 1, categoriaActualizada);
            this.axios.patch(process.env.VUE_APP_API_URL`/categorias/${categoriaActualizada.id}`, categoriaActualizada)
                .then(response => {
                    const index = this.categorias.findIndex(c => c.id === response.data.id);
                    if (index !== -1) {
                        this.$set(this.categorias, index, response.data);
                    }
                })
                .catch(error => {
                    console.error('Error al actualizar categoría:', error);
                });
            this.cerrarModal();
        },
        eliminarCategoria(index) {
            if (confirm('¿Está seguro de eliminar esta categoría?')) {
                //this.categorias.splice(index, 1);
                this.axios.delete(process.env.VUE_APP_API_URL`/categorias/${this.categorias[index].id}`)
                    .then(() => {
                        this.categorias.splice(index, 1);
                    })
                    .catch(error => {
                        console.error('Error al eliminar categoría:', error);
                    });
            }
        }
    }
};
</script>