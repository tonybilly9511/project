<template>
  <div class="cartcontrol">
    <transition name='move'>
      <div class="cart-decrease" @click.stop.prevent='decreaseCart' v-show='food.count'>
      <span class='inner icon-remove_circle_outline'></span>
      </div>
    </transition>
    <div class="cart-count" v-show='food.count'>{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent='addCart'>
      <i class='bg'></i>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'

export default{
  props:{
    food:{
      type: Object,
    }
  },
  computed:{},
  methods:{
    decreaseCart(){
      if(this.food.count<=0){
        return
      };
      this.food.count--;
    },
    addCart(){
      if(!this.food.count){
        Vue.set(this.food,'count',1)
      }else{
        this.food.count++;
      }
    }
  },
  component:{
  Vue
  }
}

</script>
<style scoped>

  .cartcontrol{
  font-size: 0;
  }
  .cartcontrol .cart-decrease{
    display:inline-block;
    width: 26px;
    height: 26px;
    font-size: 26px;
    color: #b4b4b4;
  }
  .cartcontrol .cart-count{
    display:inline-block;
    width: 25px;
    font-size: 15px;
    text-align: center;
    line-height: 26px;
    vertical-align: top;
  }
  .cartcontrol .cart-add{
    position: relative;
    display:inline-block;
    width: 26px;
    height: 26px;
    font-size: 26px;
    color: #ffd161;
  }
  .cartcontrol .cart-add .bg{
    position: absolute;
    left: 3px;
    top: 3px;
    z-index: -10;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #000;
  }

  .move-enter-active,.move-leave-active{
    transition: all 0.5s linear;
  }
  .move-enter,.move-leave-to{
    transform: translateX(28px) rotate(180deg)
  }



</style>
