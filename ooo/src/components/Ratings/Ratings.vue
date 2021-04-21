<template>
  <div class="ratings" ref='ratingView'>
    <div class="ratings-wrapper">
      <div class="overview">
        <div class="overview-left">
          <div class="comment-score">
            <p class='score'>{{ratings.comment_score}}</p>
            <p class='text'>商家評分</p>
          </div>
          <div class="other-score">
            <div class="quality-score item">
              <span class='text'>口味</span>
              <Star :score='ratings.quality_score' class='star'></Star>
              <span class='score'>{{ratings.quality_score}}</span>
            </div>
            <div class="pack-score item">
              <span class='text'>包裝</span>
              <Star :score='ratings.pack_score' class='star'></Star>
              <span class='score'>{{ratings.pack_score}}</span>
            </div>
          </div>
        </div>
        <div class="overview-right">
          <div class="delivery-score">
            <p class='score'>{{ratings.delivery_score}}</p>
            <p class='text'>配送評分</p>
          </div>
        </div>
      </div>
      <Split></Split>
      <div class="content">
        <div class="rating-select" v-if='ratings.tab'>
          <span class='item' @click='selectTypeFn(2)' :class='{"active":selectType==2}'>
            {{ratings.tab[0].comment_score_title}}
          </span>
          <span class='item' @click='selectTypeFn(1)' :class='{"active":selectType==1}'>
            {{ratings.tab[1].comment_score_title}}
          </span>
          <span class='item' @click='selectTypeFn(0)' :class='{"active":selectType==0}'>
            <img src="./icon_sub_tab_dp_normal@2x.png" alt="" v-show='selectType!=0'>
            <img src="./icon_sub_tab_dp_highlighted@2x.png" alt="" v-show='selectType==0'>
            {{ratings.tab[2].comment_score_title}}
          </span>
        </div>
        <div class="labels-view">
          <span class='item' v-for='item in ratings.labels' :class='{"highligh":item.label_star>0}'>
            {{item.content}}{{item.label_count}}
          </span>
        </div>


        <ul class="rating-list">
           <li v-for='comment in selectComments' class='comment-item'>
             <div class="comment-header">
               <img :src="comment.user_pic_url" alt="" v-if='comment.user_pic_url'>
               <img src="./icon_poi_review_anonymity@2x.png" v-if='!comment.user_pic_url' alt="">
             </div>
             <div class="comment-main">
               <div class="user">
                 {{comment.user_name}}
               </div>
               <div class="time">
                 {{fotmatDate(comment.comment_time)}}
               </div>
               <div class="star-wrapper">
                 <span class='text'>評分</span>
                 <Star :score='comment.order_comment_score' class='star'></Star>
               </div>
               <div class="c_content" v-html='commentStr(comment.comment)'>
               </div>
               <div class="img-wrapper" v-if='comment.comment_pics.length'>
                 <img v-for='item in comment.comment_pics' :src='item.thumbnail_url'>
               </div>
             </div>
           </li>
        </ul>


      </div>
    </div>
  </div>
</template>
<script>
  import Star from 'components/Star/Star'
  import Split from 'components/Split/Split'
  import BScroll from 'better-scroll'

  const ALL = 2;      //全部
  const PICTURE = 1;  //帶圖片
  const COMMENT = 0;  //點評

  export default{
    data(){
      return{
        ratings: {},
        selectType: ALL,

      }
    },
    created(){
      let that = this;
      this.$axios.get('/api/ratings')
        .then(function(response){
          var dataSource = response.data.data;
          if(dataSource.code == 0){
            that.ratings = dataSource.data;

            //初始化滾動
            that.$nextTick(()=>{
              if(!that.scroll){
                that.scroll = new BScroll(that.$refs.ratingView,{click:true});
              }else{
                that.scroll.refresh();
              }
            })

          }

        })
        .catch(function(error){
          console.log(error);
        })

    },
    methods:{
      selectTypeFn(type){
        this.selectType = type;

        //刷新操作
        this.$nextTick(()=>{
          this.scroll.refresh();
        })
      },
      fotmatDate(time){
        let date = new Date(time*1000);

        //時間格式
        let fmt = 'yyyy.MM.dd';

        if(/(y+)/.test(fmt)){
          let year = date.getFullYear().toString();
          fmt = fmt.replace(RegExp.$1,year);
        };
        if(/(M+)/.test(fmt)){
          let month = date.getMonth() + 1;
          if(month < 10){
            month = '0' + month;
          }
          fmt = fmt.replace(RegExp.$1,month);
        };
        if(/(d+)/.test(fmt)){
          let mydate = date.getDate();
          if(mydate < 10){
            mydate = '0' + mydate;
          }
          fmt = fmt.replace(RegExp.$1,mydate);
        }
        return fmt;
      },
      commentStr(content){
        let rel = /#[^#]+#/g;

        return content.replace(rel,'<i>$&</i>');
      }
    },
    computed:{
      selectComments(){
        if(this.selectType == ALL){
          return this.ratings.comments;
        }else if(this.selectType == PICTURE){
          let arr = [];
          this.ratings.comments.forEach((comment) => {
            if(comment.comment_pics.length){
              arr.push(comment)
            }
          });
          return arr;
        }else{
          return this.ratings.comments_dp.comments;
        }
      }
    },
    components:{
      Star,Split,BScroll
    }
  }

