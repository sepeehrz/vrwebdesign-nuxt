<style lang="scss">
#editItem {
  .header {
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 25px;
    border-bottom: 1px solid #ebedf2;
    background: #fff;
    h3 {
      margin: 0;
      padding: 0;
      font-size: 1.2rem;
      font-weight: 500;
      color: #464457;
    }
    .head-toolbar {
      display: flex;
      align-items: center;
    }
    .btn-group {
      display: flex;
      position: relative;
      .v-menu__content {
        box-shadow: 0px 0px 50px 0px rgba(82, 63, 105, 0.15);
        padding: 10px 0;
        .v-list__item {
          height: 34px;
          font-size: 0.875rem;
          .v-icon {
            font-size: 22px;
          }
        }
      }
      .action-btn {
        position: relative;
        &:after {
          display: block;
          content: ' ';
          border-left: 1px solid #ffffff;
          position: absolute;
          top: 3px;
          right: -1px;
          bottom: 3px;
          opacity: 0.3;
        }
      }
      .v-btn {
        margin: 0;
        border-radius: 0;
      }
      .btn-dropdown {
        padding: 0 8px;
        min-width: auto;
        border-right: 1px solid #fff;
        .v-icon {
          font-size: 0.875rem;
        }
      }
    }
  }
  .content {
    padding: 25px;
    min-height: 400px;
  }
}
</style>

