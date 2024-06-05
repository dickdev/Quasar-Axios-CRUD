<template>
  <q-form @submit="!selectedRow.id ? addTodo() : updateTodo()" class="q-ma-md">
    <q-input outlined v-model="form.title" label="Input Field" />
    <!-- BUTTON SUBMIT OR UPDATE -->
    <br />
    <q-btn
      type="submit"
      :label="!selectedRow.id ? 'Submit' : 'Update'"
      color="primary"
    />

    <!-- BUTTON SELECTING AND DELETE TO DO -->
    <q-btn
      :loading="deleteBtnLoadingState"
      v-if="selectedRow.id"
      @click="deleteTodo()"
      class="q-ml-md"
      label="Delete"
      color="negative"
    />
  </q-form>
  <q-page class="q-ma-xl">
    <q-table
      title="Todos"
      :rows="rows"
      class="bg-blue-grey"
      :columns="columns"
      row-key="name"
      @row-click="onRowClick"
    />
  </q-page>
</template>

<script>
import { ref } from "vue";
import { api } from "../boot/axios";

export default {
  setup() {
    let rows = ref([]);
    let columns = ref([
      {
        name: "title",
        label: "Title",
        align: "left",
        field: "title",
      },
      {
        name: "completed",
        label: "Completed",
        align: "left",
        field: "completed",
      },
    ]);
    //GET API REQUEST TO DO
    const getTodos = () => {
      api.get("https://jsonplaceholder.typicode.com/todos").then((response) => {
        rows.value = response.data;
      });
    };

    getTodos();

    //INITIALIZATION
    let form = ref({
      userId: 1,
      title: null,
      completed: false,
    });
    //ADD TO DO IN API FUNCTION RESPONCE
    let btnLoadingState = ref(false);
    const addTodo = () => {
      btnLoadingState.value = true;
      api
        .post("https://jsonplaceholder.typicode.com/todos", form.value)
        .then((response) => {
          if (response.status === 201) {
            rows.value.unshift(response.data);
            form.value.title = null;
          }
          btnLoadingState.value = false;
        });
    };

    // SELECTING ROW IN TO DO
    let selectedRow = ref({});
    const onRowClick = (evt, row) => {
      selectedRow.value = row;
      form.value.title = row.title;
    };

    //UPDATING TO DO API FUNCTION RESPONCE
    const updateTodo = () => {
      btnLoadingState.value = true;
      api
        .put(
          `https://jsonplaceholder.typicode.com/todos/${selectedRow.value.id}`,
          {
            title: form.value.title,
          }
        )
        .then((response) => {
          if (response.status === 200) {
            let index = rows.value.findIndex(
              (row) => row.id === selectedRow.value.id
            );
            rows.value[index].title = response.data.title;
            form.value.title = null;
            selectedRow.value = {};
          }
          btnLoadingState.value = false;
        });
    };

    //DELETE BUTTON API RESPONCE
    let deleteBtnLoadingState = ref(false);
    const deleteTodo = () => {
      deleteBtnLoadingState.value = true;
      api
        .delete(
          `https://jsonplaceholder.typicode.com/todos/${selectedRow.value.id}`
        )
        .then((response) => {
          if (response.status === 200) {
            rows.value = rows.value.filter(
              (row) => row.id !== selectedRow.value.id
            );
            form.value.title = null;
          }
          deleteBtnLoadingState.value = false;
        });
      btnLoadingState.value = false;
    };

    return {
      rows,
      columns,
      form,
      btnLoadingState,
      addTodo,
      selectedRow,
      onRowClick,
      updateTodo,
      deleteBtnLoadingState,
      deleteTodo,
    };
  },
};
</script>
