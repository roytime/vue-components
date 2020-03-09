<template>
  <div id="app">
    <div id="nav">
      <ul>
        <li><router-link to="/">Home</router-link></li>
        <li><router-link to="/render">render函数制作的无限级别树</router-link></li>
      </ul> 
    </div>
    <router-view id="view"/>
  </div>
</template>

<style lang="stylus">
#app
  font-family Avenir, Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  text-align center
  color #2c3e50
  margin-top 0px
  text-align left
  display flex
  justify-items start
  font-size 16px

#nav
  width 30%
  padding-top 60px

#view
  padding 30px

input
  width 100px
  border 0px
  border-bottom 1px solid #42B983
  outline none
  background-color transparent
  line-height 24px
  font-size 16px

</style>
