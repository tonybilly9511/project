<template>
  <div class="goods">
    <!-- 分類 -->
    <div class="menu-wrapper" ref='menuScroll'>
      <ul>
        <!-- 專場 -->
        <li class='menu-item' :class='{"current":currentIndex===0}' @click='selectMenu(0)'>
          <p class='text'>
            <img src="container.tag_icon" v-if='container.tag_icon' class='icon'>
            {{container.tag_name}}
          </p>
        </li>

        <li class='menu-item' v-for='(item, index) in goods' :class='{"current":currentIndex===index+1}' @click='selectMenu(index+1)'>
          <p class='text'>
            <img src="item.icon" v-if='item.icon' class='icon'>
            {{item.name}}
          </p>
          <i class='num' v-show='calculateCount(item.spus)'>{{calculateCount(item.spus)}}</i>
        </li>
      </ul>
    </div>

    <!-- 商品列表 -->
    <div class="foods-wrapper" ref='foodScroll'>
      <ul>
        <!-- 專場 -->
        <li class='container-list food-list-hook'>
          <div v-for='item in container.operation_source_list'>
            <img :src="item.pic_url" alt="">
          </div>
        </li>

        <!-- 具體分類 -->
        <li v-for='item in goods' class='food-list food-list-hook'>
          <h3 class='title'>{{item.name}}</h3>
          <!-- 具體商品列表 -->
          <ul>
            <li v-for='food in item.spus' class='food-item' @click='showDetail(food)'>
              <div class="icon" :style='head_bg(food.picture)'></div>

              <div class="content">
                <h3 class='name'>{{food.name}}</h3>
                <p class='desc' v-if='food.description'>{{food.description}}</p>
                <div class="extra">
                  <span class='saled'>{{food.month_saled_content}}</span>
                  <span class='praise'>{{food.praise_content}}</span>
                </div>
                <img :src="food.product_label_picture" class='product'/>
                <p class='price'>
                  <span class='text'>￥{{food.min_price}}</span>
                  <span>/{{food.unit}}</span>
               </p>
               <div class="cartcontrol-wrapper">
                 <Cartcontrol :food='food'></Cartcontrol>
               </div>
              </div>
            </li>
          </ul>

        </li>
      </ul>
    </div>

    <!-- 購物車 -->
    <Shopcart :poiInfo='poiInfo' :selectFoods='selectFoods'></Shopcart>

    <!-- 商品詳情 -->
    <Food :food='selectFood' ref='foodView'></Food>
  </div>
