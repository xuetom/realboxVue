<template>
  <div class="home">
    <nav-bar></nav-bar>
    <breadcrumb></breadcrumb>
    <div class="content">
      <ul class="fastNav">
        <li>
          <a @click="newPlay">
            <i class="iconfont icon-server-kuaisufabu"></i>
            <b>{{$t('Content.ID_QUICK_PUBLISH')}}</b>
          </a>
        </li>
        <li>
          <router-link to="auditList">
            <i class="iconfont icon-shenhe"></i>
            <b>{{$t('Content.ID_AUDIT_LIST')}}</b>
          </router-link>
        </li>
        <li>
          <router-link to="programList">
            <i class="iconfont icon-jiemu"></i>
            <b>{{$t('Content.ID_PROGRAM')}}</b>
          </router-link>
        </li>
        <li>
          <router-link to="terminal">
            <i class="iconfont icon-zhongduan"></i>
            <b>{{$t('Content.ID_TERMINAL')}}</b>
          </router-link>
        </li>
        <li>
          <router-link to="resource">
            <i class="iconfont icon-ziyuanguanli"></i>
            <b>{{$t('Content.ID_RESOURCE')}}</b>
          </router-link>
        </li>
      </ul>
      <div style="font-size: 2rem;padding-left: 100px;margin-bottom: 20px;">{{$t('Content.ID_RELEASE_STEPS')}}：</div>
      <el-steps :active="4" align-center>
        <el-step :title="$t('Content.ID_SELECT_TEMPLATE')" :description="$t('Msg.ID_MSG_1')"></el-step>
        <el-step :title="$t('Content.ID_SELECT_RESOURCE')" :description="$t('Msg.ID_MSG_2')"></el-step>
        <el-step :title="$t('Content.ID_SETTING_PROPERTY')" :description="$t('Msg.ID_MSG_3')"></el-step>
        <el-step :title="$t('Content.ID_PUBLISH_PROGRAM')" :description="$t('Msg.ID_MSG_4')"></el-step>
      </el-steps>
    </div>
    <footer-bar></footer-bar>
    <el-dialog
      :title="$t('Content.ID_SELECT_TEMPLATE')"
      :visible.sync="setTem"
      width="58%"
      top="1vh"
    >
      <div class="temDialog">
        <div id="templateTree">
          <div class="title">{{$t('Content.ID_TEMPLATE')}}</div>
          <el-tree :data="templateTree" node-key="id" @node-click="temTreeClick" :expand-on-click-node="false"
                   default-expand-all></el-tree>
        </div>
        <div id="templateList">
          <div class="controlBox">
            <div class="search">
              <div style="width: 110px">
                <el-select v-model="tValue" :placeholder="$t('Content.ID_IMAGE_MODE')">
                  <el-option v-for="item in tSelect"
                             :key="item.value"
                             :label="item.label"
                             :value="item.value"></el-option>
                </el-select>
              </div>
              <div style="width:200px;">
                <el-input :placeholder="$t('Msg.ID_MSG_5')">
                  <template slot="prepend">{{$t('Content.ID_TEMPLATE_NAME')}}</template>
                </el-input>
              </div>
              <el-button>{{$t('Content.ID_RESEARCH')}}</el-button>
            </div>
          </div>
          <div v-if="tValue == '2'" class="tableList">
            <el-table :data="templates" style="width: 100%">
              <el-table-column prop="name" align="center" :label="$t('Content.ID_TEMPLATE_NAME')"></el-table-column>
              <el-table-column prop="preview" align="center" :label="$t('Content.ID_THUMBNAIL')">
                <template slot-scope="scope">
                  <img :src="scope.row.preview" width="30" height="50"/>
                </template>
              </el-table-column>
              <el-table-column prop="resolution" align="center" :label="$t('Content.ID_RESOLUTION')"></el-table-column>
              <el-table-column prop="address" align="center" :label="$t('Content.ID_TERMINAL_TYPE')"></el-table-column>
              <el-table-column prop="creator" align="center" :label="$t('Content.ID_CREATOR')"></el-table-column>
              <el-table-column prop="updateTime" align="center" :label="$t('Content.ID_UPDATE_TIME')"></el-table-column>
              <el-table-column prop="desc" align="center" :label="$t('Content.ID_DESCRIPTION')"></el-table-column>
            </el-table>
          </div>
          <div v-else class="templateBox">
            <ul class="templateList">
              <li v-for="(template,id) in templates" :key="id" :id="template.id"
                  @click="selectedTem($event)" @dblclick="editPlay(template.name)">
                <label class="el-upload-list__item-status-label">
                  <i class="el-icon-upload-success el-icon-check"></i>
                </label>
                <div class="imgBox">
                  <img :src="template.preview">
                </div>
                <p>{{template.name}}</p>
              </li>
            </ul>
          </div>
          <div class="page">
            <el-pagination
              @current-change="temChange"
              :page-size="tPageCount"
              layout="total, prev, pager, next, jumper"
              :total="tTotal">
            </el-pagination>
          </div>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="selectTem">{{$t('Content.ID_OK')}}</el-button>
  </span>
    </el-dialog><!--选择模板-->
  </div>
