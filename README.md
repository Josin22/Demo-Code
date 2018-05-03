[TOC]

# pc管理系统代码展示:

UI:

![](https://raw.githubusercontent.com/Josin22/Demo-Code
/master/images/pc1.png)

Code:

```
import React from 'react';
import { connect } from 'dva';
import { routerRedux } from 'dva/router';
import { Form, Popconfirm, Button, Card, message, Divider } from 'antd';
import _ from 'lodash';
import qs from 'qs';
import SearchForm from 'components/SearchForm';
import StandardTable from 'components/StandardTable';
import PageHeaderLayout from 'layouts/PageHeaderLayout';
import StaffModal from './StaffModal';

const StaffsIndex = ({ dispatch, location, staffs, form }) => {
  const { data, selectedRows, pagination } = staffs;

  const handleQuery = (params = {}) => {
    dispatch(routerRedux.push({
      pathname: location.pathname,
      search: qs.stringify(params),
    }));
  };

  const searchFormProps = {
    form,
    setupValue: qs.parse(location.search.replace(/^[?]*(.*)$/, '$1')).q,
    handleSearch: (formData) => { handleQuery(formData); },
    formItems: [
      {
        label: '姓名',
        key: 'q[name_cont]',
        type: 'Input',
      },
      {
        label: '手机号',
        key: 'q[mobile_cont]',
        type: 'Input',
      },
    ],
  };

  const columns = [
    {
      title: '姓名',
      dataIndex: 'name',
    },
    {
      title: '手机号',
      dataIndex: 'mobile',
    },
    {
      title: '详细地址',
      dataIndex: 'address',
    },
    {
      title: '开通时间',
      dataIndex: 'created_at',
    },
    {
      title: '类型',
      dataIndex: 'genre',
    },
    {
      title: '状态',
      dataIndex: 'status',
    },
    {
      title: '操作',
      key: 'operation',
      render: (text, record) => (
        <span>
          <StaffModal actionType="edit" record={record} onOk={editHandler.bind(null, record.id)}>
            <Button icon="edit" type="primary">编辑</Button>
          </StaffModal>
          <Divider type="vertical" />
          <Popconfirm title="确定要删除吗?" onConfirm={deleteHandler.bind(null, record.id)}>
            <Button icon="delete" type="danger">删除</Button>
          </Popconfirm>
        </span>
      ),
    },
  ];

  const pageChangeHandler = (page) => {
    const params = {
      ...form.getFieldsValue(),
      page: page.current,
      per_page: page.pageSize,
    };
    handleQuery(params);
  };

  const createHandler = (values) => {
    dispatch({
      type: 'staffs/create',
      payload: { staff: values },
    });
  };

  const deleteHandler = (id) => {
    dispatch({
      type: 'staffs/remove',
      payload: id,
    }).then((res) => {
      if (res.success) {
        message.success('删除成功！');
        const params = {
          ...form.getFieldsValue(),
          page: pagination.current,
          per_page: pagination.pageSize,
        };
        handleQuery(params);
      }
    });
  };

  const editHandler = (id, values) => {
    dispatch({
      type: 'staffs/create',
      payload: { staff: _.merge(values, { id }) },
    });
  };

  return (
    <PageHeaderLayout title="员工列表">
      <Card bordered={false}>
        <div>
          <SearchForm {...searchFormProps} />
          <StaffModal actionType="create" record={{}} onOk={createHandler}>
            <Button icon="plus" type="primary">
              新建
            </Button>
          </StaffModal>
          <StandardTable
            data={data}
            columns={columns}
            selectedRows={selectedRows}
            pagination={pagination}
            onChange={pageChangeHandler}
          />
        </div>
      </Card>
    </PageHeaderLayout>
  );
};

export default connect(({ staffs, loading }) => ({ staffs, loading }))(Form.create()(StaffsIndex));


```

# 微信小程序代码展示:

UI:

![](https://raw.githubusercontent.com/Josin22/Demo-Code
/master/images/wx1.png)

Code:

```
<template lang="wxml">
  <view class="page" >
    <form bindsubmit="submit" class="page__bd">
      <view class="zan-panel">
            <view class="weui-cell weui-cell_access">
            <view class="weui-cell__bd">
                <button class="zan-btn zan-btn--primary">
                    <picker mode="date" value="{{date}}" bindchange="bindDateChange">
                        <view class="picker">自提时间 : {{date}}</view>
                    </picker>
                </button>
            </view>
        </view>
      </view>
      <view class="zan-panel">
        <navigator
          url="product/index?gift=0"
          class="weui-cell weui-cell_access" hover-class="weui-cell_active">
          <view class="weui-cell__bd">
            <button class="zan-btn zan-btn--primary">选择商品</button>
          </view>
        </navigator>
        <view>
          <lineItemList :line_items.sync="line_items"/>
        </view>
      </view>
      <view class="zan-panel">
        <navigator
          url="product/index?gift=1"
          class="weui-cell weui-cell_access" hover-class="weui-cell_active">
          <view class="weui-cell__bd">
            <button class="zan-btn zan-btn--primary">选择赠品</button>
          </view>
        </navigator>
        <view>
          <gift_lineItemList :line_items.sync="gift_line_items"/>
        </view>
      </view>
      <view class="zan-panel">
        <view class="page-section">
          <textarea bindinput="bindinput" auto-height placeholder="下单留言" />    
        </view>
      </view>
    
      <view class="zan-panel my-30">
        <button disabled="{{disabled}}" type="primary" formType="submit">提交订单</button>
      </view>
    </form>
  </view>
</template>

<script>
  import wepy from 'wepy'
  import _ from 'lodash'
  import api from '../api/api.js'
  import LineItemList from '../components/lineItemList'

  export default class addSelfHelpOrder extends wepy.page {
    config = {};

    components = {
      lineItemList: LineItemList,
      gift_lineItemList: LineItemList,
    };

    data = {
      title: '自提下单',
      id: '',
      line_items: [],
      gift_line_items: [],
      total: 0,
      disabled: true,
      moment:'',
      date: '- - -',
    };
    methods = {
         bindDateChange: function(e) {
            console.log('picker发送选择改变，携带值为', e.detail.value)
            this.date = e.detail.value
            this.disabled = _.isEmpty(this.line_items) || _.isEmpty(this.date)
        },
       
    };

    events = {
      'line_item_remove': (lineItem) => {
        if (lineItem.gift) {
          _.remove(this.gift_line_items, (item) => item.product_id === lineItem.product_id)
        } else {
          _.remove(this.line_items, (item) => item.product_id === lineItem.product_id)
        }

        this.disabled =  _.isEmpty(this.line_items) || _.isEmpty(this.date)
      },
  }
  
    async onLoad() {
      wx.setNavigationBarTitle({ title: this.title, })
    };

    bindinput(e) {
      console.log('bindinput',e.detail.value)
      this.moment = e.detail.value
    }

    onRoute() {
      // this.shop = this.$parent.globalData.current_shop
      this.line_items = this.$parent.globalData.line_items
      this.gift_line_items = this.$parent.globalData.gift_line_items
      this.total = _.reduce(this.line_items, (sum, lineItem) => sum + lineItem.price * lineItem.package_quantity, 0)

      this.disabled = _.isEmpty(this.date) || _.isEmpty(this.line_items)

      this.$apply()
    }

    async submit(e) {
      // Object.assign(this.order, e.detail.value)
      console.log('submit',e);
    
      let cart = _.map(this.line_items, (lineItem) => _.omit(lineItem, [ 'stock_item', ]))
      cart = _.concat(cart, _.map(this.gift_line_items, (lineItem) => _.omit(lineItem, [ 'stock_item', ])))

      let current_user = wepy.getStorageSync('current_user')

      let params = {
        'ziti_at':this.date,
        'genre':'ziti',
        'cart':JSON.stringify(cart),
        'staff_id':current_user.id,
        'comment':this.moment
      }

      let rep = await api.addOrder({data:params})

      if(rep.statusCode === 201) {
        wx.showToast({
          title: '自提下单成功',
          icon: 'success',
        })
         await this.$parent.sleep(1)
        wx.navigateBack()
      } else {
         wx.showToast({
          title: '自提下单失败',
          icon: 'info',
        })
      }

      console.log('ziti ret',rep)

    };
  }
</script>

<style lang="less">
  .page {
    background: #F9F9F9;
  }
  .comment .zan-field--wrapped {
    margin-top: 15px;
  }
  .comment__note {
    padding: 0 15px;
    font-size: 12px;
    color: #999;
  }
  .radio .zan-icon {
    margin-right: 8px;
  }
  .radio .zan-icon::before {
    color: #4b0;
  }
  .my-30 {
    margin-top: 30px;
    margin-bottom: 30px;
  }
  .page-section {
    margin-top: 20px;
    margin-bottom: 20px;
  }
</style>


```