<template>
  <v-card ref="editItem" id="editItem">
    <div sticky-container>
      <div v-sticky sticky-offset="offset" sticky-side="top" class="header">
        <div class="head-lable">
          <h3 v-once>{{custom_title}}</h3>
        </div>
        <div class="head-toolbar">
          <div class="btn-group">
            <v-menu offset-y attach bottom left min-width="180">
             <template v-slot:activator="{ on }">
                <v-btn class="btn-dropdown" depressed color="info" v-on="on">
                <v-icon>la-angle-down</v-icon>
              </v-btn>
             </template>
              <v-list>
                <v-list-item
                  v-for="(item,index) in action_list"
                  @click="action(item.action)"
                  :key="index"
                >
                  <v-icon class="pl-2">{{item.icon}}</v-icon>
                  <v-list-item-title>{{ item.title }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
            <v-btn @click="save" class="action-btn" depressed color="info">ذخیره</v-btn>
          </div>
          <v-btn text color="accent" @click="goBack">
            <span>بازگشت</span>
            <v-icon class="pr-2">la-arrow-left</v-icon>
          </v-btn>
        </div>
      </div>
      <div ref="loaderWrapper" class="content">
        <v-layout row wrap>
          <v-flex xl3 lg2></v-flex>
          <v-flex xl6 lg8 xs12>
            <v-form-generator
              :form="form"
              v-model="initItem"
              :formData="formData"
              :minimal="minimal"
            ></v-form-generator>
          </v-flex>
          <v-flex xl3 lg2></v-flex>
        </v-layout>
      </div>
    </div>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue'
import vFormGenerator from './v-form-generator.vue'
import Sticky from 'vue-sticky-directive'
import { setTimeout } from 'timers'
import { NuxtAxiosResource } from '../../nuxt-axios/types'
import { NuxtLoaderElement } from '../../nuxt-loader/types'
import { AxiosResponse } from 'axios'
interface ISaveFunction {
  renew_after?: boolean
  exit_after?: boolean
}
export default Vue.extend({
  directives: { Sticky },
  components: { vFormGenerator },
  props: {
    title: {
      type: String
    },
    service: {
      required: true,
      type: Object as () => NuxtAxiosResource<any>
    },
    item: {
      require: true,
      default: {},
      type: Object
    },
    formData: {
      required: true
    },
    minimal: {
      type: Boolean,
      default: false
    },
    loading: {},
    customSave: {
      type: Function
    },
    customExit: {
      type: Function
    },
    customDelete: {
      type: Function
    },
    beforeSave: {
      type: Function
    },
    beforeExit: {
      type: Function
    },
    beforeDelete: {
      type: Function
    },
    editUrl: {}
  },
  data() {
    let action_list = [
      { icon: 'la-save', action: 'save', title: 'ذخیره' },
      {
        icon: 'la-power-off',
        action: 'save & exit',
        title: 'ذخیره و خارج شدن'
      },
      {
        icon: 'la-refresh',
        action: 'save & create',
        title: 'ذخیره و ساخت جدید'
      },
      { icon: 'la-arrow-left', action: 'back', title: 'بازگشت' }
    ]
    if (this.$route.params.id !== 'create') {
      action_list.splice(3, 0, {
        icon: 'la-trash',
        action: 'delete',
        title: 'حذف'
      })
    }
    return {
      form: {},
      initItem: this.item,
      freezItem: JSON.parse(JSON.stringify(this.item)),

      offset: { top: 64 },
      loader: <NuxtLoaderElement>(<unknown>null),
      action_list: action_list
    }
  },
  mounted() {
    if (this.loading) {
      (<any>this).loader = (<any>this).$loader.show((<any>this).$refs.loaderWrapper)
    }
  },
  watch: {
    item: {
      handler: function(value, oldValue) {
        if (JSON.stringify(value) == JSON.stringify(oldValue)) {
          return
        }

        (<any>this).initItem = { ...value }
      },
      deep: true
    },
    loading: function(value) {
      if (value) {
        this.loader = (<any>this).$loader.show((<any>this).$refs.loaderWrapper)
      } else if (this.loader) {
        (<any>this).loader.hide()
      }
    }
  },
  methods: {
    async goBack() {
      if ((<any>this).customExit) {
        return (<any>this).customExit((<any>this).item)
      }
      if ((<any>this).beforeExit) {
        await (<any>this).beforeExit()
      }
      (<any>this).$loader.destroy()
      return (<any>this).$router.go(-1)
      let current_route = <string>this.$router.currentRoute.name
      let route_array = current_route.split('-')
      route_array.pop()
      let name = route_array.join('-')
      this.$router.push({ name })
    },
    async save({
      renew_after = false,
      exit_after = false
    }: ISaveFunction = {}) {
      if ((<any>this).beforeSave) {
        (<any>this).initItem = await (<any>this).beforeSave((<any>this).initItem)
      }
      (<any>this).form
        .validate()
        .then(valid => {
          if (valid) {
            if ((<any>this).customSave) {
              return (<any>this).customSave((<any>this).initItem, {
                renew_after,
                exit_after
              })
            }
            (<any>this).loader = this.$loader.show(this.$refs.loaderWrapper)
            let result: Promise<AxiosResponse<any>>
            if ((<any>this).initItem.id) {
              result = (<any>this).service.update((<any>this).initItem.id, (<any>this).initItem)
            } else {
              result = (<any>this).service.save((<any>this).initItem)
            }
            result
              .then(({ data, status }) => {
                this.$toast
                  .success()
                  .timeout(1000)
                  .showSimple('با موفقیت ذخیره شد')
                  .then(() => {
                    if (renew_after) {
                      if (this.$route.path.includes('create')) {
                        (<any>this).initItem = JSON.parse(
                          JSON.stringify((<any>this).freezItem)
                        )
                        setTimeout(() => {
                          (<any>this).form.resetError()
                          this.$scrollTo((<any>this).$refs.editItem, 1000)
                        }, 100)
                      } else {
                        this.$router.push('create')
                      }
                    } else if (exit_after) {
                      (<any>this).goBack()
                    } else if (status == 201) {
                      let route = this.$route.path.replace('create', data.id)
                      if ((<any>this).editUrl) {
                        route = (<any>this).editUrl.replace(/:[a-z]+/g, p => {
                          let param = p.replace(':', '')
                          return data[param]
                        })
                      } else {
                        let route = this.$route.path.replace('create', data.id)
                      }
                      (<any>this).$router.push(route)
                      (<any>this).initItem = data
                    } else {
                      (<any>this).initItem = data
                    }
                  })
              })
              .catch(err => {
                this.$toast.error().showSimple('خطایی رخ داده است')
              })
              .then(() => {
                (<any>this).loader.hide()
              })
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    async delete() {
      if ((<any>this).customDelete) {
        return (<any>this).customDelete((<any>this).initItem)
      }
      if ((<any>this).beforeDelete) {
        await (<any>this).beforeDelete()
      }
      this.$dialog.confirm().then(() => {
        (<any>this).service
          .delete((<any>this).initItem.id)
          .then(res => {
            this.$toast
              .success()
              .timeout(1000)
              .showSimple('با موفقیت حذف شد')
              .then(() => {
                (<any>this).goBack()
              })
          })
          .catch(err => {
            this.$toast.error().showSimple('خطایی رخ داده است')
          })
      })
    },
    action(action_name) {
      switch (action_name) {
        case 'save':
          (<any>this).save()
          break
        case 'save & exit':
          (<any>this).save({ exit_after: true })
          break
        case 'save & create':
          (<any>this).save({ renew_after: true })
          break
        case 'delete':
          (<any>this).delete()
          break
        case 'back':
          (<any>this).goBack()
          break
        default:
          break
      }
    }
  },
  computed: {
    custom_title(): any {
      if (!this.title) {
        return null
      }
      let title = (<any>this).title.replace(/{{[a-z\.0-9]+}}/g, p => {
        let param = p.replace(/{|}/g, '')
        let array_param = param.split('.')
        let title = this.initItem
        for (const param of array_param) {
          title = title[param]
        }
        return title
      })
      return title
    }
  }
})
</script>