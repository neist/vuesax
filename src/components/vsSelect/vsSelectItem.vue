<template lang="html">
  <li
    v-show="visible"
    class="vs-component">
    <button
      ref="item"
      :disabled="disabled"
      v-bind="$attrs"
      :style="styles"
      :class="{
        'activex':$parent.vsMultiple?getValue.indexOf(vsValue) != -1:getValue == vsValue,
        'con-icon':$parent.vsMultiple,
        'disabledx':disabledx
      }"
      class="vs-select-item-btn"
      type="button"
      name="button"
      v-on="listeners"
      @keydown.backspace.prevent="backspace"
      @keydown.down.prevent="navigateOptions('next')"
      @keydown.up.prevent="navigateOptions('prev')"
      @keydown.enter.prevent="clickOption()">
      <i
        v-if="$parent.vsMultiple"
        class="material-icons icon-item">
        check_circle
      </i>
      <span
        v-html="getText"></span>
    </button>
  </li>
</template>

<script>
import _color from '../../utils/color.js'
export default {
  name:'VsSelectItem',
  inheritAttrs:false,
  props:{
    disabled:{
      type:Boolean,
      default:false
    },
    vsValue:{
      default:null,
    },
    vsText:{
      default:null,
    }
  },
  data:()=>({
    hoverx:false,
    visible:true,
    getText:null,
    valueInputx:''
  }),
  computed:{
    disabledx(){
      if(this.$parent.vsMultiple){
        if(this.isActive){
          return false
        } else {
          return this.$parent.vsMaxSelected == this.$parent.value.length
        }
      } else {
        return false
      }
    },
    isActive(){
      return this.$parent.vsMultiple?this.getValue.indexOf(this.vsValue) != -1:this.getValue == this.vsValue
    },
    listeners() {
      return {
        ...this.$listeners,
        blur: (event) => {
          if(event.relatedTarget?!event.relatedTarget.closest('.vs-select-options'):true) {
            this.$parent.closeOptions()
          }
        },
        click: (event) => this.clickOption(event),
        mouseover: (event) => {
          this.$emit('mouseover',event)
          this.changeHover(true)
        },
        mouseout: (event) => {
          this.$emit('mouseover',event)
          this.changeHover(false)
        }
      }
    },
    styles(){
      return {
        background: this.isActive?_color.getColor(this.$parent.vsColor,.1):null,
        color: this.isActive?_color.getColor(this.$parent.vsColor,1):null,
        fontWeight: this.isActive?'bold':null
      }
    },
    getValue(){
      return this.$parent.value
    }
  },
  watch:{
    '$parent.active': function () {
      this.$nextTick(() => {
        if( this.$parent.vsMultiple?this.getValue.indexOf(this.vsValue) != -1:this.getValue == this.vsValue ) {
          this.$emit('update:isSelected', true)
          this.getText = this.vsText
          this.putValue()
        } else {
          this.$emit('update:isSelected', false)
          this.getText = this.vsText
          this.putValue()
        }
      })
    },
    valueInputx(){
      if(this.visible){
        let valueInputx = this.valueInputx.split(',')
        if(valueInputx[valueInputx.length-1] == ''){
          this.getText = this.vsText
          return
        }
        let valuex = valueInputx[valueInputx.length-1]
        var re = new RegExp(valuex,"i");
        if(this.vsText.toUpperCase().indexOf(valuex.toUpperCase()) == 0){
          valuex = this.MaysPrimera(valuex)
        }
        let text = this.vsText.replace(re,`<span class="searchx">${valuex}</span>`)
        this.getText = text
      } else {
        this.getText = this.vsText
      }
    }
  },
  created(){
    this.putValue()
    this.$nextTick(() => {
      if( this.$parent.vsMultiple?this.getValue.indexOf(this.vsValue) != -1:this.getValue == this.vsValue ) {
        this.$emit('update:isSelected', true)
        this.getText = this.vsText
        this.putValue()
      } else {
        this.$emit('update:isSelected', false)
        this.getText = this.vsText
        this.putValue()
      }
    })
  },
  updated(){
    this.putValue()
  },
  methods:{
    changeHover(booleanx){
      this.hoverx = booleanx
    },
    MaysPrimera(string){
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    backspace(){
      if(this.$parent.vsAutocomplete){
        let valueInput = this.$parent.$refs.inputselect.value
        this.$parent.$refs.inputselect.value = valueInput.substr(0,valueInput.length-1)
        this.$parent.$refs.inputselect.focus()
      }
    },
    navigateOptions(orientation){
      let
        orientationObject = 'nextSibling',
        lengthx = 0

      function getNextLi(li,orientationObject){
        if(li){
          let lix = li[orientationObject]
          if(li.style){
            if(li.style.display == 'none'){
              return getNextLi(lix,orientationObject)
            } else {
              return li
            }
          } else {
            return li
          }
        } else {
          return false
        }
      }

      const children = this.$parent.$slots.default
      if(orientation == 'prev'){
        orientationObject = 'previousSibling'
        lengthx = children.length
      }
      let nextElement = getNextLi(this.$el[orientationObject],orientationObject)
      if(nextElement){
        nextElement.querySelector('.vs-select-item-btn').focus()
      } else {
        if (lengthx === children.length) lengthx--
        getNextLi(children[lengthx].elm,orientationObject).querySelector('.vs-select-item-btn').focus()
      }
    },
    focusValue(index){
      if(this.$parent.vsMultiple?this.$parent.value.indexOf(this.vsValue) != -1:this.vsValue == this.$parent.value){
        if(!this.$parent.vsAutocomplete){
          setTimeout( () => {
            this.$refs.item.focus()
          }, 50);
        }
      } else if (index === 0) {
        if(!this.$parent.vsAutocomplete){
          setTimeout( () => {
            this.$refs.item.focus()
          }, 50);
        }
      }
    },
    putValue(){
      if(this.vsValue == this.$parent.value){
        this.$parent.valuex = this.vsText
      }

    },
    clickOption(){
      if(this.disabledx){
        return
      }
      let text = this.vsText
      if(!this.$parent.vsMultiple){
        this.$parent.active = false
        document.removeEventListener('click',this.$parent.clickBlur)
        this.$parent.valuex = text
        this.$parent.$emit('input',this.vsValue)
        this.$parent.changeValue()
      } else if (this.$parent.vsMultiple) {
        this.$parent.valuex = text
        this.$parent.addMultiple(this.vsValue)
      }
      this.$parent.$children.map((item)=>{
        item.valueInputx = ''
      })
    },

    // methods colors
    isColor(){
      return _color.isColor(this.vsColor)
    },
  }
}
</script>
