<template>
  <div class="banner">
    <div class="banleft">
      <img src="@/static/sbclogo.png" align="Sailboat Companion logo" />
    </div>
    <div class="banright">
      <img src="@/static/hand.jpg" align="demo hand" />
    </div>
    <div class="clear" />
  </div>
</template>

<style lang="scss">
.clear {
  clear: both;
}
.banner {
  background: #e0e0e0;
  display: flex;
  align-items: center;
  //   height: 100%;
  //   width: 100%;
}
.banleft {
  /* background: #d0d0d0; */
  background: #e0e0e0;
  float: left;
  /*   height: 400px; */
  width: 50%;
  min-width: 200px;
  text-align: center;
}
.banleft img {
  width: 60%;
}
.banright {
  background: #e0e0e0;
  float: left;
  /*   height: 400px; */
  width: 50%;
  min-width: 200px;
  text-align: center;
}
.banright img {
  width: 60%;
}
</style>
