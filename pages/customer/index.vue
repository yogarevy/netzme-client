<template>
  <div>
    <h1 class="title">Customers</h1>
    <hr />
    <div class="row">
      <div class="col-md-12">
        <b-card>
          <div class="row">
            <div class="col">
              <b-card-title>List Customers</b-card-title>
            </div>
            <div class="col">
              <b-button
                variant="primary"
                size="sm"
                class="float-right"
                @click="loadTable()"
              >
                <font-awesome-icon icon="sync-alt" /> Load Data
              </b-button>
            </div>
          </div>
          <hr />
          <div class="row my-3">
            <!-- SELECT PAGE LIMIT -->
            <div class="col-md-4">
              <b-form-group
                label="Per page"
                label-for="per-page-select"
                label-cols-sm="6"
                label-cols-md="4"
                label-cols-lg="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-form-select
                  id="per-page-select"
                  v-model="perPage"
                  :options="pageOptions"
                  size="sm"
                ></b-form-select>
              </b-form-group>
            </div>
            <!-- FORM SEARCH DATA -->
            <div class="col-md-8">
              <b-form-group
                label="Filter"
                label-for="filter-input"
                label-cols-sm="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-input-group size="sm">
                  <b-form-input
                    id="filter-input"
                    v-model="filter"
                    type="search"
                    placeholder="Type to Search"
                  ></b-form-input>

                  <b-input-group-append>
                    <b-button :disabled="!filter" @click="filter = ''"
                      >Clear</b-button
                    >
                  </b-input-group-append>
                </b-input-group>
              </b-form-group>
            </div>
          </div>
          <div class="row my-3">
            <!-- TABLE DATA CUSTOMER -->
            <div class="col-md-12">
              <b-table
                ref="tabelCustomer"
                striped
                hover
                :items="items"
                :fields="fields"
                :current-page="currentPage"
                :per-page="0"
                :filter="filter"
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
                :sort-direction="sortDirection"
                :busy="isBusy"
                stacked="md"
                show-empty
                outlined
              >
                <!-- THIS WILL SHOW LOADING SPINNER EVERY LOAD DATA -->
                <template #table-busy>
                  <div class="text-center text-primary my-2">
                    <b-spinner class="align-middle"></b-spinner>
                    <strong>Loading...</strong>
                  </div>
                </template>
                <template #cell(photo)="row">
                  <img :src="row.value" class="photo" />
                </template>
                <!-- ADD HTML TO DATA -->
                <template #cell(percentage)="row">
                  <b-progress
                    :value="row.value"
                    :max="max"
                    :precision="2"
                    :variant="
                      row.value < 5
                        ? 'danger'
                        : row.value < 10
                        ? 'warning'
                        : 'success'
                    "
                    show-value
                    show-progress
                    class="mb-3"
                  ></b-progress>
                </template>
              </b-table>
            </div>
          </div>
          <!-- PAGINATION TABLE -->
          <div class="row justify-content-end">
            <div class="col-md-6">
              <b-pagination
                v-model="currentPage"
                :total-rows="totalRows"
                :per-page="perPage"
                align="fill"
                size="sm"
                class="my-0"
              ></b-pagination>
            </div>
          </div>
          <!-- INFORMATION DATA IN TABLE -->
          <b-card-text class="small text-muted">
            Showing {{ from }} to {{ to }} of {{ totalRows }} rows
          </b-card-text>
        </b-card>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  layout: 'app',
  data() {
    return {
      fields: [
        {
          key: 'name',
          label: 'NAME',
          sortable: true,
          sortDirection: 'desc',
        },
        {
          key: 'photo',
          label: 'PROFILE PHOTO',
          sortable: false,
        },
        {
          key: 'trx_count',
          label: '# TRANSACTION',
          sortable: true,
        },
        {
          key: 'percentage',
          label: '% Amount Transaction',
          sortable: false,
        },
      ], // Field for head table
      isBusy: false,
      items: this.items, // inital data for table
      totalRows: 0, // START initial pagination table
      currentPage: 1,
      perPage: 5,
      pageOptions: [5, 10, 15, 20, 50],
      sortBy: '',
      sortDesc: false,
      sortDirection: 'asc',
      from: 0,
      to: 0,
      filter: '', // END initial pagination table
      max: 100,
    }
  },
  computed: {
    // Create param for API URL
    params() {
      let params = '?limit=' + this.perPage + '&page=' + this.currentPage
      if (this.sortBy) {
        params += '&sort=' + this.sortBy
        if (this.sortDesc) {
          params += '&order=DESC'
        }
      }
      if (this.filter !== '') {
        params += '&search=' + this.filter
      }
      return params
    },
  },
  watch: {
    params(newParams, oldParams) {
      this.fetchData(newParams) // watch every changes of parameter API URL
    },
  },
  created() {
    this.fetchData(this.params) // initial fetch data on load page
  },
  methods: {
    // Function Fetch Data From API
    fetchData(newParams) {
      this.isBusy = true
      this.items = []
      this.$axios
        .get('/api/v1/customer' + newParams)
        .then((res) => {
          if (res.status === 200) {
            this.isBusy = false
            const data = res.data.data.items
            this.items = data
            this.from = res.data.page_info.from
            this.to = res.data.page_info.to
            this.totalRows = res.data.page_info.total
            return this.items
          } else {
            this.$swal({
              text: res.data.errors,
              icon: 'error',
              allowOutsideClick: false,
            })
          }
        })
        .catch((err) => {
          this.$swal({
            text: err.response.statusText,
            icon: 'error',
            allowOutsideClick: false,
          })
        })
    },
    loadTable() {
      this.fetchData(this.params)
    },
  },
}
</script>

<style scoped>
.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 50px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

.photo {
  width: 50px;
  height: 50px;
}
</style>
