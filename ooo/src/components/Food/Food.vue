<template>
  <transition name='food-detail'>
    <div class="food" v-show='showFlag' ref='foodView'>
      <div class="food-wrapper" >
        <div class="food-content">
          <div class="img-wrapper">
            <img src="food.picture" class='food-img'>

            <span class='close-bt icon-close' @click='closeView'></span>
            <img src="./share.png" class='share-bt' alt="">
            <img src="./more.png" class='more-bt' alt="">
          </div>
          <div class="content-wrapper">
            <h3 class='name'>{{food.name}}</h3>
            <p class='saled'>{{food.month_saled_content}}</p>
            <img :src="food.product_label_picture" class='product' v-show='food.product_label_picture'>
            <p class='price'>
              <span class='text'>￥{{food.min_price}}</span>
              <span class='unit'>/{{food.unit}}</span>
            </p>
            <div class="cartcontrol-wrapper">
              <Cartcontrol :food='food'></Cartcontrol>
            </div>
            <div class="buy" v-show='!food.count || food.count==0' @click='addFirst()'>選規格</div>
          </div>
        </div>

        <Split></Split>

        <div class="rating-wrapper">
          <div class="rating-title">
            <div class="like-ratio" v-if='food.rating'>
              <span class='title'>{{food.rating.title}}</span>
              <span class=ratio>
                (
                  {{food.rating.like_ratio_desc}}
                <i>{{food.rating.like_ratio}}</i>
                )
              </span>
            </div>
            <div class="snd-title" v-if='food.rating'>
              <span class=text>{{food.rating.snd_title}}</span>
              <span class='icon icon-keyboard_arrow_right'></span>
            </div>
          </div>
          <ul class='rating-content' v-if='food.rating'>
            <li v-for='comment in food.rating.comment_list' class='rating-comment'>
              <div class="comment-header">
                <img :src="comment.user_icon" alt="" v-if='comment.user_icon'>
                <img src="./icon_poi_review_anonymity@2x.png" v-if='!comment.user_icon' alt="">
              </div>
              <div class="comment-main">
                <div class="user">
                  {{comment.user_name}}
                </div>
                <div class="time">
                  {{comment.comment_time}}
                </div>
                <div class="content">
                  {{comment.comment_content}}
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>


    </div>
  </transition>
</template>
<script>
import Cartcontrol from 'components/Cartcontrol/Cartcontrol'
import BScroll from 'better-scroll'
import Vue from 'vue'
import Split from 'components/Split/Split'

export default{
  data(){
    return{
      showFlag: false,
    }
  },
  props:{
    food: {
      type: Object
    }
  },
  methods:{
    //父組件可以調用子組件的方法
    showView(){
      this.showFlag = true;
      //初始化BScroll
      this.$nextTick(() => {
        if(!this.foodscroll){
          this.foodscroll = new BScroll(this.$refs.foodView,{click:true})
        }else{
          this.foodscroll.refresh();
        };
      })
    },
    closeView(){
      this.showFlag = false;
    },
    addFirst(){
      Vue.set(this.food,'count',1)
    }
  },
  components:{
    Cartcontrol,
    BScroll,
    Vue,
    Split
  }
}
</script>
<style >
  .food{
  z-index: 90;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  bottom: 51px;
  background: #fff;
}

/* 動畫 */
.food-detail-enter-active,.food-detail-leave-active{
  transition: 1s;
}
.food-detail-enter,.food-detail-leave-to{
  transform: translateX(100%) rotate(180deg);
}


/* 詳情 */