# H5项目代码展示:

UI:

![](https://raw.githubusercontent.com/Josin22/Demo-Code
/master/images/h5.png)

Code:

```
import React from 'react'
import Metrics from "../../utils/Metrics";
import GameMainTabbar from "../main_tabbar/GameMainTabbar";
import NavBar from "../main_tabbar/NavBar";

var Ons = require('react-onsenui');
import {setCurrentTabbarSign} from "../../utils/TabbarLocalStorage";
import PropTypes from 'prop-types';
import {dispatch} from 'redux'
import {routerRedux} from 'dva/router';
import {connect} from "dva";
import Images from "../../utils/Images";
import {getAssets, getNewIntegral} from '../../services/CenterService'
import {chainbeans, getAvailabilityNumber, getVirtualLimit} from '../../services/HomeService'
import {font,convert_unit} from '../../utils/Metrics'

import UserProtocol from "../user_protocol/UserProtocol";
import {userProtocol} from "../../services/Market";
import {setAuthenticationToken} from "../../utils/authentication";
import _ from 'lodash'
import {get_limit_info, get_assets_yesterday} from '../../services/HomeService'
import moment from 'moment'
import {show_info} from "../reuse_view/HudTools";
import Styles from './integral.css'

class YDBIntegralView extends React.Component {

  constructor(props) {
    super(props)
    let hash = _.get(props,'location.hash')
    if (!_.isEmpty(hash)){
      let hash_array = _.split(hash,'/')
      let last_value = _.last(hash_array)
      if (hash_array.length==3&&last_value){
        setAuthenticationToken(last_value)
      }
    }
    
    this.state = ({
      modale_text: '',
      XianeshowDialog: false,      //限额说明 弹窗开关
      Assets: [],                  //用户资产
      AvailabilityNumber: '',      //兑换指数
      protocolState: false,
      start_time: '',              //禁止转入 开始时间
      end_time: '',                 //禁止转入 结束时间
      assets_yesterday: 0,          //昨日收益
      cant_go: true,
      limit_info: '',
      order_data: [],
      transfer_num: 0,    //手动转积分剩余次数
      forbid: false,        //是否禁止转入 false允许转入    true 禁止转入
      virtualLimit:'',
      total_ledou:0,    //联查总的乐豆余额
      total_yidou:0,    //联查总的益豆余额
      total_letaolian:0, //联查总的乐淘链余额
      xiane_dialog_content:''
    })
  }

  //接口获取并赋值
  componentWillMount() {

    /**
     * 查询乐豆,益豆,乐淘链各自总和
     * 2018年1月3日11:13:34
     */
    chainbeans().then(res=>{
      let total_ledou =  _.get(res,'data.HAPPY_BEANS');
      let total_yidou = _.get(res,'data.YI_BEANS');
      let total_letaolian = _.get(res,'data.NAUGHTY_MONEY');

      //console.log('查询乐豆,益豆,乐淘链各自总和====',total_ledou,total_yidou,total_letaolian)
      res.code == 200
        ?this.setState({total_ledou:total_ledou,total_yidou:total_yidou,total_letaolian:total_letaolian})
        :this.setState({total_ledou:0,total_yidou:0,total_letaolian:0})
    }).catch(error=>{
      console.log('获取查询乐豆,益豆,乐淘链各自总和 失败',error)
    })

    //获取最新积分转入信息
    getNewIntegral()
      .then(
        result => {
          if (result.code == 200) {
            this.setState({order_data: result.data})
          } else {
            this.setState({order_data: null})
          }
        }
      ).catch(error => {
      console.log('错误', error)
    })
    //昨日收益
    get_assets_yesterday()
      .then(
        res => {
          res.code == 200 ? this.setState({assets_yesterday: res.data.YESTERDAY_PROFIT}) : this.setState({assets_yesterday: 0})
        }
      )
      .catch(error => {
        console.log('昨日收益错误', error)
      })
    //获取限额详情
    get_limit_info()
      .then(res => {

        //限额弹窗提示内容
        let xiane_dialog_content = _.get(res,'data.describe')
        this.setState({xiane_dialog_content:xiane_dialog_content})

        let cant_go = this.state.cant_go
        if (res.code == 200) {
          let limit_info = _.get(res, 'data')
          console.log('limit_info====/119',limit_info)

          if (limit_info) {
            if (limit_info.INPUT_TYPE == '2') {

              limit_info.xiane = limit_info.MIN_LIMIT + ' - ' + limit_info.MAX_LIMIT

            } else if (limit_info.INPUT_TYPE == '1') {

              limit_info.xiane = limit_info.MIN_LIMIT + '%-' + limit_info.MAX_LIMIT + '%'

            }
          }
          if ((_.isEmpty(limit_info.ban_start_time)) || (_.isEmpty(limit_info.ban_end_time))) {
            this.setState({
              cant_go: true
            })
          }
          console.log('limit_info', limit_info)
          console.log('ENTER_NUMBER///////', limit_info.ENTER_NUMBER)
          this.setState({
            limit_info: limit_info, transfer_num: limit_info.ENTER_NUMBER
          })
        }
      })
      .catch(error => {
        console.log('获取限额详情错误信息', error)
      })

    getVirtualLimit()
      .then(
        result => {
          if (result.code == 200) {
            let virtualLimit = result.data.VIRTUAL_LIMIT+'%'
            this.setState({virtualLimit})
          }
        }
      )

    //获取积分余额、益豆余额、昨日释放益豆、兑换指数
    getAssets()
      .then((res) => {
        res.code == 200 ? this.setState({Assets: res.data}) : this.setState({Assets: []})
      })
      .catch(error => {
        // console.log('getAssets error',error)
      })

    //获取兑换指数
    getAvailabilityNumber()
      .then((res) => {
        res.code == 200 ? this.setState({AvailabilityNumber: res.data.AVAILABILITY_NUMBER}) : this.setState({AvailabilityNumber: '0.0000'})
      }).catch(error => {
      // console.log('getAvailabilityNumber error',error)
    })
  }


  /**
   *  提示限额时间不能自转积分
   */
  show_limit_hint = () => {
    let order_data = this.state.order_data
    let integral_auto = _.get(order_data, 'zidong.INTEGRAL_INTEGRAL') || '0.00'
    return (
      <div onClick={this.toAutoMove} style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        /*height: font(188)*/
        padding:font(20)+' 0'
      }}>
        <span style={{fontSize: font(30), marginBottom: 9}}>积分自动转入<span style={{color: 'red'}}>(荐)</span></span>
        <span style={{color: Metrics.textLightColor, fontSize: font(26)}}>自动从益联益家总积分定时转入</span>
        {
          order_data.zidong != null ? <span style={{
            color: Metrics.textLightColor,
            fontSize: font(26),
            marginTop: 9
          }}>{moment(_.get(order_data, 'zidong.CREATE_TIME')).format('MM-D') + ' ' + integral_auto}积分 已转</span> : null
        }
      </div>
    )
  }

  pushPage = (navigator) => {
    navigator.pushPage({
      hasBackButton: true
    });
  }



  renderIntegralValueView = () => {
    let yidou = this.state.total_yidou == undefined || this.state.total_yidou == null ? 0 : this.state.total_yidou

    let datas = [
      {
        title: '积分余额',
        value: convert_unit(this.state.Assets.INTEGRAL) || 0,
        type: 'type_jifen'
      }, {
        title: '兑换指数',
        value: `${_.floor(this.state.AvailabilityNumber * 100, 4).toFixed(4)}%` || '0.0000%',
        type: 'exchange'
      }, {
        title: '益豆余额',
        value: convert_unit(yidou) || 0,
        type: 'type_yidou'
      },

    ]
    return (

      <div style={{
        width: '100%',
        height: '30vh',
        background: `url(${Images.sy_bg})`,
        backgroundSize: '100% 100%',
        paddingTop: '5vh'
      }}>
        <div style={{
          flexDirection: 'column',
          width: '100%',
          height: '70%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center'
        }}>
          <div style={{marginBottom: 10}}><span style={{color: 'white', fontSize: font(24)}}>昨日释放 (益豆)</span></div>
          <span style={{color: 'white', fontSize: font(60)}}>{this.state.assets_yesterday || 0}</span>
        </div>
        <div style={{
          display: 'flex',
          flexDirection: 'row',
          justifyContent: 'space-around',
          height: '30%',
          alignItems: 'center',
          backgroundColor: 'rgba(255,255,255,0.16)'
        }}>
          {
            datas.map((item, i) => {
              return (
                <div onClick={() => {
                  this.toBeanDetail(item)
                }} key={i} style={{display: 'flex', flexDirection: 'column',}}>
                  <span
                    style={{
                      color: '#ccc',
                      textAlign: 'center',
                      paddingBottom: 2,
                      fontSize: font(23)
                    }}>{item.title}</span>
                  <span
                    style={{color: 'white', textAlign: 'center', paddingTop: 5, fontSize: font(28)}}>{item.value}</span>
                </div>
              )
            })
          }
        </div>
      </div>
    )
  }
//中部积分管理
  renderIntegralManageView = () => {
    let { forbid, transfer_num, order_data } = this.state
    let str = !forbid ? `还可以转入${transfer_num || 0}次` : null
    let integral_shoudong = _.get(order_data, 'shoudong.INTEGRAL_INTEGRAL') || 0
    return (
      <div style={{
        borderRadius: 5,
        backgroundColor: 'white',
        color: "#333",
        padding: '0px 10px',
        width: '92vw',
        boxSizing: 'border-box',
        margin: '10px auto'
      }}>
        <div style={{display: 'flex', flexDirection: 'row', borderBottom: '1px solid #eee',padding:font(20)+' 0' /*height: font(90)*/}}>
          <div style={{display: 'flex', alignItems: 'center'}}>
            <img style={{width: font(39), marginRight: font(20.6), height: font(39)}} src={Images.icon_jifen}/>
            <span style={{fontSize: font(32)}}>积分管理</span>
          </div>
        </div>
        {this.show_limit_hint()}
        <div onClick={()=>this.toManualMove()} style={{
          display: 'flex',
          flexDirection: 'column',
          borderTop: '1px solid #E8E8E8',
          justifyContent: 'center',
          padding:font(20)+' 0'
        }}>
          <span style={{fontSize: font(30), marginBottom: 9}}>手动转入</span>
          <span style={{color: Metrics.textLightColor, fontSize: font(26)}}>手动从益联益家总积分转入{str}</span>
          {
            order_data != '' ? <span style={{
              fontSize: font(26),
              color: Metrics.textLightColor,
              marginTop: 9
            }}>{moment(_.get(order_data, 'shoudong.CREATE_TIME')).format('MM-D') + ' ' + integral_shoudong}积分 已转</span> : null
          }
        </div>
      </div>

    )
  }

  //控制弹窗显示/关闭
  XianehideDialog = () => {
    this.setState({XianeshowDialog: false})
  }
  XianeshowDialog = () => {
    this.setState({XianeshowDialog: true})
  }


//限额
  renderIntegralLimmitView = () => {
    let {virtualLimit} = this.state

    return (
      <div>
        <div style={{
          borderRadius: 5,
          backgroundColor: 'white',
          color: "#333",
          padding: '0px 10px',
          width: '92vw',
          boxSizing: 'border-box',
          margin: '10px auto'
        }}
             onClick={()=>this.showProtocol()}
        >

          <div style={{display: 'flex', flexDirection: 'row', borderBottom: '1px solid #eee', padding:font(20)+' 0'/*height: font(90)*/}}>
            <div style={{display: 'flex', alignItems: 'center'}}>
              <img style={{width: font(36), marginRight: font(22), height: font(36)}} src={Images.yidou_icon}/>
              <span style={{fontSize: font(32)}}>益豆宝转入限额</span>
            </div>
          </div>

          <div style={{display: 'flex', flexDirection: 'column', padding:font(20)+' 0'/*height: font(130)*/, justifyContent: 'center'}}>
            <div style={{
              display: 'flex',
              justifyContent: 'space-between',
              alignItems: 'center',
              marginBottom: 5
            }}>
              <span style={{color: 'red', marginBottom: 2, fontSize: font(30)}}>{virtualLimit||''}</span>
            </div>
            <span style={{color: Metrics.textLightColor, fontSize: font(26)}}>每日可转入益联积分</span>
          </div>
        </div>
      </div>
    )
  }

//底部按钮
  renderBottmBar = () => {
    let datas = [
      {
        title: '行情中心',
        type: 'redTab',
        text_color: '#333',
        backgroundColor: 'white'
      }, {
        title: '装备集市',
        type: 'greenTab',
        text_color: 'white',
        backgroundColor: '#2160e4'
      }
      // , {
      //   title: '新手练习',
      //   type: 'noobTab',
      //   text_color: '#333',
      //   backgroundColor: 'white'
      // }
    ]
    return (
      <div className={Styles.bottom_tab}
           style={{textAlign: 'center', fontSize: font(34), color: '#333', padding:font(20)+' 0'/*height: font(90),*/ ,boxSizing: 'border-box'}}>
        {
          datas.map((item, i) => {
            return (
              <div key={i} style={{width: 'calc(100% / 2)'}}
                   onClick={() => this.bottomButtonClick(item.type)}>{item.title}</div>
            )
          })
        }
      </div>
    )
  }


  renderPage = () => {
    return (
      <Ons.Page renderToolbar={() => <NavBar title="益豆宝" backButton={false}/>}>
        <div style={{
          position: 'absolute',
          background: '#f7f7f7',
          right: 0,
          left: 0,
          top: 0,
          bottom: 0,
          width: '100%',
          height: '100%',
        }}>
          {this.renderIntegralValueView()}
          {this.renderIntegralManageView()}
          {this.renderIntegralLimmitView()}
        </div>
        {this.renderBottmBar()}
      </Ons.Page>
    );
  }
//跳转函数块
  bottomButtonClick = (type) => {

    if (type === 'noobTab') {
      //console.log(type);
      this.props.dispatch(routerRedux.push({
        pathname: `/novices_login`,
        query: {
//此处传参
        },
      }))
    } else {
      setCurrentTabbarSign(type)
      this.props.dispatch(routerRedux.push({
        pathname: `/game`,
        query: {
//此处传参
        },
      }))
    }

  }
  callback_right = () => {
    //跳转到列表详情页
    this.props.dispatch(routerRedux.push({
      pathname: `/bean_detail`,
      query: {
        type: 'type_index_mx'
      },
    }))
  }
  toBeanDetail = (item) => {
    if (item.type == 'exchange') {
      return
    }
    this.props.dispatch(routerRedux.push({
      pathname: `/bean_detail`,
      query: {
        type: item.type
      },
    }))
  }
  //跳转至自动转入
  toAutoMove = () => {
    let limit_info = this.state.limit_info
    this.props.dispatch(routerRedux.push({
      pathname: `/auto_move_integral`,
      state: {
        limit_info: limit_info
      },
    }))
  }
  //跳转至手动
  toManualMove = () => {
    get_limit_info()
      .then(
        result => {
          let code = _.get(result, 'code')
          let limit_info = _.get(result, 'data')
          console.log('get_limit_info===========////////=======', limit_info)
          if (limit_info.transIntegral == 1) {
            if (limit_info.ENTER_NUMBER > 0) {
              if(code == 200){
                if (limit_info.is_no == 1) {
                  this.setState({forbid: false})
                  this.props.dispatch(routerRedux.push({
                    pathname: `/manual_move_integral`,
                    query: {},
                  }))
                } else {
                  this.setState({forbid: true})
                  show_info(Images.tanhao, `禁止转入时间段为${limit_info.ban_start_time}-${limit_info.ban_end_time}`)
                  return false
                }
              }else if(code == 401){
                show_info(Images.tanhao, _.get(result,'desc'))
              }

            } else {
              show_info(Images.tanhao, '已超过转入次数')
              return false
            }
          } else {
            show_info(Images.tanhao, '超出转入限额禁止转入')
            return false
          }
        }
      )
      .catch(
        error => {

        }
      )
  }


  /**
   * 益豆宝转入限额 点击提示弹窗开关
   */
  hideProtocol = () => {this.setState({protocolState: false})}
  showProtocol = () => {this.setState({protocolState: true})}

  /**
   * 益豆宝转入限额 点击提示弹窗
   * @returns {*}
   */
  rederUserProtocol = () => {
    return (<UserProtocol hideProtocol={this.hideProtocol} limit_info={this.state.xiane_dialog_content}></UserProtocol>)
  }
  renderTopNav = () => {
    return (
      <div style={{
        display: 'flex',
        position: 'fixed',
        height: font(67),
        background: 'transparent',
        justifyContent: 'center',
        left: 0,
        right: 0,
        top: 10,
        alignItems: 'center'
      }}>
        <div style={{width: '33.3%'}}></div>
        <h3
          style={{color: '#fff', fontSize: font(36), width: '33.3%', margin: 0, textAlign: 'center', fontWeight: 100}}>
          益豆宝</h3>
        <span style={{
          color: '#fff',
          fontSize: font(30),
          width: '33.3%',
          textAlign: 'right',
          paddingRight: '15px',
          boxSizing: 'border-box'
        }} onClick={this.callback_right}>明细</span>
      </div>
    )
  }

  render() {
    return (
      <div>
        <div
          style={{background: '#f7f7f7', position: 'absolute', bottom: 0, top: 0, left: 0, right: 0, overflow: 'scroll',width:'100%',height:'100%'}}>
          {this.renderTopNav()}
          {this.renderIntegralValueView()}
          {this.renderIntegralManageView()}
          {this.renderIntegralLimmitView()}
          {this.state.protocolState ? this.rederUserProtocol() : null}
          {this.renderBottmBar()}
        </div>
      </div>
    )
  }


}

YDBIntegralView.propTypes = {
  history: PropTypes.object
};
export default connect()(YDBIntegralView)

```

