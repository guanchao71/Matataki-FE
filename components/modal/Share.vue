<template>
  <el-dialog
    :visible.sync="showModal"
    :lock-scroll="false"
    :show-close="false"
    custom-class="gray-bg br10 p-share"
    center
    @close="change"
  >
    <div
      v-if="widgetModalStatus === 0"
      class="padding1"
    >
      <div class="widget-content-button">
        <div
          class="widget-button"
          @click="createWidget"
        >
          <div class="widget-button-img">
            <img
              src="@/assets/img/widget/widget.svg"
              alt="widget"
            >
          </div>
          <p>{{ $t('p.createWidget') }}</p>
        </div>
        <div
          class="widget-button"
          @click="widgetModalStatus = 4"
        >
          <div class="widget-button-img">
            <img
              src="@/assets/img/widget/share.svg"
              alt="widget"
            >
          </div>
          <p>{{ $t('p.createLongImg') }}</p>
        </div>
        <div
          class="widget-button"
          @click="copyCode(clipboard)"
        >
          <div class="widget-button-img">
            <img
              src="@/assets/img/widget/link.svg"
              alt="link"
            >
          </div>
          <p>{{ $t('p.copyInviteLink') }}</p>
        </div>
      </div>
      <SocialShare
        v-if="socialShow"
        :article="article"
      />
      <wechat
        :link="shareLink"
        style="margin: 60px 0 0 0;"
      />
    </div>
    <div
      v-if="widgetModalStatus === 1"
      class="padding1 widget-writecontent"
    >
      <p class="widget-title">
        {{ $t('p.createWidget') }}
      </p>
      <el-input
        v-model="widgetContent"
        :rows="4"
        type="textarea"
        placeholder="添加简介(选填)"
      />
      <div class="widget-footer">
        <a
          class="help"
          href="javascript:;"
          @click="reviewHelp"
        >
          {{ $t('p.widgetHelp') }}
        </a>
        <a
          class="create"
          href="javascript:;"
          @click="createWidgetContent"
        >
          {{ $t('p.createWidget') }}
        </a>
      </div>
    </div>
    <div
      v-if="widgetModalStatus === 2"
      class="padding1 widget-help"
    >
      <p class="widget-help-title">
        {{ $t('p.articleWidgetHelp') }}
      </p>
      <p class="widget-help-content">
        {{ $t('p.articleWidgetContent') }}
      </p>
      <br>
      <p class="widget-help-title">
        {{ $t('p.stepTitle') }}
      </p>
      <p class="widget-help-content">
        1.{{ $t('p.stepContent1') }}<br>
        2.{{ $t('p.stepContent2') }}<br>
        3.{{ $t('p.stepContent3') }}<br>
        4.{{ $t('p.stepContent4') }}<br>
      </p>

      <a
        class="widget-help-button"
        href="javascript:;"
        @click="backPage"
      >
        {{ $t('p.confirmPopover') }}
      </a>
    </div>
    <div
      v-if="widgetModalStatus === 3"
      class="padding1 widget-review"
    >
      <p class="widget-title">
        {{ $t('p.widgetView') }}
      </p>
      <div
        class="widget-review-content"
        v-html="widgetContentIframe"
      />
      <p class="widget-review-des">
        {{ $t('p.widgetCopyDes') }}
      </p>
      <el-input
        id="codeIframe"
        v-model="widgetContentIframe"
        :rows="4"
        class="widget-textarea"
        type="textarea"
        placeholder=""
        @focus="selectValue($event)"
      />
      <div class="widget-footer">
        <a
          class="help"
          href="javascript:;"
          @click="reviewHelp"
        >
          {{ $t('p.widgetHelp') }}
        </a>
        <a
          class="create"
          href="javascript:;"
          @click="copyCode(widgetContentIframe)"
        >
          {{ $t('p.copyCode') }}
        </a>
      </div>
    </div>
    <div
      v-if="widgetModalStatus === 4"
      class="padding2"
    >
      <QRCodeDialog
        :share-info="{
          title: article.title,
          avatar: article.avatar,
          name: article.nickname || article.username || '&nbsp;',
          content: article.content,
          shareLink,
          cover
        }"
        @change="change"
      />
    </div>
  </el-dialog>
</template>

<script>
import { mapGetters } from 'vuex'
import QRCodeDialog from './QRCodeDialog'
import SocialShare from './SocialShare'
import wechat from '@/components/scan/wechat.vue'

