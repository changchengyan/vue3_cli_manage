<template>
  <div id="form">
    <div class="main">
      <div class="table-msg">
        <div class="top-head">
          <div class="actions-btn">
            <template v-if="problemStatus=='true'">
              <template v-if="dispatchFlag=='false'">
                <span @click="dispatch_">派发</span>
              </template>
              <template v-else>
                <span class="finish">派发完成</span>
              </template>
            </template>
            <span @click="exportPdf">导出</span>
            <span @click="print">打印</span>
            <template v-if="dispatchFlag=='false'&&problemStatus=='true'">
              <span class="upload gray">
                <i class="el-icon-upload"></i>
                上传文件
              </span>
            </template>
            <template v-else>
              <span class="upload">
                <i class="el-icon-upload"></i>
                上传文件
                <input
                  type="file"
                  class="fullTab"
                  @change="getFile($event)"
                  accept=".pdf, .jpg, .png"
                >
              </span>
            </template>
            <span @click="goHome">返回</span>
          </div>
        </div>
        <template v-if="problemStatus=='true'&&dispatchFlag=='false'">
          <div class="tables">
            <div class="table-item">
              <div class="inner-wrap">
                <div class="mainHead">
                  <span>{{companyName}}检修派工单</span>
                </div>
                <div class="subHead">
                  <div class="col">
                    <span>检修单号:</span>
                    <span>{{workOrder.orderNum}}</span>
                  </div>
                  <div class="col">
                    <span>派单时间:</span>
                    <span>{{workOrder.orderTime}}</span>
                  </div>
                </div>
                <table>
                  <tr>
                    <td>
                      <span>检修地点</span>
                    </td>
                    <td>
                      <input type="text" placeholder="检修地点" v-model="workOrder.address">
                    </td>
                    <td>
                      <span>最迟处理时间</span>
                    </td>
                    <td colspan="2">
                      <!-- <span>2018-09-10 09:10</span> -->
                      <el-date-picker
                        value-format="yyyy-MM-dd HH:mm:ss"
                        v-model="workOrder.lastHandleTime"
                        @change="lastToDoTime(workOrder.lastHandleTime)"
                        type="datetime"
                        placeholder="选择日期"
                      ></el-date-picker>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>紧急程度</span>
                    </td>
                    <td>
                      <!-- <span>{{workOrder.hurrySelect|formateHurry}}</span> -->
                      <el-select v-model="workOrder.hurrySelect" placeholder="请选择">
                        <el-option
                          v-for="item in workOrder.hurryList"
                          :key="item.value"
                          :label="item.label"
                          :value="item.value"
                        ></el-option>
                      </el-select>
                    </td>
                    <td>
                      <span>检修类型</span>
                    </td>
                    <td colspan="2">
                      <span>{{workOrder.typeSelect|formateRepairType}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>上报人</span>
                    </td>
                    <td>
                      <span>{{workOrder.reporter}}</span>
                    </td>
                    <td>
                      <span>联系电话</span>
                    </td>
                    <td colspan="2">
                      <span>{{workOrder.contacts}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>派单人</span>
                    </td>
                    <td>
                      <span>{{workOrder.orderer}}</span>
                    </td>
                    <td>
                      <span>处理人</span>
                    </td>
                    <td colspan="2" class="getWorker">
                      <input
                        type="text"
                        placeholder="处理人"
                        v-model="workOrder.doerName"
                        @focus="getWorkers"
                      >
                      <div class="subLists" v-if="workers.length&&colseFlag">
                        <div
                          class="listItem"
                          v-for="(item,index) in workers"
                          :key="index"
                          @click="selectThisWorker(item)"
                        >{{item.NAME}}</div>
                      </div>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>隐患描述</span>
                    </td>
                    <td colspan="4">
                      <!-- <input type="text" placeholder="隐患描述" v-model="workOrder.desc"> -->
                      <span class="alignLeft">{{workOrder.desc}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>检修内容</span>
                    </td>
                    <td colspan="4">
                      <input type="text" placeholder="检修内容" v-model="workOrder.repireContent">
                    </td>
                  </tr>
                  <tr>
                    <td :rowspan="workOrder.productList.length+2">
                      <span>材料清单</span>
                    </td>
                    <td>
                      <span>材料名称</span>
                    </td>
                    <td>
                      <span>单位</span>
                    </td>
                    <td>
                      <span>数量</span>
                    </td>
                    <td>
                      <span>操作</span>
                    </td>
                  </tr>
                  <tr v-for="(item,index) in workOrder.productList" :key="index">
                    <td>
                      <input v-model="item.name" type="text" placeholder="材料名称">
                    </td>
                    <td>
                      <input v-model="item.unit" type="text" placeholder="单位">
                    </td>
                    <td>
                      <input v-model="item.number" type="number"  min='0' @change="verifyNum($event,index)" placeholder="数量">
                    </td>
                    <td class="del">
                      <span class="btn-add" @click="workOrder_del(index)">删除</span>
                    </td>
                  </tr>
                  <tr>
                    <td colspan="4" class="addCol">
                      <span class="btn-add" @click="workOrder_add">新增</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>派发意见</span>
                    </td>
                    <td colspan="4">
                      <input type="text" v-model="workOrder.orderAdvice" placeholder="请输入派发意见">
                    </td>
                  </tr>
                  <tr class="noBorder">
                    <td colspan="2">
                      <span>派发部门:{{workOrder.orderDept}}</span>
                    </td>
                    <td colspan="3">
                      <span>处理部门:{{workOrder.repairDept}}</span>
                    </td>
                  </tr>
                  <tr class="noBorder">
                    <td colspan="2">
                      <span>签字:</span>
                    </td>
                    <td colspan="3">
                      <span>签字:</span>
                    </td>
                  </tr>
                </table>
              </div>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="tables">
            <div class="table-item">
              <div class="inner-wrap">
                <div class="mainHead">
                  <span>{{companyName}}检修派工单</span>
                </div>
                <div class="subHead">
                  <div class="col">
                    <span>检修单号:</span>
                    <span>{{workOrder.orderNum}}</span>
                  </div>
                  <div class="col">
                    <span>派单时间:</span>
                    <span>{{workOrder.orderTime}}</span>
                  </div>
                </div>
                <table>
                  <tr>
                    <td>
                      <span>检修地点</span>
                    </td>
                    <td :title="workOrder.address">
                      <span>{{workOrder.address}}</span>
                    </td>
                    <td>
                      <span>最迟处理时间</span>
                    </td>
                    <td>
                      <span>{{workOrder.lastHandleTime}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>紧急程度</span>
                    </td>
                    <td>
                      <span>{{workOrder.hurrySelect|formateHurry}}</span>
                    </td>
                    <td>
                      <span>检修类型</span>
                    </td>
                    <td>
                      <span>{{workOrder.typeSelect|formateRepairType}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>上报人</span>
                    </td>
                    <td>
                      <span>{{workOrder.reporter}}</span>
                    </td>
                    <td>
                      <span>联系电话</span>
                    </td>
                    <td>
                      <span>{{workOrder.contacts}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>派单人</span>
                    </td>
                    <td>
                      <span>{{workOrder.orderer}}</span>
                    </td>
                    <td>
                      <span>处理人</span>
                    </td>
                    <td class="getWorker">
                      <span>{{workOrder.doerName}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>隐患描述</span>
                    </td>
                    <td colspan="3" :title="workOrder.desc">
                      <span class="alignLeft">{{workOrder.desc}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>检修内容</span>
                    </td>
                    <td colspan="3" :title="workOrder.repireContent">
                      <span class="alignLeft">{{workOrder.repireContent}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td :rowspan="workOrder.productList.length+1">
                      <span>材料清单</span>
                    </td>
                    <td>
                      <span>材料名称</span>
                    </td>
                    <td>
                      <span>单位</span>
                    </td>
                    <td>
                      <span>数量</span>
                    </td>
                  </tr>
                  <tr v-for="(item,index) in workOrder.productList" :key="index">
                    <td>
                      <span>{{item.name}}</span>
                    </td>
                    <td>
                      <span>{{item.unit}}</span>
                    </td>
                    <td>
                      <span>{{item.number}}</span>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <span>派发意见</span>
                    </td>
                    <td colspan="3" :title="workOrder.orderAdvice">
                      <span class="alignLeft">{{workOrder.orderAdvice}}</span>
                    </td>
                  </tr>
                  <tr class="noBorder">
                    <td colspan="2">
                      <span>派发部门:{{workOrder.orderDept}}</span>
                    </td>
                    <td colspan="2">
                      <span>处理部门:{{workOrder.repairDept}}</span>
                    </td>
                  </tr>
                  <tr class="noBorder">
                    <td colspan="2">
                      <span>签字:</span>
                    </td>
                    <td colspan="2">
                      <span>签字:</span>
                    </td>
                  </tr>
                </table>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>

    <dse-save-status :showModel="save_model.toggleShowModel"
                     :type="true"
                     :flag="save_model.save_flag"
                     :content="save_model.save_msg"
                     @delThis="hiddenSaveModel"/>
  </div>
</template>

<script>

import DseSaveStatus from '../../common/components/toast/DseSaveStatus';
export default {
  name:'standardForm',
  components: {
    DseSaveStatus
  },
  data() {
    return {
      problemID: '', //隐患ID
      companyName: '', //公司名字
      dispatchFlag: 'false', //派单状态
      deptId: '',
      problemStatus: '', // 问题状态
      msg_tabs: [], //工单
      file: null, //上传的文件
      workOrder: {
        id: '', //派单的ID
        orderNum: '', //订单单号
        orderTime: '', // 派单时间
        address: '',
        lastHandleTime: '', //最迟处理时间
        hurryList: [], // 紧急程度
        hurrySelect: '',
        typeList: [],  // 检修类型
        typeSelect: '',
        dangerDesc: '',
        repireContent: '', //检修内容
        productList: [
          {
            name: '',
            unit: '',
            number: '',
            content: '',
            remark: ''
          },
          {
            name: '',
            unit: '',
            number: '',
            content: '',
            remark: ''
          },
          {
            name: '',
            unit: '',
            number: '',
            content: '',
            remark: ''
          }
        ],
        reporter:'',//上报人
        contacts:'',//联系电话
        orderer: '', //派单人
        doer: '', //处理人ID
        doerName: '', //处理人名字
        desc: '', //隐患描述
        orderAdvice: '' //检修内容
      },
      workers: [],
      workerName: '',
      colseFlag: true,
      save_model: {
        toggleShowModel: false,
        save_flag: false,
        save_msg: ''
      }
    };
  },
  methods: {
    goBack() {
      this.$router.push({
        name: 'pipe_rushrepair_information'
      });
      this.$store.commit('setAside_status', true);
      this.$emit('showAside');
    },
    showThisTab(index) {
      this.main_selectTab = index;
      this.$store.commit('setToptab', index);
    },
    toggoleShow() {
      this.tableShow = !this.tableShow;
    },
    showInfo() {
      this.showInfoModel = true;
    },
    hiddenInfoModel(val) {
      this.showInfoModel = val;
    },
    workOrder_add() {
      this.workOrder.productList.push({
        name: '',
        unit: '',
        number: '',
        content: '',
        remark: ''
      });
    },
    workOrder_del(index) {
      this.workOrder.productList.splice(index, 1);
    },
    gotoEdit() {
      this.$router.push({
        name: 'pipe_rushrepair_edit'
      });
    },
    print() {

        let temp_arr = this.workOrder.productList;
        let type = this.workOrder.typeSelect;
        let temp_type = null;
        let list = window.DSE.problem_type;
        for(let i =0;i<list.length;i++){
            if(list[i].value===type){
                temp_type = list[i].label;
            }
        }

        let temp_data = {
          orgName: this.companyName,
          orderNum: this.workOrder.orderNum,
          orderTime: this.workOrder.orderTime,
          lastDoTime: this.workOrder.lastHandleTime,
          address: this.workOrder.address,
          hurrySelect:
            this.workOrder.hurrySelect == '0'
              ? '紧急'
              : this.workOrder.hurrySelect == '1'
              ? '较急'
              : '一般',
          typeSelect: temp_type,
          reporter: this.workOrder.reporter,
          contacts: this.workOrder.contacts,
          orderer: this.workOrder.orderer,
          doerName: this.workOrder.doerName,
          dangerDesc: this.workOrder.desc,
          repireContent: this.workOrder.repireContent,
          productList: temp_arr,
          orderAdvice: this.workOrder.orderAdvice,
          orderDept: this.workOrder.orderDept,
          repairDept: this.workOrder.repairDept
        };
        sessionStorage.setItem('tableContent', JSON.stringify(temp_data));
        let pageUrl = window.DSE.static_baseUrl + '/tables/standard.html';
        window.open(pageUrl);
    },
    formate_products(arr) {
      let temp_arr = arr.length > 0 ? arr : [];
      let formateArr = [];
      if (temp_arr.length > 0) {
        for (let i = 0; i < temp_arr.length; i++) {
          let temp_json = {};
          temp_json.name =
            temp_arr[i].materialName == null ? '' : temp_arr[i].materialName;
          temp_json.content =
            temp_arr[i].workContent == null ? '' : temp_arr[i].workContent;
          temp_json.unit =
            temp_arr[i].materialUnit == null ? '' : temp_arr[i].materialUnit;
          temp_json.num =
            temp_arr[i].materialNum == null ? '' : temp_arr[i].materialNum;
          temp_json.remark =
            temp_arr[i].remark == null ? '' : temp_arr[i].remark;
          formateArr.push(temp_json);
        }
      }
      return formateArr;
    },
    exportPdf() {
      let that = this;
      let temp_id =  that.workOrder.id;
      this.expWorkWorder2xls_(temp_id);
    },
    goHome() {
      this.$emit('goback','');
    },
    getOrderInputs() {
      let that = this;
      let temp_list = that.workOrder.productList;
      let temp_arr = [];
      if (temp_list.length > 0) {
        for (let i = 0; i < temp_list.length; i++) {
          if (temp_list[i].name !='') {
            temp_arr.push({
              name: '' + temp_list[i].name,
              unit: '' + temp_list[i].unit,
              num: '' + temp_list[i].number,
              content: '',
              remark: ''
            });
          }
        }
      }
      return temp_arr;
    },
    queryPersonByDept_() {

    },
    getWorkers(val) {
      this.colseFlag = true;
      this.queryPersonByDept_();
    },
    selectThisWorker(val) {
      this.workOrder.doer = val.ID;
      this.workOrder.doerName = val.NAME;
      this.colseFlag = false;
    },
    saveOrUpdateWorkorder_(formData_) {

    },
    dispatch_() {
      let that = this;
        let lastDoTime=that.workOrder.lastHandleTime;
        if(lastDoTime){
            if(lastDoTime==''){
                this.$message.error('请填写派单处理时间');
                return;
            }else{
                lastDoTime=lastDoTime.replace(/-/g,'/');
                if(new Date(lastDoTime).getTime()<new Date().getTime()){
                    this.$message.error('派单最迟处理时间应该大于当前时间');
                    return;
                }
            }
        }
      if (this.workOrder.doerName == '') {
        this.$message.error('请填写处理人!');
        return;
      }
      if (
        this.workOrder.address == '' ||
        this.workOrder.hurrySelect == '' ||
        this.workOrder.typeSelect == '' ||
        this.workOrder.repireContent == '' ||
        this.workOrder.orderAdvice == ''
      ) {
        this.$message.error('请把基本填写完整!');
        return;
      }

      if(that.workOrder.repireContent==''){
        this.$message.error('检修内容不能为空!');
        return;
      }
      let temp_arr = this.getOrderInputs();
      let formData_ = new FormData();
      formData_.append('checkAddr', that.workOrder.address);
      // console.log(that.workOrder.lastHandleTime)
      formData_.append('lastHandleTime', that.workOrder.lastHandleTime);
      formData_.append('lev', that.workOrder.hurrySelect);
      formData_.append('orderType', that.workOrder.typeSelect);
      formData_.append('handleUser', that.workOrder.doer);
      formData_.append('checkContent', that.workOrder.repireContent);
      formData_.append('materialArr', JSON.stringify(temp_arr));
      formData_.append('dispatchAdvice', that.workOrder.orderAdvice);
      formData_.append('orderStatus', '1');
      formData_.append('questionId', that.problemID);
      // formData_.append('dispatchAdvice', that.workOrder.orderAdvice);
      formData_.append('proFrom', '1');
      formData_.append('type', '1');

      this.saveOrUpdateWorkorder_(formData_);
    },
    hiddenSaveModel(val) {
      this.save_model.toggleShowModel = false;
    },
    getFile(e) {
      let that = this;
      let temp_file = e.target.files[0];
      let temp_fileName = temp_file.name;

      let temp_fileType = temp_file.type;
      if(temp_file.size/1024/1024>10){
          that.$message.error('您上传的文件不能大于10M!');
          return;
      }
      if(temp_fileType=='image/jpeg'||temp_fileType=='application/pdf'||temp_fileType=='image/png'||temp_fileType=='application/msword'||temp_fileType=='application/vnd.ms-excel'||temp_fileType=='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'){
        let post_formdata = new FormData();
        post_formdata.append('fileBiz', that.workOrder.id);
        post_formdata.append('fileName', temp_fileName);
        post_formdata.append('files', temp_file);


      }else{
          that.$message.error('您上传的文件格式应该为：png、jpg、pdf、word、excel');
      }

    },
    expWorkWorder2xls_(id) {
      let  formdata = new FormData();
      formdata.append('id',id);
      formdata.append('questionId','');
      formdata.append('exportType','1');
    },
    verifyNum(event,i){
        let temp_value =  event.target.value;
        temp_value = Math.abs(temp_value);
        this.workOrder.productList[i].number = temp_value;
    },
      lastToDoTime(event){
          if(event){
              if(event==''){
                  this.$message.error('请填写派单处理时间');
                  return;
              }else{
                  event=event.replace(/-/g,'/');
                  if(new Date(event).getTime()<new Date().getTime()){
                      this.$message.error('派单最迟处理时间应该大于当前时间');
                      return;
                  }
              }
          }
      },
  },
  filters: {
    formateHurry(val) {
      let temp = '';
      let list = window.DSE.dangerlist;
      for(let i =0;i<list.length;i++){
          if(list[i].value===val){
              temp = list[i].label;
          }
      }
      return temp;
    },
    formateRepairType(val) {
      let temp = '';
      let list = window.DSE.problem_type;
      for(let i =0;i<list.length;i++){
          if(list[i].value===val){
              temp = list[i].label;
          }
      }
      return temp;
    }
  },
  created() {
    this.main_selectTab = this.$store.getters.getTopActivedTab;
    let temp_data = JSON.parse(sessionStorage.getItem('hiddenDanger'));
    this.companyName = this.$store.getters.getCompyName;
    this.deptId = this.$store.getters.get_deptId;
    this.workOrder.reporter = temp_data.createUser;
    this.workOrder.desc  = temp_data.proDescrip;
    this.workOrder.id  = temp_data.id;
    this.workOrder.address = temp_data.place;
    this.workOrder.contacts = this.$store.getters.get_tel;
      // dispatchUser
    this.workOrder.orderer = temp_data.dispatchUser;
    this.workOrder.repireContent = temp_data.checkContent || '';
    this.workOrder.orderAdvice = temp_data.dispatchAdvice || '';
    this.workOrder.typeSelect = temp_data.proInfoType||temp_data.type;
    this.workOrder.orderDept = this.workOrder.repairDept = this.$store.getters.get_dept;
    this.problemID = temp_data.id;
    this.workOrder.hurryList = window.DSE.dangerlist;
    this.workOrder.typeList = window.DSE.problem_type;

    if (temp_data.handleUser) {
      this.workOrder.doerName = temp_data.handleUser;
    }
    if (temp_data.lev) {
        this.workOrder.hurrySelect = temp_data.lev;
    }
    if (temp_data.dispatchTime) {
       this.workOrder.orderTime = temp_data.dispatchTime;
    }
    if (temp_data.materialList) {
      let temp_arr = [];

      for (let i = 0; i < temp_data.materialList.length; i++) {
        let temp_json = {};
        temp_json.name = temp_data.materialList[i].materialName;
        temp_json.content = temp_data.materialList[i].workContent;
        temp_json.unit = temp_data.materialList[i].materialUnit;
        temp_json.number = temp_data.materialList[i].materialNum;
        temp_json.remark = temp_data.materialList[i].remark;
        temp_arr.push(temp_json);
      }

    this.workOrder.productList = temp_arr;
    }
    if (temp_data.orderCode) {
      this.workOrder.orderNum = temp_data.orderCode;
    }
    if (temp_data.lastHandleTime) {
      this.workOrder.lastHandleTime = temp_data.lastHandleTime;
    }
    if (temp_data.checkAddr) {
      this.workOrder.address = temp_data.checkAddr;
    }
    if (temp_data.type) {
      this.workOrder.typeSelect = temp_data.type;
    }
    if (temp_data.proUser) {
      this.workOrder.reporter = temp_data.proUser;
    }
    // this.orderStatus = temp_data.
    let temp_flag = this.$route.params.editFlag;
    this.problemStatus = temp_flag;
    this.dispatchFlag = sessionStorage.getItem('dispatchFlag') || 'false';

    console.log(this.workOrder,temp_data);
  },
  beforeDestroy() {
    sessionStorage.setItem('dispatchFlag', 'false');
  }
};
</script>

<style lang="scss" scoped>
$baseFontSize: 14px;
$baseColor: #333;
$baseBorderColor: #ccc;

#form {
  width: 100%;
  height: 100%;
  background: #ebebeb;
  font-size: $baseFontSize;
  color: $baseColor;

  .main {
    width: 100%;
    height: calc(100%);
    position: relative;

    .table-msg {
      width: 100%;
      height: 100%;

      .top-head {
        width: 100%;
        display: flex;
        justify-content: flex-end;
        padding: 0 20px;
        height: 40px;
        line-height: 40px;
        background: #caf2fc;

        .subTabs {
          font-weight: 600;

          span {
            display: inline-block;
            line-height: 38px;
            cursor: pointer;
            margin-right: 15px;

            &.actived {
              color: #0b83fe;
              border-bottom: 2px solid #0b83fe;
            }
          }
        }

        .actions-btn {
          cursor: pointer;

          span {
            display: inline-block;
            padding: 0px 15px;
            line-height: 25px;
            border-radius: 5px;
            background: #0b83fe;
            color: #fff;
            margin-right: 10px;

            &:last-child {
              background: #fe8a01;
            }

            &.finish {
              background: #67c23a;
            }

            &.upload {
              position: relative;
              width: 110px;
              height: 25px;

              input {
                display: inline-block;
                width: 100%;
                height: 100%;
                position: absolute;
                top: 0;
                left: 0;
                opacity: 0;
                cursor: pointer;
              }
            }

            &.gray {
              background: gray;
              cursor: not-allowed;
            }
          }
        }
      }

      .tables {
        input {
          display: inline-block;
          height: 30px;
          width: 100%;
          border: none;
          padding-left: 5px;
        }

        textarea {
          border: none;
          width: 100%;
        }

        width: 100%;
        height: calc(100% - 40px);
        padding: 0 10px;
        padding-bottom: 10px;
        display: flex;
        justify-content: center;

        .table-item {
          width: 800px;
          height: 100%;

          .inner-wrap {
            width: 100%;
            height: 100%;
            background: #fff;
            font-size: 14px;
            padding: 0 10px;
            overflow-y: auto;

            .mainHead {
              font-size: 16px;
              line-height: 40px;
              text-align: center;
              font-weight: 600;
              border-bottom: 1px solid #ccc;

              &.old {
                border-bottom: none;
              }
            }

            .subHead {
              width: 100%;
              padding: 0 10px;
              height: 30px;
              line-height: 30px;
              display: flex;
              justify-content: space-between;
              font-size: 12px;
              color: #666;
            }

            table {
              width: 100%;
              border-spacing: 0;
              border-collapse: collapse;
              border-top: 1px solid #ccc;
              margin-bottom: 20px;

              tr {
                border: 1px solid #ccc;
                border-top: none;

                &.noBorder {
                  border: none;

                  td {
                    padding-left: 20px;
                    border-right: none;
                    text-align: center;
                  }
                }

                td {
                  // min-height: 250px;
                  line-height: 30px;
                  height: 30px;
                  text-align: center;
                  border-right: 1px solid #ccc;
                  text-align: center;

                  span {
                    display: inline-block;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    width: 100%;
                  }
                  span.btn-del,
                  span.btn-add {
                    width: 60px;
                  }

                  &.sm {
                    width: 80px;
                  }

                  &.md {
                    width: 120px;
                  }

                  .inLine {
                    width: calc(100% - 76px);
                    display: inline-block;
                  }

                  &.result {
                    word-break: break-all;

                    span {
                      display: inline-block;
                    }
                  }

                  &.middle {
                    width: 100px;
                  }

                  &.long {
                    width: calc(100% - 300px);
                  }

                  &.largeText {
                    div {
                      height: 100px;
                    }
                  }

                  &.long-sm {
                    width: 50px;
                  }

                  &.long-md {
                    width: 200px;
                  }

                  &.long-long {
                    width: calc(100% - 450px);
                  }

                  &.store {
                    div {
                      height: 100px;
                    }
                  }

                  &.addCol {
                    span {
                      padding: 0 5px;
                      line-height: 20px;
                      border: 1px solid #0b83fe;
                      color: #0b83fe;
                      border-radius: 5px;
                      cursor: pointer;
                    }
                  }

                  &.md {
                    width: 300px;
                  }

                  &.del {
                    color: #333;
                    font-size: 12px;

                    span {
                      padding: 0 8px;
                      line-height: 20px;
                      border: 1px solid #0b83fe;
                      color: #0b83fe;
                      border-radius: 5px;
                      cursor: pointer;
                    }
                  }

                  &.getWorker {
                    position: relative;

                    .subLists {
                      width: 80%;
                      position: absolute;
                      top: 40px;
                      left: 10px;
                      background: #fff;
                      min-height: 100px;
                      max-height: 200px;
                      overflow-y: auto;
                      box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);

                      .listItem {
                        width: 100px;
                        line-height: 30px;
                        height: 30px;
                        border-bottom: 1px solid #ccc;
                        cursor: pointer;

                        &:hover {
                          background: #0b83fe;
                          color: #fff;
                        }
                      }
                    }
                  }

                  span.alignLeft{
                    text-align: left;
                    padding-left: 1em;
                  }
                }
              }
            }
          }
        }
      }
    }
  }

  .notice {
    width: 100%;
    height: 400px;

    .subContent {
      width: 100%;
      padding: 10px;
      height: calc(100% - 40px);
    }

    .footer {
      width: 100%;
      height: 40px;
      background: #e3e3e3;
      display: flex;
      justify-content: center;
      align-items: center;
      border-bottom-left-radius: 5px;
      border-bottom-right-radius: 5px;

      span {
        display: inline-block;
        padding: 0px 10px;
        height: 30px;
        line-height: 30px;
        background: #0b83ff;
        color: #fff;
        border-radius: 5px;
        margin-right: 10px;
        cursor: pointer;

        &.cancel {
          background: #ccccce;
        }
      }
    }
  }
}
</style>
