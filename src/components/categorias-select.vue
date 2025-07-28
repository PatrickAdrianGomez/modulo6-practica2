<template>
  <div class="mb-3">
    <label class="form-label fw-bold d-block">
      <i class="bi bi-tags me-2"></i>Categorías
    </label>
    <div class="d-flex flex-wrap gap-3 align-items-center">
      <div v-for="categoria in categorias" :key="categoria.id" class="form-check form-check-inline">
        <input
          class="form-check-input"
          type="checkbox"
          :id="'cat-'+categoria.id"
          :value="categoria.id"
          v-model="selectedCategorias"
        >
        <label 
          class="form-check-label d-flex align-items-center"
          :for="'cat-'+categoria.id"
        >
          <span 
            class="color-badge me-2"
            :style="{backgroundColor: categoria.color}"
          ></span>
          {{ categoria.nombre }}
        </label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CategoriaSelect',
  props: {
    categorias: {
      type: Array,
      required: true,
      default: () => []
    },
    value: { // Para v-model
      type: Array,
      default: () => []
    }
  },
  data() {
    console.log('Categorias:', this.categorias);
    console.log('Valor inicial:', this.value);
    return {
      selectedCategorias: [...this.value]
    }
  },
  watch: {
    selectedCategorias(newVal) {
      this.$emit('input', newVal)
    },
    value(newVal) {
      this.selectedCategorias = [...newVal]
    }
  }
}
</script>

<style scoped>
.color-badge {
  display: inline-block;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 1px solid #dee2e6;
}

.form-check-input:checked {
  background-color: var(--bs-primary);
  border-color: var(--bs-primary);
}

.form-check-label {
  cursor: pointer;
  user-select: none;
}
</style>