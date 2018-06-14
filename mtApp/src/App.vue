<template>
  <div id="app">
    <!-- 顶部 -->
    <app-header :poiInfo="poiInfo"></app-header>

    <!-- 导航部分 -->
    <app-nav :commentNum="commentNum"></app-nav>

    <keep-alive>
      <router-view></router-view> 
    </keep-alive>
    
  </div>
</template>

<script>
import Header from './components/header/Header'
import Nav from './components/nav/Nav'
export default {
  name: 'App',
  data(){
    return{
      poiInfo:{},
      commentNum:0
    }
  },
  components: {
      'app-header':Header,
      'app-nav':Nav
  },
  created(){
    fetch('api/goods').then(res => {
      return res.json()
    }).then(response => {
      // console.log(response);
      if(response.code === 0) this.poiInfo = response.data.poi_info
      console.log(this.poiInfo)
    }),
    
    fetch('api/ratings').then(res => {
      return res.json()
    }).then(response => {
       this.commentNum = response.data.comment_num
    })
  }
}
</script>

<style>

</style>
