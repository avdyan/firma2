<script lang="ts" setup>
// Importa las funciones necesarias de vue, vue-i18n y otros archivos locales.
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import SignPopup from '@/components/SignPopup.vue';
import SignPassword from '@/components/SignPassword.vue';
import { useConfigStore } from '@/store';
import { downloadPDF } from '@/utils/jspdf';
import toast from '@/utils/toast';
import type { PDF } from '@/types/pdf';

// Define las propiedades que se esperan para este componente.
interface Props {
  file: PDF | null;
}

// Define las propiedades y emisores de eventos para este componente.
const props = defineProps<Props>();
const emit = defineEmits(['close-encrypt-popup']);

// Define las referencias reactivas para las contraseñas.
const password = ref('');
const confirmPassword = ref('');

// Utiliza la función de internacionalización y la tienda de configuración.
const { t } = useI18n();
const { toggleLoading, setLoadingCompleteness } = useConfigStore();

// Define la función de descarga que se encarga de la lógica de encriptación y descarga del PDF.
function download(isEncrypt: boolean) {
  // Si se solicita la encriptación y las contraseñas están vacías, muestra un mensaje de error y termina la función.
  if (isEncrypt && (!password.value || !confirmPassword.value)) {
    toast.showToast(t('password_required'), 'error');
    return;
  }
  // Si se solicita la encriptación y las contraseñas no coinciden, muestra un mensaje de error y termina la función.
  if (isEncrypt && password.value !== confirmPassword.value) {
    toast.showToast(t('password_not_match'), 'error');
    return;
  }
  // Emite el evento para cerrar el popup de encriptación.
  emit('close-encrypt-popup');
  // Solicita un nuevo frame de animación y luego realiza la descarga del PDF.
  window.requestAnimationFrame(async () => {
    // Si no hay un archivo para descargar, termina la función.
    if (!props.file) return;
    // Define la contraseña del usuario en función de si se solicita la encriptación.
    const userPassword = isEncrypt ? password.value : '';

    // Muestra la pantalla de carga y realiza la descarga del PDF.
    toggleLoading({ isShow: true, title: 'download', content: 'file_downloading', isProcess: true });
    await downloadPDF(props.file, setLoadingCompleteness, userPassword);
    // Oculta la pantalla de carga.
    toggleLoading({ isShow: false });
  });
}
</script>

<template>
  <!-- Define el marcado para el popup de encriptación. -->
  <SignPopup :title="$t('encryption')">
    <!-- Muestra un mensaje al usuario. -->
    <p class="text-center my-5">{{ $t('prompt.encryption_file') }}</p>
    <!-- Define los campos de entrada para las contraseñas. -->
    <div class="mb-4 flex flex-col justify-center items-center gap-3">
      <sign-password
        v-model="password"
        placeholder="placeholder.password"
      />
      <sign-password
        v-model="confirmPassword"
        placeholder="placeholder.confirm_password"
      />
    </div>
    <!-- Define los botones para cancelar o confirmar la encriptación. -->
    <div class="flex justify-between md:justify-evenly">
      <button
        class="btn btn_base"
        @click="() => download(false)"
      >
        {{ $t('not_yet') }}
      </button>
      <button
        class="btn btn_primary"
        @click="() => download(true)"
      >
        {{ $t('confirm') }}
      </button>
    </div>
  </SignPopup>
</template>