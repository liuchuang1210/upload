<template>
    <div class="main" id="main">
        <img :src="item" v-for="item in imgArr" class="img-item">
        <input type="file" id="choose" accept="image/*" capture="camera" @change="chooseImg($event)"/>

    </div>
</template>
<script>

    export default {
        data(){
            return{
                imgArr:[],
                imgInfo:{
                    orientation:''
                }
            }
        },
        methods: {
            async chooseImg(e) {

                //判断大于9张
                if (this.imgArr.length > 8) {
                    layer.open({'content': '最多同时只可上传9张图片'});
                    return;
                }
                //判断是否有值
                if (!e.target.files.length) {
                    return false
                }

                var maxsize = 100 * 1024;
                var _self = this
                var file = e.target.files[0]
                console.log(file)
                //判断图片格式
                if (!/\/(?:jpeg)/i.test(file.type)) {
                    layer.open({'content': '图片格式错误'});
                    return false
                }
                var orientation = EXIF.getData(file, async function () {
                    let orientation = EXIF.getTag(this, 'Orientation')

                    // 获取图片大小
                    var size = file.size / 1024 > 1024 ? (~~(10 * file.size / 1024 / 1024)) / 10 + "MB" : ~~(file.size / 1024) + "KB";
                    var type=file.type

                    var reader = new FileReader();

                    reader.onload =  function (e) {
                        var result = this.result;
                        var img = new Image();
                        img.src = result;
                        _self.compress(img, orientation,type)


                    }
                    reader.readAsDataURL(file)

                });


            },
            compress(img, orientation,type) {
                //用于压缩图片的canvas
                var _self = this
                var canvas = document.createElement("canvas");
                // var canvas = this.$refs.canvas
                var ctx = canvas.getContext('2d');
                // //瓦片canva
                // var tCanvas = document.createElement("canvas");
                // var tctx = tCanvas.getContext("2d");

                img.onload = function () {
                    var initSize = this.src.length;
                    var width = this.width;
                    var height = this.height;

                    canvas.width = width;
                    canvas.height = height;
                    // 铺底色
                    ctx.fillStyle = "#fff";
                    ctx.fillRect(0, 0, canvas.width, canvas.height);
                    if (orientation && orientation != 1) {
                        switch (orientation) {
                            case 6:     // 旋转90度
                                canvas.width = height;
                                canvas.height = width;

                                ctx.rotate(Math.PI / 2);
                                // (0,-imgHeight) 从旋转原理图那里获得的起始点
                                ctx.drawImage(img, 0, -height, width, height);
                                break;
                            case 3:     // 旋转180度
                                ctx.rotate(Math.PI);
                                ctx.drawImage(img, -width, -height, width, height);
                                break;
                            case 8:     // 旋转-90度
                                canvas.width = width;
                                canvas.height = height;
                                ctx.rotate(3 * Math.PI / 2);
                                ctx.drawImage(img, -width, 0, width, height);
                                break;
                        }
                    } else {
                        canvas.width = width;
                        canvas.height = height;
                        ctx.drawImage(img, 0, 0, width, height);
                    }
                    var ndata = canvas.toDataURL('image/jpeg', 0.5);
                    _self.upload(ndata,type)
                }
            },
            getBlob(buffer, format) {
                try {
                    return new Blob(buffer, {type: format});
                } catch (e) {
                    var bb = new (window.BlobBuilder || window.WebKitBlobBuilder || window.MSBlobBuilder);
                    buffer.forEach(function (buf) {
                        bb.append(buf);
                    });
                    return bb.getBlob(format);
                }
            },
            FormDataShim() {
                console.warn('using formdata shim');
                var o = window,
                    parts = [],
                    boundary = Array(21).join('-') + (+new Date() * (1e16 * Math.random())).toString(36),
                    oldSend = XMLHttpRequest.prototype.send;
                this.append = function(name, value, filename) {
                    parts.push('--' + boundary + '\r\nContent-Disposition: form-data; name="' + name + '"');
                    if (value instanceof Blob) {
                        parts.push('; filename="' + (filename || 'blob') + '"\r\nContent-Type: ' + value.type + '\r\n\r\n');
                        parts.push(value);
                    }
                    else {
                        parts.push('\r\n\r\n' + value);
                    }
                    parts.push('\r\n');
                };
            },
            getFormData() {
                var isNeedShim = ~navigator.userAgent.indexOf('Android')
                    && ~navigator.vendor.indexOf('Google')
                    && !~navigator.userAgent.indexOf('Chrome')
                    && navigator.userAgent.match(/AppleWebKit\/(\d+)/).pop() <= 534;
                return isNeedShim ? new this.FormDataShim() : new FormData()
            },

            upload(url,type){
                console.log(type)
                var text = window.atob(url.split(",")[1]);
                var buffer = new Uint8Array(text.length);
                for (var i = 0; i < text.length; i++) {
                    buffer[i] = text.charCodeAt(i);
                    // console.log(text.charCodeAt(i))
                }
                var blob1 = this.getBlob([buffer],type);
                var formdata = this.getFormData();

                formdata.append("uploadFile", blob1,'image.jpeg');
                $.ajax({
                    url:'',
                    type:'post',
                    dataType:'json',
                    contentType:false,
                    processData:false,
                    data:formdata,
                    success:function(data){
                        console.log(data)
                    }
                })
            }

        }
    }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
    .main
        width 100%
        height 100%
        overflow-y scroll
        -webkit-overflow-scrolling touch
        .img-item
            width 100%
</style>