</template>

<script>
  import NavBar from '@/components/common/Navbar'
  import FooterBar from '@/components/common/footer'
  import Breadcrumb from '@/components/common/Breadcrumb'
  import {getTemplate} from '@/api/template'
  import {getTree} from '@/api/Tree'

  export default {
    data() {
      return {
        //模板选择
        setTem: false,
        tTreeId: 22,
        templates: [],          //模板列表
        templateTree: [],
        tValue: '',
        tSelect: [{
          value: '1',
          label: this.$t('Content.ID_IMAGE_MODE')
        }, {
          value: '2',
          label: this.$t('Content.ID_LIST_MODE')
        }],
        tPageCount: 12,     //每页显示数目
        tPageNo: 1,          //当前页
        tTotal: 0,            //总数目
        temName: '',          //模板名
      }
    },
    components: {
      NavBar,
      FooterBar,
      Breadcrumb
    },
    methods: {
      /*模板事件*/
      newPlay() {
        this.setTem = true;
        this.queryTemList();
        this.getTemTree()
      },                       //新建节目
      queryTemList() {
        let _this = this;
        this.templates = [];
        let params = {
          groupId: this.tTreeId,
          pageNo: this.tPageNo,
          pageCount: this.tPageCount
        };
        getTemplate(params).then(response => {
          let templates = response.cust.templates;
          _this.tTotal = response.cust.pages.count;
          for (let template of templates) {
            _this.templates.push(template);
          }
        })
      },                  //获取模板列表
      temTreeClick(val) {
        this.tTreeId = val.id;
        this.queryTemList()
      },               //点击模板树回调
      temChange(val) {
        this.tPageNo = val;
        this.queryTemList()
      },                  //模板翻页回调
      getTemTree() {
        let _this = this;
        let params = {
          id: 20
        };
        getTree(params).then(response => {
          _this.templateTree = response.cust.trees
        })
      },                    //查询模板树
      selectedTem(e) {
        let dom = e.currentTarget.children[2].innerHTML;
        let target = e.currentTarget;
        let children = target.children[0];
        //单选判定所需
        let p = target.parentNode.childNodes;
        let id = e.currentTarget.id;

        if (children.style.display != 'block') {
          children.style.display = 'block';
          target.style.backgroundColor = "#ebebeb";
          this.temName = dom;
          //去掉兄弟元素的选择项
          if (p.length > 1) {
            for (let i = 0; i <= p.length; i++) {
              if (p[i].id !== id) {
                p[i].children[0].style.display = 'none';
              }
            }
          }
        } else {
          children.style.display = 'none';
          target.style.backgroundColor = "white";
          this.temName = ''
        }
      },                  //单击选择文件
      selectTem() {
        if (this.temName === '') {
          this.$message({
            message: '未选择模板！',
            showClose: true,
            center: true,
            type: 'warning'
          });
          return false;
        }
        this.setTem = false;
        this.$router.push({path: '/programMack', query: {name: this.temName, groupId: this.tTreeId, type: 0}})
      },                     //新建模板
    }
  }
