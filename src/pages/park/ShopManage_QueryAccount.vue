<template>
    <section>
        <common-table
                :queryapi="queryapi"
                :tableheight="tableheight"
                :fieldsstr="fieldsstr"
                :tableitems="tableitems"
                :btswidth="btswidth"
                :hide-export="hideExport"
                :hide-options="hideOptions"
                :searchtitle="searchtitle"
                :hideTool="hideTool"
                :orderfield="orderfield"

                :showParkInfo="showParkInfo"
                :hideSearch="hideSearch"
                :hideAdd="hideAdd"
                ref="bolinkuniontable"
        ></common-table>
    </section>
</template>


<script>
    import {path, checkURL, checkUpload, checkNumber, operateType} from '../../api/api';
    import util from '../../common/js/util'
    import common from '../../common/js/common'
    import {AUTH_ID} from '../../common/js/const'
    import CommonTable from '../../components/CommonTable'

    export default {
        components: {
            CommonTable
        },
        data() {
            return {
            	orderfield:'id',
                loading: false,
                hideExport: true,
                hideSearch: true,
                hideAdd: true,
                tableheight: '',
                showdelete: true,
                hideOptions: true,
                showParkInfo: false,
                hideTool: false,
                queryapi: '/shopaccount/quickquery',
                btswidth: '100',
                fieldsstr: 'shop_name__operate_time',
                tableitems: [
                    {

                        hasSubs: false,
                        subs: [{
                            label: '商户编号',
                            prop: 'shop_id',
                            width: '123',
                            type: 'number',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    }, 
                    {
                        hasSubs: false, subs: [
                            {
                                label: '商户名称',
                                prop: 'shop_name',
                                width: '180',
                                type: 'str',
                                editable: false,
                                searchable: true,
                                addable: true,
                                unsortable: true,
                                align: 'center',
                            },
                        ]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '额度(分钟)',
                            prop: 'ticket_limit',
                            width: '123',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function(row){
                            	 if(row.ticket_unit==1){
                            	 	if(row.ticket_limit==0){
                            	 		return "";
                            	 	}else{
                            	 		return row.ticket_limit;
                            	 	}
                            	 }else{
                            	 	return "";
                            	 }
                            }
                        }]
                    },{

                        hasSubs: false,
                        subs: [{
                            label: '额度(小时)',
                            prop: 'ticket_limit',
                            width: '123',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function(row){
                            	 if(row.ticket_unit==2){
                            	 	if(row.ticket_limit==0){
                            	 		return "";
                            	 	}else{
                            	 		return row.ticket_limit;
                            	 	}
                            	 }else{
                            	 	return "";
                            	 }
                            }
                        }]
                    }, {

                        hasSubs: false,
                        subs: [{
                            label: '额度(天)',
                            prop: 'ticket_limit',
                            width: '123',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function(row){
                            	 if(row.ticket_unit==3){
                            	 	if(row.ticket_limit==0){
                            	 		return "";
                            	 	}else{
                            	 		return row.ticket_limit;
                            	 	}
                            	 }else{
                            	 	return "";
                            	 }
                            }
                        }]
                    },{

                        hasSubs: false,
                        subs: [{
                            label: '额度(张)',
                            prop: 'ticketfree_limit',
                            width: '123',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function (row) {
                                if(row.ticketfree_limit==0){
                                	return ""
                                }else{
                                	return row.ticketfree_limit;
                                }
                            }
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '额度(元)',
                            prop: 'ticket_money',
                            width: '123',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function (row) {
                                if(row.ticket_money==0){
                                	return ""
                                }else{
                                	return row.ticket_money;
                                }
                            }
                        }]
                    },{

                        hasSubs: false,
                        subs: [{
                            label: '缴费金额(元)',
                            prop: 'add_money',
                            width: '180',
                            type: 'str',
                            editable: true,
                            searchable: false,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    }, {

                        hasSubs: false,
                        subs: [{
                            label: '操作人',
                            prop: 'nickname',
                            width: '180',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                        }]
                    }, {

                        hasSubs: false,
                        subs: [{
                            label: '操作时间',
                            prop: 'operate_time',
                            width: '180',
                            type: 'date',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function (row) {
                                return common.dateformat(row.operate_time)
                            }
                        }]
                    },  {

                        hasSubs: false,
                        subs: [{
                            label: '流水类型',
                            prop: 'operate_type',
                            width: '100',
                            type: 'selection',
                            selectlist:operateType,
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            format:function (row) {
                                if(row.operate_type==1){
                                	return "续费"
                                }else{
                                	return "回收充值";
                                }
                            }
                        }]
                    }

                ],
                searchtitle: '流水查询',

            }
        },      
        mounted() {
            window.onresize = () => {
                this.tableheight = common.gwh() - 143;
            };
            this.tableheight = common.gwh() - 143;
            var user = sessionStorage.getItem('user');
            if (user) {
                user = JSON.parse(user);
                for (var item of user.authlist) {
                	
                    if (AUTH_ID.shopManage_QueryAccount == item.auth_id) {
                        this.hideSearch = !common.showSubSearch(item.sub_auth)                       
                        break;
                    }
                }
            }
        },
        activated() {
            window.onresize = () => {
                this.tableheight = common.gwh() - 143;
            }
            this.tableheight = common.gwh() - 143;
            this.$refs['bolinkuniontable'].$refs['search'].resetSearch()
            this.$refs['bolinkuniontable'].getTableData({})
        }
    }

</script>

<style>
    .gutter {
        display: none
    }
</style>

