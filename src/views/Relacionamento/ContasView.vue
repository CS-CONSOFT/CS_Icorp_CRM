<template>
    <v-container fluid class="border-sm rounded bg- ma-0">
        <div>
            <v-row class="pa-4 pt-7">
                <p class="text-h2 mb-5">Contas</p>
                <v-divider></v-divider>
            </v-row>

            <div class="px-4 mb-5">
                <v-row class="align-center">
                    <v-col cols="3">
                        <v-text-field
                            v-model="search"
                            append-inner-icon="mdi-magnify"
                            label="Pesquisar"
                            variant="solo-filled"
                            single-line
                            hide-details
                            clearable
                        />
                    </v-col>
                    <v-spacer></v-spacer>

                    <v-col cols="auto" class="d-flex align-center">
                        <v-btn v-if="active" prepend-icon="mdi-cancel" flat class="mr-2 bg-error" @click="fetchInactive">Inativos</v-btn>
                        <v-btn v-else prepend-icon="mdi-delete-empty" flat class="bg-secondary mr-2" @click="fetchActive">Ativos</v-btn>
                        <BtnAdicionar @click="redirectToCreate" />
                    </v-col>

                    <v-col cols="auto" class="d-flex align-center justify-end">
                        <v-btn icon @click="toggleView">
                            <v-icon>{{ isListView ? 'mdi-view-grid' : 'mdi-view-list' }}</v-icon>
                        </v-btn>
                    </v-col>
                </v-row>
            </div>

            <v-card class="border-sm mt-1" elevation="0" v-if="isListView">
                <v-data-table
                    :headers="headers"
                    :items="items"
                    :items-per-page="itemsPerPage"
                    :loading="loading"
                    @update:items-per-page="updateItemsPerPage"
                    hide-default-footer
                    loading-text="Carregando dados..."
                    class="month-table"
                    :search="search"
                >
                    <template v-slot:bottom>
                        <v-row class="d-flex align-center">
                            <v-col cols="2">
                                <v-select
                                    v-model="itemsPerPage"
                                    class="pa-2 mr-4"
                                    label="Itens por página"
                                    :items="[5, 10, 15, 25, 50]"
                                    hide-details
                                ></v-select>
                            </v-col>
                            <v-col cols="8" class="d-flex justify-center">
                                <Pagination
                                    :currentPage="currentPage"
                                    :totalPages="totalPages"
                                    :itemsPerPage="itemsPerPage"
                                    :totalItems="totalItems"
                                    @update:currentPage="updatePage"
                                />
                            </v-col>
                        </v-row>
                    </template>
                    <template v-slot:no-data>
                        <p>Nenhum dado encontrado.</p>
                    </template>
                    <template v-slot:progress>
                        <v-progress-linear v-if="loading" color="blue" height="10" indeterminate></v-progress-linear>
                    </template>
                    <template v-slot:item.Ativo="{ item }">
                        <v-chip :color="item.Ativo ? 'success' : 'error'" class="font-weight-bold" rounded="lg" size="small">{{
                            item.Ativo ? 'Ativo' : 'Inativo'
                        }}</v-chip>
                    </template>
                    <template v-slot:item.actions="{ item }">
                        <v-menu v-model="item.menu" :close-on-content-click="false" offset-y>
                            <template v-slot:activator="{ props }">
                                <v-icon small v-bind="props" class="v-btn-icon">mdi-dots-vertical</v-icon>
                            </template>

                            <v-list style="width: 210px">
                                <v-list-item v-for="(opcao, index) in opcoesMenu" :key="index" @click="opcao.acao(item)">
                                    <div class="d-flex">
                                        <v-col cols="3">
                                            <v-icon>{{ opcao.icone(item) }}</v-icon>
                                        </v-col>
                                        <v-col cols="9">
                                            <v-list-item-title>{{ opcao.nome(item) }}</v-list-item-title>
                                        </v-col>
                                    </div>
                                </v-list-item>
                            </v-list>
                        </v-menu>
                    </template>
                </v-data-table>
            </v-card>

            <v-container fluid v-else class="pa-0">
                <v-row>
                    <v-col cols="12">
                        <v-row class="d-flex">
                            <v-col
                                cols="12"
                                md="4"
                                v-for="item in items"
                                :key="item.ID"
                                class="mb-1"
                                @update:items-per-page="updateItemsPerPage"
                            >
                                <v-card class="border-sm border-opacity-50 pa-0 mt-1" flat>
                                    <v-card-title class="font-weight-bold bg-lightprimary">{{ item.Nome }}</v-card-title>
                                    <v-card-text class="mt-5">
                                        <div class="pa-0 my-1">
                                            <v-chip
                                                :color="item.IsActive ? 'success' : 'error'"
                                                class="font-weight-bold"
                                                rounded="lg"
                                                size="small"
                                                >{{ item.IsActive ? 'Ativo' : 'Inativo' }}</v-chip
                                            >
                                        </div>
                                    </v-card-text>
                                    <v-card-actions class="pl-4">
                                        <v-icon small @click="" class="v-btn-icon">mdi-pencil</v-icon>
                                        <v-icon small @click="" class="v-btn-icon">mdi-delete</v-icon>
                                        <v-icon small @click="" class="v-btn-icon">{{ active ? 'mdi-cancel' : 'mdi-check' }}</v-icon>
                                    </v-card-actions>
                                </v-card>
                            </v-col>
                        </v-row>
                    </v-col>
                </v-row>
                <v-row class="d-block-inline align-center">
                    <v-col cols="2">
                        <v-select
                            v-model="itemsPerPage"
                            class="pa-2 mr-4"
                            label="Itens por página"
                            :items="[5, 10, 15, 25, 50]"
                            hide-details
                        ></v-select>
                    </v-col>
                    <v-col cols="8" class="d-flex justify-center">
                        <Pagination
                            :currentPage="currentPage"
                            :totalPages="totalPages"
                            :itemsPerPage="itemsPerPage"
                            :totalItems="totalItems"
                            @update:currentPage="updatePage"
                        />
                    </v-col>
                </v-row>
            </v-container>
        </div>
    </v-container>

    <v-dialog v-model="confirmDialog" max-width="400">
        <v-card>
            <v-card-title class="text-h5 pa-4 bg-error">Confirmar Exclusão</v-card-title>
            <v-card-text>Tem certeza de que deseja excluir esta conta?</v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <BtnCancelar @click="cancelDelete" />
                <BtnExcluir @click="deleteContaConfirmed" />
            </v-card-actions>
        </v-card>
    </v-dialog>

    <v-dialog v-model="confirmSoftDeleteDialog" max-width="400">
        <v-card>
            <v-card-title class="text-h5 pa-4 bg-secondary"> Confirmar {{ active ? 'Inativação' : 'Ativação' }} </v-card-title>
            <v-card-text> Tem certeza de que deseja {{ active ? 'inativar' : 'ativar' }} esta conta? </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <BtnCancelar @click="cancelSoftDelete" />
                <BtnIsActive :IsActive="active" @click="softDeleteContaConfirmed" />
            </v-card-actions>
        </v-card>
    </v-dialog>

    <v-snackbar v-model="snackbar" :timeout="3000" top v-bind:color="snackbarColor" multi-line>
        {{ snackbarMessage }}
        <template v-slot:actions>
            <v-btn color="white" icon="mdi-close" @click="snackbar = false"></v-btn>
        </template>
    </v-snackbar>
