<template>
  <div>
    <b-spinner class="loading-spinner" v-show="loading"></b-spinner>
    <div class="container" v-bind:class="{'blured': loading}">
      <div class="journal-logo" @click="openJournalSite()"></div>
      <h2 class="my-3 text-center">{{ title }}</h2>
      <b-form-input
        v-model="filter"
        type="search"
        id="filterInput"
        placeholder="Начните вводить название или код..." class="my-3 search-form" v-if="showSearch" v-bind:disabled="loading" autofocus="autofocus"></b-form-input>
      <div class="text-center" v-else>
      <b-button class="mb-3 mx-auto back-button" variant="outline-primary" @click="goBack()">← Назад</b-button>  <b-button class="mb-3 mx-auto home-button-wrapper" variant="primary" @click="goHome()"><img src="/icons/home.svg" alt="На главную" class="home-button"></b-button>
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
         <div class="mt-3"><b-button variant="outline-primary" @click="clearSearch()">Показать все</b-button></div>
      </div>
      <footer><div class="text-center pb-3">© 2017-{{ new Date().getFullYear() }} <a href="http://perviy-vestnik.ru/?utm_source=udc" target="_blank">НАУЧНО-ИЗДАТЕЛЬСКИЙ ЦЕНТР "ВЕСТНИК НАУКИ"</a></div></footer>
    </div>
  </div>
</template>

<style>
.journal-logo{
  width: 100%;
  height: 12rem;
  margin: 1.2rem auto;
  background: url("/images/journal-logo.svg");
  background-position: center;
  background-size: contain;
  background-repeat: no-repeat;
  cursor: pointer;
}

.search-form {
  width: 70%;
  text-align: center;
  margin: 0 auto;
}

.back-button {
  width: 8rem;
}

.home-button-wrapper {
  width: 3.1rem;
  text-align: center;
}

.home-button{
  width: 1.5rem;
  height: 1.5rem;
}

.copy-icon {
  background-image: url("/icons/copy.svg");
  width: 1.5em;
  height: 1.5em;
}

.loading-spinner{
  position: fixed;
  left: 47%;
  top: 44%;
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
  mounted(){
    window.addEventListener('keypress', function(event){
      let filterField = document.getElementById('filterInput');
      filterField.focus();
    })
  },
  methods: {
    openJournalSite(){
      window.open('http://perviy-vestnik.ru/?utm_source=udc', '_blank')
    },
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
    },
    showToast(text) {
      this.$bvToast.toast(text, {
        toaster: 'b-toaster-bottom-right',
        solid: true,
        appendToast: true,
        noCloseButton: true,
      })
    },
    clearSearch(){
      this.filter = ''
    }
  },
  watch: {
    filter: {
      handler: function (value) {
        this.fetchSearchResults();
      }
    }
  },
  head() {
    return {
      title: 'УДК',
      meta: [
        // hid is used as unique identifier. Do not use `vmid` for it as it will not work
        {
          hid: 'description',
          name: 'description',
          content: 'Справочник по УДК для статьи. Данный ресурс является информационно-справочной системой УДК, описывающей универсальную десятичную классификацию УДК. Определить индекс УДК для статьи онлан УДК и БКБ.'
        },
        {
          hid: 'keywords',
          name: 'keywords',
          content: 'УДК, классификатор УДК, код УДК, УДК для статьи, индекс УДК, определить УДК, УДК онлайн, УДК БКБ',
        }
      ]
    }
  }
}
</script>
