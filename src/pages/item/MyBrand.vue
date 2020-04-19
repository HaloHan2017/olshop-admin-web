<template>
  <v-card>
    <!-- 卡片的头部 -->
    <v-card-title>
      <v-btn @click="addBrand" color="primary">新增品牌</v-btn>
      <!--空间隔离组件-->
      <v-spacer/>
      <!--搜索框，与search属性关联-->
      <v-text-field
        append-icon="search"
        label="搜索"
        single-line
        hide-details
        v-model.lazy="search"
      />
    </v-card-title>
    <!-- 分割线 -->
    <v-divider/>
    <!--卡片的中部-->
    <v-data-table
      :headers="headers"
      :items="brands"
      :search="search"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td>{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img :src="props.item.image"></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="justify-center layout">
          <v-btn icon @click="editBrand(props.item)">
            <i class="el-icon-edit"/>
          </v-btn>
          <v-btn icon @click="deleteBrand(props.item)">
            <i class="el-icon-delete"/>
          </v-btn>
        </td>
      </template>
      <!--alert警告框-->
      <template slot="no-data" slot-scope="props">
        <v-alert :value="true" color="error" icon="warning">
          对不起，没有查询到任何数据 :(
        </v-alert>
      </template>
      <!--分页条中文显示-->
      <template slot="pageText" slot-scope="props">
        共{{props.itemsLength}}条,当前:{{ props.pageStart }} - {{ props.pageStop }}
      </template>
    </v-data-table>

    <!--新增、修改品牌 模态框-->
    <v-dialog v-model="show" max-width="600" scrollable v-if="show">
      <v-card>
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="show = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5 py-2">
          <!-- 表单 -->
          <brand-form :oldBrand="brand" :isEdit="isEdit" @close="show = false" :reload="getDataFromServer"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  import BrandForm from './BrandForm'

  export default {
    name: "MyBrand",
    components: {
      BrandForm
    },
    data() {
      return {
        search: '', // 搜索过滤字段
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,}
        ],
        show: false,// 是否弹出窗口
        oldBrand: {}, // 品牌信息
        isEdit: false // 判断是编辑还是新增
      }
    },
    mounted() { // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    methods: {
      getDataFromServer() {
        this.loading = true;
        this.$http.get("item/brand/page", {
          params: {
            key: this.search, // 搜索条件
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending// 是否降序
          }
        }).then(resp => {
          console.log(resp);
          this.brands = resp.data.items;
          this.totalBrands = resp.data.total;
          this.loading = false;
        })
      },
      addBrand() {
        this.brand = {};
        this.isEdit = false;
        this.show = true;
      },
      editBrand(item) {
        this.brand = item;
        this.isEdit = true;
        this.show = true;
        // 查询商品分类信息，进行回显
        this.$http.get("/item/category/bid/" + item.id)
          .then(resp => {
            this.brand.categories = resp.data;
          })
      },
      deleteBrand(item) {
        this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(() => {
          // 发起删除请求
          this.$http.delete("/item/brand?id=" + item.id,)
            .then(() => {
              // 删除成功，重新加载数据
              this.$message.success("删除成功！");
              this.getDataFromServer();
            })
        }).catch(() => {
          this.$message.info("删除已取消！");
        });
      }
    },
    watch: {
      pagination: {
        deep: true,
        handler() {
          this.getDataFromServer();
        }
      },
      search: {
        handler() {
          this.getDataFromServer();
        }
      },
      show(val, oldVal) {
        // 表单关闭后重新加载数据
        if (oldVal) {
          this.getDataFromServer();
        }
      },
      oldBrand: {// 监控Brand的变化
        handler(val) {
          if(val){
            // 注意不要直接复制，否则这边的修改会影响到父组件的数据，copy属性即可
            this.brand =  Object.deepCopy(val)
          }else{
            // 为空，初始化brand
            this.brand = {
              name: '',
              letter: '',
              image: '',
              categories: [],
            }
          }
        },
        deep: true
      }
    }
  }
</script>

<style scoped>

</style>
