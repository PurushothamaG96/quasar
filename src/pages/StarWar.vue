<template>
  <div>
    <p v-if="errorMessage">{{ errorMessage }}</p>
    <div v-if="isLoading" class="q-pa-md q-gutter-md">
      <q-spinner-dots color="primary" size="3em" />
    </div>
    <q-table
      class="q-mt-md my-sticky-table"
      :rows="people"
      :columns="columns"
      row-key="name"
      :loading="isLoading"
      title="Star war Table"
      virtual-scroll
      :virtual-scroll-sticky-size-start="48"
    >
    <template v-slot:body-cell-avatar="props">
        <q-td>
          <img class="img-fluid" :src="props.row.avatar" :alt="props.row.name" />
        </q-td>
      </template>
    </q-table>
    <q-card class="my-card">
  <q-card-section>
    <div class="row justify-center">
      <div class="col-lg-8">
        <div class="q-pa-md q-gutter-sm">
          <q-btn
            :disable="currentPage === 1"
            color="primary"
            label="Previous"
            @click="previousPage"
          />
          <template v-for="(_, index) in totalPages" :key="index">
            <q-btn
              v-if="Math.abs(currentPage - (index + 1)) <= 2 || index === 0 || index === totalPages - 1"
              color="primary"
              :class="{ 'bg-primary text-white': index + 1 === currentPage }"
              @click="perticularPage(index + 1)"
            >
              {{ index + 1 }}
            </q-btn>
          </template>
          <q-btn
            :disable="currentPage === totalPages"
            color="primary"
            label="Next"
            @click="nextPage"
          />
        </div>
      </div>
    </div>
  </q-card-section>
</q-card>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { QSpinnerDots, QTable, useQuasar } from 'quasar'
import axios from 'axios'

export default {
  name: 'PeopleList',
  components: {
    QSpinnerDots,
    QTable
  },
  setup () {
    const apiUrl = 'https://swapi.dev/api/people/'
    const currentPage = ref(1)
    const previousUrl = ref(null)
    const nextUrl = ref(null)
    const people = ref([])
    const totalPages = ref(1)
    const errorMessage = ref('')
    const isLoading = ref(true)
    const $q = useQuasar()

    const fetchPeople = () => {
      isLoading.value = true
      axios
        .get(apiUrl, {
          params: {
            page: currentPage.value
          }
        })
        .then((response) => {
          const data = response.data
          people.value = data.results.map((person) => ({
            ...person,
            avatar: `https://i.pravatar.cc/150?img=${Math.floor(Math.random() * 70)}`
          }))
          previousUrl.value = data.previous
          nextUrl.value = data.next
          totalPages.value = Math.ceil(data.count / 10)
          // $q.notify('Data success fully loaded')
          $q.notify({
            message: 'Data successfully loaded',
            caption: 'a seconds ago',
            color: 'secondary'
          })
        })
        .catch((error) => {
          errorMessage.value = error.message
          $q.notify({
            message: 'Error occured',
            caption: 'a seconds ago',
            color: 'Danger'
          })
        })
        .finally(() => {
          isLoading.value = false
        })

      // $q.dialog({
      //   title: 'Confirm',
      //   message: 'Would you like to turn on the wifi?',
      //   cancel: true,
      //   persistent: true
      // }).onOk(() => {
      //   console.log('>>>> OK')
      // })
    }

    const perticularPage = (pageNumber) => {
      if (currentPage.value !== pageNumber) {
        currentPage.value = pageNumber
        fetchPeople()
      }
    }

    const previousPage = () => {
      if (previousUrl.value) {
        currentPage.value--
        fetchPeople()
      }
    }

    const nextPage = () => {
      if (nextUrl.value) {
        currentPage.value++
        fetchPeople()
      }
    }

    onMounted(fetchPeople)

    const columns = [
      { name: 'avatar', required: true, label: 'Image', align: 'center' },
      { name: 'name', required: true, label: 'Name', align: 'left', field: 'name' },
      { name: 'height', required: true, label: 'Height', align: 'left', field: 'height' },
      { name: 'mass', required: true, label: 'Mass', align: 'left', field: 'mass' },
      { name: 'hair_color', required: true, label: 'Hair Color', align: 'left', field: 'hair_color' },
      { name: 'skin_color', required: true, label: 'Skin Color', align: 'left', field: 'skin_color' },
      { name: 'eye_color', required: true, label: 'Eye Color', align: 'left', field: 'eye_color' },
      { name: 'birth_year', required: true, label: 'Year of Birth', align: 'left', field: 'birth_year' },
      { name: 'gender', required: true, label: 'Gender', align: 'left', field: 'gender' }
    ]

    return {
      currentPage,
      previousUrl,
      nextUrl,
      people,
      totalPages,
      errorMessage,
      isLoading,
      columns,
      fetchPeople,
      perticularPage,
      previousPage,
      nextPage
    }
  }
}
</script>

<style lang="sass">
.img-fluid
  width: 50px
  height: 50px
  object-fit: cover

.my-sticky-table
  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th /* bg color is important for th; just specify one */
    background-color: #00b4ff

  thead tr th
    position: sticky
    z-index: 1
  /* this will be the loading indicator */
  thead tr:last-child th
    /* height of all previous header rows */
    top: 48px
  thead tr:first-child th
    top: 0

  /* prevent scrolling behind sticky top row on focus */
  tbody
    /* height of all previous header rows */
    scroll-margin-top: 48px
</style>
