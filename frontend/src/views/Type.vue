<template>
  <div class="type">
    <div class="container">
      <div class="row">
        <div class="col-xs-12">
          <!-- 考题类型开始 -->
          <div class="panel panel-default" v-show="!editing">
            <div class="panel-heading">
              <h3 class="panel-title">考题类型
              <button class="btn btn-primary btn-xs pull-right" @click="create">新建</button>
              </h3>
            </div>
            <table class="table table-bordered">
              <thead>
                <tr>
                  <th>编号</th>
                  <th>类型</th>
                  <th>日期</th>
                  <th>操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(type, index) in types" :key="index">
                  <td>{{type.id}}</td>
                  <td>{{type.name}}</td>
                  <td>{{type.date}}</td>
                  <td>
                    <button class="btn btn-info btn-xs" @click="edit(index)">
                      <span class="glyphicon glyphicon-edit"></span>
                    </button>&nbsp;
                    <button class="btn btn-danger btn-xs" @click="remove(index)">
                      <span class="glyphicon glyphicon-trash"></span>
                    </button>
                  </td>
                </tr>
                <tr v-if="!types.length">
                  <td colspan="4" class="text-center">暂无题型</td>
                </tr>
              </tbody>
            </table>
          </div>
          <!-- 考题类型结束 -->
          <!-- 编辑类型开始 -->
          <form class="panel panel-default form-horizontal" v-show="editing">
            <div class="panel-heading">
              <h3 class="panel-title">考题类型</h3>
            </div>
            <div class="panel-body">
              <div class="form-group">


                <label for="type" class="col-xs-1 control-label">类型：</label>
                <div class="col-xs-5">
                  <input type="text" class="form-control" id="type" placeholder="请输入题型" v-model.trim="editingType.name">
                </div>


                <label for="view" class="col-xs-1 control-label">视图：</label>
                <div class="col-xs-5">
                  <select id="view" class="form-control" v-model="editingType.view">
                    <option disabled>请选择视图</option>
                    <option v-for="view in views" :key="view.id" :value="view.value">{{view.name}}</option>
                  </select>
                </div>

              </div>
            </div>
            <div class="panel-footer">
              <button type="button" class="btn btn-success btn-sm" @click="save">保存</button>
              <button type="button" class="btn btn-warning btn-sm pull-right" @click="editing = false">取消</button>
            </div>
          </form>
          <!-- 编辑类型结束 -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'type',
  data: function() {
    return {
      editing: false,
      editingType: {},
      types: [],
      views: [],
      url: 'http://localhost:3001',
      error: ''
    }
  },
  created: function() {

    // 获取所有视图
    this.$http.get(this.url + '/views').then(function(res){
      this.views = res.body;
    },
    function(){
      this.error = '获取视图失败';
    });

    // 获取所有类型
    this.$http.get(this.url + '/types').then(function(res){
      this.types = res.body;
    },
    function(){
      this.error = '获取类型失败';
    });
  },
  methods: {
    create: function() {
      this.editingType = {
        name: ''
      };
      this.editing = true;
    },
    edit: function(index) {

      // 将指定索引的类型赋值给 editingType
      this.editingType = Object.assign({}, this.types[index]);

      // 打开编辑界面
      this.editing = true;
    },
    save: function() {
      // 修改或创建时间
      this.editingType.date = new Date();

      if(this.editingType.id) {
        // 修改已有题型
        this.$http.put(this.url + '/types/' + this.editingType.id, this.editingType).then(function(res){

          // 更新页面中的题型
          this.types = res.body;

          // 关闭编辑页面
          this.editing = false;

        },function(){
          this.error = '修改题型失败';
        });

      } else {

        // 创建ID
        this.editingType.id = Date.now();

        // 创建新题型
        this.$http.post(this.url + '/types', this.editingType).then(function(res){

          // 更新页面中的题型
          this.types = res.body;

          // 关闭编辑界面
          this.editing = false;
        },function(){
          this.error = '创建新题型失败';
        });
      }
    },
    remove: function(index) {

      // 删除数据库中的题型
      this.$http.delete(this.url + '/types/' + this.types[index].id).then(function(){

        // 删除界面中的题型
        this.types.splice(index, 1);
      },function(){

      });
    }
  }
}
</script>
