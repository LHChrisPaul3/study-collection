<template>
  <page-container>
    <template slot="toolbar">
      <v-btn icon @click="$router.go(-1)">
        <v-icon>fa-times</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
      <v-btn icon @click="submitTopic">
        <v-icon>fa-paper-plane</v-icon>
      </v-btn>
    </template>
    <v-form v-model="valid" class="new" ref="form" lazy-validation>
      <v-select
        v-model="formTab"
        :rules="tabRules"
        :items="tabs"
        item-text="text"
        item-value="value"
        label="选择板块"
        required
      >
      </v-select>
      <v-text-field
        v-model="formTitle"
        :rules="titleRules"
        label="标题"
        required
      ></v-text-field>
      <v-textarea
        v-model="formContent"
        :rules="contentRules"
        solo
        flat
        no-resize
        required
      ></v-textarea>
    </v-form>
  </page-container>
</template>

<style lang="stylus" scoped>
.new
  padding: 15px
</style>


<script>
export default {
  props: {
    id: {
      default: '',
      type: String
    },
    isEdit: {
      default: false,
      type: Boolean
    }
  },
  data () {
    return {
      tabs: [
        {
          value: 'ask',
          text: '问答'
        },
        {
          value: 'share',
          text: '分享'
        },
        {
          value: 'job',
          text: '招聘'
        },
        {
          value: 'dev',
          text: '开发测试'
        }
      ],
      valid: true,
      formTab: '',
      formTitle: '',
      formContent: '',
      tabRules: [
        v => !!v || '请选择板块'
      ],
      titleRules: [
        v => !!v || '请输入标题',
        v => (v.trim().length > 10) || '标题必须大于10个字'
      ],
      contentRules: [
        v => !!v || '内容不能为空'
      ],
      accessToken: null
    }
  },
  created () {
    this.accessToken = this.getToken(this.$route.path)
    if (this.isEdit) {
      this.ajax(`/topic/${this.id}`, {
        showloading: true,
        params: {
          mdrender: 'false'
        }
      }).then(data => {
        if (data.success) {
          this.formTab = data.data.tab
          this.formTitle = data.data.title
          this.formContent = data.data.content
        }
      })
    }
  },
  methods: {
    submitTopic () {
      let path = this.isEdit ? '/topics/update' : '/topics'
      let data = {
        accesstoken: this.accessToken,
        tab: this.formTab,
        title: this.formTitle,
        content: this.formContent
      }

      data.topic_id = this.id

      if (this.$refs.form.validate()) {
        if (!this.isEdit) {
          data.content += '\n\n来自 [CNode-V](https://github.com/oodzchen/CNode-V)'
        }
        this.ajax(path, {
          method: 'post',
          showloading: true,
          data: data
        }).then(data => {
          if (data.success) {
            this.$router.replace(`/topic/${data.topic_id}`)
          }
        })
      }
    }
  }
}
</script>

