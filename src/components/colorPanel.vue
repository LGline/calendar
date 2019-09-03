<template>
  <div 
    v-if="show" 
    class="color-panel" 
    :style="configData.containerStyle + positionStyle"
    @click.stop="colorPanelClick">
    <span 
      v-for="color in colorList"
      class="color-item"
      @click.stop="colorItemClick"
      :data-color="color"
      :style="configData.itemStyle+'background-color:'+color+';'">
    </span>
  </div>
</template>
<script type="text/ecmascript-6">
  export default{
    name: 'color-panel',
    components: {},
    props: {
      show: {type:Boolean, default:false},
      colorList: {type:Array, default:[]},
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
      }
    },
    mounted(){
      let target = document.querySelector('[data-mark="'+this.targetElementMark+'"]');
      this.addEvent(document.getElementById('app'),'click',this.wrapperClickHandle);
      this.getOffsetParent(target);
      this.targetElementWidth = this.getTargetElementWidth(target);
      this.targetElementHeight = this.getTargetElementHeight(target);
      this.setPosition(this.position, target);
    },
    methods: {
      // 设置面板定位位置
      setPosition(position, target) {
        let noonNumber = target.getAttribute('data-mark').split('--')[1];
        let count = target.getAttribute('data-mark').split('--')[2];
        let week = this.getWeek(target);
        this.positionStyle = this.returnPositionStyle(noonNumber, count, position, this.targetElementWidth, this.targetElementHeight, week);
      },
      // 返回positionStyle
      returnPositionStyle(noonNumber, count, position, targetElementWidth, targetElementHeight, week) {
        let left = 0;
        if (week === '周六') {
          left = position.left + targetElementWidth - 50 - 15;
        } else {
          left = position.left + targetElementWidth;
        }
        return 'left:'+ left +'px;top:'+position.top+'px;';
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
      wrapperClickHandle(event) {this.$emit('close', {targetElementMark:this.targetElementMark});},
      // 
      getWeek(target) {return target.getAttribute('data-week');},
      // 
      getTargetElementWidth(target) {return target.offsetWidth;},
      // 
      getTargetElementHeight(target) {return target.offsetHeight;},
      // 面板容器的点击事件 
      colorPanelClick(event) {return false;},
      // 面板元素的点击事件
      colorItemClick(event) {
        let target = event.target || target.srcElement;
        let color = target.getAttribute('data-color');
        this.$emit('colorItemClick', {color:color, targetElementMark:this.targetElementMark});
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

</style>
