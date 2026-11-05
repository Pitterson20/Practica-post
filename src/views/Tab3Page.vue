<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Crear Post</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <ion-card>
        <ion-card-header>
          <ion-card-title>Nuevo post</ion-card-title>
        </ion-card-header>

        <ion-card-content>
          <ion-button
            expand="block"
            fill="outline"
            class="ion-margin-bottom"
            :disabled="escaneando"
            @click="escanearQr"
          >
            <ion-icon slot="start" :icon="qrCodeOutline" />
            {{ escaneando ? 'Escaneando...' : 'Escanear QR' }}
          </ion-button>

          <ion-input v-model="title" label="Título" label-placement="floating" fill="outline" placeholder="foo" />

          <ion-textarea
            v-model="body"
            class="ion-margin-top"
            label="Cuerpo"
            label-placement="floating"
            fill="outline"
            :auto-grow="true"
            placeholder="bar"
          />

          <ion-select
            v-model="userId"
            class="ion-margin-top"
            label="Usuario"
            label-placement="floating"
            fill="outline"
            :disabled="cargandoUsuarios"
            :placeholder="cargandoUsuarios ? 'Cargando usuarios...' : 'Selecciona un usuario'"
          >
            <ion-select-option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.id">
              {{ usuario.name }} — {{ usuario.email }}
            </ion-select-option>
          </ion-select>

          <ion-button
            expand="block"
            class="ion-margin-top"
            :disabled="cargando || cargandoUsuarios || !title || !body || userId === null"
            @click="crearPost"
          >
            <ion-spinner v-if="cargando" name="crescent" />
            <span v-else>Crear</span>
          </ion-button>

          <ion-text v-if="error" color="danger">
            <p>{{ error }}</p>
          </ion-text>
        </ion-card-content>
      </ion-card>

      <ion-card v-if="respuesta">
        <ion-card-header>
          <ion-card-title>Respuesta del servidor</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <pre class="respuesta-json">{{ JSON.stringify(respuesta, null, 2) }}</pre>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption,
  IonButton,
  IonSpinner,
  IonText,
  IonIcon,
} from '@ionic/vue';
import { CapacitorBarcodeScanner, CapacitorBarcodeScannerTypeHint } from '@capacitor/barcode-scanner';
import { qrCodeOutline } from 'ionicons/icons';
import { onMounted, ref } from 'vue';
import type { Posts } from '@/interfaces/Posts';
import type { Usuarios } from '@/interfaces/usuarios';

type NuevoPost = Omit<Posts, 'id'>;

const title = ref('');
const body = ref('');
const userId = ref<number | null>(null);
const usuarios = ref<Usuarios[]>([]);

const cargandoUsuarios = ref(true);
const cargando = ref(false);
const escaneando = ref(false);
const error = ref('');
const respuesta = ref<Posts | null>(null);

async function cargarUsuarios() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    if (!response.ok) throw new Error(`Error ${response.status}`);
    usuarios.value = await response.json();
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'No se pudieron cargar los usuarios';
  } finally {
    cargandoUsuarios.value = false;
  }
}

async function escanearQr() {
  error.value = '';
  escaneando.value = true;

  try {
    const resultado = await CapacitorBarcodeScanner.scanBarcode({
      hint: CapacitorBarcodeScannerTypeHint.QR_CODE,
      scanInstructions: 'Apunta al QR con los datos del post',
    });

    let datos: Partial<NuevoPost>;

    try {
      datos = JSON.parse(resultado.ScanResult);
    } catch {
      throw new Error('El QR no contiene un JSON válido');
    }

    const usuarioId = Number(datos.userId);

    if (
      typeof datos.title !== 'string' ||
      typeof datos.body !== 'string' ||
      !Number.isInteger(usuarioId) ||
      usuarioId <= 0
    ) {
      throw new Error('El QR debe tener los campos title, body y userId');
    }

    title.value = datos.title;
    body.value = datos.body;
    userId.value = usuarioId;
    respuesta.value = null;
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'No se pudo escanear el QR';
  } finally {
    escaneando.value = false;
  }
}

async function crearPost() {
  const usuarioSeleccionado = userId.value;
  if (usuarioSeleccionado === null) return;

  cargando.value = true;
  error.value = '';
  respuesta.value = null;

  try {
    const payload: NuevoPost = {
      title: title.value,
      body: body.value,
      userId: usuarioSeleccionado,
    };

    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'POST',
      body: JSON.stringify(payload),
      headers: {
        'Content-type': 'application/json; charset=UTF-8',
      },
    });

    if (!response.ok) throw new Error(`Error ${response.status}`);

    respuesta.value = await response.json();
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'No se pudo crear el post';
  } finally {
    cargando.value = false;
  }
}

onMounted(cargarUsuarios);
</script>

<style scoped>
.respuesta-json {
  margin: 0;
  white-space: pre-wrap;
  word-break: break-word;
}
</style>