</script>

<style scoped>
  .home {
    width: 100%;
    height: 100%;
    background-color: #eeeeee;
  }

  .content {
    height: 73%;
    border-radius: 10px;
    margin: 20px 20px 0px;
    background-color: white;
    padding: 5% 270px 0;
    box-shadow: 0px 7px 4.8px 3.2px rgba(0, 0, 0, 0.1);
  }

  .fastNav {
    display: flex;
    justify-content: space-around;
    margin-bottom: 200px;
  }

  .fastNav li {
    text-align: center;
  }

  .fastNav li a b {
    color: #d33a31;
    display: block;
    font-size: 2rem;
  }

  .fastNav li a i {
    font-size: 100px;
    color: #d33a31;
  }

  /*模板样式*/
  .temDialog {
    width: 100%;
    height: 100%;
    text-align: center;
    overflow: hidden;
    display: flex;
    justify-content: space-between;
  }

  #templateTree {
    border: 1px solid #c1c1c1;
    overflow: hidden;
    border-radius: 10px;
    width: 12%;
    min-width: 120px;
  }

  #templateList {
    border: 1px solid #c1c1c1;
    height: 98%;
    overflow: hidden;
    border-radius: 10px;
    width: 87%;
  }

  .controlBox {
    height: 40px;
    line-height: 40px;
    text-align: right;
    padding: 10px;
    border-bottom: 1px solid #e0e0e0;
    display: flex;
    justify-content: space-between;
  }

  .control a {
    margin-left: 10px;
    font-size: 1.4rem;
    border-right: 1px solid #cfcfcf;
    padding-right: 10px;
  }

  .control > a > i {
    margin-right: 5px;
  }

  .search {
    display: flex;
  }

  .search > div {
    margin-right: 10px;
  }

  .templateBox, .tableList {
    padding: 20px;
    height: 580px;
  }

  .templateList {
    display: flex;
    flex-wrap: wrap;
  }

  .templateList li {
    width: 150px;
    height: 150px;
    overflow: hidden;
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    margin: 0 30px 40px;
    cursor: pointer;
    padding: 10px 5px;
    border-radius: 5px;
    position: relative;
  }

  .imgBox {
    width: 130px;
    height: 130px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: white;
    border: 1px solid #e7e7e7;
  }

  .templateList li img {
    max-width: 120px;
    max-height: 120px;
  }

  .templateList li:hover {
    background-color: #ebebeb !important;
  }

  .templateList p {
    min-width: 20px;
  }

  .controlTree {
    height: 40px;
    line-height: 40px;
    text-align: right;
    padding-right: 10px;
    border-bottom: 1px solid #dedede;
  }

  .controlTree > a {
    margin-left: 15px;
  }

  .controlTree > a > i {
    margin-right: 3px;
  }

  .controlTree > a:hover {
    color: #d33a31;
  }

  .title {
    height: 40px;
    background-color: #d33a31;
    line-height: 40px;
    padding-left: 10px;
    font-size: 1.4rem;
    color: white;
    letter-spacing: 3px;
  }

  .el-upload-list__item-status-label {
    position: absolute;
    right: -15px;
    top: -7px;
    width: 46px;
    height: 26px;
    background: #13ce66;
    text-align: center;
    transform: rotate(45deg);
    box-shadow: 0 1px 1px #ccc;
  }

  .el-upload-list__item-status-label i {
    font-size: 12px;
    margin-top: 12px;
    transform: rotate(-45deg);
    color: white;
  }
</style>
