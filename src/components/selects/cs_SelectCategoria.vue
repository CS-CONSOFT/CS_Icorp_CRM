<template>
    <v-select
        v-model="internalSelectedCategoria"
        :items="formattedCategoria"
        item-value="value"
        :rules="props.rules"
        item-text="title"
        variant="solo-filled"
        hide-details
        @change="emitSelection"
    >
        <template v-slot:label>
            <span class="d-flex align-center" style="font-size: 12px; font-weight: 500; padding-bottom: 0.1em; color: #808080">
                {{ computedLabel }}<span v-if="props.Prm_isObrigatorio" class="text-error">*</span>
            </span>
        </template>
    </v-select>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';
import { getListCategorias } from '../../services/basico/categoria/bb029_categoria';
import type { Lista_bb029 } from '../../types/basico/bb029_categoria';
import type { User } from '../../types/login/Login';

function getUserFromLocalStorage(): User | null {
    const storedUser = localStorage.getItem('user');

    if (storedUser) {
        try {
            return JSON.parse(storedUser) as User;
        } catch (e) {
            console.error('Erro ao parsear o usuário do localStorage', e);
            return null;
        }
    }

    return null;
}
const user = getUserFromLocalStorage();

const emit = defineEmits<{
    (e: 'update:modelValue', value: string | null): void;
}>();

const props = defineProps<{
    Prm_etiqueta?: string;
    Prm_isObrigatorio: boolean;
    rules?: Array<(v: string) => true | string>;
}>();

const categoria = ref<Lista_bb029[]>([]);
const internalSelectedCategoria = ref<string | null>(null);
const errors = ref<string[]>([]);

const computedLabel = computed(() => props.Prm_etiqueta || 'Selecione uma categoria');

const formattedCategoria = computed(() => {
    return [
        { title: '', value: null },
        ...categoria.value.map((item) => ({
            title: item.BB029_Categoria,
            value: item.ID
        }))
    ];
});

const fetchCategoria = async () => {
    try {
        const response = await getListCategorias(user?.TenantId);
        if (response.status === 200) {
            categoria.value = response.data.Lista_bb029;
            if (internalSelectedCategoria.value) {
                const selected = categoria.value.find((categoria) => categoria.ID === internalSelectedCategoria.value);
                if (selected) {
                    internalSelectedCategoria.value = selected.ID;
                }
            }
        } else {
            console.error('Erro ao buscar as categorias', response.statusText);
        }
    } catch (error) {
        console.error('Erro ao buscar as categorias:', error);
    }
};

onMounted(async () => {
    getUserFromLocalStorage();
    await fetchCategoria();
});

watch(internalSelectedCategoria, (newVal) => {
    emit('update:modelValue', newVal);
});

function emitSelection() {
    emit('update:modelValue', internalSelectedCategoria.value);
}

function validate() {
    errors.value = [];
    if (props.rules) {
        for (const rule of props.rules) {
            const result = rule(internalSelectedCategoria.value || '');
            if (result !== true) {
                errors.value.push(result);
            }
        }
    }
    return errors.value.length === 0;
}

defineExpose({ validate });
</script>
