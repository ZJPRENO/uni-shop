<template>
  <view>
    <!-- 使用自定义搜索组件 -->
    <!-- <my-search :bgcolor="'pink'" :radius="9"></my-search> -->
    <my-search @click="gotoSearch"></my-search>
    <view class="scroll-view-container">
      <!-- 左侧滑动区域 -->
      <scroll-view class="left-scroll-view" scroll-y="true" :style="{height:windowH+'px'}">
        <block v-for="(item,index) in cateList" :key="index">
           <view :class="['left-scroll-view-item',index===active?'active':'']" @click="activeChange(index)">{{item.cat_name}}</view>
        </block>
        
      </scroll-view>
      <!-- 右侧滑动区域 -->
      <scroll-view scroll-y="true" :style="{height:windowH+'px'}" :scroll-top="scrollTop">
        
        <view class="cateList2" v-for="(item2,index2) in cateSubList" :key="index2">
          <!-- 二级分类标题 -->
          <view class="cateList2-title">/ {{item2.cat_name}} /</view>
          <!-- 二级分类的三级分类列表 -->
          <view class="cateList3-list">
            
            <view class="cateList3-item" v-for="(item3,index3) in item2.children" :key="index3" @click="gotoGoodsList(item3)">
              <image :src="item3.cat_icon"></image>
              <text>{{item3.cat_name}}</text>
            </view>
          </view>
        </view>
        
      </scroll-view>
    </view>
    
  </view>
</template>

<script>
  export default {
    data() {
      return {
        //当前设备可用的高度
        windowH:0,
        cateList:[],
        cateSubList:[],//二级分类
        active:0,
        scrollTop:0
      };
    },
    onLoad() {
      const sysInfo = uni.getSystemInfoSync()
      
        console.log(sysInfo)
        //获取屏幕可以高度 ,搜索高度50
        this.windowH = sysInfo.windowHeight - 50
         // 获取分类数据
        this.getCataList()
    },
    methods:{
      // 获取分类数据
      async getCataList(){
       const {data:res} = await uni.$http.get('/api/public/v1/categories')
       if(res.meta.status!=200) return uni.$showMsg()
       this.cateList = res.message
       this.cateSubList = res.message[0].children
        
      },
      activeChange(index){
        this.active = index
        this.cateSubList = this.cateList[index].children
        this.scrollTop = this.scrollTop===0?1:0
      },
      gotoGoodsList(item){//跳转商品列表页面
        uni.navigateTo({
          url:'/subpkg/goods_list/goods_list?cid='+item.cat_id
        })
      },
      gotoSearch(){
        uni.navigateTo({
          url:'/subpkg/search/search'
        })
        console.log('ddddddddd')
      }
    }
  }
</script>

<style lang="scss">
.scroll-view-container{
  display: flex;
  .left-scroll-view{
    width: 120px;
    .left-scroll-view-item{
      background-color: #f7f7f7;
      line-height: 60px;
      text-align: center;
      font-size: 12px;
      &.active{
        background-color: #FFFFFF;
        position: relative;
        
        &::before{
          content: '';
          display: block;
          width: 3px;
          height: 30px;
          background-color: #c00000;
          position: absolute;
          top: 50%;
          left: 0%;
          transform: translateY(-50%);
        }
      }
    }
  }
}
.cateList2-title{
  font-size: 12px;
  font-weight: bold;
  text-align: center;
  padding: 15px 0;
}
.cateList3-list{
  display: flex;
  flex-wrap: wrap;
  .cateList3-item{
    width: 33.33%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin-bottom: 10px;
    
    image{
      width: 60px;
      height:60px
    }
    text{
      font-size: 12px;
    }
  }
}
</style>
