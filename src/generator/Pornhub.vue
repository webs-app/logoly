<style lang="stylus" scoped>
.pornhub {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.box {
  border: 1px solid #333;
  border-radius: 10px;
  padding: 40px;
  margin: 40px 10px;
  max-width: 100%;
  overflow: auto;

  .editarea {
    padding: 20px;
    text-align: center;
    font-size: 60px;
    // font-weight: bold;
    white-space: nowrap;
    display: inline-block;
    .prefix {
      color: #fff;
      padding: 5px 5px;
    }

    .postfix {
      color: #000;
      background-color: #f90;
      padding: 5px 10px;
      border-radius: 7px;
    }
  }
}

// customize things
.customize {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin-bottom: 50px;
  flex-flow: column;

  .customize-color > div, .customize-misc > div {
    padding: 8px 0;
  }
}

// download and share buttons
.download-share {
  display: flex;
  justify-content: space-around;
  width: 80%;

  & > div {
    width: 100px;
    height: 40px;
    border-radius: 3px;
    line-height: 40px;
    text-align: center;
    cursor: pointer;
  }

  .download {
    color: black;
    background: #f90;
  }

  .share {
    color: #fff;
    background: #1da1f2;
  }
}
</style>

<template>
  <div class="pornhub">
    <div
      class="box"
      v-tooltip="{
        content: '编辑文字',
        show: true,
        classes: 'tooltipClasses',
      }"
    >
      <div
        class="editarea"
        id="logo"
        :style="{ 'font-size': fontSize + 'px', 'background-color': transparentBgColor,'font-family':fontFamily }"
      >
        <template v-if="!reverseHighlight">
          <span
            @input="updatePrefix"
            class="prefix"
            :style="{ color: prefixColor }"
            contenteditable
            spellcheck="false"
          >
            {{
            prefixText
            }}
          </span>
          <!-- HACK: meaningless text: ".", just to split input area, see: #269 -->
          <span style="font-size: 0;">.</span>
          <span
            class="postfix"
            :style="{ color: suffixColor, 'background-color': postfixBgColor }"
            contenteditable
            @input="updateSuffix"
            spellcheck="false"
          >{{ suffixText }}</span>
        </template>
        <template v-else>
          <span
            class="postfix"
            :style="{ color: suffixColor, 'background-color': postfixBgColor }"
            contenteditable
            @input="updatePrefix"
            spellcheck="false"
          >{{ prefixText }}</span>
          <span
            class="prefix"
            @input="updateSuffix"
            :style="{ color: prefixColor }"
            contenteditable
            spellcheck="false"
          >
            {{
            suffixText
            }}
          </span>
        </template>
      </div>
    </div>

    <div class="customize">
      <div class="customize-misc">
        <div>
          字体大小:
          <input type="range" min="30" max="200" v-model="fontSize" />
          {{ fontSize }}px
        </div>
        <div>
          字体:
          <FontSelector v-on:update-font="(font)=>fontFamily = font" />
        </div>
        <div>
          调转方向:
          <input type="checkbox" v-model="reverseHighlight" />
        </div>
      </div>
      <div
        class="customize-color"
        id="prefixColor">
        <div>
          前缀颜色: &nbsp;
          <input type="color" v-model="prefixColor" />
        </div>
        <div>
          后缀颜色: &nbsp;
          <input type="color" v-model="suffixColor" />
        </div>
        <div>
          后缀背景色: &nbsp;
          <input type="color" v-model="postfixBgColor" />
        </div>
        <div>
          是否透明背景: &nbsp;
          <input
            type="checkbox"
            value="transparentBg"
            v-model="transparentBg"
          />
        </div>
      </div>

   
    </div>

    <div class="download-share">
      <div
        class="download"
        @click="download"
      >导出</div>

      <!-- <div class="share" @click="twitter">
        <i class="iconfont icon-twitter"></i> Tweet
      </div> -->
    </div>
  </div>
</template>

<script>
import domtoimage from 'dom-to-image'
import FontSelector from '../components/FontSelector'

export default {
  name: 'pornhub',
  data() {
    return {
      prefixColor: '#ffffff',
      suffixColor: '#000000',
      postfixBgColor: '#ff9900',
      fontSize: '60',
      fontFamily: 'sans-serif',
      transparentBg: false,
      reverseHighlight: false,
      prefixText: this.$store.state.prefix,
      suffixText: this.$store.state.suffix,
    }
  },
  components: { FontSelector },
  mounted: function() {
    //   this.$tours['pornhub'].start()
  },
  methods: {
    updatePrefix(e) {
      this.$store.commit('updatePrefix', e.target.childNodes[0].nodeValue)
    },
    updateSuffix(e) {
      this.$store.commit('updateSuffix', e.target.childNodes[0].nodeValue)
    },
    downloadImage(imgsrc, name) {
      //下载图片地址和图片名
      let image = new Image()
      // 解决跨域 Canvas 污染问题
      image.setAttribute('crossOrigin', 'anonymous')
      image.onload = function() {
        let canvas = document.createElement('canvas')
        canvas.width = image.width
        canvas.height = image.height
        let context = canvas.getContext('2d')
        context.drawImage(image, 0, 0, image.width, image.height)
        let url = canvas.toDataURL('image/png')
        let a = document.createElement('a')
        let event = new MouseEvent('click')
        a.download = name || 'photo'
        a.href = url
        a.dispatchEvent(event)
      }
      image.src = imgsrc
    },
    download() {
      var that = this
      var node = document.getElementById('logo')
      domtoimage.toPng(node).then(function(res) {
        console.log(res)
        that.downloadImage(res, 'logo.png')
      })
    },
    twitter() {
      this.$ga.event('social', 'action', 'twitter', 1)
      let url = 'https://logoly.pro'
      let text = encodeURIComponent(`Built with #LogolyPro, by @xiqingongzi ${url}`)
      window.open(`https://twitter.com/intent/tweet?text=${text}`)
    },
  },
  computed: {
    transparentBgColor() {
      if (this.transparentBg) {
        return 'transparent'
      } else {
        return '#000000'
      }
    },
  },
}
</script>
