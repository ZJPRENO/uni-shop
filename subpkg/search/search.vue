<template>
  <view>
    
    <view class="search-box">
      <uni-search-bar @confirm="search" @input="input" :radius="100" cancelButton="none"></uni-search-bar>
    </view>
    
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
     <!-- 搜索历史 -->
     <view class="history-box" v-else>
       <!-- 标题区域 -->
       <view class="history-title">
         <text>搜索历史</text>
         <uni-icons type="trash" size="17" @click="clean"></uni-icons>
       </view>
       <!-- 列表区域 -->
       <view class="history-list">
         <uni-tag :text="item" v-for="(item, i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
       </view>
     </view>
     
    
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer:null,//定时器
        kw:'',//搜索关键字
        // 搜索的结果列表
        searchResults: [],
        // 搜索历史的数组
        historyList: []
        
      };
    },
    onLoad() {
      // 获取存储的数据，如果没有就赋值默认数组【】
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      
    input(e){
      clearTimeout(this.timer)
       
       this.timer =  setTimeout(()=>{
         //如果500毫秒内没有触发新的输入事件，则为搜索关键字赋值
        this.kw = e
        console.log(this.kw)
        this.getSearchList() 
        },500)
    },
    async getSearchList(){
      // 判断搜索关键词是否为空
      if (this.kw.length === 0) {
        this.searchResults = []
        return
      }
    const {data:res} = await uni.$http.get('/api/public/v1/goods/qsearch',{ query:this.kw })
    if(res.meta.status!==200) return uni.$showMsg()
    this.searchResults = res.message
    
    this.saveSearchHistory()
      
    },
    gotoDetail(item) {
      uni.navigateTo({
        // 跳转到指定详情页面URL地址并传递 gods_id地址参数
        url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
      })
    },
    saveSearchHistory() {
      // this.historyList.push(this.kw)
      //1.将array转化为set对象
      const set = new Set(this.historyList)
      // 2.调用set对象的delete方法，删除元素
      set.delete(this.kw)
      // 3.调用set对象的add方法，添加元素
      set.add(this.kw)
     // 4.将set对象转换为array数组
      this.historyList = Array.from(set)
      // 对搜索历史数据，进行持久化的存储
      // uni.setStorageSync(key,value）,JSON.stringify转成字符串
      uni.setStorageSync('kw', JSON.stringify(this.historyList))
    },
    clean() {
      this.historyList = []
      uni.setStorageSync('kw', '[]')
    },
    gotoGoodsList(kw) {
      uni.navigateTo({
        url: '/subpkg/goods_list/goods_list?query=' + kw
      })
    }
    
    },
    computed: {
      histories() {
        return [...this.historyList].reverse()
      }
    }
  }
</script>

<style lang="scss">
.search-box {
  position: sticky;
  top: 0;
  z-index: 999;
}
.sugg-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;

      .goods-name {
        //文字不允许换行
        white-space: nowrap;
        //超出部分隐藏
        overflow: hidden;
        // 文字超出部分以...代替
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
    .history-box {
      padding: 0 5px;
    
      .history-title {
        display: flex;
        justify-content: space-between;
        height: 40px;
        align-items: center;
        font-size: 13px;
        border-bottom: 1px solid #efefef;
      }
    
      .history-list {
        display: flex;
        flex-wrap: wrap;
    
        .uni-tag {
          margin-top: 5px;
          margin-right: 5px;
          background-color: #dddddd;
          color: black;
          border: #efefef;
          font-size: 13px;
        }
      }
    }

</style>
