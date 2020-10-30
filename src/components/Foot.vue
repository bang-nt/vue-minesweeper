<template>
  <div class="footer">
    <div>
      <b-icon-flag></b-icon-flag>Minesweeper 1.2.1
      <br />From
      <a href="https://www.facebook.com/bang.sulfuric">B</a> With
      <b-icon-heart></b-icon-heart>
    </div>
    <div>
      Thanks to Internet for providing the excellent source code and tutorials
      <b-icon-emoji-smile></b-icon-emoji-smile>
    </div>
    <div class="copy-right-container" style="z-index: 2; position: relative;">
      <div class="line"></div>
      <div class="viewInfoFooter">
        <span>No Copyright &copy;</span>
      </div>
      <div class="privacy">
        <span>No Rights Reserved.</span>
      </div>
    </div>
  </div>
</template>
<style lang="scss" scoped>
$white-color: #fff;

.footer {
  width: 100%;
  padding: 20px;
  background-color: rgba(0, 0, 0);
  position: relative;
  border-top: 1px solid #000;
  color: rgb(80, 80, 80);
  text-shadow: 0 1px 0 rgba(0, 0, 0, 0.15);
  margin: 20px 0 0 0;
  box-sizing: border-box;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 14px;
  vertical-align: baseline;
  text-align: left;
}

.footer a {
  color: rgba(255, 255, 255, 0.5);
  border-bottom: dotted 1px #a5d8c78e;
  text-decoration: none;
  background-color: transparent;
  box-sizing: border-box;
  cursor: pointer;
}

.footer a:hover {
  color: $white-color;
  border-bottom: dotted 1px #a5d8c7;
}
.footer .copy-right-container .line {
  opacity: 0.6;
  height: 1px;
  width: 100%;
  margin: 1rem 0;
  background-color: $white-color !important;
}
.footer .copy-right-container .viewInfoFooter {
  display: inline-block;
  padding-right: 10px;
  font-size: 100%;
  vertical-align: baseline;
}
.footer .copy-right-container .privacy {
  padding-left: 10px;
  border-left: 1px solid $white-color;
  display: inline-block;
}
</style>