export default {
  name: 'ShareModal',
  components: {
    QRCodeDialog,
    SocialShare,
    wechat
  },
  props: {
    article: {
      type: Object,
      required: true
    },
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      showModal: false,
      amount: 0,
      comment: '',
      // 0 默认
      // 1 创建widget
      // 2 widget help
      // 3 review widget
      widgetModalStatus: 0,
      oldWidgetModalStatus: 0,
      widgetContent: '',
      widgetContentIframe: '',
      socialShow: false,
      shareLink: process.env.VUE_APP_URL,
    }
  },
  computed: {
    ...mapGetters(['currentUserInfo', 'isLogined', 'isMe']),
    clipboard() {
      const { article, currentUserInfo } = this
      const { protocol, host } = window.location
      const articleUrl = `${protocol}//${host}/p/${article.id}`
      const shareLink = this.isLogined ? `${articleUrl}/?invite=${currentUserInfo.id}&referral=${currentUserInfo.id}` : articleUrl
      return `《${article.title}》by ${article.nickname || article.username} \n${shareLink}\n${this.$t('p.clipboardText1')}`
    },
    id() {
      return this.article.id
    },
    cover() {
      if (!this.article) return ''
      return this.article.cover ? this.$ossProcess(this.article.cover) : ''
    }
  },
  watch: {
    article() {
      if (process.browser) {
        // console.log(this.$route.params.id)
        const { origin } = window.location
        // let url = `${process.env.WX_SHARE_HOST}/p/${this.$route.params.id}`
        let url = `${origin}/p/${this.$route.params.id}`
        if (this.isLogined) url += `?invite=${this.currentUserInfo.id}&referral=${this.currentUserInfo.id}`
        this.shareLink = url
      } else this.shareLink = process.env.VUE_APP_URL
    },
    showModal(val) {
      if (val) document.querySelector('body').style.overflow = 'hidden'
      else document.querySelector('body').style.overflow = ''
      this.$emit('input', val)
    },
    value(val) {
      this.showModal = val
    },
    widgetModalStatus(newVal, oldVal) {
      // console.log(newVal, oldVal)
      this.oldWidgetModalStatus = oldVal
      // 如果显示创建widget 但是没有内容
      if (newVal === 3 && !this.widgetContent) {
        let invite = this.currentUserInfo.id
        invite = invite ? `&invite=${invite}&referral=${invite}` : ''
        this.widgetContentIframe = `<iframe width="100%" height="180" src='${window.location.origin}/widget/?id=${this.id}${invite}' frameborder=0></iframe>`
      }
    },
    widgetContent(newVal) {
      let content = ''
      let invite = this.currentUserInfo.id
      invite = invite ? `&invite=${invite}&referral=${invite}` : ''
      // 去前后空格防止空内容
      if (newVal.trim()) content = `&content=${newVal}`
      this.widgetContentIframe = `<iframe width="100%" height="180" src='${window.location.origin}/widget/?id=${this.id}${content}${invite}' frameborder=0></iframe>`
    }
  },
  mounted() {
    this.socialShow = true
  },
  methods: {
    createWidget() {
      this.widgetModalStatus = 1
    },
    reviewHelp() {
      this.widgetModalStatus = 2
    },
    backPage() {
      this.widgetModalStatus = this.oldWidgetModalStatus
    },
    createWidgetContent() {
      this.widgetModalStatus = 3
    },
    copyCode(code) {
      this.$copyText(code).then(
        () => {
          this.$message({ showClose: true, message: this.$t('success.copy'), type: 'success' })
        },
        () => {
          this.$message({ showClose: true, message: this.$t('error.copy'), type: 'error' })
        }
      )
    },
    selectValue() {
      event.currentTarget.select()
    },
    resetStatus() {
      this.widgetModalStatus = 0
      this.widgetContent = ''
      this.widgetContentIframe = ''
    },
    async change(status) {
      this.showModal = false
      await this.$utils.sleep(300)
      !status && this.resetStatus()
    }
  }
}
</script>

<style src="./share.less" scoped lang="less">
</style>

<style lang="less">
.p-share .el-dialog__header,
.p-share .el-dialog__body {
  padding: 0;
}
</style>

<style lang="less" scoped>
/deep/ .p-share {
  width: 400px;
}

@media screen and (max-width: 460px) {
  /deep/ .p-share {
    width: 350px;
  }
}
</style>
