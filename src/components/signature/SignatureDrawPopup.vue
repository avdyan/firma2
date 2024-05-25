<script setup lang="ts">
// Importa las funciones necesarias de vue, vue-i18n y otros archivos locales.
import { ref, defineAsyncComponent } from 'vue';
import { useI18n } from 'vue-i18n';
import { useSignatureStore } from '@/store';
import SignIcon from '@/components/SignIcon.vue';
import toast from '@/utils/toast';
import { isDesktop } from '@/utils/common';

// Define los emisores de eventos para este componente.
const emit = defineEmits(['close']);

// Define las referencias reactivas para el lienzo, la herramienta actual y el estado de dibujo.
const canvasDraw = ref<HTMLCanvasElement | null>(null);
const currentTool = ref('black');
const isDraw = ref(false);

// Utiliza la función de internacionalización.
const { t } = useI18n();

// Define el componente SignPopup de forma asíncrona.
const SignPopup = defineAsyncComponent(() => import('@/components/SignPopup.vue'));

// Define el contexto de dibujo.
const drawCtx = {
  ctx: null as CanvasRenderingContext2D | null | undefined,
  isPainting: false,
};

// Define la función para seleccionar una herramienta de dibujo.
function selectTool(tool: string) {
  currentTool.value = tool;
  if (!drawCtx.ctx) return;
  drawCtx.ctx.strokeStyle = tool;
}

// Define la función para agregar una firma.
function addSignature() {
  const signature = canvasDraw.value?.toDataURL('image/png');
  if (!signature) return;
  useSignatureStore().addSignature(signature);
  emit('close');
  toast.showToast(t('signature_add_success'), 'success');
}

// Define la función para configurar el lienzo.
function setCanvas() {
  if (!canvasDraw.value) return;
  if (isDesktop()) {
    canvasDraw.value.height = 235;
    canvasDraw.value.width = 466;
  } else {
    canvasDraw.value.height = window.innerHeight - 400;
  }
  drawCtx.ctx = canvasDraw.value?.getContext('2d');
  if (!drawCtx.ctx) return;
  drawCtx.ctx.lineWidth = 2;
  drawCtx.ctx.lineCap = 'round';
}

// Define la función para obtener la posición de pintura.
function getPaintPosition(event: MouseEvent | TouchEvent) {
  if (!canvasDraw.value) return { x: 0, y: 0 };
  const { left, top } = canvasDraw.value.getBoundingClientRect();

  if (event.type === 'mousemove') {
    const { clientX, clientY } = event as MouseEvent;
    return { x: clientX - left, y: clientY - top };
  }

  const { touches } = event as TouchEvent;

  return {
    x: touches[0].clientX - left,
    y: touches[0].clientY - top,
  };
}

// Define la función para iniciar la posición de dibujo.
function startPosition() {
  drawCtx.isPainting = true;
  drawCtx.ctx?.beginPath();
}

// Define la función para finalizar la posición de dibujo.
function finishedPosition() {
  drawCtx.isPainting = false;
  drawCtx.ctx?.closePath();
}

// Define la función para dibujar en el lienzo.
function draw(event: MouseEvent | TouchEvent) {
  if (!drawCtx.isPainting) return;
  const { x, y } = getPaintPosition(event);

  drawCtx.ctx?.lineTo(x, y);
  drawCtx.ctx?.stroke();
  isDraw.value = true;
}

// Define la función para limpiar el lienzo.
function clear() {
  if (!canvasDraw.value) return;
  drawCtx.ctx?.clearRect(0, 0, canvasDraw.value.width, canvasDraw.value.height);
  isDraw.value = false;
}
</script>

<template>
  <!-- Define el marcado para el popup de firma. -->
  <sign-popup
    :title="$t('create_signature_file')"
    @child-mounted="setCanvas"
  >
    <!-- Define la barra de herramientas para seleccionar el color y limpiar el lienzo. -->
    <ul class="toolbar">
      <li>
        <sign-icon
          :name="currentTool === 'black' ? 'color_black_h' : 'color_black'"
          class="w-7 h-7"
          hover-change-svg
          @click="selectTool('black')"
        />
      </li>
      <li>
        <sign-icon
          :name="currentTool === 'blue' ? 'color_blue_h' : 'color_blue'"
          class="w-7 h-7"
          hover-change-svg
          @click="selectTool('blue')"
        />
      </li>
      <li>
        <sign-icon
          :name="currentTool === 'red' ? 'color_red_h' : 'color_red'"
          class="w-7 h-7"
          hover-change-svg
          @click="selectTool('red')"
        />
      </li>
      <li>
        <sign-icon
          name="trash"
          class="w-7 h-7"
          @click="clear"
        />
      </li>
    </ul>
    <!-- Define el lienzo para dibujar la firma. -->
    <canvas
      ref="canvasDraw"
      class="bg-secondary-tint border-2 border-gray-30 rounded-[20px] mb-6"
      @mousedown.prevent="startPosition"
      @mouseup="finishedPosition"
      @mouseleave="finishedPosition"
      @mousemove="draw"
      @touchstart.prevent="startPosition"
      @touchend="finishedPosition"
      @touchcancel="finishedPosition"
      @touchmove="draw"
    ></canvas>
    <!-- Define los botones para cancelar o confirmar la firma. -->
    <div class="flex justify-between md:justify-evenly">
      <button
        class="btn btn_base"
        @click="emit('close')"
      >
        {{ $t('cancel') }}
      </button>
      <button
        class="btn btn_primary"
        :disabled="!isDraw"
        @click="addSignature"
      >
        {{ $t('confirm') }}
      </button>
    </div>
  </sign-popup>
</template>