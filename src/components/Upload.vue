/*jshint -W065 */
<template lang="html">
  <div>
    <form style="display: none" name=theform>
      <input type="radio" name="myradio" value="local_name" checked=true/> 上传文件名字保持本地文件名字
      <input type="radio" name="myradio" value="random_name"/> 上传文件名字是随机文件名, 后缀保留
    </form>

    <h4>您所选择的文件列表：</h4>
    <div id="ossfile"></div>

    <br/>

    <div id="container">
      <a id="selectfiles" href="javascript:void(0);" class='btn'>选择文件</a>
      <a id="postfiles" href="javascript:void(0);" class='btn'>开始上传</a>
    </div>

    <pre id="console">{{message}}</pre>

    <p>&nbsp;</p>
  </div>
</template>

<script>
  import plupload from '../../static/pupload/plupload-2.1.2/js/plupload.full.min.js';

  export default {
    name: 'upload',
    data() {
      return {
        ossFile: '',
        uploadMethod: '',
        accessid: '',
        accesskey: '',
        host: '',
        policyBase64: '',
        signature: '',
        callbackbody: '',
        filename: '',
        key: '',
        expire: 0,
        g_object_name: '',
        g_object_name_type: '',
        now: Date.parse(new Date()) / 1000,
        message: '',
      };
    },
    props: {
      fileId: '',
      beforeUpload: Function,
      onSuccess: Function,
      onError: Function,
      onProgress: Function,
    },
    beforeCreate() {
    },
    mounted() {
      this.$nextTick(() => {
        this.upload();
      });
    },
    methods: {
      send_request() {
        let xmlhttp = null;
        if (window.XMLHttpRequest) {
          xmlhttp = new XMLHttpRequest();
        }

        if (xmlhttp != null) {
          // 你的服务端接口地址:参考:http://oss-demo.aliyuncs.com/oss-h5-upload-js-php/
          // 服务端签名后直传:https://help.aliyun.com/document_detail/31926.html
          const serverUrl = 'http://oss-demo.aliyuncs.com/oss-h5-upload-js-php/php/get.php';
          xmlhttp.open('GET', serverUrl, false);
          xmlhttp.send(null);
          return xmlhttp.responseText;
        }
        return '';
      },
      get_signature() {
        // 可以判断当前expire是否超过了当前时间,如果超过了当前时间,就重新取一下.3s 做为缓冲
        this.now = Date.parse(new Date()) / 1000;
        if (this.expire < this.now + 3) {
          const body = this.send_request();
          const e = eval;
          const obj = e(`(${body})`);
          this.host = obj.host;
          this.policyBase64 = obj.policy;
          this.accessid = obj.accessid;
          this.signature = obj.signature;
          this.expire = parseInt(obj.expire, 10);
          this.callbackbody = obj.callback;
          this.key = obj.dir;
          return true;
        }
        return false;
      },
      check_object_radio() {
        const tt = document.getElementsByName('myradio');
        for (let i = 0; i < tt.length; i += 1) {
          if (tt[i].checked) {
            this.g_object_name_type = tt[i].value;
            break;
          }
        }
      },
      random_string(len = 32) {
        const chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
        const maxPos = chars.length;
        let pwd = '';
        for (let i = 0; i < len; i += 1) {
          pwd += chars.charAt(Math.floor(Math.random() * maxPos));
        }
        return pwd;
      },
      get_suffix(filename) {
        const pos = filename.lastIndexOf('.');
        let suffix = '';
        if (pos !== -1) {
          suffix = filename.substring(pos);
        }
        return suffix;
      },
      calculate_object_name(filename) {
        if (this.g_object_name_type === 'local_name') {
          this.g_object_name += `${filename}`;
        } else if (this.g_object_name_type === 'random_name') {
          const suffix = this.get_suffix(filename);
          this.g_object_name = this.key + this.random_string(10) + suffix;
        }
        return '';
      },
      get_uploaded_object_name(filename) {
        if (this.g_object_name_type === 'local_name') {
          let tmpName = this.g_object_name;
          tmpName = tmpName.replace(`${filename}`, filename);
          return tmpName;
        } else if (this.g_object_name_type === 'random_name') {
          return this.g_object_name;
        }
        return '';
      },
      set_upload_param(up, filename, ret) {
        if (ret === false) {
          this.get_signature();
        }
        this.g_object_name = this.key;
        if (filename !== '') {
          this.calculate_object_name(filename);
        }
        const newMultipartParams = {
          key: this.g_object_name,
          policy: this.policyBase64,
          OSSAccessKeyId: this.accessid,
          // 让服务端返回200,不然，默认会返回204
          success_action_status: '200',
          signature: this.signature,
          callback: this.callbackbody,
        };
        up.setOption({
          url: this.host,
          multipart_params: newMultipartParams,
        });
        up.start();
      },
      upload() {
        const that = this;

        const uploader = new plupload.Uploader({
          runtimes: 'html5,flash,silverlight,html4',
          browse_button: 'selectfiles',
          multi_selection: true,
          container: document.getElementById('container'),
          flash_swf_url: '../../static/pupload/plupload-2.1.2/js/Moxie.swf',
          silverlight_xap_url: '../../static/pupload/plupload-2.1.2/js/Moxie.xap',
          url: 'http://oss.aliyuncs.com',
          filters: {
            mime_types: [{
              title: '允许上传文件类型',
              extensions: 'jpg,gif,png,bmp',
            }],
            // 最大只能上传200mb的文件
            max_file_size: '200mb',
            // 不允许队列中存在重复文件
            prevent_duplicates: true,
          },
          init: {
            PostInit: () => {
              this.ossFile = '';
              document.getElementById('postfiles').onclick = () => {
                that.set_upload_param(uploader, '', false);
                // console.log('...');
                return false;
              };
            },
            FilesAdded: (up, files) => {
              plupload.each(files, (file) => {
                console.log('file_name: ', file.name, ',    ', 'file_size:  ', file.size);
                document.getElementById('ossfile').innerHTML += `<div id="${file.id}">${file.name} (${plupload.formatSize(file.size)})<b></b><div class="progress"><div class="progress-bar" style="width: 0%"></div></div></div>`;
                that.message = '';
              });
            },

            BeforeUpload: (up, file) => {
              that.check_object_radio();
              that.set_upload_param(up, file.name, true);
            },
            UploadProgress: (up, file) => {
              const d = document.getElementById(file.id);
              d.getElementsByTagName('b')[0].innerHTML = `<span>${file.percent}%</span>`;
              const prog = d.getElementsByTagName('div')[0];
              const progBar = prog.getElementsByTagName('div')[0];
              progBar.style.width = `${2 * file.percent}px`;
              progBar.setAttribute('aria-valuenow', file.percent);
            },
            FileUploaded: (up, file, info) => {
              if (info.status === 200) {
                document.getElementById(file.id).getElementsByTagName('b')[0].innerHTML
                  = `upload to oss success, object name:${this.get_uploaded_object_name(file.name)}`;
              } else {
                document.getElementById(file.id).getElementsByTagName('b')[0].innerHTML = info.response;
              }
            },
            Error: (up, err) => {
              console.log('上传失败：', err, that.onError, up);
              if (err.code === -600) {
                that.message = '文件大小超出限制，限制大小为200M';
              } else if (err.status === 403) {
                // console.log('tocken过期，请刷新页面后再上传文件!');
                that.message = '页面失效，请刷新页面后重新上传文件!';
              } else {
                // console.log('文件上传失败，请刷新页面后重新上传文件！');
                that.message = '上传失败，请刷新页面后重新上传文件！';
              }
              if (that.onError) {
                that.onError(err.message, up, err);
              }
            },
          },
        });
        uploader.init();
      },
    },
  };
