<template>
<div class="tabbar-wrap">
    <!-- banner -->
    <c-top-back :mar='true'></c-top-back>
    <div v-if="dataReady">
        <c-product-wrap type="normal">
          <c-product :title="goods.name"  :des="logistics_fee1=='0.00'?'包邮':'运费：¥'+logistics_fee1" :price="goods.price" :pic="goods.pics"></c-product>
        </c-product-wrap>
       <!-- <c-cell-wrap>
            <c-cell :title="storeInfo.name" value="进入店铺" is-link>
                <img class="wq-img" slot="icon" src='http://static.taggole.com/sithbrobot/poster/1489997146660.jpg'/>
            </c-cell>
        </c-cell-wrap>-->

        <c-title title="商品详情"></c-title>
        <div class="wrap-detail">
            <p>{{goods.product_detail}}</p>
            <p v-for='i in goods.product_desc'><img v-bind:src="i" alt="" style="width:100%; height:auto;"></p>
        </div>

        <c-sku v-model="sku.status" fillable @submit="submit" :title="goods.name" :img="goods.pics" :price="goods.price" :text="sku.buttonTxt" :sku='skuList' ></c-sku>
        <c-button text="立即购买" size="block" @click.native="barClick" type="primary" style="position:fixed; bottom:0"></c-button>
    </div>

</div>

</template>
<script>
    import utils from '../../libs/utils.js';
    import constants from '../../libs/constants.js';
    export default {
        data (){
            return {
                dataReady:false,
                goods:{},//商品信息
                skuList:[],//商品sku列表
                storeInfo:{name:'测试'},//店铺信息
                content:{},//商品详细信息
                goods_id:0,//商品id
                sku:{
                    status:false,
                    url:basepath + "/mall/store/shelf_product_sku_get",
                    buttonTxt:"下一步",
                },
                submitType:"buy",
                hasCart:true,//是否显示购物车
                limitNum:'',
                rule_flag:'',
                pageId:'',
                invoiceTypes:[],
                logistic_flg:1,
                logistics_fee1:0

            }
        },
        mounted: function () {
            this.$nextTick(function () {
                this.$vux.alert.hide();
                //utils.MenuShare();    
                let that = this;
                utils.ajax({
                    url: basepath + "/seller/product/get",
                    async:false,
                    data:{"id":that.$route.query.product_id},
                    success: function(data){
                        if(data.code=='SUCESS'){
                            that.goods=data.result;
                            that.goods.logistics_fee=that.goods.logistics_fee;
                            that.logistics_fee1=utils.formatPrice(that.goods.logistics_fee);
                            that.goods.product_desc=JSON.parse(that.goods.product_desc);
                            utils.ajax({
                                url: basepath + "/seller/product/sku/get",
                                async:false,
                                data:{"product_id":data.result.id},
                                success: function(res){
                                    if(res.code=='SUCESS'){
                                        var max=res.result[0].price;
                                        var min=res.result[0].price;
                                        var len=res.result.length;

                                        for(var ii=1; ii<len; ii++){
                                          if(res.result[ii].price > max){
                                            max = res.result[ii].price;
                                          }
                                        }

                                        for(var i=1; i<len; i++){
                                          if(res.result[i].price < min){
                                            min = res.result[i].price;
                                          }
                                        }
                                        if(min!=max){
                                            that.goods.price='¥'+utils.formatPrice(min)+' ~ ¥'+utils.formatPrice(max);
                                        }else{
                                            that.goods.price='¥'+utils.formatPrice(min);
                                        }
                                        $.each(res.result,function(i,v){
                                            that.skuList.push({
                                                id:v.id,
                                                name:v.sku_str,
                                                price:utils.formatPrice(v.price),
                                                stock:v.stock
                                            })
                                        })
                                        

                                    }else{
                                        that.$vux.alert.show({content:res.message,onHide :function(){
                                                            return false
                                                          }});
                                    }
                                }
                                
                            });

                            that.dataReady = true;
                        }else if(data.code=='auth_seller_error'){
                                utils.wang(that,utils,data.message);

                        }else{
                            that.$vux.alert.show({content:data.message,onHide :function(){
                                                            return false
                                                          }});
                        }
                    }
                    
                    
                });

                // utils.ajax({
                //   url: "/mall/wxconfig/get",
                //   data:{'url':window.location.href}),
                //   success: function(data) {
                //     if(data.success) {
                //       wx.config({
                //           debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                //           appId: data.obj.appId, // 必填，公众号的唯一标识
                //           timestamp: data.obj.timestamp, // 必填，生成签名的时间戳
                //           nonceStr: data.obj.nonceStr, // 必填，生成签名的随机串
                //           signature: data.obj.signature,// 必填，签名，见附录1
                //           jsApiList: ['onMenuShareAppMessage','onMenuShareTimeline'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
                //       });
                //       wx.ready(function(){
                //         wx.onMenuShareAppMessage({
                //             title: that.goods.title, // 分享标题
                //             desc: '【'+that.storeInfo.name+'】发现好商品，立即分享给你，进店有惊喜呦。', // 分享描述
                //             link: window.location.href+'?xv=enter', // 分享链接
                //             imgUrl: that.goods.pic // 分享图标
                //         });

                //         wx.onMenuShareTimeline({
                //             title: that.goods.title, // 分享标题
                //             link: window.location.href+'?xv=enter', // 分享链接
                //             imgUrl: that.goods.pic // 分享图标
                //         });
                //       })
                //     } else {
                //       that.$vux.alert.show(data.message);
                //     }
                //   },
                // });

                // if(utils.getSession('pageId')){
                //     that.pageId=utils.getSession('pageId');
                // }else{
                //     utils.ajax({
                //         url: basepath + "/mall/store/get",
                //         dataType: 'json',
                //         type: 'POST',
                //         success: function(data){
                //             if(data.success){
                //                 that.pageId=data.obj.page_id;
                //             }
                //         }
                //     });
                // }
            })
        },
        methods:{

            barClick(type){
                this.sku.status = true;
                this.submitType = "buy";
            },
            submit(){
                this.$vux.alert.show({content:'无法购买自己店铺的商品。',onHide :function(){
                                            return false
                                          }});
            }
            
        },
        components:{
            "cProduct": require('../../components/x-product/x-product.vue'),
            "cProductWrap": require('../../components/x-product/x-product-wrap.vue'),
            "cTopBack":require('../../components/x-top-back/x-top-back.vue'),
            "cCellWrap":require('../../components/cell/cell-wrap.vue'),
            "cCell":require('../../components/cell/cell.vue'),
            "cButton":require('../../components/button/button.vue'),
            "cTitle":require('../../components/x-title/x-title.vue'),
            "cPayBar1":require('../../components/x-pay-bar/x-pay-bar1.vue'),
            "cPayBar1":require('../../components/x-pay-bar/x-pay-bar2.vue'),
            "cSku":require('../../components/x-sku/x-sku.vue')
        }
    }
</script>
<style>
    .tabbar-wrap .wq-img{
        width:20px; margin-right:10px;
    }
    .tabbar-wrap .weui_cell_ft.with_arrow {
        margin-right:0 !important;
    }
    .tabbar-wrap .weui_cell_ft.with_arrow:after{
        right:0 !important;
    }
</style>
