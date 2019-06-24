<style lang="scss">
.form-section {
  h3 {
    display: block;
    font-size: 1.5rem;
    font-weight: 500;
    padding: 0;
    margin: 1rem 0 1rem 0;
    color: #464457;
  }
  .form-group {
    display: flex;
    > label {
      flex: 0 0 25%;
      color: #646c9a;
      padding: 12px;
    }
    .v-input {
      font-size: 13px;
      &.v-input--is-focused {
        .v-input__slot {
          border-color: $primary-color !important;
        }
      }
      .v-input__slot {
        min-height: 38px;
        border-width: 1px !important;
        border-color: #e2e5ec !important;
        align-items: center;
      }
      .v-input__prepend-inner {
        margin-top: 6px;
      }
      .v-input__append-inner {
        margin-top: 6px;
      }

      .v-label {
        top: 7px;
        font-size: 13px;
      }
      input {
        margin-top: 0;
      }
    }
    .v-input--selection-controls {
      margin-top: 0;
      padding-top: 0;
      .v-label {
        top: 0;
      }
    }
    .v-messages {
      min-height: 18px;
    }
    .v-text-field__suffix {
      background: #f5f5f5;
      height: 38px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-left: -12px;
      min-width: 40px;
      border-radius: 4px 0 0 4px;
    }
  }
}
</style>

<template>
  <section>
    <div class="form-section" v-for="(row, sectionIndex) in formData" :key="sectionIndex">
      <h3 v-if="row.title">{{row.title}}</h3>
      <template v-for="(field, fieldIndex) in row.rows">
        <component
          v-if="field.model.includes('.')"
          :key="fieldIndex"
          :is="field.component || `form-controll-${field.type}`"
          v-validate="field.validation"
          :error-messages="errors.collect(field.model)"
          :name="field.model"
          :field="field"
          v-model="item[field.model.split('.')[0]][field.model.split('.')[1]]"
        ></component>
        <component
          v-else
          :key="fieldIndex"
          :is="field.component || `form-controll-${field.type}`"
          v-validate="field.validation"
          :name="field.model"
          :error-messages="errors.collect(field.model)"
          :field="field"
          v-model="item[field.model]"
        ></component>
      </template>
    </div>
  </section>
</template>
<script >
import FormControlls from './form-controlls/index'
export default {
  components: FormControlls,
  props: {
    value: {
      require: true
    },
    formData: {
      required: true
    }
  },
  data() {
    return {
      item: { ...this.value }
    }
  },

  watch: {
    value: {
      handler: function(value, oldValue) {
        if (JSON.stringify(value) == JSON.stringify(oldValue)) {
          return
        }
        this.item = { ...value }
      },
      deep: true
    },
    item: {
      handler: function() {
        this.$emit('input', this.item)
      },
      deep: true
    }
  }
}
</script>