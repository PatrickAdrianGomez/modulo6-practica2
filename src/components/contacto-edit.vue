<template>
  <form @submit.prevent="guardarCambios">
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input type="text" class="form-control" v-model="contactoEditado.nombre" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Apellido</label>
      <input type="text" class="form-control" v-model="contactoEditado.apellido" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Teléfono</label>
      <input type="tel" class="form-control" v-model="contactoEditado.telefono" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Correo</label>
      <input type="text" class="form-control" v-model="contactoEditado.correo" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Dirección</label>
      <textarea class="form-control" v-model="contacto.direccion"></textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Fecha de Nacimiento</label>
      <input type="date" class="form-control" v-model="contactoEditado.fechaNacimiento" required>
    </div>
    <div class="mb-3">
      <label class="form-label fw-bold">Categorías</label>
      <div class="d-flex flex-wrap gap-2">
        <div v-for="categoria in categorias" :key="categoria.id" class="form-check">
          <input class="form-check-input" type="checkbox" :id="'cat-' + categoria.id" :value="categoria.id"
            v-model="contactoEditado.categorias">
          <label class="form-check-label badge" :for="'cat-' + categoria.id" :style="{
            backgroundColor: contactoEditado.categorias.includes(categoria.id) ? categoria.color + '20' : 'transparent',
            color: contactoEditado.categorias.includes(categoria.id) ? categoria.color : '#495057',
            border: '1px solid ' + categoria.color
          }">
            {{ categoria.nombre }}
          </label>
        </div>
      </div>
    </div>
    <div class="mb-3">
      <input type="checkbox" class="form-check-input" v-model="contactoEditado.favorito">
      <label class="form-check-label">Favorito</label>
    </div>

    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" @click="$emit('cancelar')">Cancelar</button>
      <button type="submit" class="btn btn-primary">Guardar Cambios</button>
    </div>
  </form>
</template>

<script>
import CategoriaSelect from './categorias-select.vue'
import categorias from '@/data/categorias'

export default {
  components: { CategoriaSelect },
  props: {
    categorias: {
      type: Array,
      required: true,
      default: () => []
    },
    contacto: Object,
    paises: Array,
    ciudades: Array
  },
  data() {
    return {
      contactoEditado: {
        ...this.contacto,
        categorias: [...(this.contacto.categorias || [])]
      }
    }
  },
  methods: {
    cambiarPais() {
      this.$emit('pais-seleccionado', this.contactoEditado.paisId)
    },
    guardarCambios() {
      this.$emit('updated', { ...this.contactoEditado })
    },
    agregarContacto(nuevoContacto) {
      const maxId = Math.max(...this.contactos.map(c => c.id), 0)
      nuevoContacto.id = maxId + 1

      // Guardar contacto
      this.contactos.push({ ...nuevoContacto })

      // Guardar relaciones con categorías
      nuevoContacto.categorias?.forEach(catId => {
        const maxRelId = Math.max(...this.contactosCategorias.map(cc => cc.id), 0)
        this.contactosCategorias.push({
          id: maxRelId + 1,
          contactoId: nuevoContacto.id,
          categoriaId: catId
        })
      })

      this.aplicarFiltro()
      this.cerrarModal()
    },
    actualizarContacto(contactoActualizado) {
      // Actualizar datos del contacto
      const index = this.contactos.findIndex(c => c.id === contactoActualizado.id)
      this.contactos.splice(index, 1, { ...contactoActualizado })

      // Actualizar relaciones con categorías
      this.contactosCategorias = this.contactosCategorias.filter(
        cc => cc.contactoId !== contactoActualizado.id
      )

      contactoActualizado.categorias?.forEach(catId => {
        const maxRelId = Math.max(...this.contactosCategorias.map(cc => cc.id), 0)
        this.contactosCategorias.push({
          id: maxRelId + 1,
          contactoId: contactoActualizado.id,
          categoriaId: catId
        })
      })

      this.aplicarFiltro()
      this.cerrarModal()
    }
  }
}
</script>