</template>
<script>
  //導入BScroll
  import BScroll from 'better-scroll'
  //導入Shopcart
  import Shopcart from 'components/Shopcart/Shopcart'
  import Cartcontrol from 'components/Cartcontrol/Cartcontrol'
  import Food from 'components/Food/Food'

  export default{
    data(){
      return{
        container:{},
        goods:[],
        poiInfo:{},
        listHeight:[],
        scrollY: 0,
        menuScroll:[],
        foodScroll:[],
        selectFood:{}
      }
    },
    created(){
      var that = this;

      this.$axios.get('/api/goods')
        .then(function(response){
          var dataSource = response.data.data;
          if(dataSource.code == 0){
            that.container = dataSource.data.container_operation_source;
            that.goods = dataSource.data.food_spu_tags;
            that.poiInfo = dataSource.data.poi_info;

            //調用滾動的初始化方法
            //that.initScroll();
            //開始時，DOM元素沒有渲染，即高度是有問題
            //在獲取數據後，並DOM已經被渲染，表示列表高度是沒問題

            //nextTick
            that.$nextTick( ()=>{
              that.initiScroll();

              //計算分類區間高度
              that.calculateHeight();
            });

          }
        })
        .catch(function(error){
          console.log(error);
        });

    },
    methods:{
      head_bg(imgName){
        return 'background-image: url(' + imgName + ');';
      },
      //滑鼠滾動初始化
      initiScroll(){
        //ref屬性是用來綁定某個dom元素或某個組建，在this.$refs裡面
        this.menuScroll = new BScroll(this.$refs.menuScroll,{click:true});
        this.foodScroll = new BScroll(this.$refs.foodScroll,{probeType:3,click:true});


        //添加滾動監聽事件
        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        })
      },
      calculateHeight(){
        //通過$refs獲取對應的元素
        let foodlist = this.$refs.foodScroll.getElementsByClassName('food-list-hook');

        //添加到數組區間
        //0~242 第一個區間（專場）
        //243~1427 第二個區間（熱銷）
        let height = 0;
        this.listHeight.push(height);
        for(let i=0; i<foodlist.length; i++){
          let item = foodlist[i];

          //累加
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },
      selectMenu(index){
        // console.log(index);

        let foodlist = this.$refs.foodScroll.getElementsByClassName('food-list-hook');

        //根據下標，滾動到相對應的元素
        let el = foodlist[index];
        this.foodScroll.scrollToElement(el,300);
      },
      calculateCount(spus){
        let count = 0;
        spus.forEach((food) => {
          if(food.count>0){
            count += food.count;
          }
        });
        return count;
      },
      showDetail(food){
        //傳值
        this.selectFood = food;
        //顯示詳情
        this.$refs.foodView.showView();
      }
    },
    computed:{    //不能傳參數
      currentIndex(){ //根據右側滾動位置，確定對應的索引下標
        for(let i=0; i<this.listHeight.length; i++){
          //獲取商品區間範圍
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i+1];

          //是否在上述區間中
          if(this.scrollY>=height1 && this.scrollY<height2){
            return i
          }
        }
          return 0;
      },
      selectFoods(){
        let foods = [];

        this.goods.forEach((good) => {
          good.spus.forEach((food) => {
            if(food.count>0){
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    components:{
      BScroll,Shopcart,Cartcontrol,Food
    }

  }
</script>
<style>
  .goods{
    position: absolute;
    top:192px;
    bottom:51px;
    display: flex;
    overflow: hidden;
    width: 100%;
  }
  .goods .menu-wrapper{
    background: #f4f4f4;
    flex: 0 0 85px;

  }
  .goods .menu-wrapper ul{
    list-style: none;
  }
  .goods .menu-wrapper .menu-item{
    position: relative;
    padding: 16px 23px 15px 10px;
    border-bottom: 1px solid #e4e4e4;
  }
  .goods .menu-wrapper .menu-item.current{
    background: #fff;
    font-weight: bold;
    margin-top: -1px;
  }
  .goods .menu-wrapper .menu-item .text{
    font-size: 13px;
    color: #333;
    line-height: 17px;
    vertical-align: middle;

    /* 用來顯示一個快元素顯示文本的行數 */
    -webkit-line-clamp: 2;
    /* 必須，將對象作為彈性伸縮盒子模型顯示 */
    display: -webkit-box;
    /* 必須，設置或檢索伸縮盒子的子元素排列方式 */
    -webkit-box-orient: vertical;

    overflow: hidden;
  }
  .goods .menu-wrapper .menu-item .icon{
    width: 15px;
    height: 15px;
  }

  .goods .menu-wrapper .menu-item .num{
    position: absolute;
    top: 5px;
    right: 5px;
    width: 13px;
    height: 13px;
    border-radius: 50%;
    color: #fff;
    background: #f00;
    text-align: center;
    font-size: 7px;
    line-height: 13px;
  }





  /* 商品列表 */
  .goods .foods-wrapper{
    /* background: blue; */
    flex: 1;
  }

  .goods .foods-wrapper .container-list{
    padding: 11px 11px 0 11px;
    border-bottom: 1px solid #e4e4e4
  }
  .goods .foods-wrapper .container-list img{
    width: 100%;
    margin-bottom: 11px;
    border-radius: 5px
  }
  .goods .foods-wrapper .food-list{
    padding: 11px
  }
  .goods .foods-wrapper .food-list .title{
    height: 13px;
    font-size: 13px;
    background: url(btn_yellow_highlighted@2x.png) no-repeat left center;
    background-size: 2px 10px;
    padding-left: 7px;
    margin-bottom: 12px;
  }
  .goods .foods-wrapper .food-list .food-item{
    position: relative;
    display: flex;
    margin-bottom: 25px;
  }
  .goods .foods-wrapper .food-list .food-item .icon{
    flex: 0 0 63px;
    height: 80px;
    background-position: center;
    background-size: 120% 100%;
    background-repeat: no-repeat;
    margin-right: 11px;
  }

  .goods .foods-wrapper .food-list .food-item .content{
    flex: 1;
  }
  .goods .foods-wrapper .food-list .food-item .content .name{
    font-size: 16px;
    line-height: 21px;
    color: #333;
    margin-bottom: 10px;
    padding-right: 27px;
    font-weight: bold;
  }
  .goods .foods-wrapper .food-list .food-item .content .desc{
    font-size: 10px;
    line-height: 19px;
    color: #bfbfbf;
    margin-bottom: 8px;

    -webkit-line-clamp: 1;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
  .goods .foods-wrapper .food-list .food-item .content .extra{
    font-size: 10px;
    color: #bfbfbf;
    margin-bottom: 7px;
  }
  .goods .foods-wrapper .food-list .food-item .content .extra .saled{
    margin-right: 14px;
  }
  .goods .foods-wrapper .food-list .food-item .content .extra .praise{

  }
  .goods .foods-wrapper .food-list .food-item .content .product{
    height: 15px;
    margin-bottom: 6px;

  }
  .goods .foods-wrapper .food-list .food-item .content .price{
    font-size: 0;
  }
  .goods .foods-wrapper .food-list .food-item .content .price .text{
    font-size: 14px;
    color: #fb4e44
  }
  .goods .foods-wrapper .food-list .food-item .content .price span{
    font-size: 12px;
    color: #bfbfbf
  }
  .goods .foods-wrapper .food-list .food-item .content .cartcontrol-wrapper{
    position: absolute;
    right: 0;
    bottom: 0;
  }

</style>
