<template lang="pug">
  .container.init.col-md-4.col-md-offset-4
    a.navbar-brand.block.m-b.m-t.text-center
      img(src="../../../images/logobig.png")
    h3.text-lt.text-center.wrapper.m-t
      translate Perfect! Let's create a new area.
    .m-b-lg
      .wrapper
        .panel.panel-default
          .panel-heading
            h4.text-lt
              translate Add New Area
          .panel-body
            p.text-muted
              translate Area is a space where you grow your plants. It could be a seeding tray, a garden bed, or a
              translate pot or anything that describes the different physical locations in your facility.
            form(@submit.prevent="validateBeforeSubmit")
              .line.line-dashed.b-b.line-lg
              .row
                .col-xs-6
                  .form-group
                    label#label-name(for="name")
                      translate Area Name
                    input#name.form-control(type="text" v-validate="'required|alpha_num_space|min:5|max:100'" :class="{'input': true, 'text-danger': errors.has('name') }" v-model="area.name" name="name")
                    span.help-block.text-danger(v-show="errors.has('name')") {{ errors.first('name') }}
                .col-xs-6
                  .from-group
                    label#label-size
                      translate Size
                    .row
                      .col-xs-6
                        input#size.form-control(type="text" v-validate="'required|decimal'" :class="{'input': true, 'text-danger': errors.has('size') }" v-model="area.size" name="size")
                        span.help-block.text-danger(v-show="errors.has('size')") {{ errors.first('size') }}
                      .col-xs-6
                        select#size_unit.form-control(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('size_unit') }" v-model="area.size_unit" name="size_unit")
                          option(v-for="size_unit in options.size_units" :value="size_unit.key") {{ size_unit.label }}
                        span.help-block.text-danger(v-show="errors.has('size_unit')") {{ errors.first('size_unit') }}
              .row
                .col-xs-6
                  .form-group
                    label#label-type(for="type")
                      translate Type
                    select#type.form-control(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('type') }" v-model="area.type" name="type")
                      option(v-for="type in options.types" :value="type.key") {{ type.label }}
                    span.help-block.text-danger(v-show="errors.has('type')") {{ errors.first('type') }}
                .col-xs-6
                  .form-group
                    label#label-location(for="location")
                      translate Locations
                    select#location.form-control(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('location') }" v-model="area.location" name="location")
                      option(v-for="location in options.locations" :value="location.key") {{ location.label }}
                    span.help-block.text-danger(v-show="errors.has('location')") {{ errors.first('location') }}
              .row
                .col-xs-6
                  .form-group
                    label#label-reservoir(for="reservoir")
                      translate Select Reservoir
                    select#reservoir.form-control(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('reservoir') }" v-model="area.reservoir_id" name="reservoir")
                      option(value = "")
                        translate Please select reservoir
                      option(:value="reservoir.uid ? reservoir.uid : reservoir.name") {{ reservoir.name }}
                    span.help-block.text-danger(v-show="errors.has('reservoir')") {{ errors.first('reservoir') }}
                .col-xs-6
                  .form-group
                    label
                      translate Select photo
                      small.text-muted (if any)
                    UploadComponent(@fileSelelected="fileSelelected")
              .form-group
                button.btn.btn-addon.btn-success.pull-right(type="submit")
                  i.fas.fa-long-arrow-alt-right
                  translate Finish Setup
                router-link#back.btn.btn-addon.btn-default(:to="{name: 'IntroReservoirCreate'}")
                  i.fas.fa-long-arrow-alt-left
                  translate Back
</template>

<script>
import { AreaTypes, AreaLocations, AreaSizeUnits } from '../../stores/helpers/farms/area'
import { StubArea, StubMessage } from '../../stores/stubs'
import { mapActions, mapGetters } from 'vuex'
import UploadComponent from '../../components/upload.vue'
export default {
  name: 'AreaIntro',
  components: {
    UploadComponent
  },
  computed: {
    ...mapGetters({
      reservoir: 'introGetReservoir',
      currentArea: 'introGetArea',
      currentFarm: 'introGetFarm',
    })
  },
  data () {
    return {
      message: Object.assign({}, StubMessage),
      area: Object.assign({}, StubArea),
      options: {
        types: Array.from(AreaTypes),
        locations: Array.from(AreaLocations),
        size_units: Array.from(AreaSizeUnits)
      }
    }
  },
  methods: {
    ...mapActions([
      'introSetArea',
      'introCreateFarm',
      'introCreateReservoir',
      'introCreateArea',
    ]),
    validateBeforeSubmit () {
      this.$validator.validateAll().then(result => {
        if (result) {
          this.introSetArea(this.area)
          this.toCreateFarm()
        }
      })
    },
    toCreateFarm () {
      this.introCreateFarm()
        .then(data => {
          this.toCreateReservoir()
        }).catch(error => {
          this.$router.push({ name: 'IntroFarmCreate' })
        })
    },
    toCreateReservoir () {
      this.introCreateReservoir()
        .then(data => {
          this.toCreateArea()
        }).catch(error => {
          this.$router.push({ name: 'IntroReservoirCreate' })
        })
    },
    toCreateArea () {
      this.introCreateArea()
        .then(data => {
          this.$router.push({ name: 'Home' })
        }).catch(error => {
          this.$toasted.error('Error in area submission')
        })
    },
    fileSelelected (file) {
      this.area.photo = file
    }
  },
  mounted () {
    if (this.currentArea) {
      this.area = Object.assign({}, this.currentArea)
      this.area.size_unit = this.options.size_units[0].key
      this.area.location = this.options.locations[0].key
      this.area.type = this.options.types[0].key
      this.area.reservoir_id = this.reservoir.uid ? this.reservoir.uid : this.reservoir.name
    }

    if (this.reservoir.name === '') {
      this.$router.push({ name: 'IntroReservoirCreate' })
    }

    if (this.currentFarm.name === '') {
      this.$router.push({ name: 'IntroFarmCreate' })
    }
  },
}
</script>

