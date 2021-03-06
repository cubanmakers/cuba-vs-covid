<template>
  <div>
    <form @submit.stop.prevent="save()">
      <!-- name -->
      <label for="" class="label">Nombre del taller</label>
      <b-field>
        <b-input v-model="$v.form.name.$model"></b-input>
      </b-field>
      <!-- end name -->

      <!-- phone -->
      <label for="" class="label">Teléfono</label>
      <b-field>
        <b-input v-model="$v.form.phone.$model"></b-input>
      </b-field>
      <!-- end phone -->

      <!-- address line 1 -->
      <label for="" class="label">Línea 1 de la dirección</label>
      <b-field>
        <b-input v-model="$v.form.address.line1.$model"></b-input>
      </b-field>
      <!-- end address line 1 -->

      <!-- address line 2 -->
      <label for="" class="label">Línea 2 de la dirección</label>
      <b-field>
        <b-input v-model="form.address.line2"></b-input>
      </b-field>
      <!-- end address line 2 -->

      <!-- address province -->
      <label for="" class="label">Provincia</label>
      <b-field>
        <b-select
          v-model="$v.form.address.province.$model"
          placeholder="Selecciona una provincia"
          expanded
          @input="loadCities()"
        >
          <option
            v-for="province in provinces"
            :key="province.id"
            :value="province.id"
          >
            {{ province.name }}
          </option>
        </b-select>
      </b-field>
      <!-- end address province -->

      <!-- address city -->
      <label for="" class="label">Municipio</label>
      <b-field>
        <b-select
          v-model="$v.form.address.city.$model"
          placeholder="Selecciona un municipio"
          expanded
          :disabled="!$v.form.address.province.$model"
        >
          <option v-for="city in cities" :key="city.id" :value="city.id">
            {{ city.name }}
          </option>
        </b-select>
      </b-field>
      <!-- end address city -->

      <!-- from time -->
      <label for="" class="label">Hora de apertura</label>
      <b-field>
        <b-timepicker
          v-model="$v.form.fromTime.$model"
          rounded
          placeholder="Clic para seleccionar"
          icon="clock"
        >
        </b-timepicker>
      </b-field>
      <!-- end from time -->

      <!-- to time -->
      <label for="" class="label">Hora de cierre</label>
      <b-field>
        <b-timepicker
          v-model="$v.form.toTime.$model"
          rounded
          placeholder="Clic para seleccionar"
          icon="clock"
        >
        </b-timepicker>
      </b-field>
      <!-- end to time -->

      <b-field>
        <b-button
          type="is-primary"
          native-type="submit"
          :loading="form.loading"
          :disabled="$v.form.$invalid"
        >
          Guardar
        </b-button>
      </b-field>
    </form>
  </div>
</template>

<script>
import gql from 'graphql-tag'
import { required } from 'vuelidate/lib/validators'
export default {
  layout: 'dashboard',
  data() {
    return {
      provinces: [],
      cities: [],
      form: {
        name: null,
        phone: null,
        fromTime: null,
        toTime: null,
        address: {
          line1: null,
          line2: null,
          province: null,
          city: null
        }
      }
    }
  },
  validations: {
    form: {
      name: {
        required
      },
      phone: {
        required
      },
      fromTime: {
        required
      },
      toTime: {
        required
      },
      address: {
        line1: {
          required
        },
        province: {
          required
        },
        city: {
          required
        }
      }
    }
  },
  async mounted() {
    // Load provinces
    const { data } = await this.$apollo.query({
      query: gql`
        query provinces {
          provinces {
            id
            name
          }
        }
      `
    })
    this.provinces = data.provinces

    // Load workshop
    this.$apollo
      .query({
        query: gql`
          query viewerWorkshop {
            viewerWorkshop {
              name
              phone
              fromTime
              toTime
              address {
                line1
                line2
                city {
                  id
                  name
                }
                province {
                  id
                  name
                }
              }
            }
          }
        `
      })
      .then(({ data }) => {
        const workshop = data.viewerWorkshop
        this.form.name = workshop.name
        this.form.phone = workshop.phone
        this.form.fromTime = new Date(workshop.fromTime)
        this.form.toTime = new Date(workshop.fromTime)
        this.form.address.line1 = workshop.address.line1
        this.form.address.line2 = workshop.address.line2
        this.form.address.province = workshop.address.province.id
        this.loadCities(workshop.address.city.id)
      })
  },
  methods: {
    save() {
      this.$apollo
        .mutate({
          mutation: gql`
            mutation createOrUpdateWorkshop(
              $name: String!
              $phone: String!
              $fromTime: Time!
              $toTime: Time!
              $addressLine1: String!
              $addressLine2: String
              $addressProvince: String!
              $addressCity: String!
            ) {
              createOrUpdateWorkshop(
                name: $name
                phone: $phone
                fromTime: $fromTime
                toTime: $toTime
                line1: $addressLine1
                line2: $addressLine2
                province: $addressProvince
                city: $addressCity
              ) {
                status
              }
            }
          `,
          variables: {
            name: this.form.name,
            phone: this.form.phone,
            fromTime: this.$moment(this.form.fromTime).format('hh:mm'),
            toTime: this.$moment(this.form.toTime).format('hh:mm'),
            addressLine1: this.form.address.line1,
            addressLine2: this.form.address.line2,
            addressProvince: this.form.address.province,
            addressCity: this.form.address.city
          }
        })
        .then(({ data }) => {
          if (data.createOrUpdateWorkshop.status === 'ok') {
            this.$buefy.toast.open('Se guardaron los cambios')
          }
        })
    },
    loadCities(city) {
      this.$apollo
        .query({
          query: gql`
            query cities($provinceId: String!) {
              cities(provinceId: $provinceId) {
                id
                name
              }
            }
          `,
          variables: {
            provinceId: this.form.address.province
          }
        })
        .then(({ data }) => {
          this.cities = data.cities
          if (city) {
            this.form.address.city = city
          }
        })
    }
  }
}
</script>

<style scoped></style>