# iOS App 代码展示:

[预加载分页demo演示](http://qiaotongxin.cc/2017/08/06/20170807/)

# RN App 代码展示:

订单详情code:

```
import React,{Component} from 'react';
import {
    View,
    Text,
    StyleSheet,
    Image,
    FlatList,
} from 'react-native';
import {
    Container,
    Icon,
    Header,
    Footer,
    Content,
    Button,
    ListItem,
    Right,
} from 'native-base';
import {calll_server, jsImageURL} from '../../utils/Metrics';
import {api_get_user_insure_order} from '../../services/PhoneInsure';
import _ from 'lodash';


class InsureOrderDetailView extends Component {
    static navigationOptions = ({navigation}) => {
        return ({
            title: '投保手机详情',
        });
    }

    constructor(props) {
        super(props);
        let insure_order = this.props.navigation.state.params && this.props.navigation.state.params.insure_order;
        this.state = ({
            insure_order:insure_order,
        });
    }

    componentWillMount() {
        this.requestInsureDetail();
    }

    requestInsureDetail = () => {
        // let orderID = '59c22c50e1382371caa74f48';
        let orderID = this.state.insure_order.id;
        api_get_user_insure_order(orderID)
            .then((data) => {
                console.log('投保详情请求', data.insure_order);
                this.setState({
                    insure_order: data.insure_order,
                });
            })
            .catch((e) => {
            });;
    }

    renderItem = ({item}) => {
        let isWallet = false;
        if (item.hasOwnProperty('detailTitle')) {
            isWallet = true;
        }
        return (
            <ListItem>
                <Text>{item.title}</Text>
                <Text>{item.detailTitle}</Text>
            </ListItem>
        );
    }

    _renderOrderInfo = () => {
        const data = [{
            title: '订单号:',
            detailTitle:this.state.insure_order.number
        }, {
            title: '投保时间:',
            detailTitle:this.state.insure_order.created_at
        }]
        return (
            <FlatList
                data={data}
                renderItem={this.renderItem}
                keyExtractor={ (item, index) => index}
            />
        );
    }


    _renderOrderInsureInfo = () => {
            let xh = `设备型号 : ${this.state.insure_order.phone_category.name}`;
            let ys = `设备颜色 : ${this.state.insure_order.color}`;
            let fa = `手机投保 : ${this.state.insure_order.insure_price.name}`;
            let imei = `手机IMEI : ${this.state.insure_order.imei}`;

            let price = `¥ ${this.state.insure_order.insure_price.price}`;

            return (
                <View style={[styles.infos, {marginTop: 10}]}>
                    <Text style={[Metrics.lineStyle, styles.infos_text]}>{xh}</Text>
                    <Text style={[Metrics.lineStyle, styles.infos_text]}>{ys}</Text>
                    <View style={[Metrics.lineStyle, {flexDirection: 'row', justifyContent: 'space-between'}]}>
                        <Text style={[styles.infos_text]}>{fa}</Text>
                        <Text style={[styles.infos_text, {marginRight: 10, color: 'red'}]}>{price}</Text>
                    </View>
                    <Text style={[Metrics.lineStyle, styles.infos_text]}>{imei}</Text>
                </View>
            );
    }

    _renderOrderUserInfo = () => {
        const data = [{
            title: '受益人:',
            detailTitle:this.state.insure_order.beneficiary
        }, {
            title: '联系方式:',
            detailTitle:this.state.insure_order.beneficiary_mobile
        }, {
            title: '身份证号:',
            detailTitle:this.state.insure_order.beneficiary_idcard
        }]
        return (
            <FlatList
                data={data}
                renderItem={this.renderItem}
                keyExtractor={ (item, index) => index}
            />
        );
    }

    renderPhotoView = () => {
        return (
            <View style={{flexDirection: 'column',backgroundColor:'white',marginBottom:5,height:150}}>
                <Text style={styles.photoText}>手机照片</Text>
                <Image style={{width:70,height:70,marginLeft:15}} source={jsImageURL(this.state.insure_order.screen_image)}/>
                <View style={{flexDirection: 'row',paddingTop:15}}>
                    <View style={{alignItems: 'flex-start', paddingLeft: 10,paddingTop:5}}>
                        <Text style={{fontSize: 13, color: 'blue'}}
                              onPress={() => this.props.navigation.navigate('Help',
                                  {
                                      title: '机派服务协议',
                                      url: 'http://www.kuaiyiyuncai.cn/system/help/about/58ecbf7fccf959417a91be5d.html'
                                  })}>
                            {'《投保手机服务协议》'}
                        </Text>
                    </View>
                </View>
            </View>
        );
    }

    render() {
        if (_.isEmpty(this.state.insure_order)) return
        return (
            <Container>
                <Content style={styles.container}>
                    { this._renderOrderInfo() }
                    <View style={styles.sep}/>
                    { this._renderOrderInsureInfo()}
                    <View style={styles.sep}/>
                    { this._renderOrderUserInfo()}
                    { this.renderPhotoView()}
                </Content>
                {/*<Footer style = {styles.footers}>*/}
                    {/*<Button full style={{backgroundColor: Metrics.themeColor,width:Metrics.screenWidth}} onPress={ () => calll_server()}>*/}
                        {/*<Text>续保</Text>*/}
                    {/*</Button>*/}
                {/*</Footer>*/}
            </Container>
        );
    }
}

const styles = StyleSheet.create({
    sep: {
        backgroundColor: Metrics.backgroundColor,
        height: 10
    },

    photoText: {
        marginLeft: 15,
        marginTop: 10,
        marginBottom: 10,
        fontSize: 14,
        // color: Metrics.textColor
    },

    container: {
        flex: 1,
        backgroundColor: 'white',
        width: Metrics.screenWidth,
    },

    infos_text: {
        // height: 40,
        paddingTop:10,
        paddingBottom:10,
        // marginLeft: 10,
        // marginTop: 10,
        marginLeft: 10,
        alignItems: 'center',
        textAlignVertical: 'center',
        includeFontPadding: false
    },
});

export default InsureOrderDetailView;


```



