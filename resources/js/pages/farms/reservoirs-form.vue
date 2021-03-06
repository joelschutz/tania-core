<template lang="pug">
  .reservoirs-form.col
    .modal-header
      span.h4.font-bold(v-if="reservoir.uid")
        translate Update Reservoir
      span.h4.font-bold(v-else)
        translate Add New Reservoir
    .modal-body
      small.text-muted
        translate Reservoir is a water source for your farm. It can be a direct line from well, or water tank that has volume/capacity.
      error-message(:message="message.error_message")
      form(@submit.prevent="validateBeforeSubmit")
        .form-group
          label#label-name(for="name")
            translate Reservoir Name
          input#name.form-control(type="text" v-validate="'required|alpha_num_space|min:5|max:100'" :class="{'input': true, 'text-danger': errors.has('name') }" v-model="reservoir.name" name="name")
          span.help-block.text-danger(v-show="errors.has('name')") {{ errors.first('name') }}
        .form-group
          label#label-type(for="type")
            translate Source
          select#type.form-control(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('type') }" v-model="reservoir.type" name="type" @change="typeChanged($event.target.value)")
            option(value="")
              translate Please select source
            option(v-for="option in options" :value="option.key") {{ option.label }}
          span.help-block.text-danger(v-show="errors.has('type')") {{ errors.first('type') }}
        .form-group(v-if="reservoir.type == 'BUCKET'")
          input#capacity.form-control(type="text" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('capacity') }" v-model="reservoir.capacity" placeholder="Capacity (litre)" name="capacity")
          span.help-block.text-danger(v-show="errors.has('capacity')") {{ errors.first('capacity') }}
        .form-group
          button.btn.btn-addon.btn-success.pull-right(type="submit")
            translate SAVE
          button.btn.btn-default(type="button" style="cursor: pointer;" @click="$parent.$emit('close')")
            translate CANCEL
</template>


<script>
import { ReservoirTypes } from '../../stores/helpers/farms/reservoir'
import { StubReservoir, StubMessage } from '../../stores/stubs'
import { mapActions } from 'vuex'
export default {
  name: "FarmReservoirForm",
  data () {
    return {
      message: Object.assign({}, StubMessage),
      reservoir: Object.assign({}, StubReservoir),
      options: Array.from(ReservoirTypes)
    }
  },
  methods: {
    ...mapActions([
      'submitReservoir'
    ]),
    validateBeforeSubmit () {
      this.$validator.validateAll().then(result => {
        if (result) {
          this.submit()
        }
      })
    },
    submit () {
      console.log(this.reservoir)
      this.submitReservoir(this.reservoir)
        .then(() => this.$parent.$emit('close'))
        .catch(() => this.$toasted.error('Error in reservoir submission'))
    },
    typeChanged (type) {
      if (type === 'bucket') {
        this.reservoir.capacity = ''
      } else {
        this.reservoir.capacity = 0
      }
    },
  },
  mounted () {
    if (typeof this.data.uid != "undefined") {
      this.reservoir.uid = this.data.uid
      this.reservoir.name = this.data.name
      this.reservoir.type = this.data.water_source.type
      this.reservoir.capacity = this.data.water_source.capacity
    }
  },
  props: ['data'],
}
</script>
