<template>
  <el-card>
    <div class="card-content">
      <div class="ms-main-div" @click="showAll">

        <!--操作按钮-->
        <div class="ms-opt-btn">
          <ms-table-button v-if="type!='add'" :is-tester-permission="true"
                           id="inputDelay"
                           type="primary"
                           :content="$t('commons.save')"
                           size="small" @click="saveCase"
                           icon=""
                           title="ctrl + s"/>
          <el-dropdown v-else split-button type="primary" class="ms-api-buttion" @click="handleCommand"
                       @command="handleCommand" size="small" style="float: right;margin-right: 20px">
            {{ $t('commons.save') }}
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="ADD_AND_CREATE">{{ $t('test_track.case.save_create_continue') }}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </div>
        <el-form :model="form" :rules="rules" ref="caseFrom" v-loading="result.loading" class="case-form">
          <div class="tip">{{ $t('test_track.plan_view.base_info') }}</div>
          <el-row>
            <el-col :span="7">
              <el-form-item
                :placeholder="$t('test_track.case.input_name')"
                :label="$t('test_track.case.name')"
                :label-width="formLabelWidth"
                prop="name">
                <el-input :disabled="readOnly" v-model="form.name" size="small" class="ms-case-input"></el-input>
              </el-form-item>
            </el-col>

            <el-col :span="7">
              <el-form-item :label="$t('test_track.case.module')" :label-width="formLabelWidth" prop="module">
                <ms-select-tree :disabled="readOnly" :data="treeNodes" :defaultKey="form.module" :obj="moduleObj"
                                @getValue="setModule" clearable checkStrictly size="small" />
              </el-form-item>
            </el-col>
            <el-col :span="7">
              <el-form-item label="状态" :label-width="formLabelWidth" prop="status">
                <el-select size="small" v-model="form.status" class="ms-case-input">
                  <el-option v-for="item in statuOptions" :key="item.id" :label="item.label" :value="item.id">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>

          <el-row>
            <!--            <el-col :span="7">-->
            <!--              <el-form-item label="评审状态" :label-width="formLabelWidth" prop="reviewStatus">-->
            <!--                <el-select size="small" v-model="form.reviewStatus" class="ms-case-input">-->
            <!--                  <el-option v-for="item in options" :key="item.id" :label="item.label" :value="item.id">-->
            <!--                  </el-option>-->
            <!--                </el-select>-->
            <!--              </el-form-item>-->
            <!--            </el-col>-->
            <el-col :span="7">
              <el-form-item :label="$t('commons.tag')" :label-width="formLabelWidth" prop="tag">
                <ms-input-tag :currentScenario="form" v-if="showInputTag" ref="tag" class="ms-case-input"/>
              </el-form-item>
            </el-col>
            <el-col :span="7">
              <el-form-item label="责任人" :label-width="formLabelWidth" prop="maintainer">
                <el-select :disabled="readOnly" v-model="form.maintainer"
                           :placeholder="$t('test_track.case.input_maintainer')" filterable class="ms-case-input">
                  <el-option
                    v-for="item in maintainerOptions"
                    :key="item.id"
                    :label="item.id + ' (' + item.name + ')'"
                    :value="item.id">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="7">
              <el-form-item :label="$t('test_track.case.priority')" :label-width="formLabelWidth" prop="priority">
                <el-select :disabled="readOnly" v-model="form.priority" clearable
                           :placeholder="$t('test_track.case.input_priority')" class="ms-case-input">
                  <el-option label="P0" value="P0"></el-option>
                  <el-option label="P1" value="P1"></el-option>
                  <el-option label="P2" value="P2"></el-option>
                  <el-option label="P3" value="P3"></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>

            <el-col :span="7">
              <el-form-item :label="$t('test_track.case.relate_test')" :label-width="formLabelWidth">
                <el-cascader :options="sysList" filterable placeholder="请选择要关联的测试" show-all-levels
                             v-model="form.selected" :props="props"
                             class="ms-case" @change="clearInput" ref="cascade"></el-cascader>
              </el-form-item>
            </el-col>
            <el-col :span="7">
              <el-form-item label="关联需求" :label-width="formLabelWidth" prop="demandId">
                <el-select filterable :disabled="readOnly" v-model="form.demandId" @visible-change="visibleChange"
                           placeholder="请选择要关联的需求" class="ms-case-input">
                  <el-option
                    v-for="item in demandOptions"
                    :key="item.id"
                    :label="item.platform + ': '+item.name"
                    :value="item.id">
                  </el-option>
                </el-select>
              </el-form-item>

            </el-col>
            <el-col :span="7">
              <el-form-item label="需求ID/名称" :label-width="formLabelWidth" prop="demandName"
                            v-if="form.demandId=='other'">
                <el-input v-model="form.demandName"></el-input>
              </el-form-item>
            </el-col>
          </el-row>

          <div class="tip">步骤信息</div>
          <el-row style="margin-top: 10px;">
            <el-col :span="1" :offset="1">{{ $t('test_track.case.prerequisite') }}:</el-col>
            <el-col :span="19">
              <el-form-item prop="prerequisite">
                <el-input :disabled="readOnly" v-model="form.prerequisite"
                          type="textarea"
                          :autosize="{ minRows: 2, maxRows: 4}"
                          :rows="2"
                          :placeholder="$t('test_track.case.input_prerequisite')"></el-input>
              </el-form-item>
            </el-col>
          </el-row>

          <el-row>
            <el-col :span="1" :offset="1">{{ $t('test_track.case.steps') }}:</el-col>
            <el-col :span="19">
              <el-table
                v-if="isStepTableAlive"
                :data="form.steps"
                class="tb-edit"
                border
                size="mini"
                :default-sort="{prop: 'num', order: 'ascending'}"
                highlight-current-row>
                <el-table-column :label="$t('test_track.case.number')" prop="num" min-width="10%"></el-table-column>
                <el-table-column :label="$t('test_track.case.step_desc')" prop="desc" min-width="35%">
                  <template v-slot:default="scope">
                    <el-input
                      class="table-edit-input"
                      size="mini"
                      :disabled="readOnly"
                      type="textarea"
                      :autosize="{ minRows: 1, maxRows: 6}"
                      :rows="2"
                      v-model="scope.row.desc"
                      :placeholder="$t('commons.input_content')"
                      clearable/>
                  </template>
                </el-table-column>
                <el-table-column :label="$t('test_track.case.expected_results')" prop="result" min-width="35%">
                  <template v-slot:default="scope">
                    <el-input
                      class="table-edit-input"
                      size="mini"
                      :disabled="readOnly"
                      type="textarea"
                      :autosize="{ minRows: 1, maxRows: 6}"
                      :rows="2"
                      v-model="scope.row.result"
                      :placeholder="$t('commons.input_content')"
                      clearable/>
                  </template>
                </el-table-column>
                <el-table-column :label="$t('commons.input_content')" min-width="25%">
                  <template v-slot:default="scope">
                    <el-button
                      type="primary"
                      :disabled="readOnly"
                      icon="el-icon-plus"
                      circle size="mini"
                      @click="handleAddStep(scope.$index, scope.row)"></el-button>
                    <el-button
                      icon="el-icon-document-copy"
                      type="success"
                      :disabled="readOnly"
                      circle size="mini"
                      @click="handleCopyStep(scope.$index, scope.row)"></el-button>
                    <el-button
                      type="danger"
                      icon="el-icon-delete"
                      circle size="mini"
                      @click="handleDeleteStep(scope.$index, scope.row)"
                      :disabled="readOnly || (scope.$index == 0 && form.steps.length <= 1)"></el-button>
                  </template>
                </el-table-column>
              </el-table>
            </el-col>
          </el-row>
          <el-row style="margin-top: 10px;">
            <el-col :span="1" :offset="1">{{ $t('commons.remark') }}:</el-col>
            <el-col :span="19">
              <el-form-item prop="remark">
                <el-input v-model="form.remark"
                          :autosize="{ minRows: 2, maxRows: 4}"
                          type="textarea"
                          :disabled="readOnly"
                          :rows="2"
                          :placeholder="$t('commons.input_content')"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <div class="tip">其他信息</div>
          <el-row>
            <el-col :span="20" :offset="1">{{ $t('test_track.case.attachment') }}:</el-col>
          </el-row>
          <el-row>
            <el-col :span="19" :offset="2">
              <el-upload
                accept=".jpg,.jpeg,.png,.xlsx,.doc,.pdf,.docx"
                action=""
                :show-file-list="false"
                :before-upload="beforeUpload"
                :http-request="handleUpload"
                :on-exceed="handleExceed"
                multiple
                :limit="8"
                :file-list="fileList">
                <el-button icon="el-icon-plus" size="mini"></el-button>
                <span slot="tip" class="el-upload__tip"> {{ $t('test_track.case.upload_tip') }} </span>
              </el-upload>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="19" :offset="2">
              <test-case-attachment :table-data="tableData"
                                    :read-only="readOnly"
                                    :is-delete="true"
                                    @handleDelete="handleDelete"
              />
            </el-col>
          </el-row>
          <el-row style="margin-top: 10px" v-if="type!='add'">
            <el-col :span="20" :offset="1">{{ $t('test_track.review.comment') }}:
              <el-button icon="el-icon-plus" type="mini" @click="openComment"></el-button>
            </el-col>
          </el-row>
          <el-row v-if="type!='add'">
            <el-col :span="20" :offset="1">

              <review-comment-item v-for="(comment,index) in comments"
                                   :key="index"
                                   :comment="comment"
                                   @refresh="getComments"/>
              <div v-if="comments.length === 0" style="text-align: center">
                <i class="el-icon-chat-line-square" style="font-size: 15px;color: #8a8b8d;">
                      <span style="font-size: 15px; color: #8a8b8d;">
                        {{ $t('test_track.comment.no_comment') }}
                      </span>
                </i>
              </div>
            </el-col>
          </el-row>
          <test-case-comment :case-id="form.id"
                             @getComments="getComments" ref="testCaseComment"/>

        </el-form>

      </div>
    </div>
  </el-card>


