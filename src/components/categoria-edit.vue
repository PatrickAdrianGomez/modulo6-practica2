<template>
    <div>
        <form @submit.prevent="guardarCambios" novalidate class="needs-validation"
            :class="{ 'was-validate': formSubmitted }">
            <div class="mb-3 d-none">
                <label for="id" class="form-label">ID</label>
                <input type="text" class="form-control" id="id" v-model="categoriaEditada.id" readonly>
            </div>
            <div class="mb-4">
                <label for="nombre" class="form-label fw-bold">Nombre de la Categoría</label>
                <input type="text" class="form-control" id="nombre" v-model="categoriaEditada.nombre" required>
                <div class="invalid-feedback">El nombre es obligatorio</div>
            </div>
            <div class="mb-4">
                <label for="descripcion" class="form-label fw-bold">Descripción</label>
                <textarea class="form-control" id="descripcion" v-model="categoriaEditada.descripcion" rows="3" required></textarea>
                <div class="invalid-feedback">La descripción es obligatoria</div>
            </div>
            <div class="mb-4">
                <label for="color" class="form-label fw-bold">Color</label>
                <textarea class="form-control" id="colot" v-model="categoriaEditada.color" rows="3"></textarea>
                <div class="invalid-feedback">El color es opcional</div>
            </div>
            <div class="d-flex justify-content-end gap-3 mt-4">
                <button type="button" class="btn btn-outline-secondary" @click="$emit('cancelar')">
                    <i class="bi bi-x-circle me-2"></i>Cancelar
                </button>
                <button type="submit" class="btn btn-primary">
                    <i class="bi bi-save me-2"></i>Guardar Cambios
                </button>
            </div>
        </form>
    </div>
</template>

<script>
export default {
    name: 'CategoriaEdit',
    props: {
        categoria: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            categoriaEditada: { ...this.categoria },
            formSubmitted: false
        }
    },
    watch: {
        categoria(newVal) {
            this.categoriaEditada = { ...newVal };
        }
    },
    methods: {
        guardarCambios() {
            this.formSubmitted = true;

            if (!this.categoriaEditada.nombre || !this.categoriaEditada.descripcion) {
                return;
            }

            this.$emit('updated', { ...this.categoriaEditada });
        }
    }
}
</script>