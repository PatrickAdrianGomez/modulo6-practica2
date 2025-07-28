<template>
    <div>
        <form @submit.prevent="guardarCategoria" novalidate class="needs-validation"
            :class="{ 'was-validated': formSubmitted }">
            <div class="mb-4">
                <label for="nombre" class="form-label fw-bold">Nombre de la Categoría</label>
                <input type="text" class="form-control" id="nombre" v-model="categoria.nombre" required>
                <div class="invalid-feedback">El nombre es obligatorio</div>
            </div>
            <div class="mb-4">
                <label for="descripcion" class="form-label fw-bold">Descripción</label>
                <textarea class="form-control" id="descripcion" v-model="categoria.descripcion" rows="3" required></textarea>
                <div class="invalid-feedback">La descripción es obligatoria</div>
            </div>
            <div class="mb-4">
                <label for="color" class="form-label fw-bold">Color</label>
                <textarea class="form-control" id="color" v-model="categoria.color" rows="3"></textarea>
                <div class="invalid-feedback">El color es opcional</div>
            </div>
            <div class="d-flex justify-content-end gap-3 mt-4">
                <button type="button" class="btn btn-outline-secondary" @click="$emit('cancelar')">
                    <i class="bi bi-x-circle me-2"></i>Cancelar
                </button>
                <button type="submit" class="btn btn-primary">
                    <i class="bi bi-check-circle me-2"></i>Guardar
                </button>
            </div>
        </form>
    </div>
</template>

<script>
export default {
    name: 'CategoriaAdd',
    props: {
        categoria: {
            type: Array,
            required: true,
            default: () => []
        }
    },
    data() {
        return {
            categoria: {
                id: null,
                nombre: '',
                descripcion: '',
                color: ''
            },
            formSubmitted: false
        }
    },
    methods: {
        guardarCategoria() {
            this.formSubmitted = true;

            if (!this.categoria.nombre || !this.categoria.descripcion) {
                return;
            }

            this.$emit('created', { ...this.categoria });
            this.resetForm();
        },
        resetForm() {
            this.categoria = {
                id: null,
                nombre: '',
                descripcion: '',
                color: ''
            };
            this.formSubmitted = false;
        }
    }
}
</script>