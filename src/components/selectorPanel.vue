<template>
  <div 
    v-if="show" 
    :class="data.optionList.length?'selector-panel':'selector-panel empty-panel'"
    :style="'width:'+targetElementWidth+'px;'+'max-height:'+data.panelHeight+'px;'+positionStyle"
    @click.stop="panelClick">
    <span class="panel-arrow"></span>
    <span class="panel-arrow-shadow"></span>
    <ul v-if="data.optionList.length" class="select-box">
      <li 
        v-for="option in data.optionList"
        :style="'background-color:' + option.color"
        class="selector-option"
        :title="option.label"
        :data-color="option.color"
        @click.stop="optionClick"
        :value="option.label">
        {{option.label}}
      </li>
    </ul>
    <div v-else class="empty-box">
      <div class="empty-content">{{emptyText}}</div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  export default{
    name: 'selector-panel',
    components: {},
    props: {
      show: {type:Boolean, default:false},
      data: {type:Object, default:{}},
      targetElementMark: {type:String, default:''},
      configData: {type:Object, default:{}}
    },
    computed: {

    },
    data(){
      return {
        positionStyle: '',
        position: {top:0, left:0, right:0, bottom:0},
        targetElementWidth: null,
        targetElementHeight: null,
        offsetParent: null,
        emptyText: '无匹配内容', // 
        selectedValue: '', // 选中项
      }
    },
    mounted(){
      let target = document.querySelector('[data-mark="'+this.targetElementMark+'"]');
      this.addEvent(document.getElementById('app'),'click',this.wrapperClickHandle);
      this.getOffsetParent(target);
      this.targetElementWidth = this.getTargetElementWidth(target);
      this.targetElementHeight = this.getTargetElementHeight(target);
      this.setPosition(this.position);
    },
    methods: {
      // 设置面板定位位置
      setPosition(position) {
        this.positionStyle = 'left:' + position.left + 'px;top:' + (position.top + this.targetElementHeight + 10) + 'px;';
      },
      // 获取定位的父元素
      getOffsetParent(target) {
        let className = target.offsetParent.getAttribute('class');
        if (className != this.configData.offsetParentClass) {
          target.offsetTop ? (this.position.top += target.offsetTop) : 0;
          target.offsetLeft ? (this.position.left += target.offsetLeft) : 0;
          target.offsetRight ? (this.position.right += target.offsetRight) : 0;
          target.offsetBottom ? (this.position.bottom += target.offsetBottom) : 0;
          this.getOffsetParent(target.offsetParent);
        } else {
          target.offsetTop ? (this.position.top += target.offsetTop) : 0;
          target.offsetLeft ? (this.position.left += target.offsetLeft) : 0;
          target.offsetRight ? (this.position.right += target.offsetRight) : 0;
          target.offsetBottom ? (this.position.bottom += target.offsetBottom) : 0;
          this.offsetParent = target.offsetParent;
        }
      },
      // 顶层元素点击事件
      wrapperClickHandle(event) {this.$emit('close', {targetElementMark: this.targetElementMark});},
      // 
      getTargetElementWidth(target) {return target.offsetWidth;},
      // 
      getTargetElementHeight(target) {return target.offsetHeight;},
      // 面板容器的点击事件 
      panelClick(event) {
        return false;
      },
      // 面板元素的点击事件
      optionClick(event) {
        let target = event.target || target.srcElement;
        let value = target.getAttribute('value');
        let color = target.getAttribute('data-color');
        this.$emit('optionClick', {value:value, targetElementMark:this.targetElementMark, targetColor:color});
      },
      // 添加事件
      addEvent(dom,eName,fun) {
        if (dom.addEventListener) {
          dom.removeEventListener(eName, fun);
          dom.addEventListener(eName, fun);
        } else if (x.attachEvent) {
          dom.detachEvent('on'+eName, fun);
          dom.attachEvent('on'+eName, fun);
        }
      },
    },
    destroyed(){},
    watch: {}
  }
</script>
<style>

</style>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  .empty-panel
    height 50px
    overflow visible!important
  .selector-panel
    position absolute
    background-color #fff
    overflow auto
    -webkit-border-radius 3px
    -moz-border-radius 3px
    -ms-border-radius 3px
    border-radius 3px
    -webkit-box-shadow 0 0 8px #ddd
    -moz-box-shadow 0 0 8px #ddd
    -ms-box-shadow 0 0 8px #ddd
    box-shadow 0 0 8px #ddd
    .select-box
      position relative
      background-color #fff
      z-index 2
      padding 5px
      li
        font-size 10px
        color #666
        height 20px
        line-height 20px
        text-align center
        cursor pointer
        overflow hidden
        text-overflow ellipsis
        white-space nowrap
        &:hover
          background-color #f1f1f1
    .empty-box
      width 100%
      height 100%
      position absolute
      top 0
      left 0
      background-color #fff
      z-index 2
      .empty-content
        position absolute
        top 50%
        left 50%
        user-select none
        -webkit-transform translate(-50%,-50%)
        -moz-transform translate(-50%,-50%)
        -ms-transform translate(-50%,-50%)
        transform translate(-50%,-50%)
        width 60px
        height 20px
        line-height 20px
        text-align center
        font-size 10px
        color #666
        background-color #fff
        z-index 3
    .panel-arrow
      width 0
      height 0
      border-width 0 8px 8px
      border-style solid
      border-color transparent transparent #fff
      position absolute
      top -6px
      left 50%
      z-index 2
      -webkit-transform translateX(-50%)
      -moz-transform translateX(-50%)
      -ms-transform translateX(-50%)
      transform translateX(-50%)
    .panel-arrow-shadow
      width 0
      height 0
      border-width 0 10px 10px
      border-style solid
      border-color transparent transparent #eee
      position absolute
      top -8px
      left 50%
      z-index 1
      -webkit-transform translateX(-50%)
      -moz-transform translateX(-50%)
      -ms-transform translateX(-50%)
      transform translateX(-50%)

</style>
