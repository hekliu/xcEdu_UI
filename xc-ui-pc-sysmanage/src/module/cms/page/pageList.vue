<template>
  <div>
    <!--查询表单-->
    <el-form :model="params">
      <el-select v-model="params.siteId" placeholder="请选择站点">
        <el-option
          v-for="item in siteList"
          :key="item.siteId"
          :label="item.siteName"
          :value="item.siteId">
        </el-option>
      </el-select>
      页面别名：<el-input v-model="params.pageAlias" style="width: 100px"></el-input>
      <el-button type="primary" v-on:click="query" size="small">查询</el-button>
      <router-link class="mui-tab-item" :to="{path:'/cms/page/add/',query:{
          page: this.params.page,
          siteId: this.params.siteId}}">
        <el-button  type="primary" size="small">新增页面</el-button>
      </router-link>
    </el-form>
    <!--data table-->
    <el-table :data="list" highlight-current-row v-loading="listLoading" style="width: 100%;">
      <el-table-column type="index" width="60">
      </el-table-column>
      <el-table-column prop="pageName" label="页面名称" width="120">
      </el-table-column>
      <el-table-column prop="pageAliase" label="别名" width="120">
      </el-table-column>
      <el-table-column prop="pageType" label="类型（静态、动态）" width="150">
      </el-table-column>
      <el-table-column prop="pageWebPath" label="访问路径" width="250">
      </el-table-column>
      <el-table-column prop="pagePhysicalPath" label="物理路径" width="250">
      </el-table-column>
      <el-table-column prop="pageCreateTime" label="创建时间" width="200" :formatter="formatCreateTime">
      </el-table-column>
      <el-table-column label="编辑" width="80">
        <template slot-scope="scope">
          <el-button
            size="small" type="primary"
            @click="edit(scope.row.pageId)">编辑
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="删除" width="80">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="danger"
            @click="del(scope.row.pageId)">删除
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="页面预览" width="100">
        <template slot-scope="scope">
          <el-button
            size="small" type="primary" plain @click="preview(scope.row.pageId)">页面预览
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="发布" width="80">
        <template slot-scope="scope">
          <el-button
            size="small" type="primary" plain @click="postPage(scope.row.pageId)">发布
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="撤销" width="80">
        <template slot-scope="scope">
          <el-button
            size="small" type="primary" plain @click="redoPage(scope.row.pageId)">撤销
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--分页条-->
    <el-col :span="30" class="toolbar">
      <el-pagination background layout="prev, pager, next"  :page-size="this.params.size"
                     :total="total" :current-page="this.params.page" @current-change="changePage"
                     style="float:right;">
      </el-pagination>
    </el-col>
  </div>
</template>

<script>
    import * as cmsApi from '../api/cms'
    import utilApi from '@/common/utils'
    export default {
      name: 'pageList',
      data () {
        return {
          params: {
            page: 1, // 页码
            size: 5, // 每页显示个数
            siteId: '', // 站点id
            pageAlias: '' // 站点别名
          },
          listLoading: false,
          list: [],
          total: 10,
          siteList: [] // 站点列表
        }
      },
      created () {
        // 从路由上获取参数
        this.params.page = Number.parseInt(this.$route.query.page || 1)
        this.params.siteId = this.$route.query.siteId || ''
      },
      mounted () {
        // 初始查询
        this.query()
        // 初始化站点列表
        this.siteList = [
          {
            siteId: '5a751fab6abb5044e0d19ea1',
            siteName: '门户主站'
          },
          {
            siteId: '102',
            siteName: '测试站'
          }
        ]
      },
      methods: {
        redoPage (pageId) {
          this.$confirm('确认撤销发布的页面吗？', '提示', {}).then(() => {
            cmsApi.pagePostRollBack(pageId).then((res) => {
              if (res.success) {
                this.$message.success('页面撤销成功，请稍后查看效果')
              } else {
                this.$message.error('页面撤销失败！')
              }
            })
          })
        },
        postPage (pageId) {
          this.$confirm('确认发布该页面吗？', '提示', {}).then(() => {
            cmsApi.pagePost(pageId).then((res) => {
              if (res.success) {
                this.$message.success('页面发布成功，请稍后查看效果')
              } else {
                this.$message.error('页面发布失败！')
              }
            })
          })
        },
        preview (pageId) {
          window.open('http://www.xuecheng.com/cms/preview/' + pageId)
        },
        del (pageId) {
          this.$confirm('确认删除此页面吗？', '提示', {}).then(() => {
            console.log(pageId)
            cmsApi.pageDel(pageId).then((res) => {
              if (res.success) {
                this.$message.success('删除成功！')
                this.query()
              } else {
                this.$message.error('删除失败！')
              }
            })
          })
        },
        edit (pageId) {
          this.$router.push({
            path: '/cms/page/edit/' + pageId,
            query: {
              page: this.params.page,
              siteId: this.params.siteId
            }
          })
        },
        query () {
          // console.log(this.params)
          cmsApi.pageList(this.params.page, this.params.size, this.params).then((resp) => {
            // console.log(resp)
            this.total = resp.queryResult.total
            this.list = resp.queryResult.list
          })
        },
        changePage (page) {
          this.params.page = page
          this.query()
        },
        formatCreateTime (row, column) {
          let createTime = new Date(row.pageCreateTime)
          if (createTime) {
            return utilApi.formatDate(createTime, 'yyyy-MM-dd hh:mm:ss')
          }
        }
      }
    }
</script>

<style scoped>

</style>
