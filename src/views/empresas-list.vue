<template>
    <div class="container mt-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h1>{{ title }}</h1>
            <button class="btn btn-primary" @click="abrirModalParaCrear()">
                <i class="bi bi-plus-circle"></i> Nueva Empresa
            </button>
        </div>

        <div class="table-responsive">
            <table class="table table-hover table-bordered border-light">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Teléfono</th>
                        <th>Correo</th>
                        <th>Dirección</th>
                        <th>País</th>
                        <th>Ciudad</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(empresa, index) in empresas" :key="empresa.id">
                        <td>{{ empresa.id }}</td>
                        <td>{{ empresa.nombre }}</td>
                        <td>{{ empresa.telefono }}</td>
                        <td>{{ empresa.correo }}</td>
                        <td>{{ empresa.direccion }}</td>
                        <td>{{ obtenerNombrePais(empresa.paisId) }}</td>
                        <td>{{ obtenerNombreCiudad(empresa.ciudadId) }}</td>
                        <td class="text-nowrap">
                            <button class="btn btn-sm btn-outline-warning me-2" @click="abrirModalParaEditar(index)">
                                <i class="bi bi-pencil"></i> Editar
                            </button>
                            <button class="btn btn-sm btn-outline-danger" @click="eliminarEmpresa(index)">
                                <i class="bi bi-trash"></i> Eliminar
                            </button>
                        </td>
                    </tr>
                </tbody>

            </table>
                <div class="modal fade" id="modalAutoEditar" tabindex="-1" ref="modalRef">
                    <div class="modal-dialog modal-lg">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">
                                    {{ modalMode === 'crear' ? 'Nueva Empresa' : 'Editar Empresa' }}
                                </h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                            </div>
                            <div class="modal-body">
                                <empresa-add v-if="modalMode === 'crear'" :paises="paises" :ciudades="ciudadesFiltradas"
                                    @created="agregarEmpresa" @cancelar="cerrarModal"
                                    @pais-seleccionado="actualizarCiudadesFiltradas" />
                                <empresa-edit v-if="modalMode === 'editar' && empresaSeleccionada"
                                    :key="empresaSeleccionada.id" :empresa="empresaSeleccionada" :paises="paises"
                                    :ciudades="ciudadesFiltradas" @updated="actualizarEmpresa" @cancelar="cerrarModal"
                                    @pais-seleccionado="actualizarCiudadesFiltradas" />
                            </div>
                        </div>
                    </div>
                </div>
        </div>
    </div>
</template>

<script>
import EmpresaAdd from '../components/empresa-add.vue';
import EmpresaEdit from '../components/empresa-edit.vue';
import empresasList from '../data/empresas.js';
import paisesList from '../data/paises.js';
import ciudadesList from '../data/ciudades.js';
export default {
    name: 'EmpresasList',
    components: {
        EmpresaAdd,
        EmpresaEdit
    },
    data() {
        return {
            title: 'Lista de Empresas',
            empresas: [],
            paises: [],
            ciudades: [],
            ciudadesFiltradas: [],
            modalBootstrapInstance: null,
            indiceSeleccionado: null,
            modalMode: 'crear',
            empresaSeleccionada: null
        };
    },
    mounted() {
        this.obtenerListaEmpresas();
        this.obtenerListaPaises();
        this.obtenerListaCiudades();
        this.initModal();
    },
    methods: {
        obtenerListaEmpresas() {
            this.axios.get(process.env.VUE_APP_API_URL+'/empresas')
                .then(response => {
                    this.empresas = response.data;
                })
                .catch(error => {
                    console.error('Error al obtener empresas:', error);
                });
        },
        obtenerListaPaises() {
            this.axios.get(process.env.VUE_APP_API_URL+'/paises')
                .then(response => {
                    this.paises = response.data;
                })
                .catch(error => {
                    console.error('Error al obtener países:', error);
                });
        },
        obtenerListaCiudades() {
            this.axios.get(process.env.VUE_APP_API_URL+'/ciudades')
                .then(response => {
                    this.ciudades = response.data;
                })
                .catch(error => {
                    console.error('Error al obtener ciudades:', error);
                });
        },
        initModal() {
            if (this.$refs.modalRef) {
                this.modalBootstrapInstance = new bootstrap.Modal(this.$refs.modalRef)
            } else {
                console.error('Elemento modal no encontrado');
            }
        },
        abrirModalParaCrear() {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }
            this.modalMode = 'crear';
            this.empresaSeleccionada = null;
            this.ciudadesFiltradas = [];
            this.modalBootstrapInstance.show();
        },
        abrirModalParaEditar(index) {
            if (!this.modalBootstrapInstance) {
                this.initModal() // Intenta reinicializar si es necesario
            }

            this.modalMode = 'editar';
            this.empresaSeleccionada = { ...this.empresas[index] };
            this.indiceSeleccionado = index;
            this.actualizarCiudadesFiltradas(this.empresaSeleccionada.paisId);
            this.modalBootstrapInstance.show();
        },
        agregarEmpresa(nuevaEmpresa) {
            this.axios.post(process.env.VUE_APP_API_URL+'/empresas', nuevaEmpresa)
                .then(response => {
                    this.empresas.push(response.data);
                })
                .catch(error => {
                    console.error('Error al agregar empresa:', error);
                });
            //const maxId = Math.max(0, ...this.empresas.map(e => e.id));
            //nuevaEmpresa.id = maxId + 1;
            //this.empresas.push(nuevaEmpresa);
            this.cerrarModal();
        },
        actualizarCiudadesFiltradas(paisId) {
            if (paisId) {
                this.ciudadesFiltradas = this.ciudades.filter(c => c.paisId === paisId);
            } else {
                this.ciudadesFiltradas = [];
            }
        },
        actualizarEmpresa(empresaActualizada) {
            //this.empresas.splice(this.indiceSeleccionado, 1, empresaActualizada);
            this.axios.patch(process.env.VUE_APP_API_URL`/empresas/${empresaActualizada.id}`, empresaActualizada)
                .then(response => {
                    const index = this.empresas.findIndex(e => e.id === empresaActualizada.id);
                    if (index !== -1) {
                        this.empresas.splice(index, 1, response.data);
                    }
                })
                .catch(error => {
                    console.error('Error al actualizar empresa:', error);
                });
            this.cerrarModal();
        },
        guardarEdicion(empresaEditada) {
            const index = this.empresas.findIndex(e => e.id === empresaEditada.id);
            if (index !== -1) {
                this.empresas.splice(index, 1, empresaEditada);
                this.modalBootstrapInstance?.hide();
            }
        },
        eliminarEmpresa(index) {
            if (confirm('¿Estás seguro de eliminar esta empresa?')) {
                //this.empresas.splice(index, 1);
                this.axios.delete(process.env.VUE_APP_API_URL`/empresas/${this.empresas[index].id}`)
                    .then(() => {
                        this.empresas.splice(index, 1);
                    })
                    .catch(error => {
                        console.error('Error al eliminar empresa:', error);
                    });
            }
        },
        obtenerNombrePais(paisId) {
            const pais = this.paises.find(p => p.id === paisId);
            return pais ? pais.nombre : 'Desconocido';
        },
        obtenerNombreCiudad(ciudadId) {
            const ciudad = this.ciudades.find(c => c.id === ciudadId);
            return ciudad ? ciudad.nombre : 'Desconocido';
        },
        cerrarModal() {
            this.modalBootstrapInstance.hide()
        }
    }
};
</script>