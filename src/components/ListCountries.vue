<template>
  <DataTable
    paginatorTemplate="RowsPerPageDropdown PrevPageLink CurrentPageReport NextPageLink"
    class="table p-datatable-sm"
    v-model:filters="filters"
    filterDisplay="menu"
    :globalFilterFields="['name.official', 'region']"
    :value="posts"
    paginator
    :rows="10"
    :rowsPerPageOptions="[10, 25, 50, 100]"
  >
    <template #header>
      <div class="table-header">
        <InputText
          v-model="filters['global'].value"
          placeholder="Keyword Search"
        />
      </div>
    </template>
    <template #empty> 
      <ProgressSpinner />
    </template>
    <Column :style="'width: 10%'" field="flag" header="Flag"></Column>
    <Column field="name.official" header="Name">
      <template #body="{ data }">
        {{ data.name.official }}
      </template>
      <template #filter="{ filterModel, filterCallback }">
        <InputText
          v-model="filterModel.value"
          type="text"
          @input="filterCallback()"
          class="p-column-filter"
          placeholder="Search by name"
        />
      </template>
    </Column>
    <Column field="region" header="Region">
      <template #body="{ data }">
        {{ data.region }}
      </template>
      <template #filter="{ filterModel, filterCallback }">
        <InputText
          v-model="filterModel.value"
          type="text"
          @input="filterCallback()"
          class="p-column-filter"
          placeholder="Search by region"
        />
      </template>
    </Column>
    <Column field="population" header="Population"></Column>
    <Column header="Language">
      <template #body="slotProps">
        <div v-if="Object.keys(slotProps.data.languages).length <= 1">
          <Chip :label="Object.values(slotProps.data.languages)[0]" />
        </div>
        <div v-if="Object.keys(slotProps.data.languages).length > 1">
          <Chip
            :style="'cursor:pointer'"
            type="button"
            icon="pi pi-image"
            label="More"
            @click="openChip"
          />

          <OverlayPanel ref="op">
            <Chip
              v-for="language in slotProps.data.languages"
              :key="language"
              :label="language"
            />
          </OverlayPanel>
        </div>
      </template>
    </Column>
  </DataTable>
</template>

<script>
import ProgressSpinner from 'primevue/progressspinner';
import DataTable from "primevue/datatable";
import OverlayPanel from "primevue/overlaypanel";
import Column from "primevue/column";
import Chip from "primevue/chip";
import InputText from "primevue/inputtext";
import { ref, onMounted } from "vue";
import axios from "axios";
import { FilterMatchMode, FilterOperator } from "primevue/api";

export default {
  components: {
    Column,
    DataTable,
    Chip,
    InputText,
    OverlayPanel,
    ProgressSpinner,
  },
  setup() {
    const posts = ref([]);
    const filters = ref({
      global: { value: null },
      "name.official": { value: null },
      region: { value: null },
    });
    const op = ref();
    const openChip = (event) => {
      op.value.toggle(event);
    };

    const initFilters = () => {
      filters.value = {
        global: { value: null },
        "name.official": {
          operator: FilterOperator.AND,
          constraints: [
            { value: null, matchMode: FilterMatchMode.STARTS_WITH },
          ],
        },
        region: {
          operator: FilterOperator.AND,
          constraints: [
            { value: null, matchMode: FilterMatchMode.STARTS_WITH },
          ],
        },
      };
    };

    const getPosts = async () => {
      try {
        const response = await axios.get("http://localhost:8080/api/all");
        posts.value = response.data;
      } catch (error) {
        console.error("Помилка отримання даних:", error);
      }
    };

    onMounted(() => {
      getPosts();
    });

    initFilters();

    return {
      posts,
      filters,
      openChip,
      op,
    };
  },
};
</script>
<style lang="scss">
.p-datatable-emptymessage td{
  text-align: center;
}
</style>
