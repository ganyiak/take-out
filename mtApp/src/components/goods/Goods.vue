<template>
  <div class="goods">
      <!-- 分类列表 -->
      <div class="menu-wrapper" ref="menuScroll">
        <ul>
          <!-- 专场 -->
          <li class="menu-item" :class="{'current':currentIndex === 0}" @click="selectMenu(0)">
            <p class="text">
              <img :src="container.tag_icon" alt="" v-if="container.tag_icon"  class="icon">
              {{container.tag_name}}
            </p>
          </li>

          <li class="menu-item" v-for="(item,index) in goods" :key="index" :class="{'current':currentIndex === index + 1}" @click="selectMenu(index + 1)">
            <p class="text">
              <img :src="item.icon" alt="" v-if="item.icon" class="icon">
              {{item.name}}
            </p>
            <span class="num" v-show="calculateCount(item.spus)">{{calculateCount(item.spus)}}</span>
          </li>
        </ul>
      </div>
      <!-- 商品列表 -->
      <div class="foods-wrapper" ref="foodsScroll">
        <ul>
          <!-- 专场 -->
          <li class="container-list food-list-hook">
            <div v-for="(item,index) in container.operation_source_list" :key="index">
              <img :src="item.pic_url" alt="">
            </div>
          </li>

          <!-- 具体分类 -->
          <li class="food-list food-list-hook" v-for="(item,index) in goods" :key="index">
            <h3 class="title">
              {{item.name}}

              </h3>

            <!-- 具体的商品列表 -->
            <ul>
              <li v-for="(item,index) in item.spus" :key="index" class="food-item" @click="showDetail(item)">
                <div class="icon" :style="head_bg(item.picture)"></div>
                <div class="content">
                  <h3 class="name">{{item.name}}</h3>
                  <p class="description" v-if="item.description">{{item.descrption}}</p>
                  <div class="extra">
                    <span class="saled">{{item.month_saled_conten}}</span>
                    <span class="praise">{{item.praise_content}}</span>
                  </div>
                  <img :src="item.product_label_picture" v-if="item.product_label_picture" alt="" class="product">
                  <p class="price">
                    <span class="text">${{item.min_price}}</span>
                    <span class="unit">/{{item.unit}}</span>
                  </p>
                </div>
                <div class="cartcontrol-wrapper">
                  <app-cart-control :item="item"></app-cart-control>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <!-- 购物车 -->
      <app-shopcart :poiInfo="poiInfo" :selectFoods="selectFoods"></app-shopcart>

      <!-- 商品详情 -->
      <app-product-detail :food="selectFood" ref="foodView"></app-product-detail>
  </div>
</template>

