<template>
  <div class="container">
    <div class="form-group">
      <input type="text" class="form-control" id="name" v-model="name" placeholder="請輸入您的姓名">
    </div>
    <div class="radio selectImage">
      <p class="text-center">請選擇要套用的圖框</p>
      <label v-for="n in 5">
        <input type="radio" name="optionsRadios"  v-bind:id="'optionsRadios' + n"  v-bind:value="'option'+ n " v-model="option" >
        <img :src="'./static/'+ $route.params.room +'/'+ n +'.png'" v-on:click="changeCover">
        <!-- <img src='../static/01/1.png'> -->
      </label>
    </div>
    <div id="preview" @touchmove.prevent>
      <img :src="userImage" id="userImage" v-if="userImage">
      <img :src="coverImage" id="coverimage">
        <div id="userName">{{name}}</div>
        <v-touch
          v-bind:enabled="{ pan:true, pinch: true, rotate: false }"
          v-on:panmove="touchmoveMethod"
          v-on:pinchmove="pinchmoveMethod"
          id='dragger'
        ></v-touch>
    </div>    
    <p v-if="userImage">單指移動照片，雙指縮放</p>
    <div class="form-group">
      <button v-if="coverImage" class="btn btn-primary" type="submit" v-on:click="uploadImageMethod">上傳圖片</button>
      <button v-if="userImage" class="btn btn-primary" type="submit" v-on:click="makeImageMethod">下載圖片</button>
    </div>
    <a id="download" class="hide" download="10.png">Download</a>
    <p v-if="outputImage">請長按圖片以儲存圖片</p>
    <img id="output" :src="outputImage" />
    
    <input type="file" name="Filedata" id="uploadInput" v-on:change="loadUploadImage">  
    <!--  cover 的圖片 -->
    <canvas id="canvasCover" class="hide"></canvas>
    <!--  使用者上傳 的圖片 -->
    <canvas id="canvasUserImage" class="hide"></canvas>
    <!-- 最後輸出的圖片 -->
    <canvas id="result" class="hide"></canvas>
    <transition name="fade">
      <div v-if="uploading" id="uploading">圖片讀取中，請稍後...</div>
    </transition>
  </div>
</template>

<script>
import Vue from 'vue'
var VueTouch = require('vue-touch')
Vue.use(VueTouch, {name: 'v-touch'})