</script>
<style>
  .ratings{
  position: absolute;
  top: 191px;
  left: 0;
  bottom: 0;
  width: 100%;
  overflow: hidden;
  }
  .ratings .overview{
    display: flex;
    padding: 20px 0 18px 0;
  }
  .ratings .overview .overview-left{
    flex: 1;
    padding-left: 26px;
  }
  .ratings .overview .overview-left .comment-score{
    float: left;
    width: 48px;
    text-align: center;
    margin-right: 26px;
  }
  .ratings .overview .overview-left .comment-score .score{
    font-size: 23px;
    font-weight: 800;
    color: #ffb000;
    margin-bottom: 9px;
  }
  .ratings .overview .overview-left .comment-score .text{
    font-size: 11px;
    color: #666;
  }

  .ratings .overview .overview-left .other-score{
    float: left;
    margin-top: 3px;
  }
  .ratings .overview .overview-left .other-score .item{
    height: 11px;
  }
  .ratings .overview .overview-left .other-score .quality-score{
    margin-bottom: 14px;
  }
  .ratings .overview .overview-left .other-score .item .text{
    float:left;
    font-size: 11px;
    color: #666;
    margin-right: 11px;
  }
  .ratings .overview .overview-left .other-score .item .star{
    float:left;
    margin-right: 11px;
  }
  .ratings .overview .overview-left .other-score .item .score{
    float:left;
    font-size: 11px;
    color: #ffb000
  }


  .ratings .overview .overview-right{
    flex: 0 0 100px;
    text-align: center;
    border-left:  1px solid #9d9d9d;
  }
  .ratings .overview .overview-right .delivery-score{

  }
  .ratings .overview .overview-right .delivery-score .score{
    font-size: 19px;
    font-weight: 500;
    color: #999;
    margin-bottom: 10px;
    margin-top: 3px;
  }
  .ratings .overview .overview-right .delivery-score .text{
    font-size: 11px;
    color: #999;
  }

  .ratings .ratings-wrapper .content{
    padding: 16px;
  }
  .ratings .ratings-wrapper .content .rating-select{
    width: 100%;
    box-sizing: border-box;
    font-size: 0;
    border: 1px solid #ffb000;
    margin-bottom: 11px;
    border-radius: 3px;
    border-right: 0;
  }
  .ratings .ratings-wrapper .content .rating-select .item{
    display:inline-block;
    width: 33.3%;
    height: 33px;
    line-height: 33px;
    font-size: 14px;
    text-align: center;
    border-right: 1px solid #ffb000;
    box-sizing: border-box;
    color: #ffb000
  }
  .ratings .ratings-wrapper .content .rating-select .item.active{
    background: #ffb000;
    color: #000;
  }
  .ratings .ratings-wrapper .content .rating-select .item:last-child{
    border-right: 0;
  }
  .ratings .ratings-wrapper .content .rating-select .item img{
    height: 14px;
    vertical-align: middle;
  }
  .ratings .ratings-wrapper .content .labels-view{
    margin-bottom: 14px;
  }
  .ratings .ratings-wrapper .content .labels-view .item{
    display: inline-block;
    height: 27px;
    line-height: 27px;
    padding: 0 10px;
    font-size: 12px;
    background: #f4f4f4;
    margin-right: 6px;
    margin-bottom: 6px;
    border-radius: 3px;
    color: #999
  }
  .ratings .ratings-wrapper .content .labels-view .item.highligh{
    color: #656565;
  }
  .ratings .ratings-wrapper .content .rating-list{}
  .ratings .ratings-wrapper .content .rating-list .comment-item{
    display: flex;
    padding: 16px 16px 16px 0;
    border-bottom: 1px solid #f4f4f4;
    width: 100%;
    box-sizing: border-box;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-header{
    flex: 0 0 35px;
    margin-right: 11px;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-header img{
    width: 35px;
    height: 35px;
    border-radius: 50%;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main{
    flex: 1;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .user{
    float:left;
    width: 50%;
    font-size: 11px;
    color: #333
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .time{
    float:right;
    width: 50%;
    text-align: right;
    font-size: 9px;
    color: #666
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .star-wrapper{
    float: left;
    margin-top: 12px;
    margin-bottom: 15px;
    width: 100%;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .star-wrapper .text{
    float: left;
    color: #999;
    font-size: 11px;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .star-wrapper .star{
    float: left;
    margin-left: 7px;
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .c_content{
    float: left;
    font-size: 13px;
    line-height: 19px;
    width: 100%;
  }

  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .c_content i{
    color: #576b95
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .img-wrapper{
    float:left;
    margin-top: 9px
  }
  .ratings .ratings-wrapper .content .rating-list .comment-item .comment-main .img-wrapper img{
    width: 175px;
  }


</style>
