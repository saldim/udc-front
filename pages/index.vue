<template>
  <div>
    <b-spinner class="loading-spinner" v-show="loading"></b-spinner>
    <div class="container" v-bind:class="{'blured': loading}">
      <h2 class="my-3 text-center">{{ title }}</h2>
      <b-form-input
        v-model="filter"
        type="search"
        id="filterInput"
        placeholder="Начните вводить название или код..." class="my-3 search-form" v-if="showSearch" v-bind:disabled="loading"></b-form-input>
      <div class="text-center" v-else>
        <b-button class="mb-3 mx-auto back-button" @click="goBack()">← Назад</b-button>
      </div>
      <b-table striped no-select-on-click hover :items="udcs" :fields="fields" :key="udcs[0].id" v-if="udcs.length"
               @row-clicked="rowClickedHandler" ref="mainTbl" class="main-table">
        <template v-slot:cell(code)="data">
          <template v-if="data.item.hasChilds">
            <a href="#" @click="linkClickedHandler(data.item)">{{ data.value }}</a>
          </template>
          <template v-else> {{ data.value }}</template>
        </template>
        <template v-slot:cell(title)="data">
          {{ data.value }}
        </template>
        <template v-slot:cell(actions)="data">
          <div class="copy-icon" @click="copyClickedHandler(data.item, $event)"></div>
        </template>
      </b-table>
      <div class="text-center" v-else>
        Ничего не найдено
      </div>
    </div>
  </div>
</template>

<style>
.search-form {
  width: 70%;
  text-align: center;
  margin: 0 auto;
}

.back-button {
  width: 8rem;
}

.copy-icon {
  background-image: url("/icons/copy.svg");
  width: 1.5em;
  height: 1.5em;
}

.loading-spinner{
  position: fixed;
  bottom: 50%;
  left: 50%;
  right: 50%;
  width: 4rem;
  height: 4rem;
}

.main-table{
  transition: 0.3s;
}

.blured{
  filter: blur(2px);
}

</style>

<script>
export default {
  async asyncData({$axios, context}) {
    const res = await $axios.get("/api/index");
    return {
      udcs: res.data,
      fields: [{'key': 'code', 'label': 'Код'}, {'key': 'title', 'label': 'Заголовок'}, {
        'key': 'description',
        'label': 'Описание',
      }, {'key': 'actions', 'label': ''}],
      filter: '',
      title: 'Классификатор УДК',
      showSearch: true,
      currentId: null,
      loading: false,
    }
  },
  methods: {
    async fetchSearchResults() {
      if (this.filter.length > 0) {
        const res = await this.$axios.get("/api/search?request=" + this.filter);
        this.udcs = res.data;
      } else {
        const res = await this.$axios.get("/api/index");
        this.udcs = res.data;
      }
      this.loading = false
    },
    async fetchChild(id, parentTitle) {
      this.loading = true
      const res = await this.$axios.get("/api/view?id=" + id)
      this.udcs = res.data
      this.title = parentTitle
      this.showSearch = false
      this.parentId = id;
      this.loading = false
      window.scrollTo(0, 0);
    },
    async linkClickedHandler(record) {
      this.fetchChild(record.id, record.title)
      // await this.$copyText(record.code.toString());
      // this.showToast("Код УДК " + record.code + " " + record.title + " скопирован в буфер обмена")
    },
    async copyClickedHandler(record, event) {
      event.stopPropagation()
      await this.$copyText(record.code.toString());
      this.showToast("Код УДК " + record.code + " " + record.title + " скопирован в буфер обмена")
    },
    async goHome() {
      const res = await this.$axios.get("/api/index");
      this.udcs = res.data;
      this.title = 'Классификатор УДК';
      this.showSearch = true;
      this.loading = false;
    },
    async goBack() {
     this.loading = true;
      if (this.parentId) {
        const parent = await this.$axios.get("/api/get-parent?id=" + this.parentId);
        if (parent.data && parent.data.id) {
          this.fetchChild(parent.data.id, parent.data.title)
        } else {
          this.goHome()
        }
      } else {
        this.goHome()
      }
    },
    async rowClickedHandler(record, index) {
      if (record.hasChilds) {
        this.fetchChild(record.id, record.title)
      }
      // await this.$copyText(record.code.toString());
      // this.showToast("Код УДК " + record.code + " " + record.title + " скопирован в буфер обмена")
    },
    showToast(text) {
      this.$bvToast.toast(text, {
        toaster: 'b-toaster-bottom-right',
        solid: true,
        appendToast: true,
        noCloseButton: true,
      })
    }
  },
  watch: {
    filter: {
      handler: function (value) {
        this.fetchSearchResults();
      }
    }
  }

}
</script>
