<template>
 
    <ejs-inplaceeditor id="inplace_editor" ref="dataObj" type="Text" :actionBegin="actionBegin" :primaryKey="data.id" :template = "Template1" :value="data.status" :actionSuccess= "actionSuccess">
    </ejs-inplaceeditor>

</template>

<script>
import Vue from 'vue';
import { InPlaceEditorPlugin } from '@syncfusion/ej2-vue-inplace-editor';

Vue.use(InPlaceEditorPlugin);
export default {
  name: 'app',
  data () {
    return {
      data: {},
      Template1: function() {
        return {
            template: Vue.component('Template1', {
            template: `<div><input type='text' id='statusValue'></input>
                        <div class='wrap'>
                        <ejs-colorpicker mode="Palette" id="colorObj" ref="colorObj" :inline='true' :change="onChange" :columns="6" :showButtons="false" :modeSwitcher="false" :presetColors="customColors" :beforeTileRender="tileRender"></ejs-colorpicker>
                        </div></div>`,
             data() { return {
                  customColors: {'custom': ['#f44336', '#e91e63', '#9c27b0', '#673ab7', '#2196f3', '#03a9f4', '#00bcd4',
                    '#009688', '#8bc34a', '#cddc39', '#ffeb3b', '#ffc107']}
              }; },
              methods: {
                 onChange: function(args) {
                  console.log(args)
                },
                  tileRender: function(args) {
                    args.element.classList.add("e-icons");
                    args.element.classList.add("e-custom-tile");
                }
              }
            })
        }
      },
    }
  },
  methods: {
      actionSuccess(e) {
          console.log(e.value)
      }, 
      actionBegin(args) {
        var temp = document.getElementsByClassName("statuscss");
        temp[args.data.primaryKey].style.backgroundColor=document.getElementById('colorObj').value;
        if (document.getElementById('statusValue').value == '') {
          this.$refs.dataObj.ej2Instances.value =   args.data.value;  
        } else {
          this.$refs.dataObj.ej2Instances.value = document.getElementById('statusValue').value;
        }
      }
  },
}
</script>

<style>

#inplace_editor {
    width: 100%;
}
#inplace_editor_wrap {
    width: 100%;
    padding: 0px;
   
}
#inplace_editor_wrap .e-editable-value {
    border-bottom: 0px;
    margin: 0px;      
}
.e-inplaceeditor-tip.e-tooltip-wrap .e-tip-content .e-editable-wrapper .e-editable-action-buttons {
    margin-top: 60px;
}
</style>