<script>
import BScroll from 'better-scroll'
import ShopCart from '../shopcart/ShopCart'
import CartControl from '../cartcontrol/CartControl'
import ProductDetail from '../productdetail/ProductDetail'
export default {
  data(){
    return {
      container:{},
      goods:[],
      poiInfo:{},
      listHeight:[],
      menuScroll:{},
      foodScroll:{},
      scrollY:0,
      selectFood:{}
    }
  },
  components:{
    'app-shopcart':ShopCart,
    'app-cart-control':CartControl,
    'app-product-detail':ProductDetail
  },
  //计算属性是不能接受参数的
  methods:{
    head_bg(url){
       return "background-image: url(" + url + ");"
    },
    initScroll(){
      //移动端无效点击事件无效的时候要传入一个参数，click：true
     this.menuScroll = new BScroll(this.$refs.menuScroll,{
       click:true
     })
     this.foodScroll = new BScroll(this.$refs.foodsScroll,{
       probeType:3,
       click:true
     })
      // console.log(new BScroll(this.$refs.foodsScroll))

      //foodScroll 监听事件
      this.foodScroll.on('scroll',(pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
        // console.log(this.scrollY)
        this.currentIndex
      })
    },
    calculateHeight(){
      //获取元素
      let foodList =  this.$refs.foodsScroll.getElementsByClassName('food-list-hook')
      // console.log(foodList)

      let height = 0
      this.listHeight.push(height)

      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        //累加
        height += item.clientHeight

        this.listHeight.push(height)
      }
      // console.log(this.listHeight)
    },
    selectMenu(index){
        console.log(index); 
        let foodList =  this.$refs.foodsScroll.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodScroll.scrollToElement(el,250)
        console.log(this.selectFoods)
        console.log(this.selectFoods.count)
      },
      calculateCount(food){
        let count = 0
        food.forEach((item,index) => {
          if(item.count > 0) count +=item.count
        })
        return count
      },
    showDetail(food){
      this.selectFood = food
      this.$refs.foodView.showView()
    }
  },
  created(){
    fetch('api/goods').then(res => {
      return res.json()
    }).then(response => {
      console.log(response);
      if(response.code === 0){
        this.container = response.data.container_operation_source
        this.goods = response.data.food_spu_tags
        this.poiInfo  = response.data.poi_info
        console.log(this.goods);
        //dom已经更新时调用
        this.$nextTick(() =>{
          this.initScroll()

        //计算分类区间的高度
        this.calculateHeight();
        //监听滚动的位置
        //根据滚动位置确认下表与左侧对应
        //通过下标实现点击左侧，滚动右侧
        })
      }
    })
    
  },
  computed:{
    currentIndex(){
      for(let i=0; i<this.listHeight.length; i++)
      {
        if(!this.listHeight[i+1] ||(this.scrollY >= this.listHeight[i] && this.scrollY < this.listHeight[i+1]))
        {
          return i
        }  
      }
       return 0
    },
    selectFoods(){
      let foods = [];
      this.goods.forEach((myFoods) => {
        myFoods.spus.forEach((food) =>{
          if(food.count > 0) foods.push(food)  
        })
      })
      return foods
    },
    totalNum(){

    },
    totalIndex(){
      let num = 0
        this.goods.forEach((myFoods) => {
        myFoods.spus.forEach((food) =>{
          if(food.count > 0)
          {
            num += food.count

          } 
        })
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.goods{
  display: flex;
  position: absolute;
  top: 190px;
  bottom: 51px;
  overflow: hidden;
  width: 100%;
}
.goods .menu-wrapper{
  flex: 0 0 85px;
  background-color: #f4f4f4;
}
.goods .foods-wrapper{
  flex: 1;
  /* background-color: blue; */
}
/* Menu item */ 
.goods .menu-wrapper .menu-item{
	padding: 16px 23px 15px 10px;
	border-bottom: 1px solid #E4E4E4;
  position: relative;
}

.goods .menu-wrapper .menu-item .text{
	font-size: 13px;
	color: #333333;
	line-height: 17px;
	vertical-align: middle;
	-webkit-line-clamp: 2;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}

.goods .menu-wrapper .menu-item .text .icon{
	width: 15px;
	height: 15px;
	vertical-align: middle;
}
/* 专场样式 */ 
.goods .foods-wrapper .container-list{
	padding: 11px 11px 0 11px;
	border-bottom: 1px solid #E4E4E4;
}

.goods .foods-wrapper .container-list img{
	width: 100%;
	margin-bottom: 11px;
	border-radius: 5px;
}

/* 具体分类商品布局 */ 
.goods .foods-wrapper .food-list{
	padding: 11px;
}
.goods .foods-wrapper .food-list .title{
	height: 13px;
	font-size: 13px;
	background: url(./img/btn_yellow_highlighted@2x.png) no-repeat left center;
	background-size: 2px 10px;
	padding-left: 7px;
	margin-bottom: 12px;
}




.goods .foods-wrapper .food-list .food-item{
	display: flex;
	margin-bottom: 25px;
  position: relative;
}

.goods .foods-wrapper .food-list .food-item  .icon{
	flex: 0 0 63px;
	background-position: center;
	background-size: 120% 100%;background-repeat: no-repeat;
	margin-right: 11px;
	height: 75px;
}
.goods .foods-wrapper .food-list .food-item .content{
	flex: 1;
}
/* 具体内容样式 */ 
.goods .foods-wrapper .food-list .food-item .content .name{
	font-size: 16px;
	line-height: 21px;
	color: #333333;
	margin-bottom: 10px;
	padding-right: 27px;
}

.goods .foods-wrapper .food-list .food-item .content .desc{
	font-size: 10px;
	line-height: 19px;
	color: #bfbfbf;
	margin-bottom: 8px;
	
	/* 超出部分显示省略号*/
	-webkit-line-clamp: 1;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}

.goods .foods-wrapper .food-list .food-item .content .extra{
	font-size: 10px;
	color: #BFBFBF;
	margin-bottom: 7px;
}
.goods .foods-wrapper .food-list .food-item .content .extra .saled{
	margin-right: 14px;
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
	color: #fb4e44;
}
.goods .foods-wrapper .food-list .food-item .content .price .unit{
	font-size: 12px;
	color: #BFBFBF;
}
/* // 选中时候的样式 */
.goods .menu-wrapper .menu-item.current{
	background: white;
	font-weight: bold;
	margin-top: -1px;
}

.goods .foods-wrapper .food-list .food-item .cartcontrol-wrapper{
	position: absolute;
	right: 0;
	bottom: 0;
}

.goods .menu-wrapper .menu-item .num{
	position: absolute;
	right: 5px;
	top: 5px;
	width: 13px;
	height: 13px;
	border-radius: 50%;
	color: white;
	background: red;
	text-align: center;
	font-size: 7px;
	line-height: 13px;
}
</style>