</template>

<script>
import {TokenKey, WORKSPACE_ID} from '@/common/js/constants';
import MsDialogFooter from '../../../common/components/MsDialogFooter'
import {getCurrentUser, getNodePath, handleCtrlSEvent, listenGoBack, removeGoBackListener} from "@/common/js/utils";
import {Message} from "element-ui";
import TestCaseAttachment from "@/business/components/track/case/components/TestCaseAttachment";
import {buildNodePath,buildTree} from "../../../api/definition/model/NodeTree";
import CaseComment from "@/business/components/track/case/components/CaseComment";
import MsInputTag from "@/business/components/api/automation/scenario/MsInputTag";
import MsPreviousNextButton from "../../../common/components/MsPreviousNextButton";
import {ELEMENTS} from "@/business/components/api/automation/scenario/Setting";
import TestCaseComment from "@/business/components/track/case/components/TestCaseComment";
import ReviewCommentItem from "@/business/components/track/review/commom/ReviewCommentItem";
import {API_STATUS, REVIEW_STATUS, TEST} from "@/business/components/api/definition/model/JsonData";
import MsTableButton from "@/business/components/common/components/MsTableButton";
import MsSelectTree from "../../../common/select-tree/SelectTree";

export default {
  name: "TestCaseEdit",
  components: {
    MsTableButton,
    MsSelectTree,
    ReviewCommentItem,
    TestCaseComment, MsPreviousNextButton, MsInputTag, CaseComment, MsDialogFooter, TestCaseAttachment
  },
  data() {
    return {
      props: {
        multiple: true,
        //lazy: true,
        //lazyLoad:this.lazyLoad
      },
      sysList: [],//一级选择框的数据
      options: REVIEW_STATUS,
      statuOptions: API_STATUS,
       comments: [],
      result: {},
      dialogFormVisible: false,
      form: {
        name: '',
        module: 'default-module',
        nodePath:'/默认模块',
        maintainer: getCurrentUser().id,
        priority: 'P0',
        type: '',
        method: '',
        prerequisite: '',
        testId: '',
        otherTestName: '',
        steps: [{
          num: 1,
          desc: '',
          result: ''
        }],
        selected: [],
        remark: '',
        tags: [],
        demandId: '',
        demandName: '',
        status: 'Prepare',
        reviewStatus: 'Prepare',
      },
      readOnly: false,
      maintainerOptions: [],
      testOptions: [],
      demandOptions: [],
      workspaceId: '',
      fileList: [],
      tableData: [],
      uploadList: [],
      rules: {
        name: [
          {required: true, message: this.$t('test_track.case.input_name'), trigger: 'blur'},
          {max: 255, message: this.$t('test_track.length_less_than') + '255', trigger: 'blur'}
        ],
        module: [{required: true, message: this.$t('test_track.case.input_module'), trigger: 'change'}],
        demandName: [{required: true, message: this.$t('test_track.case.input_demand_name'), trigger: 'change'}],
        maintainer: [{required: true, message: this.$t('test_track.case.input_maintainer'), trigger: 'change'}],
        priority: [{required: true, message: this.$t('test_track.case.input_priority'), trigger: 'change'}],
        method: [{required: true, message: this.$t('test_track.case.input_method'), trigger: 'change'}],
        prerequisite: [{max: 500, message: this.$t('test_track.length_less_than') + '500', trigger: 'blur'}],
        remark: [{max: 1000, message: this.$t('test_track.length_less_than') + '1000', trigger: 'blur'}]
      },
      formLabelWidth: "120px",
      operationType: '',
      isCreateContinue: false,
      isStepTableAlive: true,
      methodOptions: [
        {value: 'auto', label: this.$t('test_track.case.auto')},
        {value: 'manual', label: this.$t('test_track.case.manual')}
      ],
      testCase: {},
      testCases: [],
      index: 0,
      showInputTag: true,
      tableType:"",
      moduleObj: {
        id: 'id',
        label: 'name',
      },
    };
  },
  props: {
    treeNodes: {
      type: Array
    },
    currentTestCaseInfo: {},
    /*readOnly: {
      type: Boolean,
      default: false
    },*/
    selectNode: {
      type: Object
    },
    selectCondition: {
      type: Object
    },
    type: String
  },
  computed: {
    projectIds() {
      return this.$store.state.projectId
    },
    moduleOptions() {
      return this.$store.state.testCaseModuleOptions;
    }
  },
  mounted() {
    this.getSelectOptions();
    if (this.type === 'edit' || this.type === 'copy') {
      this.open(this.currentTestCaseInfo)
      this.getComments(this.currentTestCaseInfo)
    }
    // Cascader 级联选择器: 点击文本就让它自动点击前面的input就可以触发选择。
    setInterval(function () {
      document.querySelectorAll('.el-cascader-node__label').forEach(el => {
        el.onclick = function () {
          if (this.previousElementSibling) this.previousElementSibling.click();
        };
      });
    }, 1000);
    if(this.selectNode && this.selectNode.data && !this.form.id){
        this.form.module = this.selectNode.data.id;
        this.form.nodePath = this.selectNode.data.path;
    }
    if((!this.form.module || this.form.module==="default-module" || this.form.module ==="root") && this.treeNodes.length > 0){
      this.form.module =  this.treeNodes[0].id;
      this.form.nodePath = this.treeNodes[0].path;
    }
  },
  // watch: {
    // treeNodes() {
    //   this.getModuleOptions();
    // },
    // moduleOptions() {
    //   this.$emit('setModuleOptions', this.moduleOptions);
    // }
  // },
  created() {
    this.loadOptions();
    this.addListener(); //  添加 ctrl s 监听
    if(this.selectNode && this.selectNode.data && !this.form.id){
      this.form.module = this.selectNode.data.id;
      this.form.nodePath = this.selectNode.data.path;
    }else{
      this.form.module =this.treeNodes && this.length>0? this.treeNodes[0].id:"";
    }
    if (this.type === 'edit' || this.type === 'copy') {
      this.form.module = this.currentTestCaseInfo.nodeId;
      this.form.nodePath = this.currentTestCaseInfo.nodePath;
    }
    if((!this.form.module || this.form.module==="default-module" || this.form.module ==="root") && this.treeNodes.length > 0){
      this.form.module =  this.treeNodes[0].id;
      this.form.nodePath = this.treeNodes[0].path;
    }
  },
  methods: {
    setModule(id,data) {
      this.form.module = id;
      this.form.nodePath = data.path;
    },
    clearInput() {
      //this.$refs['cascade'].panel.clearCheckedNodes()
    },
    async loadOptions(sysLib) {
      sysLib = TEST
        .map(item => ({
          value: item.id,
          label: item.name,
        }));
      let array = [];
      for (let i = 0; i < sysLib.length; i++) {
        if (sysLib.length > 0) {
          let res = await this.getTestOptions(sysLib[i].value);
          sysLib[i].children = res;
        }
        array.push(sysLib[i]);
      }
      this.sysList = array;
    },
    getTestOptions(val) {
      this.form.type = val
      this.projectId = this.projectIds
      this.testOptions = [];
      let url = '';
      if (this.form.type === 'testcase' || this.form.type === 'automation') {
        url = '/api/' + this.form.type + '/list/' + this.projectId
      } else if (this.form.type === 'performance' || this.form.type === 'api') {
        url = '/' + this.form.type + '/list/' + this.projectId
      }
      if (!url) {
        return;
      }
      return new Promise((resolve, reject) => {
        this.$get(url).then(res => {
          const data = res.data.data.map(item => ({
            value: item.id,
            label: item.name,
            leaf: true
          }))
          resolve(data)
        }).catch((err) => {
          reject(err)
        })
      });
    },
    /* lazyLoad(node, resolve){
      const { level } = node;
      if(node.level==0){
        const nodes = TEST
          .map(item => ({
            value: item.id,
            label: item.name,
            leaf: level >= 1
          }));
        resolve(nodes)
      }
      if(node.level==1){
        this.projectId = getCurrentProjectID()
        this.testOptions = [];
        let url = '';
        this.form.type=node.data.value
        if (this.form.type === 'testcase' || this.form.type === 'automation') {
          url = '/api/' + this.form.type + '/list/' + this.projectId
        } else if (this.form.type === 'performance' || this.form.type === 'api') {
          url = '/' + this.form.type + '/list/' + this.projectId
        }
        if (this.projectId && this.form.type != '' && this.form.type != 'undefined') {
          this.$get(url, response => {
              const nodes = response.data
                .map(item => ({
                  value: item.id,
                  label: item.name,
                  leaf: level >= 1
                }));
              resolve(nodes)
          });
        }
      }
    },*/
    handleCommand(e) {
      if (e === "ADD_AND_CREATE") {
        this.$refs['caseFrom'].validate((valid) => {
          if (!valid) {
            this.saveCase();
          } else {
            this.saveCase();
            let tab = {}
            tab.name = 'add'
            this.$emit('addTab', tab)
          }
        })
      } else {
        this.saveCase();
      }
    },
    openComment() {
      this.$refs.testCaseComment.open()
    },
    getComments(testCase) {
      let id = '';
      if (testCase) {
        id = testCase.id;
      } else {
        id = this.form.id;
      }
      this.result = this.$get('/test/case/comment/list/' + id, res => {
        this.comments = res.data;
      })
    },
    showAll() {
      if (!this.customizeVisible) {
        this.operatingElements = ELEMENTS.get("ALL");
        this.selectedTreeNode = undefined;
      }
      //this.reload();
    },
    reload() {
      this.isStepTableAlive = false;
      this.$nextTick(() => (this.isStepTableAlive = true));
    },
    open(testCase) {
      /*
             this.form.selected=[["automation", "3edaaf31-3fa4-4a53-9654-320205c2953a"],["automation", "3aa58bd1-c986-448c-8060-d32713dbd4eb"]]
      */
      this.projectId = this.projectIds;
      if (window.history && window.history.pushState) {
        history.pushState(null, null, document.URL);
        window.addEventListener('popstate', this.close);
      }
      this.resetForm();
      listenGoBack(this.close);
      this.operationType = 'add';
      if (testCase) {
        //修改
        this.operationType = 'edit';
        //复制
        if (this.type === 'copy') {
          this.operationType = 'add';
          this.setFormData(testCase);
          this.setTestCaseExtInfo(testCase);
          this.getSelectOptions();
          this.reload();
        } else {
          this.initTestCases(testCase);
        }
      } else {
        if (this.selectNode.data) {
          this.form.module = this.selectNode.data.id;
        } else {
          if (this.moduleOptions.length > 0) {
            this.form.module = this.moduleOptions[0].id;
          }
        }
        let user = JSON.parse(localStorage.getItem(TokenKey));
        this.form.priority = 'P3';
        this.form.type = 'functional';
        this.form.method = 'manual';
        this.form.maintainer = user.id;
        this.form.tags = [];
        this.getSelectOptions();
        this.reload();
      }
    },
    handlePre() {
      this.index--;
      this.getTestCase(this.index)
    },
    handleNext() {
      this.index++;
      this.getTestCase(this.index);
    },
    initTestCases(testCase) {
      this.result = this.$post('/test/case/list/ids', this.selectCondition, response => {
        this.testCases = response.data;
        for (let i = 0; i < this.testCases.length; i++) {
          if (this.testCases[i].id === testCase.id) {
            this.index = i;
            this.getTestCase(i);
          }
        }
      });
    },
    getTestCase(index) {
      this.showInputTag = false;
      let testCase = this.testCases[index];
      this.result = this.$get('/test/case/get/' + testCase.id, response => {
        let testCase = response.data;
        this.setFormData(testCase);
        this.setTestCaseExtInfo(testCase);
        this.getSelectOptions();
        this.reload();
        this.$nextTick(() => {
          this.showInputTag = true
        })
      })
    },
    async setFormData(testCase) {
      try {
        testCase.selected = JSON.parse(testCase.testId);
      } catch (error) {
        testCase.selected = testCase.testId
      }
      let tmp = {};
      Object.assign(tmp, testCase);
      tmp.steps = JSON.parse(testCase.steps);
      if (tmp.steps == null) {
        tmp.steps = [{
          num: 1,
          desc: '',
          result: ''
        }];
      }
      tmp.tags = JSON.parse(tmp.tags);
      Object.assign(this.form, tmp);
      this.form.module = testCase.nodeId;
      this.getFileMetaData(testCase);
      await this.loadOptions(this.sysList)

    },
    setTestCaseExtInfo(testCase) {
      this.testCase = {};
      if (testCase) {
        // 复制 不查询评论
        this.testCase = testCase.isCopy ? {} : testCase;
      }
    },
    getFileMetaData(testCase) {
      this.fileList = [];
      this.tableData = [];
      this.uploadList = [];
      this.result = this.$get("test/case/file/metadata/" + testCase.id, response => {
        let files = response.data;

        if (!files) {
          return;
        }
        // deep copy
        this.fileList = JSON.parse(JSON.stringify(files));
        this.tableData = JSON.parse(JSON.stringify(files));
        this.tableData.map(f => {
          f.size = f.size + ' Bytes';
        });
      })
    },
    handleAddStep(index, data) {
      let step = {};
      step.num = data.num + 1;
      step.desc = "";
      step.result = "";
      this.form.steps.forEach(step => {
        if (step.num > data.num) {
          step.num++;
        }
      });
      this.form.steps.splice(index + 1, 0, step);
    },
    handleCopyStep(index, data) {
      let step = {};
      step.num = data.num + 1;
      step.desc = data.desc;
      step.result = data.result;
      this.form.steps.forEach(step => {
        if (step.num > data.num) {
          step.num++;
        }
      });
      this.form.steps.splice(index + 1, 0, step);
    },
    handleDeleteStep(index, data) {
      this.form.steps.splice(index, 1);
      this.form.steps.forEach(step => {
        if (step.num > data.num) {
          step.num--;
        }
      });
    },
    close() {
      //移除监听，防止监听其他页面
      removeGoBackListener(this.close);
      this.dialogFormVisible = false;
    },
    saveCase() {
      this.$refs['caseFrom'].validate((valid) => {
        if (valid) {
          let param = this.buildParam();
          if (this.validate(param)) {
            let option = this.getOption(param);
            this.result = this.$request(option, (response) => {
              this.$success(this.$t('commons.save_success'));
              this.operationType="edit"
              this.form.id=response.id;
              this.$emit("refreshTestCase",)
              /*if (this.operationType == 'add' && this.isCreateContinue) {
                this.form.name = '';
                this.form.prerequisite = '';
                this.form.steps = [{
                  num: 1,
                  desc: '',
                  result: ''
                }];
                this.form.remark = '';
                this.uploadList = [];
                this.fileList = [];
                this.tableData = [];
                this.$emit("refresh");
                return;
              }*/
              this.tableType='edit';
              this.$emit("refresh",this.form);
              this.form.id=response.data
              if (this.type === 'add' || this.type === 'copy') {
                param.id = response.data;
                this.$emit("caseCreate", param);
                this.close();
              } else {
                this.$emit("caseEdit", param);
              }
            });
          }
        } else {
          return false;
        }
      });
    },
    buildParam() {
      let param = {};
      Object.assign(param, this.form);
      param.steps = JSON.stringify(this.form.steps);
      param.nodeId = this.form.module;
      param.nodePath = getNodePath(this.form.module, this.moduleOptions);
      if (this.projectId) {
        param.projectId = this.projectId;
      }
      param.name = param.name.trim();

      if (this.form.tags instanceof Array) {
        this.form.tags = JSON.stringify(this.form.tags);
      }
      param.testId = JSON.stringify(this.form.selected)
      param.tags = this.form.tags;
      param.type = 'functional';
      return param;
    },
    getOption(param) {
      let type={}
      if(this.tableType==='edit'){
        type='edit'
      }else if(this.type === 'copy'){
        type = 'add'
      }else{
        type=this.type
      }
      let formData = new FormData();
      let url = '/test/case/' + type;
      this.uploadList.forEach(f => {
        formData.append("file", f);
      });

      if (param.isCopy) {
        // 如果是copy，则把文件的ID传到后台进行文件复制
        param.fileIds = this.fileList.map(f => f.id);
      }

      param.updatedFileList = this.fileList;

      let requestJson = JSON.stringify(param, function (key, value) {
        return key === "file" ? undefined : value
      });

      formData.append('request', new Blob([requestJson], {
        type: "application/json"
      }));

      return {
        method: 'POST',
        url: url,
        data: formData,
        headers: {
          'Content-Type': undefined
        }
      };
    },
    validate(param) {
      for (let i = 0; i < param.steps.length; i++) {
        if ((param.steps[i].desc && param.steps[i].desc.length > 300) ||
          (param.steps[i].result && param.steps[i].result.length > 300)) {
          this.$warning(this.$t('test_track.case.step_desc') + ","
            + this.$t('test_track.case.expected_results') + this.$t('test_track.length_less_than') + '300');
          return false;
        }
      }
      if (param.name == '') {
        this.$warning(this.$t('test_track.case.input_name'));
        return false;
      }
      return true;
    },
    typeChange() {
      this.form.testId = '';
      this.getTestOptions()
    },
    getMaintainerOptions() {
      let workspaceId = localStorage.getItem(WORKSPACE_ID);
      this.$post('/user/ws/member/tester/list', {workspaceId: workspaceId}, response => {
        this.maintainerOptions = response.data;
      });
    },

    visibleChange(flag) {
      if (flag) {
        this.getDemandOptions();
      }
    },
    getDemandOptions() {
      if (this.demandOptions.length === 0) {
        this.result = {loading: true};
        this.$get("demand/list/" + this.projectId).then(response => {
          this.demandOptions = response.data.data;
          this.demandOptions.unshift({id: 'other', name: this.$t('test_track.case.other'), platform: 'Other'})
          this.result = {loading: false};
        }).catch(() => {
          this.demandOptions.unshift({id: 'other', name: this.$t('test_track.case.other'), platform: 'Other'})
          this.result = {loading: false};
        })
      }
    },
    getSelectOptions() {
      this.getMaintainerOptions();
      this.getTestOptions();
      if (this.type === 'edit') {
        this.getDemandOptions();
      }
    },

    resetForm() {
      //防止点击修改后，点击新建触发校验
      if (this.$refs['caseFrom']) {
        this.$refs['caseFrom'].validate((valid) => {
          this.$refs['caseFrom'].resetFields();
          this._resetForm();
          return true;
        });
      } else {
        this._resetForm();
      }
    },
    _resetForm() {
      this.form.name = '';
      this.form.module = '';
      this.form.type = '';
      this.form.method = '';
      this.form.maintainer = '';
      this.form.priority = '';
      this.form.prerequisite = '';
      this.form.remark = '';
      this.form.testId = '';
      this.form.testName = '';
      this.form.steps = [{
        num: 1,
        desc: '',
        result: ''
      }];
      this.uploadList = [];
      this.fileList = [];
      this.tableData = [];
    },
    handleExceed() {
      this.$error(this.$t('load_test.file_size_limit'));
    },
    beforeUpload(file) {
      if (!this.fileValidator(file)) {
        /// todo: 显示错误信息
        return false;
      }

      if (this.tableData.filter(f => f.name === file.name).length > 0) {
        this.$error(this.$t('load_test.delete_file') + ', name: ' + file.name);
        return false;
      }

      let type = file.name.substring(file.name.lastIndexOf(".") + 1);

      this.tableData.push({
        name: file.name,
        size: file.size + ' Bytes', /// todo: 按照大小显示Byte、KB、MB等
        type: type.toUpperCase(),
        updateTime: new Date().getTime(),
      });

      return true;
    },
    handleUpload(uploadResources) {
      this.uploadList.push(uploadResources.file);
    },
    handleDownload(file) {
      let data = {
        name: file.name,
        id: file.id,
      };
      let config = {
        url: '/test/case/file/download',
        method: 'post',
        data: data,
        responseType: 'blob'
      };
      this.result = this.$request(config).then(response => {
        const content = response.data;
        const blob = new Blob([content]);
        if ("download" in document.createElement("a")) {
          // 非IE下载
          //  chrome/firefox
          let aTag = document.createElement('a');
          aTag.download = file.name;
          aTag.href = URL.createObjectURL(blob);
          aTag.click();
          URL.revokeObjectURL(aTag.href)
        } else {
          // IE10+下载
          navigator.msSaveBlob(blob, this.filename)
        }
      }).catch(e => {
        Message.error({message: e.message, showClose: true});
      });
    },
    handleDelete(file, index) {
      this.$alert(this.$t('load_test.delete_file_confirm') + file.name + "？", '', {
        confirmButtonText: this.$t('commons.confirm'),
        callback: (action) => {
          if (action === 'confirm') {
            this._handleDelete(file, index);
          }
        }
      });
    },
    _handleDelete(file, index) {
      this.fileList.splice(index, 1);
      this.tableData.splice(index, 1);
      let i = this.uploadList.findIndex(upLoadFile => upLoadFile.name === file.name);
      if (i > -1) {
        this.uploadList.splice(i, 1);
      }
    },
    fileValidator(file) {
      /// todo: 是否需要对文件内容和大小做限制
      return file.size > 0;
    },
    addListener() {
      document.addEventListener("keydown", this.createCtrlSHandle);
    },
    removeListener() {
      document.removeEventListener("keydown", this.createCtrlSHandle);
    },
    createCtrlSHandle(event) {
      handleCtrlSEvent(event, this.saveCase);
    },
  }
}
</script>

<style scoped>

.el-switch {
  margin-bottom: 10px;
}

.case-name {
  width: 194px;
}

.container {
  height: 100vh;
}

.case-form {
  height: 95%;
  overflow: auto;
}

.case-dialog >>> .el-dialog__body {
  padding: 0 20px 10px 20px;
}

.container >>> .el-card__body {
  height: calc(100vh - 120px);
}

.comment-card >>> .el-card__header {
  padding: 27px 20px;
}

.comment-card >>> .el-card__body {
  height: calc(100vh - 120px);
}

.head-right {
  text-align: right;
}

.tip {
  padding: 3px 5px;
  font-size: 16px;
  border-radius: 4px;
  border-left: 4px solid #783887;
}

.ms-main-div {
  background-color: white;
}

.ms-opt-btn {
  position: fixed;
  right: 50px;
  z-index: 1;
}

.ms-case-input {
  width: 100%;
}

.ms-case {
  width: 100%;
}

/deep/ .el-button-group > .el-button:first-child {
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
  height: 32px;
  width: 56px;
}

</style>