export default {
  name: 'index',
  data () {
    return {
      name: '',
      option: null,
      coverImage: '',
      userImage: '',
      uploading: false,
      userImageOrientation: -1,
      userImageTranslateX: 0,
      userImageTranslateY: 0,
      userImageScale: 1,
      userImageRotate: 0,
      making: false,
      outputImage: ''
    }
  },
  methods: {
    changeCover: function (event) {
      //  當點擊縮圖時，切換預覽的圖片
      this.outputImage = ''
      var preview = document.getElementById('preview')
      var previewWidth = preview.clientWidth
      var vm = this
      var newImg = new Image()
      newImg.src = event.target.src
      newImg.onload = (e) => {
        // console.log(e.target.result)
        document.getElementById('preview').style.height = ((previewWidth / newImg.width * newImg.height) + 'px')
        var canvasCover = document.getElementById('canvasCover')
        canvasCover.width = newImg.width
        canvasCover.height = newImg.height
        var ctx = canvasCover.getContext('2d')
        ctx.drawImage(newImg, 0, 0)
        var base64 = canvasCover.toDataURL('image/png')
        vm.coverImage = base64
      }
    },
    uploadImageMethod: function () {
      // 點擊上傳按鈕
      document.getElementById('uploadInput').click()
    },
    loadUploadImage: function (e) {
      // 選擇了圖片上傳後的動作
      this.uploading = true
      var files = e.target.files || e.dataTransfer.files
      var vm = this
      if (!files.length) {
        this.uploading = false
        return
      }
      this.userImageTranslateX = 0
      this.userImageTranslateY = 0
      this.userImageScale = 1
      this.getOrientation(files[0], function (orientation) {
        vm.userImageOrientation = orientation
        console.log('userImageOrientation:' + vm.userImageOrientation)
        vm.imageLoaded(files[0])
      })
    },
    imageLoaded: function (file) {
      // 讀取上傳圖片
      console.log('imageLoaded')
      var reader = new FileReader()
      var vm = this
      reader.onload = (e) => {
        // console.log(e.target.result)
        vm.createUserImage(e)
      }
      reader.readAsDataURL(file)
    },
    createUserImage: function (file) {
      // 讀取上傳圖片完成建立圖片
      console.log('createUserImage')
      var vm = this
      var userImage = new Image()
      userImage.onload = (e) => {
        console.log('userImageonload')
        var canvasUserImage = document.getElementById('canvasUserImage')
        canvasUserImage.width = userImage.width
        canvasUserImage.height = userImage.height
        var ctx = canvasUserImage.getContext('2d')
        var width = userImage.width
        var height = userImage.height
        var styleWidth = userImage.width
        var styleHeight = userImage.height
        if (vm.userImageOrientation > 4) {
          canvasUserImage.width = height
          canvasUserImage.height = width
          canvasUserImage.style.width = styleHeight
          canvasUserImage.style.height = styleWidth
        }
        switch (vm.userImageOrientation) {
          case 2:
            // horizontal flip
            ctx.translate(width, 0)
            ctx.scale(-1, 1)
            break
          case 3:
            // 180° rotate left
            ctx.translate(width, height)
            ctx.rotate(Math.PI)
            break
          case 4:
            // vertical flip
            ctx.translate(0, height)
            ctx.scale(1, -1)
            break
          case 5:
            // vertical flip + 90 rotate right
            ctx.rotate(0.5 * Math.PI)
            ctx.scale(1, -1)
            break
          case 6:
            // 90° rotate right
            ctx.rotate(0.5 * Math.PI)
            ctx.translate(0, -height)
            break
          case 7:
            // horizontal flip + 90 rotate right
            ctx.rotate(0.5 * Math.PI)
            ctx.translate(width, -height)
            ctx.scale(-1, 1)
            break
          case 8:
            // 90° rotate left
            ctx.rotate(-0.5 * Math.PI)
            ctx.translate(-width, 0)
            break
        }
        ctx.drawImage(userImage, 0, 0)
        var base64 = canvasUserImage.toDataURL('image/png')
        // console.log(base64)
        vm.userImage = base64
        vm.uploading = false
      }
      userImage.src = file.target.result
    },
    getOrientation: function (file, callback) {
      // 獲得上傳照片的EXIF方向
      var reader = new FileReader()
      reader.onload = function (e) {
        var view = new DataView(e.target.result)
        if (view.getUint16(0, false) !== 0xFFD8) return callback(-2)
        var length = view.byteLength
        var offset = 2
        while (offset < length) {
          var marker = view.getUint16(offset, false)
          offset += 2
          if (marker === 0xFFE1) {
            if (view.getUint32(offset += 2, false) !== 0x45786966) return callback(-1)
            var little = view.getUint16(offset += 6, false) === 0x4949
            offset += view.getUint32(offset + 4, little)
            var tags = view.getUint16(offset, little)
            offset += 2
            for (var i = 0; i < tags; i++) {
              if (view.getUint16(offset + (i * 12), little) === 0x0112) {
                return callback(view.getUint16(offset + (i * 12) + 8, little))
              }
            }
          } else if ((marker & 0xFF00) !== 0xFF00) {
            break
          } else {
            offset += view.getUint16(offset, false)
          }
        }
        return callback(-1)
      }
      reader.readAsArrayBuffer(file)
    },
    touchmoveMethod: function (e) {
      // 點擊移動
      console.log('touchmove')
      // console.log(e)
      this.userImageTranslateX = this.userImageTranslateX + e.deltaX / 24
      this.userImageTranslateY = this.userImageTranslateY + e.deltaY / 24
      document.getElementById('userImage').style.transform = 'translate(' + this.userImageTranslateX + 'px,' + this.userImageTranslateY + 'px) scale(' + this.userImageScale + ') rotate(' + this.userImageRotate + 'deg)'
    },
    pinchmoveMethod: function (e) {
      // 縮放
      // console.log(e)
      this.userImageScale = e.scale
      document.getElementById('userImage').style.transform = 'translate(' + this.userImageTranslateX + 'px,' + this.userImageTranslateY + 'px) scale(' + this.userImageScale + ') rotate(' + this.userImageRotate + 'deg)'
    },
    rotatemoveMethod: function (e) {
      // 旋轉圖片
      console.log(e)
      this.userImageRotate = this.userImageRotate + e.rotation
      document.getElementById('userImage').style.transform = 'translate(' + this.userImageTranslateX + 'px,' + this.userImageTranslateY + 'px) scale(' + this.userImageScale + ') rotate(' + this.userImageRotate + 'deg)'
    },
    makeImageMethod: function (e) {
      // 開始合成照片
      this.making = true
      // console.log(this.making)
      // console.log(document.getElementById('preview').clientWidth)
      // console.log(this.userImageScale)
      var scale = 1000 / document.getElementById('preview').clientWidth
      var w = document.getElementById('userImage').clientWidth * scale * this.userImageScale
      var h = document.getElementById('userImage').clientHeight * scale * this.userImageScale
      var x = this.userImageTranslateX * scale
      var y = this.userImageTranslateY * scale
      // console.log(w)
      // console.log(h)
      // console.log(x)
      // console.log(y)
      this.createImage(x, y, w, h)
    },
    createImage: function (x, y, w, h) {
      // 建立合成照片
      var userImage = new Image()
      userImage.src = this.userImage
      var coverImage = new Image()
      coverImage.src = this.coverImage
      var previewHeight = document.getElementById('preview').clientHeight
      var previewWidth = document.getElementById('preview').clientWidth
      var resizeCanvas = document.getElementById('result')
      resizeCanvas.width = 1000
      resizeCanvas.height = previewHeight / previewWidth * 1000
      this.reset()

      var ctx = resizeCanvas.getContext('2d')
      // 上底色
      ctx.rect(0, 0, resizeCanvas.width, resizeCanvas.height)
      ctx.fillStyle = '#ffffff'
      ctx.fill()
      // ctx.rotate(this.userImageRotate / 180 * Math.PI)
      // 描繪使用者上傳圖片
      ctx.drawImage(userImage, x, y, w, h)
      // ctx.rotate(-this.userImageRotate / 180 * Math.PI)
      // 描繪封面圖片
      ctx.drawImage(coverImage, 0, 0, resizeCanvas.width, resizeCanvas.height)

      // 上字
      var size = resizeCanvas.width / 30
      ctx.font = size + 'px sans-serif'

      var text = ctx.measureText(this.name)
      var textW = text.width * 1.15
      var textH = size * 1.75
      var textX = resizeCanvas.width
      var textY = resizeCanvas.height

      ctx.fillStyle = 'rgba(0, 0, 0, 0.75)'
      ctx.fillRect(
          textX - textW,
          textY - textH,
          textW, textH
      )
      ctx.fillStyle = 'rgba(255, 255, 255, 0.9)'
      ctx.fillText(
          this.name,
          textX - (text.width * 1.075),
          textY - (size * 0.5)
      )
      var base64 = resizeCanvas.toDataURL('image/png')
      this.outputImage = base64
      this.making = true
    },
    reset: function () {
      // 重置資料
      this.userImage = ''
      this.coverImage = ''
      this.userImageRotate = 0
      this.userImageScale = 1
      this.userImageTranslateX = 0
      this.userImageTranslateY = 0
      document.getElementById('preview').style.height = 0
      document.getElementById('uploadInput').value = ''
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
.selectImage img{
  width: 100%;
  height: auto;
}
.selectImage label{
  padding:0;
  width: 18%;
  margin: 0 1%;
}
.selectImage input{
  display: none;
}
#preview{
  width: 100%;
  margin: 0 auto;
  max-width: 600px;
  overflow: hidden;
  position: relative;
  margin-bottom: 20px;
}
#userImage{
  display: block;
  width: 100%;
  height: auto;
  position: absolute;
  top: 0;
  left: 0;
  transform-origin: 0 0;
  transition-duration: 0s;
}
#coverimage{
  display: block;
  width: 100%;
  height: auto;
  position: absolute;
  top: 0;
  left: 0;
}
#userName{
  position: absolute;
  padding: 5px 10px;
  font-size: 15px;
  color:#fff;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,.5);
  border-radius: 5px 0 0 5px;
}
#dragger{
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
input#uploadInput{
  display: none;
}

#uploading,
#making{
  position: fixed;
  bottom: 0;
  left: 0;
  background: rgba(0,0,0,.5);
  font-size: 20px;
  line-height: 1.7;
  width: 100%;
  color: #fff;
  z-index: 999;
}
#output {
  max-width: 100%;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in below version 2.1.8 */ {
  opacity: 0
}
.hide{
  /*display: block!important;*/
}
</style>
