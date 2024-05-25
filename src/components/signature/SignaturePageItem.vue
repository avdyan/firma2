<script setup lang="ts">
// Importa las funciones necesarias de vue y otros archivos locales.
import { ref, onMounted, onBeforeUnmount } from 'vue';
import useFabric from '@/hooks/useFabric';
import type { PDF } from '@/types/pdf';

// Define las propiedades para este componente.
interface Props {
  file: PDF;
  page: number;
}

// Define las propiedades del componente.
const props = defineProps<Props>();

// Define la referencia reactiva para mostrar el lienzo.
const isShowCanvas = ref(false);

// Define el ID del lienzo.
const canvasId = `canvas_page_${props.page - 1}`;

// Define las funciones para trabajar con el lienzo.
const { createCanvas, specifyPage, renderImage, deleteCanvas } = useFabric(canvasId);

// Define la función para configurar el PDF.
async function setPDF() {
  const { file, page } = props;

  // Crea el lienzo.
  createCanvas();

  // Si el archivo PDF es una imagen, renderiza la imagen.
  // De lo contrario, especifica la página del PDF a renderizar.
  file.PDFBase64.startsWith('data:image')
    ? renderImage({ url: file.PDFBase64, scale: 0.2 })
    : await specifyPage({ page, PDF: file, scale: 0.3 });

  // Muestra el lienzo.
  isShowCanvas.value = true;
}

// Cuando el componente se monta, configura el PDF.
onMounted(setPDF);

// Antes de que el componente se desmonte, elimina el lienzo.
onBeforeUnmount(deleteCanvas);
</script>

<template>
  <!-- Define el marcado para el lienzo. -->
  <canvas
    v-show="isShowCanvas"
    :id="canvasId"
    class="border-2 border-gray-20"
  ></canvas>
</template>