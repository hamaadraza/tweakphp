<script setup lang="ts">
  import Divider from '../components/Divider.vue'
  import TextInput from '../components/TextInput.vue'
  import { onBeforeUnmount, onMounted, Ref, ref } from 'vue'
  import PrimaryButton from '../components/PrimaryButton.vue'
  import SelectInput from '../components/SelectInput.vue'
  import { ConnectionConfig } from '../../types/kubectl.type'
  import events from '../events'
  import DropDown from '../components/DropDown.vue'
  import DropDownItem from '../components/DropDownItem.vue'
  import { useSettingsStore } from '../stores/settings'
  import { useKubectlStore } from '../stores/kubectl'
  import { ActionReply } from '../../types/client.type'

  const kubectlStore = useKubectlStore()
  const settingsStore = useSettingsStore()
  const emit = defineEmits(['done'])
  const props = defineProps({
    id: {
      type: Number,
      required: false,
    },
  })
  const contexts = ref<string[]>([])
  const namespaces = ref<string[]>([])

  const colors = [
    'slate',
    'gray',
    'red',
    'orange',
    'amber',
    'yellow',
    'lime',
    'green',
    'emerald',
    'teal',
    'cyan',
    'sky',
    'blue',
    'indigo',
    'violet',
    'purple',
    'fuchsia',
    'pink',
    'rose',
  ]
  const form: Ref<ConnectionConfig> = ref({
    type: 'kubectl',
    id: Date.now(),
    name: '',
    color: 'rose',
    context: '',
    namespace: '',
    pod: '',
    path: '',
    php: undefined,
    client_path: undefined,
  })

  onMounted(() => {
    events.addEventListener('client.action.reply', actionReply)
    getContexts()
    if (props.id) {
      const connection = kubectlStore.getConnection(props.id)
      if (connection) {
        form.value = { ...connection }
        getNamespaces()
      }
    }
  })

  const getContexts = () => {
    window.ipcRenderer.send('client.action', {
      type: 'getContexts',
      connection: { ...form.value },
    })
  }

  const getNamespaces = () => {
    window.ipcRenderer.send('client.action', {
      type: 'getNamespaces',
      connection: { ...form.value },
    })
  }

  const save = () => {
    if (props.id) {
      kubectlStore.updateConnection(props.id, form.value)
      emit('done')
      return
    }
    kubectlStore.addConnection(form.value)
    emit('done')
  }

  const actionReply = (e: any) => {
    const reply = e.detail as ActionReply
    if (reply.error) {
      alert(reply.error)
      return
    }

    if (reply.type === 'getContexts') {
      contexts.value = reply.result
    }

    if (reply.type === 'getNamespaces') {
      namespaces.value = reply.result
    }
  }

  onBeforeUnmount(() => {
    events.removeEventListener('client.action.reply', actionReply)
  })
</script>

<template>
  <div class="mt-3 w-full mx-auto">
    <form class="mx-auto space-y-3">
      <div class="grid grid-cols-2 items-center">
        <div>Name</div>
        <div class="flex items-center justify-between">
          <TextInput class="flex-grow mr-3" id="name" v-model="form.name" placeholder="production-server" />
          <DropDown align="right" class="flex-grow-0">
            <template #trigger>
              <div
                class="!w-full h-7 text-sm border-transparent py-1 px-2 outline focus:!outline-primary-500 rounded-md flex items-center"
                :style="{
                  backgroundColor: settingsStore.colors.backgroundLight,
                  color: settingsStore.colors.foreground,
                  outlineColor: settingsStore.colors.border,
                }"
              >
                <div class="size-4 rounded-full" :class="[`bg-${form.color}-500`]"></div>
              </div>
            </template>
            <div class="space-y-1">
              <DropDownItem
                v-for="color in colors"
                :key="`color-${color}`"
                @click="form.color = color"
                class="flex items-center"
              >
                <span class="size-4 rounded-full mr-1" :class="[`bg-${color}-500`]"></span>
                <span>{{ color }}</span>
              </DropDownItem>
            </div>
          </DropDown>
        </div>
      </div>
      <Divider />
      <div class="grid grid-cols-2 items-center">
        <div>Context</div>
        <SelectInput id="context" v-model="form.context" placeholder="Select context" @change="getNamespaces()">
          <option v-for="context in contexts" :value="context" :key="`context-${context}`">{{ context }}</option>
        </SelectInput>
      </div>
      <Divider />
      <div class="grid grid-cols-2 items-center">
        <div>Namespace</div>
        <SelectInput id="namespace" v-model="form.namespace" placeholder="Select namespace">
          <option v-for="namespace in namespaces" :value="namespace" :key="`namespace-${namespace}`">
            {{ namespace }}
          </option>
        </SelectInput>
      </div>
      <Divider />
      <div class="grid grid-cols-2 items-center">
        <div>Working Directory</div>
        <TextInput id="path" v-model="form.path" placeholder="/var/www" />
      </div>
      <Divider />
      <div class="flex items-center justify-end">
        <PrimaryButton @click="save"> Save </PrimaryButton>
      </div>
    </form>
  </div>
</template>

<style scoped></style>
