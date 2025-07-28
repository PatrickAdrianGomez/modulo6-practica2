<template>
    <form @submit.prevent="guardaCambios">
        <div class="mb-3">
            <label class="from-label">Nombre</label>
            <input type="text" class="form-control" v-model="empresaEditada.nombre" required>
        </div>
        <div class="mb-3">
            <label class="from-label">Teléfono</label>
            <input type="tel" class="form-control" v-model="empresaEditada.telefono" required>
        </div>
        <div class="mb-3">
            <label class="from-label">correo</label>
            <input type="email" class="form-control" v-model="empresaEditada.correo" required>
        </div>
        <div class="mb-3">
            <label class="from-label">Dirección</label>
            <textarea class="form-control" v-model="empresaEditada.direccion"></textarea>
        </div>

        <div class="mb-3">
            <label class="form-label">País</label>
            <select class="form-select" v-model="empresaEditada.paisId" @change="actualizarCiudades" required>
                <option value="" disabled>Seleccione un país</option>
                <option v-for="pais in paises" :key="pais.id" :value="pais.id">
                    {{ pais.nombre }}
                </option>
            </select>
        </div>
        <div class="mb-3">
            <label class="form-label">Ciudad</label>
            <select class="form-select" v-model="empresaEditada.ciudadId" :disabled="!empresaEditada.paisId" required>
                <option value="" disabled>Seleccione una ciudad</option>
                <option v-for="ciudad in ciudades" :key="ciudad.id" :value="ciudad.id">
                    {{ ciudad.nombre }}
                </option>
            </select>
        </div>

        <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="$emit('cancelar')">Cancelar</button>
            <button type="submit" class="btn btn-primary">Guardar Cambios</button>
        </div>
    </form>
</template>

<script>
export default {
    props: {
        empresa: {
            type: Object,
            required: true
        },
        paises: Array,
        ciudades: Array
    },
    data() {
        return {
            empresaEditada: { ...this.empresa }
        };
    },
    methods: {
        actualizarCiudades() {
            this.$emit('pais-seleccionado', this.empresaEditada.paisId);
        },
        guardaCambios() {
            this.$emit('updated', { ...this.empresaEditada });
        },
        guardarEmpresa() {
            this.$emit('created', { ...this.empresaEditada });
        }
    }
};
</script>