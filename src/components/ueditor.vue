<template>
  <div class="mod-demo-ueditor">
    <script :id="ueId" class="ueditor-box" type="text/plain" style="width: 100%; height: 260px;"></script>

    <!-- 获取内容 -->
    <el-dialog
      title="内容"
      :visible.sync="dialogVisible"
      :append-to-body="true">
      {{ ueContent }}
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import ueditor from 'ueditor'
  export default {
    data () {
      return {
        ue: null,
        ueId: `J_ueditorBox_${new Date().getTime()}`,
        ueContent: '',
        dialogVisible: false
      }
    },
    watch:{
      editorStatus(){
        this.getContent();
        this.backContent();
      },
      DatabaseUeContent(){
        this.ue.setContent(this.DatabaseUeContent)
      }
    },
    props:["editorStatus","DatabaseUeContent"],
    mounted () {
      this.ue = ueditor.getEditor(this.ueId, {
        // serverUrl: '', // 服务器统一请求接口路径
        zIndex: 3000,
        toolbars: [
            [
            'anchor', //锚点
            'undo', //撤销
            'redo', //重做
            'bold', //加粗
            'indent', //首行缩进
            'snapscreen', //截图
            'italic', //斜体
            'underline', //下划线
            'strikethrough', //删除线
            'subscript', //下标
            'fontborder', //字符边框
            'superscript', //上标
            'formatmatch', //格式刷
            'blockquote', //引用
            'pasteplain', //纯文本粘贴模式
            'selectall', //全选
            'preview', //预览
            'time', //时间
            'date', //日期
            'mergecells', //合并多个单元格
            'deletetable', //删除表格
            'fontfamily', //字体
            'fontsize', //字号
            'paragraph', //段落格式
            'edittable', //表格属性
            'edittd', //单元格属性
            'emotion', //表情
            'spechars', //特殊字符
            'searchreplace', //查询替换
            'map', //Baidu地图
            'insertvideo', //视频
            'justifyleft', //居左对齐
            'justifyright', //居右对齐
            'justifycenter', //居中对齐
            'justifyjustify', //两端对齐
            'forecolor', //字体颜色
            'backcolor', //背景色
            'insertorderedlist', //有序列表
            'insertunorderedlist', //无序列表
            'imagenone', //默认
            'imageleft', //左浮动
            'imageright', //右浮动
            'attachment', //附件
            'imagecenter', //居中
            'wordimage', //图片转存
            'lineheight', //行间距
            'autotypeset', //自动排版
            'touppercase', //字母大写
            'tolowercase', //字母小写
            'background', //背景
             'scrawl', //涂鸦
            'music', //音乐
            'inserttable', //插入表格
            'charts', // 图表]
        ]
        ]
      })
       this.ue.ready(() => {
          this.ue.setContent(this.DatabaseUeContent)
       })
    },
    methods: {
      backContent(){
        this.$emit('backContent',this.ueContent)
      },
      getContent () {
        this.ue.ready(() => {
          this.ueContent = this.ue.getContent()
        })
      }
    }
  }
</script>

<style lang="scss">
  .mod-demo-ueditor {
    position: relative;
    z-index: 510;
    > .el-alert {
      margin-bottom: 10px;
    }
  }
  .edui-default .edui-toolbar .edui-combox .edui-combox-body{height: 22px;}
</style>
