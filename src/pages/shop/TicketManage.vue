<template>
    <section>
       <el-form :inline="true" v-model="formItem" class="demo-form-inline">

           <el-form-item label="创建时间" style="margin-bottom: 5px;">
              <el-date-picker
                      v-model="formItem.datesselector"
                      type="datetimerange"
                      align="right"
                      style="float:left"
                      unlink-panels
                      range-separator="至"
                      :start-placeholder="start_placeholder"
                      :end-placeholder="end_placeholder"
                      value-format="yyyy-MM-dd HH:mm:ss"
                      :picker-options="pickerOptions3"
                      :default-time="['00:00:00', '23:59:59']">
              </el-date-picker>
          </el-form-item>
           <el-form-item label="车牌号" class="carnumner-buttom">
               <el-input v-model="formItem.car_number" placeholder="车牌号"></el-input>
           </el-form-item>
            <el-form-item  class="member-buttom">
             <el-select placeholder="全部" v-model="formItem.shopuser">
                 <el-option
                         label="全部员工"
                         value="">
                 </el-option>
                 <el-option
                         v-for="item in shopusers"
                         :key="item.value_no"
                         :label="item.value_name"
                         :value="item.value_no">
                 </el-option>
             </el-select>
          </el-form-item>



           <el-form-item class="inp-margin-buttom">
               <el-button type="primary" @click="search">搜索</el-button>
               <el-tooltip class="item" effect="dark" content="导出内容为当前查询条件下所有数据" placement="bottom">
                   <el-button type="primary" @click="handleExport" v-if="!hideExport">导出</el-button>
               </el-tooltip>
               </el-button>
           </el-form-item>
       </el-form>
       <el-table :data="table" border highlight-current-row style="width:100%;" :height="tableheight"
                 v-loading="loading" @sort-change="sortChange">
           <!--子项折叠最终通过rowStyle实现。实际是项的显示/隐藏-->
           <el-table-column v-for="(items, index) in tableitems" :key="items.subs[0].prop"
                            :label="items.subs[0].label" :align="items.subs[0].prop=='name'?'left':'center'"
                            min-width="50"
                            :width="items.subs[0].prop=='car_number'||items.subs[0].prop=='mobile'||items.subs[0].prop=='state'?130:200"
                            :sortable="!items.subs[0].unsortable" v-if="!items.subs[0].hidden">
               <!--设置部分列宽度-->

               <template scope="scope">
                   <span v-if="items.subs[0].prop=='create_time'">{{common.dateformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else-if="items.subs[0].prop=='limit_day'">{{common.dateformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else-if="items.subs[0].prop=='use_time'">{{common.dateformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else-if="items.subs[0].prop=='state'">{{stateformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else-if="items.subs[0].prop=='type'">{{typeformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else-if="items.subs[0].prop=='money'">{{moneyformat(scope.row)}}</span>
                   <span v-else-if="items.subs[0].prop=='umoney'">{{umoneyformat(scope.row)}}</span>
                   <span v-else-if="items.subs[0].prop=='uin'">{{nameformat(scope.row[items.subs[0].prop])}}</span>
                   <span v-else>{{scope.row[items.subs[0].prop]}}</span>
                   <!--不同列的表现形式、格式化-->
               </template>

           </el-table-column>
       </el-table>

        <!--工具条-->
       <el-col :span="24" align="bottom" style="margin-top:5px;margin-bottom:5px">
           <el-col :span="24" align="right">
               <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                              :current-page="currentPage" :page-sizes="[20, 40, 80]" :page-size="pageSize"
                              layout="total, sizes, prev, pager, next, jumper" :total="total"></el-pagination>
           </el-col>
       </el-col>
    </section>
</template>


<script>
    import {path, checkURL, checkUpload, checkNumber, payType, operateType,ticketQueryType,statType} from '../../api/api';
    import util from '../../common/js/util'
    import common from '../../common/js/common'
    import {AUTH_ID_SHOP} from '../../common/js/const'
    import CommonTable from '../../components/CommonTable'
    import axios from 'axios';

    export default {
        components: {
            CommonTable
        },
        data() {
            var _this = this;
            return {
                createtime:'between',
                total:0,
                start_placeholder: '',
                end_placeholder: '',
                datesselector: '',
                minDate : '',
                maxDate : '',
                pickerOptions3:{
                    onPick(dates) {
                      _this.minDate = dates.minDate;
                      _this.maxDate = dates.maxDate;
                    },

                  disabledDate(time){
                        return time.getTime() >  new Date(_this.minDate).getTime()+30*24*3600000||time.getTime()>Date.now()||time.getTime() <  new Date(_this.minDate).getTime()-30*24*3600000;
                  }
                },
                formItem:{
                    car_number:'',
                    state:'',
                    datesselector:'',
                    shopuser:'',
                },
                table: [],
                pageSize: 20,
                currentPage: 1,
                orderby: 'desc',
                orderfield: 'id',
                shopusers:[],
                loading: false,
                hideExport: true,
                hideSearch: false,
                showTicketInfo: false,
                hideAdd: true,
                tableheight: '',
                showdelete: true,
                hideOptions: true,
                hidden_type:0,
                hideTool: false,
                showEdit: true,
                showdelete: true,
                queryapi: '/shopticket/getticketlog',
                exportapi: '/shopticket/exportlog',
                btswidth: '100',
                fieldsstr: 'id__money__umoney__limit_day__car_number__create_time__type__use_time__state__uin',
                tableitems: [
                    {
                        hasSubs: false,
                        subs: [{
                            label: '车牌号',
                            prop: 'car_number',
                            width: '180',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            hidden:'',
                            align: 'center'
                        }]
                    },
                    {
                        hasSubs: false,
                        subs: [{
                            label: '优惠时长',
                            prop: 'money',
                            width: '180',
                            type: 'number',
                            editable: true,
                            searchable: false,
                            addable: true,
                            hidden:'',
                            unsortable: true,
                            align: 'center',

                        }]
                    },
                    {
                        hasSubs: false,
                        subs: [{
                            label: '优惠金额',
                            prop: 'umoney',
                            width: '180',
                            type: 'number',
                            hidden:'',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',

                        }]
                    },
                    {
                        hasSubs: false,
                        subs: [{
                            label: '发券人',
                            prop: 'uin',
                            width: '180',
                            type: 'selection',
                            selectlist: this.shopusers,
                            editable: true,
                            searchable: true,
                            addable: true,
                            hidden:'',
                            unsortable: true,
                            align: 'center',

                        }]
                    },
                     {

                        hasSubs: false,
                        subs: [{
                           label: '状态',
                            prop: 'state',
                            width: '100',
                            type: 'selection',
                            selectlist:statType,
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            hidden:'',
                            align: 'center',
                            format:function (row) {
                                if(row.state==0){
                                    return "未使用"
                                }else if(row.state==1){
                                    return "已使用";
                                }else{
                                    return "回收作废";
                                }
                            }
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '优惠类型',
                            prop: 'type',
                            width: '100',
                            type: 'selection',
                            selectlist:ticketQueryType,
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            hidden:'',
                            align: 'center',
                            format:function (row) {
                                if(row.type==3){
                                    return "时长减免"
                                }else if(row.type==5){
                                    return "金额减免";
                                }else if(row.type==4){
                                    return "全免券";
                                }else{
                                    return row.type;
                                }
                            }
                        }]
                    },

                   {
                          hasSubs: false,
                          subs: [{
                              label: '使用时间',
                              prop: 'use_time',
                              width: '180',
                              type: 'date',
                              editable: true,
                              hidden:'',
                              searchable: true,
                              addable: true,
                              unsortable: true,
                              align: 'center',

                          }]
                      },
                    {
                        hasSubs: false,
                        subs: [{
                            label: '到期时间',
                            prop: 'limit_day',
                            width: '180',
                            type: 'date',
                            editable: true,
                            searchable: false,
                            hidden:'',
                            addable: true,
                            unsortable: true,
                            align: 'center',

                        }]
                    },
                    {
                          hasSubs: false,
                          subs: [{
                              label: '创建时间',
                              prop: 'create_time',
                              width: '180',
                              type: 'date',
                              editable: true,
                              searchable: true,
                              addable: true,
                              hidden:'',
                              unsortable: true,
                              align: 'center',

                          }]
                      },

                ],
                searchtitle: '查询明细',
                moneyformat:function(row){
                    if(row.ticket_unit==1&&row.money!=0){
                        return row.money+'分钟';
                     }else if(row.ticket_unit==2&&row.money!=0){
                        return row.money+'小时';
                     }else if(row.ticket_unit==3&&row.money!=0){
                        return row.money+'天';
                     }else{
                        return "";
                     }
                },
                umoneyformat:function(row){
                    if(row.ticket_unit==4&&row.umoney!=0){
                        return row.umoney+'元';
                     }else{
                        return "";
                     }
                },
                stateformat:function(state){
                    if(state==0){
                        return "未使用"
                    }else if(state==1){
                        return "已使用";
                    }else{
                        return "回收作废";
                    }
                },
                typeformat:function(type){
                    if(type==3){
                        return "时长减免"
                    }else if(type==5){
                        return "金额减免";
                    }else if(type==4){
                        return "全免券";
                    }else{
                        return "";
                    }
                },
                nameformat:function(uin){
                    var shopusers = this.shopusers;
                    for (let x in shopusers) {
                        if (shopusers[x].value_no == uin) {
                            return shopusers[x].value_name;
                        }
                    }
                }
            }
        },
        methods:{

            search:function () {
                this.sform = {};
                if(this.formItem.car_number != ""){
                    this.sform.car_number = this.formItem.car_number;
                }
                if(this.formItem.state != ""){
                    this.sform.state = this.formItem.state;
                    this.sform.state_start = this.formItem.state;
                }
                if(this.formItem.datesselector != ""){
                     this.createtime_start = (new Date(this.formItem.datesselector[0].replace(new RegExp(/-/gm) ,"/"))).getTime();
                     this.createtime_end = (new Date(this.formItem.datesselector[1].replace(new RegExp(/-/gm) ,"/"))).getTime();

                     this.sform.create_time = this.createtime;
                     this.sform.create_time_start = this.createtime_start;
                     this.sform.create_time_end = this.createtime_end;
                }
                if(this.formItem.shopuser != ""){
                    this.sform.uin = this.formItem.shopuser;
                    this.sform.uin_start = this.formItem.shopuser;
                }
                //hidden_type  1 隐藏时长  2 隐藏金额
                this.sform.hidden_type=this.hidden_type
                this.getTableData(this.sform)
            },
            //分页变动
            handleSizeChange(val) {
                this.pageSize = val;
                this.getTableData(this.sform);
            },
            handleCurrentChange(val) {
                this.currentPage = val;
                console.log('page change')
                this.sform.date = this.searchDate
                this.getTableData(this.sform);
            },
            //排序变动
            sortChange(val) {
                if (val.order != null && val.order.substring(0, 1) == "a") {
                    this.orderby = "asc";
                } else {
                    this.orderby = "desc";
                }
                this.orderfield = val.prop;
                console.log('sort change')
                this.getTableData(this.sform);
            },
             //拉取表格数据
            getTableData(sform1) {
                let vm = this;
                this.loading = false;
                let api = this.queryapi;
                sform1.rp = this.pageSize;
                sform1.page = this.currentPage;
                sform1.orderby = this.orderby;
                sform1.orderfield = this.orderfield;
                sform1.fieldsstr = this.fieldsstr;
                sform1.hidden_type = this.hidden_type;
                this.sform = common.generateForm(sform1);
                //保证5秒后把loading干掉
                setTimeout(() => {
                    vm.loading = false;
                }, 5000);
                vm.$axios.post(path + api, vm.$qs.stringify(this.sform), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                    }
                }).then(function (response) {
                    vm.loading = false;
                    // console.log('resset loading!!!!!');
                    let ret = response.data;
                    if (ret.validate != 'undefined' && ret.validate == '0') {
                        vm.loading = false;
                        //未携带令牌.重新登录
                        setTimeout(() => {
                            vm.alertInfo('未携带令牌,请重新登录!');
                        }, 150);
                    } else if (ret.validate != 'undefined' && ret.validate == '1') {
                        vm.loading = false;
                        //过期.重新登录
                        setTimeout(() => {
                            vm.alertInfo('登录过期,请重新登录!');
                        }, 150);
                    } else if (ret.validate != 'undefined' && ret.validate == '2') {
                        vm.loading = false;
                        //令牌无效.重新登录
                        setTimeout(() => {
                            vm.alertInfo('登录异常,请重新登录!');
                        }, 150);
                    } else {
                        //  console.log('这是查询出来的结果'+ret);
                        if (ret.total == 0) {
                            vm.table = [];
                        } else {
                            vm.table = ret.rows;
                        }
                        if (ret.actReceivable != undefined) {
                            //月卡续费记录实收
                            vm.actualpay = ret.actReceivable + '元';
                        }
                        if (ret.amountReceivable != undefined) {
                            //月卡续费记录应收
                            vm.shouldpay = ret.amountReceivable + '元';
                        }
                        if (ret.blank != undefined) {
                            //订单记录 车位统计-空车位
                            vm.parkspace_blank = ret.blank;
                        }
                        if (ret.parktotal != undefined) {
                            //订单记录 车位统计-场内停车
                            vm.parkspace_park = ret.parktotal;
                        }
                        if (ret.cashpay != undefined) {
                            //集团 业务订单-订单记录-现金支付
                            vm.cashpay = ret.cashpay + '元';
                        }
                        if (ret.elepay != undefined) {
                            //集团 业务订单-订单记录-手机支付
                            vm.elepay = ret.elepay + '元';
                        }
                        if (ret.sumtotal != undefined) {
                            //集团 业务订单-订单记录-订单总金额
                            vm.sumtotal = ret.sumtotal + '元';
                        }

                        vm.total = ret.total;

                        vm.loading = false;
                    }
                    //  console.log("get table 55555:",vm.$refs['search'].searchForm);
                }).catch(function (error) {
                    vm.loading = false;
                    setTimeout(() => {
                        vm.alertInfo('请求失败!' + error);
                    }, 150);
                });

            },
            //导出表格数据
            handleExport() {
                let vm = this;
                let api = this.exportapi;
                let params = '';
                if (common.getLength(this.sform) == 0) {
                    params = 'fieldsstr=' + this.fieldsstr + '&token=' + sessionStorage.getItem('token');
                } else {
                    for (var x in this.sform) {
                        if(x=='car_number'||x=='nickname1'){
                            params += x + '=' + encodeURI(encodeURI(this.sform[x])) + '&';
                        }else{
                            params += x + '=' + this.sform[x] + '&';
                        }
                    }
                }
                let groupid = sessionStorage.getItem('groupid');
                let cityid = sessionStorage.getItem('cityid');
                if (groupid != 'undefined' && !(params.indexOf('groupid=') > -1)) {
                    params += '&groupid=' + groupid;
                }
                if (cityid != 'undefined' && !(params.indexOf('cityid=') > -1)) {
                    params += '&cityid=' + cityid;
                }
                // params += '&groupid=' + groupid + '&cityid=' + cityid
                if (params.indexOf('comid=') > -1) {
                    window.open(path + api + '?' + params);
                } else {
                    window.open(path + api + '?' + params + '&comid=' + sessionStorage.getItem('comid'));
                }

            },
            alertInfo(msg) {
                this.$alert(msg, '提示', {
                    confirmButtonText: '确定',
                    type: 'warning',
                    callback: action => {
                        sessionStorage.removeItem('user');
                        sessionStorage.removeItem('token');
                        localStorage.removeItem('comid');
                        localStorage.removeItem('groupid');
                        if(this.$router){
                            this.$router.push('/login');
                        }

                    }
                });
            },
            getShopAccountInfo(){
              let vm = this;
              vm.$axios.post(path+"/shopaccount/shopinfo?id="+sessionStorage.getItem('shopid'),{
                  headers: {
                      'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                  }
              }).then(function (response) {
                let ret = response.data;
                if(ret.ticket_unit==1||ret.ticket_unit==2||ret.ticket_unit==3){
                    //时长
                    vm.tableitems[2].subs[0].hidden = "true";
                    vm.sform.hidden_type=2
                    vm.hidden_type = 2;
                }

                else if(ret.ticket_unit==4){
                    //金额  隐藏时长
                    vm.tableitems[1].subs[0].hidden = "true";
                    vm.sform.hidden_type=1
                    vm.hidden_type = 1;
                 }

              });
            },
        },
         beforeMount(){
                        this.tableheight=common.gwh()-150;
                    },
        mounted() {

            this.formItem.datesselector = common.currentDateArray(1);
            window.onresize = () => {
                this.tableheight = common.gwh() - 143;
            }
            this.tableheight = common.gwh() - 143;

            var user = sessionStorage.getItem('user');
            if (user) {
                user = JSON.parse(user);
                for (var item of user.authlist) {
                    if (AUTH_ID_SHOP.ticketManage == item.auth_id) {
                        this.hideExport = !common.showSubExport(item.sub_auth)
                        break;
                    }
                }
            }
        },
        watch: {
            shopusers: function (val) {
                this.tableitems[5].subs[0].selectlist = val;
            }
        },
        activated() {
            this.getShopAccountInfo()
            window.onresize = () => {
                this.tableheight = common.gwh() - 143;
            }
            this.tableheight = common.gwh() - 143;
            //this.$refs['bolinkuniontable'].$refs['search'].resetSearch()
            //this.$refs['bolinkuniontable'].getTableData({})

            this.formItem.car_number = '';
            this.formItem.shopuser = '';
            this.formItem.datesselector = common.currentDateArray(1);

            this.getTableData({})

             let _this = this;
               axios.all([common.getShopUsers()])
                .then(axios.spread(function (ret) {
                    _this.shopusers = ret.data;
                }));

        }
    }

</script>

<style>
    .gutter {
        display: none
    }
    .member-buttom{
         margin-bottom: 10px!important;
         width:140px;
     }
</style>

