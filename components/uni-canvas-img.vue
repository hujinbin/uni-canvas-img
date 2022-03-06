<template>
<view>
    <view class="personal_li"
                  @click="shareClick">
              <image src="../../static/address.png"
                     mode="widthFix"
                     class="iconImage"></image>
              <text class="font14">分享生成图片</text>
              <image src="../../static/jt.png"
                     mode="widthFix"
                     class="jtIcon"></image>
            </view>
        <view class="canvasContent" v-if="canvasShow">
            <canvas canvas-id="shareCanvas" class="canvasName"></canvas>
            <view class="canvasText">图片已保存到相册，可分享给好友</view>
            <image src="../../static/error.png" class="errorImage" @click="canvasShow = false"></image>
        </view>
</view>
</template>

<script>
export default {
  data() {
    return {
            canvasShow: false
    }
  },
  methods: {
        //这是一个封装好的方法 
        promisify: api => {
            return (options, ...params) => {
                return new Promise((resolve, reject) => {
                    const extras = {
                        success: resolve,
                        fail: reject
                    }
                    api({ ...options, ...extras }, ...params)
                })
            }
        },
        shareClick() {         
            const wxGetImageInfo = this.promisify(uni.getImageInfo)
            Promise.all([
                    // 图片目前只随机找了几张图片，后期可自行替换
                    wxGetImageInfo({
                            src: 'http://pics.ksudi.com/pic/2019/soms/companycard/jd2.png'   // 背景图片
                    }),
                    wxGetImageInfo({
                            src: 'http://pics.ksudi.com/pic/2019/soms/companycard/st2.png'   // 二维码图片，二维码图片如需要携带参数，可根据接口将需要扫码进入页面的路径+参数传入后端，后端可根据生产小程序二维码路径，将路径放入这里就ok了,<a href="https://www.jianshu.com/p/5f96a4f91b9c" target="_blank" rel="noopener">可参考</a>
                    })
            ]).then(res => {
                    console.log(3454)
                    const ctx = wx.createCanvasContext('shareCanvas')
                    console.log(ctx)
                    // 底图
                    ctx.drawImage(res[0].path, 0, 0, 600, 700)
                    // 作者名称
                    ctx.setTextAlign('center')    // 文字居中
                    ctx.setFillStyle('#000000')  // 文字颜色：黑色
                    ctx.setFontSize(22)         // 文字字号：22px
                    ctx.fillText('作者：张杰', 300 / 2, 100)
                    // 小程序码
                    const qrImgSize = 150
                    ctx.drawImage(res[1].path, (340 - qrImgSize) / 2, 230, qrImgSize, qrImgSize)
                    ctx.stroke()
                    // 绘图生成临时图片
                    console.log('res', res)
                    ctx.draw(false,() => {
                        this.tempFileImage()
                    })
                    this.canvasShow = true
            })
        },
        tempFileImage() {
            let that = this
            uni.canvasToTempFilePath({
                    canvasId: 'shareCanvas',
                    success: (res) => {
                            uni.hideLoading()
                            that.savePic(res.tempFilePath)
                    },
                    fail:function () {
                            //TODO
                    }
            })
        },
        //保存
        savePic (filePath) {
            console.log('filePath', filePath)
            uni.showLoading({
                    title: '正在保存'
            });
            uni.saveImageToPhotosAlbum({
                    filePath: filePath,
                    success: function () {
                            uni.showToast({
                                    title: '图片保存成功～'
                            });
                    },
                    fail: function (e) {
                            //TODO
                    },
                    complete: function (){
                            uni.hideLoading()
                    }
            });
        }
  }
}
</script>

<style scoped>
.canvasContent{
        position: fixed;
        bottom: 0;
        left: 0;
        right: 0;
        top: 0;
        background: rgba(0,0,0,0.5);
        display: flex;
        align-items: center;
        flex-direction: column;
        padding-top: 50upx;
}
    .canvasContent .canvasName{
            width: 80%;
            height: calc(100vh - 300upx);
        }
    .canvasContent .canvasText{
            margin: 30upx 0 20upx;
            color: #FFFFFF;
        }
    .canvasContent .errorImage{
            width: 80upx;
            height: 80upx;
        }
</style>