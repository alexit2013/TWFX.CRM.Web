<template>
  <section>
    <!--搜索-->
    <el-col :span="24" :class="this.Global.pageToolBarClass" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item  label="统计年份">
          <el-select v-model="filters.year" placeholder="统计年份" filterable  style="width: 120px;">
            <el-option
              v-for="item in yearList"
              :key="item.Yearid"
              :label="item.Yearname"
              :value="item.Yearid">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select v-model="filters.bunkId" placeholder="请选择商铺" clearable filterable style="width: 400px;">
            <el-option
              v-for="item in bunkAllList"
              :key="item.Bunkid"
              :label="item.Bunkname"
              :value="item.Bunkid">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="el-icon-search" v-on:click="f_Search">搜索</el-button>
        </el-form-item>
      </el-form>
    </el-col>


    <!--统计图-->
    <el-col :span="24" v-loading="listLoading">
      <div style="width:100%; margin-top: 30px;">
        <div style="float:left; width:2%;">&nbsp;</div>
        <div id="chartDiv1" style=" float:left; width:96%; height:500px;"></div>
        <div style="float:left; width:2%;">&nbsp;</div>
      </div>
    </el-col>



  </section>
</template>

<script>
  import echarts from 'echarts'
  export default {
    data() {
      return {
        //表格高度
        tableHeight:this.Global.pagetableHeight,
        //列表界面
        filters: {
          year: '',
          bunkId: '',
        },
        listLoading: false,
        //资源
        yearList:[],
        bunkAllList:[],
        //统计图
        chartStatistics: null,
        chartData: {stockInList:[0,0,0,0,0,0,0,0,0,0,0,0],stockOutList:[0,0,0,0,0,0,0,0,0,0,0,0]},
      }
    },
    methods: {

      //统计图
      f_Statistics: function (data) {
        this.chartStatistics = echarts.init(document.getElementById('chartDiv1'));
        this.chartStatistics.setOption({
          title: {
            text: '产品出入库统计图',
            subtext: '数量统计',
            x: 'left'
          },
          tooltip: {
            trigger: 'axis',
          },
          legend: {
            data: ['出库', '入库']
          },
          toolbox: {
            show : true,
            feature : {
              dataView : {
                show: true, readOnly: true,
                lang:['产品出入库统计数据','关闭','刷新'],
                optionToContent: function (opt) {
                  let axisData = opt.xAxis[0].data; //坐标数据
                  let series = opt.series; //类型数据
                  //组装表头
                  let tdHeads = '<td  style="width:8%;">时间</td>';
                  var pwidth = 92 / series.length;
                  for (let i = 0; i < series.length; i++) {
                    tdHeads = tdHeads + '<td style="width:' + pwidth + '%">' + series[i].name + '</td>';
                  }
                  let table = '<table border="1" style="width:100%; border-collapse:collapse;font-size:14px;text-align:center">';
                  table = table + '<tbody><tr style="background-color: #ccc; height: 32px; line-height: :32px;">'+tdHeads+'</tr>';
                  //组装数据
                  let tdBodys = '';
                  for (let i = 0; i < axisData.length; i++) {
                    for (let j = 0; j < series.length; j++) {
                      tdBodys += '<td>' + series[j].data[i] + '</td>';
                    }
                    table += '<tr style="height: 24px; line-height: 24px;"><td style="text-align: center;">' + axisData[i] + '</td>' + tdBodys + '</tr>';
                    tdBodys = '';
                  }
                  table += '</tbody></table>';
                  return table;
                }
              },
              magicType : {show: true, type: ['line', 'bar']},
              restore : {show: true},
              saveAsImage : {show: true,title:'下载'}
            }
          },
          calculable : true,
          xAxis : [
            {
              type : 'category',
              data : ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月']
            }
          ],
          yAxis : [
            {
              type : 'value'
            }
          ],
          series : [
            {
              name:'出库',
              type:'bar',
              data:data.stockOutList,
              markPoint : {
                data : [
                  {type : 'max', name: '最大值'},
                  {type : 'min', name: '最小值'}
                ]
              },
              markLine : {
                data : [
                  {type : 'average', name: '平均值'}
                ]
              }
            },
            {
              name:'入库',
              type:'bar',
              data:data.stockInList,
              markPoint : {
                data : [
                  {type : 'max', name: '最大值'},
                  {type : 'min', name: '最小值'}
                ]
              },
              markLine : {
                data : [
                  {type : 'average', name : '平均值'}
                ]
              }
            }
          ]
        });
      },




      //===============================================================
      //资源加载
      //获取全部商铺
      f_DataSelectBunkAll:function(){
        let pars= {
          page: 1,
          size: 1000000,
          bunkName: '',
          traderName:'',
          businessState:1,
          intention:1,
        };
        let _this=this;
        this.$httpPost('Bunk/GBunkList',pars).then((res)=> {
          if(res.success=='ok') {
            _this.bunkAllList = res.dtData;
          }
          else {
          }
        }) .catch(function (error) {
          _this.$message.error('数据错误！');
        });

      },
      f_ConfigLoadYear:function(){
        let _this=this;
        _this.yearList=[];
        this.$httpPost('Report/GYear',{}).then((res)=> {
          if(res.success=='ok') {
            if (res.years != null && res.years.length > 0) {
              for (let m = 0; m < res.years.length; m++) {
                let model = {
                  Yearname: res.years[m],
                  Yearid: res.years[m]
                };
                _this.yearList.push(model);
              }
            }
          }
        }) .catch(function (error) {
          _this.$message.error('数据错误！');
        });
      },
      //搜索
      f_Search: function () {
        this.listLoading = true;
        this. chartData={stockInList:[0,0,0,0,0,0,0,0,0,0,0,0],stockOutList:[0,0,0,0,0,0,0,0,0,0,0,0]};
        let curDate=new Date();
        if (this.filters.year == '')
          this.filters.year = curDate.getFullYear();

        //资源加载
        this.f_DataSelectBunkAll();
        this.f_ConfigLoadYear();
        let _this=this;
        let parms = {
          year:this.filters.year,
          bunkId:this.filters.bunkId,
        };
        _this.$httpPost('Report/GBunkStatistics', parms).then((res) => {
          if (res.success == 'ok') {

            for(let i=0;i<res.stockInList.length;i++) {
              _this.chartData.stockInList[res.stockInList[i].Smonth - 1] = res.stockInList[i].Scount;
            }
            for(let i=0;i<res.stockOutList.length;i++) {
              _this.chartData.stockOutList[res.stockOutList[i].Smonth - 1] = res.stockOutList[i].Scount;
            }
            _this.f_Statistics(_this.chartData);
            _this.listLoading = false;
          }
          else {
            _this.listLoading = false;
          }
        }).catch(function (error) {
          _this.$message.error('获取数据出错！');
          _this.listLoading = false;
        });
      },
    },
    mounted() {
      this.f_Search();
    },
  }
</script>

<style scoped>

</style>
