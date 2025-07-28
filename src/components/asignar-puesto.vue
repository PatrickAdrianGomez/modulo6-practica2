<template>
    <div class="modal fade" id="asignarPuestoModal" tabindex="-1" aria-hidden="true" ref="modal">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Asignar Puesto a Contacto</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form @submit.prevent="guardar">
                        <div class="mb-3">
                            <label class="form-label">Empresa</label>
                            <select v-model="relacion.empresaId" class="form-select" required>
                                <option value="" disabled>Seleccione una empresa</option>
                                <option v-for="empresa in empresas" :key="empresa.id" :value="empresa.id">
                                    {{ empresa.nombre }}
                                </option>
                            </select>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Puesto</label>
                            <input v-model="relacion.puesto" type="text" class="form-control"
                                placeholder="Ej: Gerente de Ventas" required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Departamento (opcional)</label>
                            <input v-model="relacion.departamento" type="text" class="form-control"
                                placeholder="Ej: Marketing">
                        </div>

                        <div class="form-check mb-3">
                            <input v-model="relacion.esActual" type="checkbox" class="form-check-input"
                                id="esActualCheck">
                            <label class="form-check-label" for="esActualCheck">
                                ¿Es el puesto actual?
                            </label>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancelar</button>
                    <button type="button" class="btn btn-primary" @click="guardar">Guardar</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    props: {
        contactoId: {
            type: Number,
            required: true
        },
        empresas: {
            type: Array,
            required: true
        }
    },
    data() {
        return {
            modalBootstrapInstance: null,
            relacion: {
                contactoId: this.contactoId,
                empresaId: null,
                puesto: '',
                departamento: '',
                esActual: true
            }
        }
    },
    mounted() {
        this.modalBootstrapInstance = new bootstrap.Modal(this.$refs.modal)
    },
    methods: {
        mostrar() {
            this.resetForm()
            this.modalBootstrapInstance.show()
        },
        ocultar() {
            this.modalBootstrapInstance.hide()
        },
        resetForm() {
            this.relacion = {
                contactoId: this.contactoId,
                empresaId: null,
                puesto: '',
                departamento: '',
                esActual: true
            }
        },
        guardar() {
            if (!this.relacion.empresaId || !this.relacion.puesto) {
                alert('Por favor complete los campos requeridos')
                return
            }

            this.$emit('guardar', { ...this.relacion })
            this.ocultar()
        }
    }
}
</script>