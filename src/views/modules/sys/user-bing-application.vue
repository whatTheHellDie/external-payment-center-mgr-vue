<template>
  <el-dialog
    title="绑定应用"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-transfer
      filterable
      :button-texts="['解除', '绑定']"
      :titles="['应用列表', '已选应用']"
      filter-placeholder="请输入应用名称"
      v-model="value"
      @change="handleChange"
      :props="{
        key: 'value',
        label: 'name'
      }"
      :data="data">
    </el-transfer>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        userId: '',
        data: this.generateData(),
        value: []
      };
    },
    methods: {
      init (id) {
        this.visible = true;
        this.userId = id;
        this.getDefaultValue();
      },
      getDefaultValue () {
        this.$http({
          url: this.$http.adornUrl('/payuserapplication/getApplicationId'),
          method: 'get',
          params: this.$http.adornParams({
            'userId': this.userId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.value = data.applicatoinIds;
          }
        })
      },
      generateData () {
        var dataArray = [];
        this.$http({
          url: this.$http.adornUrl('/payapplication/allList'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            data.list.forEach((d, index) => {
                dataArray.push({value: d.applicationId, name: d.applicationName});
            })
          }
        })
        return dataArray;
      },
      // 表单提交
      handleChange (value, direction, movedKeys) {
             if(direction === "right") {
                this.movedRight(movedKeys);
             }
             if(direction === "left") {
               this.movedLeft(movedKeys);
             }
      },
      movedRight (data) {
        this.$http({
            url: this.$http.adornUrl(`/payuserapplication/save`),
            method: 'get',
            params: {'ids': data.join(','), 'userId': this.userId}
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '绑定成功',
                type: 'success',
                duration: 1500
              });
            } else {
              this.$message.error(data.msg);
            }
          })
      },
      movedLeft (data) {
        this.$http({
            url: this.$http.adornUrl(`/payuserapplication/delete`),
            method: 'get',
            params: {'ids': data.join(','), 'userId': this.userId}
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '解除成功',
                type: 'success',
                duration: 1500
              });
            } else {
              this.$message.error(data.msg);
            }
          })
      }
    }
  }
</script>
<style>
.el-transfer-panel {
  width: 43%;
}
.el-transfer-panel__body {
  height: 450px;
}
</style>