.food .food-wrapper{}
.food .food-wrapper .food-content{
  position: relative;
}
.food .food-wrapper .food-content .img-wrapper{
  position: relative;
  width: 100%;
  height: 0;
  /*高度如何撐開？
  在定位中，使用padding-top or padding-bottom設置為100%，
  其盒子高度會根據盒子的寬度進行計算
  */
  padding-top: 100%;
}
.food .food-wrapper .food-content .img-wrapper .food-img{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.food .food-wrapper .food-content .img-wrapper .close-bt{
  position: absolute;
  top: 10px;
  left: 10px;
  width: 30px;
  height: 30px;
  text-align: center;
  font-size: 30px;
  color: #fff;
  line-height: 30px;
  background: #7f7f7f;
  border-radius: 50%;
}
.food .food-wrapper .food-content .img-wrapper .share-bt,.food .food-wrapper .food-content .img-wrapper .more-bt{
  width: 30px;
  height: 30px;
  position: absolute;
  top: 10px;
}
.food .food-wrapper .food-content .img-wrapper .share-bt{
  right: 10px;
}
.food .food-wrapper .food-content .img-wrapper .more-bt{
  right: 50px;
}
.food .food-wrapper .food-content .content-wrapper{
  height: 75px;
  padding: 5px 0 24px 16px
}
.food .food-wrapper .food-content .content-wrapper .name{
  float: none;
  font-size: 15px;
  color: #333;
  line-height: 30px;
  font-weight: bold;
  padding:0;
}
.food .food-wrapper .food-content .content-wrapper .saled{
  font-size: 11px;
  color: #9d9d9d;
  margin-bottom: 6px;
}
.food .food-wrapper .food-content .content-wrapper .product{
  height: 15px;
  margin-bottom: 5px;
}
.food .food-wrapper .food-content .content-wrapper .price{
  font-size: 0;
}
.food .food-wrapper .food-content .content-wrapper .price .text{
  font-size: 20px;
  color: #fb4e44;
}
.food .food-wrapper .food-content .content-wrapper .price .unit{
  font-size: 11px;
  color: #9d9d9d
}
.food .food-wrapper .food-content .content-wrapper .cartcontrol-wrapper{
  position: absolute;
  right: 12px;
  bottom: 20px;
  padding: 2px;
}

.food .food-wrapper .food-content .content-wrapper .buy{
  position: absolute;
  right: 12px;
  bottom: 20px;
  width: 64px;
  height: 30px;
  font-size: 13px;
  line-height: 30px;
  text-align: center;
  background: #ffd161;
  border-radius: 30px;
}

.food .food-wrapper .rating-wrapper{
  padding-left: 16px;
}
.food .food-wrapper .rating-wrapper .rating-title{
  padding: 16px 16px 16px 0;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio{
  float: left;
  font-size: 0;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .title{
  font-size: 13px;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .ratio{
  font-size: 11px;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .ratio i{
  font-size: 11px;
  color: #fb4e44;
}
.food .food-wrapper .rating-wrapper .rating-title .snd-title{
  float: right;
  font-size: 0;
}
.food .food-wrapper .rating-wrapper .rating-title .snd-title .text{
  display:inline-block;
  font-size: 13px;
  color: #9d9d9d;
  height: 15px;
  line-height: 15px;
}
.food .food-wrapper .rating-wrapper .rating-title .snd-title .icon{
  display:inline-block;
  font-size: 15px;
  color: #9d9d9d;
  margin-left: 12px;
  vertical-align: top;
}
.food .food-wrapper .rating-wrapper .rating-content{

}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment{
  display: flex;
  padding: 15px 14px 17px 0;
  border-bottom: 1px solid #f4f4f4;
  box-sizing: border-box;
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-header{
  flex: 0 0 41px;
  margin-right: 10px;
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-header img{
  width: 41px;
  height: 41px;
  border-radius: 50%;
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-main{
  flex: 1;
  margin-top: 6px;
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-main .user{
  float: left;
  width: 50%;
  font-size: 12px;
  color: #333;
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-main .time{
  float: right;
  width: 50%;
  font-size: 10px;
  color: #666
}
.food .food-wrapper .rating-wrapper .rating-content .rating-comment .comment-main .content{
  /* padding-top: 10px; */
  margin-top: 17px;
  font-size: 13px;
  line-height: 19px;
}


</style>
