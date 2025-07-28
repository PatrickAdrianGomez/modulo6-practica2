<template>
    <form @submit.prevent="guardarEmpresa">
        <div class="mb-3">
            <label class="from-label">Nombre</label>
            <input type="text" class="form-control" v-model="empresa.nombre" required>
        </div>
        <div class="mb-3">
            <label class="from-label">Teléfono</label>
            <input type="tel" class="form-control" v-model="empresa.telefono" required>
        </div>
        <div class="mb-3">
            <label class="from-label">correo</label>
            <input type="email" class="form-control" v-model="empresa.correo" required>
        </div>
        <div class="mb-3">
            <label class="from-label">Dirección</label>
            <textarea class="form-control" v-model="empresa.direccion"></textarea>
        </div>

        <div class="mb-3">
            <label class="form-label">País</label>
            <select class="form-select" v-model="empresa.paisId" @change="cambiarPais" required>
                <option value="" disabled>Seleccione un país</option>
                <option v-for="pais in paises" :key="pais.id" :value="pais.id">
                    {{ pais.nombre }}
                </option>
            </select>
        </div>
        <div class="mb-3">
            <label class="form-label">Ciudad</label>
            <select class="form-select" v-model="empresa.ciudadId" :disabled="!empresa.paisId" required>
                <option value="" disabled>Seleccione una ciudad</option>
                <option v-for="ciudad in ciudades" :key="ciudad.id" :value="ciudad.id">
                    {{ ciudad.nombre }}
                </option>
            </select>
        </div>

        <div class="mb-3">
            <label class="from-label">Sitio Web</label>
            <input type="text" class="form-control" v-model="empresa.sitioWeb">
        </div>
        <div class="mb-3">
            <label class="from-label">Notas</label>
            <textarea class="form-control" v-model="empresa.notas"></textarea>
        </div>

        <!-- Botones con mejor espaciado -->
        <div class="d-flex justify-content-between mt-5">
            <button type="button" class="btn btn-outline-secondary px-4" @click="$emit('cancelar')"
                data-bs-dismiss="modal">
                <i class="bi bi-x-circle me-2"></i>Cancelar
            </button>
            <button type="submit" class="btn btn-primary px-4">
                <i class="bi bi-check-circle me-2"></i>Guardar
            </button>
        </div>
    </form>
</template>

<script>
export default {
    props: {
        paises: Array,
        ciudades: Array
    },
    data() {
        return {
            empresa: {
                id: null,
                nombre: '',
                telefono: '',
                correo: '',
                direccion: '',
                paisId: null,
                ciudadId: null,
                sitioWeb: '',
                notas: ''
            }
        }
    },
    methods: {
        cambiarPais() {
            this.empresa.ciudadId = null;
            this.$emit('pais-seleccionado', this.empresa.paisId);
        },
        guardarEmpresa() {
            this.$emit('created', { ...this.empresa });
        },
        agregarEmpresa(nuevaEmpresa) {
            const maxId = Math.max(0, ...this.empresa.map(e => e.id));
            nuevaEmpresa.id = maxId + 1;
            this.empresa.push({ ...nuevaEmpresa });
            this.cerrarModal();
        },
        actualizarEmpresa(empresaActualizada) {
            const index = this.empresa.findIndex(e => e.id === empresaActualizada.id);
            if (index !== -1) {
                this.empresa.splice(index, 1, { ...empresaActualizada });
            }
            this.cerrarModal();
        },
        resetForm() {
            this.empresa = {
                id: null,
                nombre: '',
                telefono: '',
                correo: '',
                direccion: '',
                paisId: null,
                ciudadId: null,
                sitioWeb: '',
                notas: ''
            };
        }
    },
}
</script>