</template>
<script setup lang="ts">
// Import de bibliotecas e etc...
import { ref, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import { validationRules } from '../../utils/ValidationRules';
import { getUserFromLocalStorage } from '../../utils/getUserStorage';
// Import de API's
import { DeleteConta, GetContasList, SoftDeleteConta } from '../../services/contas/bb012_conta';
// Import de Types
import type { AxiosResponse } from 'axios';
import type { ContaCompleta, ApiResponse, Lista_csicp_bb012 } from '../../types/crm/bb012_conta';
//Import de componentes
import Pagination from '../../components/navigation/Pagination.vue';
import BtnAdicionar from '../../components/botoes/cs_BtnAdicionar.vue';
import BtnExcluir from '../../components/botoes/cs_BtnExcluir.vue';
import BtnIsActive from '../../components/botoes/cs_BtnIsActive.vue';
import BtnCancelar from '../../components/botoes/cs_BtnCancelar.vue';

interface Item {
    ID: string;
    Nome: string;
    Endereco: string;
    Contato: string;
    Modalidade: string;
    CPF: string;
    Status: string;
    IsActive: boolean;
}

//Declaração do Header para montagem da tabela
const headers = ref([
    {
        title: 'Nome',
        value: 'Nome',
        sortable: true,
        width: '25%',
        align: 'start'
    },
    {
        title: 'Endereço',
        value: 'Endereco',
        width: '25%',
        align: 'start'
    },
    {
        title: 'Contato',
        value: 'Contato',
        width: '10%',
        align: 'start'
    },
    {
        title: 'Modalidade',
        value: 'Modalidade',
        width: '10%',
        align: 'center'
    },
    {
        title: 'CPF / IE / CNPJ',
        value: 'CPF',
        width: '10%',
        align: 'center'
    },
    {
        title: 'Status',
        value: 'Status',
        width: '10%',
        align: 'center'
    },
    {
        title: 'Ações',
        align: 'center',
        value: 'actions',
        sortable: false,
        width: '10%'
    }
]);

const items = ref<Item[]>([]);
const confirmDialog = ref(false);
const confirmSoftDeleteDialog = ref(false);
const itemToDelete = ref<Item | null>(null);
const itemToSoftDelete = ref<Item | null>(null);
const loading = ref(false);
const active = ref(true);
const count = false;
const search = ref('');

const rules = {
    codigo: [validationRules.required, validationRules.numeric],
    nome: [validationRules.required]
};

const user = getUserFromLocalStorage();
const tenant = user?.TenantId;
const router = useRouter();

//variaveis da paginação
const currentPage = ref(1);
const itemsPerPage = ref(50);
const totalItems = ref(0);
const totalPages = ref(0);

//Variavel para controlar o modo de exibição da tela
const isListView = ref(true);

//Variaveis do Snackbar
const snackbar = ref(false);
const snackbarMessage = ref('');
const snackbarColor = ref('');

//Funções
const showSnackbar = (message: string, color: string) => {
    snackbarMessage.value = message;
    snackbarColor.value = color;
    snackbar.value = true;
};

const toggleView = () => {
    isListView.value = !isListView.value;
};

const fetchActive = () => {
    active.value = true;
    fetchData();
};

const fetchInactive = () => {
    active.value = false;
    fetchData();
};

const fetchData = async () => {
    loading.value = true;
    try {
        const response: AxiosResponse<ApiResponse<ContaCompleta>> = await GetContasList(
            tenant,
            active.value,
            count,
            search.value,
            currentPage.value,
            itemsPerPage.value,
            3,
            '',
            ''
        );
        const data = response.data;
        items.value = data.Lista_csicp_bb012.map((item: Lista_csicp_bb012) => ({
            ID: item.csicp_bb012.csicp_bb012.ID,
            Nome: `${item.csicp_bb012.csicp_bb012.BB012_Codigo} - ${item.csicp_bb012.csicp_bb012.BB012_Nome_Cliente}`,
            Endereco: item.BB01206_Endereco.csicp_bb01206.BB012_Logradouro,
            Contato: item.csicp_bb012.csicp_bb012.BB012_FaxCelular,
            Modalidade: item.csicp_bb012.csicp_bb012_MRel.Label,
            CPF: item.csicp_bb012.csicp_bb001.BB001_CPF_Oficial,
            Status: item.csicp_bb012.csicp_bb012_StaCta.Label,
            IsActive: item.csicp_bb012.csicp_bb012.BB012_Is_Active
        }));

        totalItems.value = data.PageSize.cs_list_total_itens;
        totalPages.value = Math.ceil(totalItems.value / itemsPerPage.value);
    } catch (error) {
        showSnackbar('Erro ao buscar dados.', 'error');
    } finally {
        loading.value = false;
    }
};

// Array de opções do menu de ações
const opcoesMenu = [
    {
        nome: () => 'Extras',
        icone: () => 'mdi-plus-circle-outline',
        acao: async (item: { ID: any }) => {
            if (item && item.ID) {
                await router.push({
                    name: 'ContasExtras',
                    params: {
                        id: item.ID
                    }
                });
            } else {
                console.error('Item indefinido');
            }
        }
    },
    {
        nome: () => 'Editar Registro',
        icone: () => 'mdi-clipboard-edit-outline',
        acao: async (item: { ID: any }) => {
            if (item && item.ID) {
                await router.push({
                    name: 'ContasEdit',
                    params: {
                        id: item.ID
                    }
                });
            } else {
                console.error('Item indefinido');
            }
        }
    },
    {
        nome: (item: Item) => (item.IsActive ? 'Inativar' : 'Ativar'),
        icone: (item: Item) => (item.IsActive ? 'mdi-cancel' : 'mdi-check'),
        acao: (item: Item) => {
            confirmSoftDeleteDialog.value = true;
            itemToSoftDelete.value = item;
        }
    },
    {
        nome: () => 'Excluir',
        icone: () => 'mdi-delete',
        acao: (item: Item) => {
            confirmDialog.value = true;
            itemToDelete.value = item;
        }
    },
    {
        nome: () => 'Ficha',
        icone: () => 'mdi-clipboard-outline',
        acao: (item: Item) => {
            console.log('Visualizando ficha para', item);
        }
    },
    {
        nome: () => 'SMS',
        icone: () => 'mdi-cellphone',
        acao: (item: Item) => {
            console.log('Enviando SMS para', item);
        }
    }
];

const redirectToCreate = async () => {
    await router.push({
        name: 'ContasCreate'
    });
};

const cancelDelete = () => {
    confirmDialog.value = false;
};

const deleteContaConfirmed = async () => {
    if (!itemToDelete.value) return;
    try {
        await DeleteConta(tenant, itemToDelete.value.ID);
        showSnackbar('Conta excluído com sucesso', 'success');
        fetchData();
        confirmDialog.value = false;
    } catch (error) {
        showSnackbar('Erro ao excluir o conta', 'error');
        confirmDialog.value = false;
    }
};

const cancelSoftDelete = () => {
    confirmSoftDeleteDialog.value = false;
};

const softDeleteContaConfirmed = async () => {
    if (!itemToSoftDelete.value) return;
    try {
        await SoftDeleteConta(tenant, itemToSoftDelete.value.ID);
        showSnackbar(`Conta ${active.value ? 'inativado' : 'ativado'} com sucesso`, 'success');
        fetchData();
        confirmSoftDeleteDialog.value = false;
    } catch (error) {
        showSnackbar(`Erro ao ${active.value ? 'inativar' : 'ativar'} a conta`, 'error');
    }
};

const updatePage = (page: number) => {
    currentPage.value = page;
    fetchData();
};

const updateItemsPerPage = (itemsPerPageValue: number) => {
    itemsPerPage.value = itemsPerPageValue;
    currentPage.value = 1;
    fetchData();
};

watch([itemsPerPage, search], fetchData);

onMounted(() => {
    fetchData();
});
</script>
