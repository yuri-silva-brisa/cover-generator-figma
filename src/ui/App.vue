<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, computed } from "vue";

import MonitorIcon from "./assets/monitor.svg";
import SmartphoneIcon from "./assets/smartphone.svg";
import GlobeIcon from "./assets/globe.svg";
import BrisaIcon from "./assets/brisa-icon.svg";

// --- CONSTANTES ---
const STATUS_OPTIONS = [
  { label: "Em andamento", colorToken: "status-info", color: "#0EA5E9" },
  { label: "Não iniciado", colorToken: "status-muted", color: "#6B7280" },
  { label: "Pausado", colorToken: "status-warning", color: "#F59E0B" },
  { label: "Concluído", colorToken: "status-success", color: "#10B981" },
  { label: "Cancelado", colorToken: "status-danger", color: "#EF4444" },
];

const DEVICE_OPTIONS = [
  { label: "Web", IconComponent: MonitorIcon },
  { label: "Mobile", IconComponent: SmartphoneIcon },
  { label: "Multiplataforma", IconComponent: GlobeIcon },
];

const DESIGNER_LIST = [
  { id: 1, name: "Anderson Nâgelo", role: "Designer", bgColor: "#4A90E2" },
  { id: 2, name: "Bárbara Barreto", role: "Designer", bgColor: "#50E3C2" },
  {
    id: 3,
    name: "Jéter Megaron Monteiro",
    role: "Designer",
    bgColor: "#F5A623",
  },
  { id: 4, name: "Lucas Alves", role: "Designer", bgColor: "#9013FE" },
  { id: 5, name: "Maria Eduarda", role: "Designer", bgColor: "#BD10E0" },
  { id: 6, name: "Ana Carolyne", role: "Designer", bgColor: "#E84F3C" },
];

// --- STATES ---
const title = ref("Novo Projeto de Interface");
const description = ref("Desenvolvimento de um novo fluxo para o usuário.");
const team = ref("Time de Produto");
const status = ref(STATUS_OPTIONS[0]);
const device = ref(DEVICE_OPTIONS[0]);
const selectedDesigners = ref([DESIGNER_LIST[0], DESIGNER_LIST[1]]);
const searchTerm = ref("");
const isDesignerDropdownOpen = ref(false);

const designerDropdownRef = ref(null);

// --- FILTRO DE DESIGNERS ---
const filteredDesigners = computed(() =>
  DESIGNER_LIST.filter(
    (d) =>
      d.name.toLowerCase().includes(searchTerm.value.toLowerCase()) &&
      !selectedDesigners.value.find((sd) => sd.id === d.id)
  )
);

const toggleDesigner = (designer) => {
  if (selectedDesigners.value.find((d) => d.id === designer.id)) {
    selectedDesigners.value = selectedDesigners.value.filter(
      (d) => d.id !== designer.id
    );
  } else {
    selectedDesigners.value = [...selectedDesigners.value, designer];
  }
};

// --- CLICK FORA DO DROPDOWN ---
const handleClickOutside = (event) => {
  if (
    designerDropdownRef.value &&
    !designerDropdownRef.value.contains(event.target)
  ) {
    isDesignerDropdownOpen.value = false;
  }
};