</script>

<style>
  .btn {
    color: #fff;
    background-color: #54b9f9;
    border-color: #54b9f9;
    display: inline-block;
    padding: 6px 12px;
    margin-bottom: 0;
    font-size: 14px;
    font-weight: 400;
    line-height: 1.42857143;
    text-align: center;
    white-space: nowrap;
    text-decoration: none;
    vertical-align: middle;
    -ms-touch-action: manipulation;
    touch-action: manipulation;
    cursor: pointer;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    background-image: none;
    /*border: 1px solid transparent;*/
    border-radius: 4px;
  }

  a.btn:hover {
    background-color: #29e0f9;
  }

  .progress {
    margin-top: 2px;
    width: 200px;
    height: 14px;
    display: inline-block;
    margin-bottom: 10px;
    overflow: hidden;
    background-color: #f5f5f5;
    border-radius: 4px;
    -webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
  }

  .progress-bar {
    background-color: rgb(84, 185, 249);
    background-image: linear-gradient(45deg, rgba(255, 255, 255, 0.14902) 25%, transparent 25%, transparent 50%, rgba(255, 255, 255, 0.14902) 50%, rgba(255, 255, 255, 0.14902) 75%, transparent 75%, transparent);
    background-size: 40px 40px;
    box-shadow: rgba(0, 0, 0, 0.14902) 0 -1px 0px 0px inset;
    box-sizing: border-box;
    color: rgb(255, 255, 255);
    display: block;
    float: none;
    font-size: 12px;
    height: 20px;
    line-height: 20px;
    text-align: center;
    transition-delay: 0s;
    transition-duration: 0.6s;
    transition-property: width;
    transition-timing-function: ease;
    width: 266px;
  }
</style>