onMounted(() => {
  document.addEventListener("mousedown", handleClickOutside);

  window.onmessage = (event) => {
    const msg = event.data.pluginMessage;
    if (msg?.type === "export-result") {
      const blob = new Blob([msg.bytes], { type: "image/png" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = msg.filename || "Capa_Exportada.png";
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    }
  };
});

onBeforeUnmount(() => {
  document.removeEventListener("mousedown", handleClickOutside);
});

// --- GERAÇÃO DE CAPA ---
const onGenerateCover = () => {
  if (!title.value) return;
  parent.postMessage(
    {
      pluginMessage: {
        type: "generate-cover",
        data: {
          title: title.value,
          description: description.value,
          team: team.value,
          status: status.value,
          device: device.value,
          selectedDesigners: selectedDesigners.value,
        },
      },
    },
    "*"
  );
};
</script>

<template>
  <div
    class="p-8 bg-neutral-50 grid grid-cols-5 grid-rows-5 gap-12 h-screen font-[Poppins]"
  >
    <div class="col-span-2 row-span-5">
      <h1 class="text-lg font-bold text-neutral-900 mb-4">Gerador de Capas</h1>

      <div class="flex-grow overflow-y-auto space-y-4">
        <!-- Título -->
        <div>
          <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
            Título do Projeto *
          </label>
          <input
            type="text"
            v-model="title"
            maxlength="50"
            placeholder="Digite aqui"
            class="block w-full border border-neutral-300 rounded-md shadow-sm py-2 px-3 text-ui-base focus:outline-none focus:ring-1 focus:ring-primary-500 focus:border-primary-500"
          />
        </div>

        <!-- Descrição -->
        <div>
          <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
            Descrição
          </label>
          <textarea
            v-model="description"
            maxlength="150"
            rows="2"
            class="block w-full border border-neutral-300 rounded-md shadow-sm py-2 px-3 text-ui-base focus:outline-none focus:ring-1 focus:ring-primary-500 focus:border-primary-500"
          />
        </div>

        <!-- Time -->
        <div>
          <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
            Time Responsável
          </label>
          <input
            type="text"
            v-model="team"
            class="block w-full border border-neutral-300 rounded-md shadow-sm py-2 px-3 text-ui-base focus:outline-none focus:ring-1 focus:ring-primary-500 focus:border-primary-500"
          />
        </div>

        <div class="grid grid-cols-2 gap-4">
          <!-- Status -->
          <div>
            <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
              Status
            </label>
            <select
              v-model="status"
              class="block w-full border border-neutral-300 rounded-md shadow-sm py-2 px-3"
            >
              <option
                v-for="opt in STATUS_OPTIONS"
                :key="opt.label"
                :value="opt"
              >
                {{ opt.label }}
              </option>
            </select>
          </div>

          <!-- Dispositivo -->
          <div>
            <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
              Dispositivo
            </label>
            <select
              v-model="device"
              class="block w-full border border-neutral-300 rounded-md shadow-sm py-2 px-3"
            >
              <option
                v-for="opt in DEVICE_OPTIONS"
                :key="opt.label"
                :value="opt"
              >
                {{ opt.label }}
              </option>
            </select>
          </div>
        </div>

        <!-- Designers -->
        <div class="relative" ref="designerDropdownRef">
          <label class="block text-ui-sm font-regular text-neutral-700 mb-1">
            Designers
          </label>
          <div
            class="mt-1 p-2 border border-neutral-300 rounded-md min-h-[42px] flex flex-wrap gap-2 cursor-pointer"
            @click="isDesignerDropdownOpen = !isDesignerDropdownOpen"
          >
            <span
              v-if="selectedDesigners.length === 0"
              class="text-neutral-700 opacity-50 px-1 py-1"
            >
              Selecione...
            </span>
            <span
              v-for="d in selectedDesigners"
              :key="d.id"
              class="flex items-center gap-2 bg-neutral-200 text-neutral-700 text-ui-sm font-regular px-2 py-1 rounded-full"
            >
              {{ d.name }}
              <button
                @click.stop="toggleDesigner(d)"
                class="text-neutral-700 opacity-50 hover:opacity-100"
              >
                ×
              </button>
            </span>
          </div>

          <div
            v-if="isDesignerDropdownOpen"
            class="absolute z-20 w-full mt-1 bg-white border border-neutral-300 rounded-md shadow-lg max-h-52 overflow-y-auto"
          >
            <input
              type="text"
              placeholder="Buscar por nome..."
              v-model="searchTerm"
              class="w-full p-2 border-b sticky top-0 text-ui-base focus:outline-none"
            />
            <ul>
              <li
                v-for="d in filteredDesigners"
                :key="d.id"
                @click="
                  toggleDesigner(d);
                  searchTerm = '';
                "
                class="p-2 hover:bg-primary-500/10 cursor-pointer text-ui-base"
              >
                {{ d.name }}
              </li>
              <li
                v-if="filteredDesigners.length === 0"
                class="p-2 text-center text-ui-sm text-neutral-700 opacity-50"
              >
                Nenhum designer encontrado
              </li>
            </ul>
          </div>
        </div>
      </div>

      <!-- Botão -->
      <div class="mt-auto pt-4 flex items-center gap-3">
        <button
          @click="onGenerateCover"
          :disabled="!title"
          class="w-full font-regular py-2 px-4 rounded-md transition-colors bg-blue-500 text-white hover:bg-blue-700 disabled:bg-neutral-300"
        >
          Gerar Capa e Exportar PNG
        </button>
      </div>
    </div>

    <!-- PREVIEW -->
    <div class="col-span-3 row-span-5 col-start-3">
      <h2 class="text-ui-sm font-semibold text-neutral-700 mb-2">
        Preview em Tempo Real
      </h2>
      <div
        class="aspect-video w-full rounded-lg shadow-md p-6 flex flex-col relative bg-gradient-to-br from-blue-800 to-sky-600 text-white"
      >
        <!-- Status e Device -->
        <div
          class="absolute top-4 left-4 flex flex-col items-start space-y-2 z-10"
        >
          <span
            class="px-4 py-1 rounded-full text-ui-sm font-bold"
            :style="{ backgroundColor: status.color }"
          >
            {{ status.label }}
          </span>
          <div class="flex gap-2 items-center">
            <component :is="device.IconComponent" class="w-6 h-6 text-white" />
            {{ device.label }}
          </div>
        </div>

        <!-- Título e Descrição -->
        <div
          class="flex-grow flex flex-col items-center justify-center text-center -mt-4"
        >
          <h3 class="text-4xl font-bold break-words px-16">
            {{ title || "Título do Projeto" }}
          </h3>
          <p class="text-lg opacity-90 break-words mt-2 px-16">
            {{ description || "Descrição de apoio" }}
          </p>
        </div>

        <!-- Logo -->
        <div class="w-24">
          <brisa-icon></brisa-icon>
        </div>

        <!-- Avatares -->
        <div class="absolute bottom-4 right-6 flex items-center">
          <div class="flex -space-x-2">
            <div
              v-for="d in selectedDesigners.slice(0, 4)"
              :key="d.id"
              :title="d.name"
              class="w-10 h-10 rounded-full flex items-center justify-center text-white font-bold text-sm border-2 border-blue-400"
              :style="{ backgroundColor: d.bgColor }"
            >
              {{
                d.name
                  .split(" ")
                  .map((n) => n[0])
                  .join("")
                  .substring(0, 2)
              }}
            </div>
            <div
              v-if="selectedDesigners.length > 4"
              class="w-10 h-10 rounded-full flex items-center justify-center bg-neutral-700 text-white font-bold text-sm border-2 border-blue-400"
            >
              +{{ selectedDesigners.length - 4 }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
