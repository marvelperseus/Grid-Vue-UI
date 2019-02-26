<template>
    <div id="app">
        <ejs-grid ref='grid' :load = 'load' :allowRowDragAndDrop="true" :aggregates="aggregates" :enableAutoFill="true" :cellSelected='cellSelected' :allowGrouping='true' :rowDrop="rowDrop" :groupSettings='groupOptions' :allowExcelExport='true' :allowPdfExport='true' :contextMenuItems="contextMenuItems" :contextMenuClick='contextMenuClick' :queryCellInfo='queryCellInfoEvent' :cellEdit="getCellEdit" gridLines='Both' :toolbarClick='clickHandler' :toolbar='toolbar' :dataSource="data" :allowReordering='true' :dataBound='dataBound' :showColumnMenu='true' :columnMenuItems='columnMenuItems' :columnMenuClick='columnMenuClick' :allowResizing= 'true' :selectionSettings="selection" :filterSettings='filterSettings' :allowPaging="true" :allowSorting='true' :allowFiltering='true' :pageSettings='pageSettings' :editSettings='editSettings' >
          <e-columns>
              <e-column :lockColumn='true' :allowFiltering='false' :allowSorting='false' width='20' :headerTemplate='headerCheckTemp' :template='checkTemp'></e-column>
              <e-column field='group' headerText='' textAlign='Center' width=100></e-column>
              <e-column field='id' headerText='ID' isPrimaryKey={true} :visible='false' textAlign='Center' width=100></e-column>
              <e-column field='groupTitle' headerText='Group Title' :customAttributes="customAttributes" textAlign='Center' width=100></e-column>
              <e-column field='personUrl' headerText='Person' defaultValue= 'pic01.png' textAlign='Center' width=30 :template='personTemp'></e-column>
              <e-column field='status' headerText='Status' :customAttributes="customAttributes1" :allowEditing='false' :template='statusTemp' textAlign='Center'  width=80></e-column>
              <e-column field='tags' headerText='Tags' textAlign='Center' width=80></e-column>
              <e-column field='numbers' headerText='Numbers' textAlign='Center' editType= 'numericedit' width=80></e-column>
              <e-column field='text' headerText='Text' textAlign='Center' width=100></e-column>
              <e-column field='date' headerText='Date' width='130' format="yMd" editType= 'datepickeredit'  type="date" textAlign='Center'></e-column>
          </e-columns>
           <e-aggregates>
                <e-aggregate>
                    <e-columns>
                        <e-column type="Count" field="groupTitle" :groupFooterTemplate='countTemplate'>
                        </e-column>
                        <e-column type="Custom" columnName="status" :customAggregate="customAggregateStatus" :groupFooterTemplate='statusTemplate'>
                        </e-column>
                          <e-column type="Sum" field="numbers" :groupFooterTemplate='sumTemplate'>
                        </e-column>
                    </e-columns>
                </e-aggregate>
            </e-aggregates>
        </ejs-grid>
        <div class="row kanban">
            <div id='content' class="col-lg-2" style="padding-top: 30px; padding-left: 30px">
              <span style="padding-bottom: 30px">Kanban Column</span>
              <ejs-dropdownlist id='fitlerByColumn' :change='onChangeByColumn' :value='value'  :dataSource='sportsData' :popupHeight='height' :placeholder='waterMark'></ejs-dropdownlist>
            </div>         
            <div class="container col-lg-10" style="padding:30px">      
              <ul class="drag-list">
                <li v-for="item in items" class="drag-column drag-column-on-hold">
                  <span class="drag-column-header">
                    <h2>{{ item }}</h2>
                  </span>
                  <div class="drag-options" id="options1"></div>
                  <ul class="drag-inner-list" :id="item">
                     <button v-on:click="addPulse(item)">+ Add pulse</button>
                    <li v-for="(data,index) in dataByItem[item]" :key="index" class="drag-item">
                      <div>{{data['groupTitle']}}</div>
                    </li>
                   
                  </ul>
                </li>
              </ul>
            </div>
        </div>
    </div>
</template>
<script>
import Vue from "vue";
import { GridPlugin,Selection,Toolbar ,ExcelExport, PdfExport,ContextMenu, ColumnMenu, Reorder, Resize, RowDD, Page, Sort, Filter, Group, Aggregate ,Edit } from "@syncfusion/ej2-vue-grids";
import { ButtonPlugin } from '@syncfusion/ej2-vue-buttons';
import { enableRipple } from '@syncfusion/ej2-base';
import { Tooltip } from "@syncfusion/ej2-vue-popups";
import { DropDownListPlugin } from "@syncfusion/ej2-vue-dropdowns";
import { InPlaceEditorPlugin, Rte, MultiSelect, ActionEventArgs } from "@syncfusion/ej2-vue-inplace-editor";
import { ColorPickerPlugin } from "@syncfusion/ej2-vue-inputs";
import personTempVue from './person-temp.vue'
import statusTempVue from './status-temp.vue'
import { data } from './datasource11.js';
import $ from 'jquery';
import vueDirectiveTooltip from 'vue-directive-tooltip';
import { DialogPlugin } from '@syncfusion/ej2-vue-popups';


Vue.use(vueDirectiveTooltip, {
    delay: 500,
    placement: 'top',
    class: 'tooltip-red',
    triggers: ['hover'],
    offset: 0
});

Vue.use(DialogPlugin);
Vue.use(ColorPickerPlugin);
Vue.use(GridPlugin);
Vue.use(DropDownListPlugin);

enableRipple(true);

 let selectedData = [];

export default {
 
  data() {
    return {
      data:data,
      waterMark: 'Kanban Column',
      value: 'status',
      height: '220px',
      aggregates: [ 
                    { 
                        columns: [ 
                            { type: "Count", field: "groupTitle", groupFooterTemplate: 'countTemplate' }, 
                            { type: "Custom", columnName: "status", customAggregate: "customAggregateStatus", groupFooterTemplate: 'statusTemplate' }, 
                            { type: "Sum", field: "numbers", groupFooterTemplate: 'sumTemplate' } 
                        ] 
                    } 
                ],
      sportsData: Object.keys(data[0]),
      selection: {mode:'Cell', type: 'Multiple', cellSelectionMode: 'Box' },
      groupOptions: { showDropArea: false, columns: ['group'] },
      editSettings: { allowEditing: true, allowAdding: true, allowDeleting: true, mode: 'Batch',  newRowPosition: 'Bottom'},
      pageSettings: { pageSizes:[5, 10, 20, 50], pageSize: 10 },
      toolbar: ["Search",{ text: 'Toolbar', tooltipText: 'Toolbar', prefixIcon: 'e-expand', id: 'Click' }],
      filterSettings: { type: 'Menu' },
      customAttributes : { class: 'customcss'},
      customAttributes1: {class: 'statuscss'},
      columnMenuItems: ['AutoFit','ColumnChooser', 'AutoFitAll', 'SortAscending', 'SortDescending',
      'Filter', {text:'Clear Sorting', id:'gridclearsorting'}, 
      {text:'Duplicate Column', id:'duplicateColumn'},
      {text:'Add Column', id:'addcolumn', items:[
                        { text: 'Status' },
                        { text: 'Text' },
                        { text: 'Person' },
                        { text: 'Date' },
                        { text: 'Tags' },
                        { text: 'Numbers' },
                    ]},
      {text:'Aggregate', id:'aggregate', items:[
          { text: 'Sum', id: 'sum' },
          { text: 'Average', id: 'average' },
          { text: 'Min', id: 'min' },
          { text: 'Max', id: 'max' },
          { text: 'Count', id: 'count' }]}
      ],
      contextMenuItems: ['AutoFit', 'AutoFitAll', 'SortAscending', 'SortDescending',
             'Edit', 'Save', 'Cancel',
            'PdfExport', 'ExcelExport', 'CsvExport', 'FirstPage', 'PrevPage',
            'LastPage', 'NextPage',{text: 'Duplicate this Row', target: '.e-content', id: 'duplicaterow'}],
      personTemp:function () {
          return { template : personTempVue}
      },
      statusTemp: function (params) {
        return { template : statusTempVue}
      },
      countTemplate: function() {
        return {
            template: Vue.component('countTemplate', {
            template: ` <span> Total Tasks: {{data.Count}}</span>`,
            data: function () {return {data: {}};}
            })
        }
      },
      sumTemplate: function() {
        return {
            template: Vue.component('sumTemplate', {
            template: ` <span>{{data.Sum}}</span>`,
            data: function () {return {data: {}};}
            })
        }
      },
      statusTemplate: function() {
        return {
            template: Vue.component('groupFooterTemplate', {
            template: ` <span> {{data.Custom}} completed</span>`,
            data: function () {return {data: {}};}
            })
        }
      },
      headerCheckTemp: function() {
        return {
            template: Vue.component('headerCheckTemp', {
              template: `<div class='wrap'>
                        <ejs-colorpicker mode="Palette" value="#7bdcb5" :columns="3" :showButtons="false" :modeSwitcher="false" :presetColors="customColors" :beforeTileRender="tileRender" :change="onChange"></ejs-colorpicker>
                        </div>`,
              data: function () {return {
                 customColors: {'custom1': ['#ff6900', '#fcb900', '#7bdcb5', '#00d084', '#8ed1fc', '#0693e3', '#abb8c3', '#eb144c', '#9900ef']},
              }},
              methods: {
                onChange: function(args) {
                  var temp = document.getElementsByClassName("customcss");
                  
                  for (let index = 0; index < temp.length; index++) {
                      temp[index].style.color=args.currentValue.hex; 
                  }
                },
                  tileRender: function(args) {
                    args.element.classList.add("e-icons");
                    args.element.classList.add("e-custom-tile");
                }
              }
            })
        }
      },
      checkTemp:function () {
         return { template : Vue.component('columnTemplate',{
             template: `
                <div class="template_checkbox">
                    <input type="checkbox" />
                </div>`,
                data: function() {
                    return {
                        data: {}
                    }
                }
          })}
      },
    };
  },
  provide: {
    grid: [Page, Selection, ColumnMenu, Toolbar, RowDD,Resize, ContextMenu, Sort, Edit, ExcelExport, PdfExport,Filter, Group,Reorder, Aggregate]
  },
  computed: {
    items: function () {
        var temp = [];
        var index = 0;
        data.forEach(element=> {
            temp[index] = element[this.value];
            index++;
        });
        return Array.from(new Set(temp))
    },
    dataByItem: function () {
        var temp = [];
        this.items.forEach(elem => {
            temp[elem] = data.filter((item) => item[this.value] === elem);
        })
        return temp;
    }
  },
  methods: {
    load() {
      this.$refs.grid.$el.addEventListener('keydown', this.keyDownHandler);
    },
    // add auto incremented value to the next cells while drag the particular cell in Grid column(like an excel feature)
    // but not working because library's issue
    cellSelected(args){
     selectedData = args.data;
      if (args.selectedRowCellIndex.length > 1 && this.$refs.grid.ej2Instances.editModule.getBatchChanges().changedRecords.length) {   // Column autoincrement 
            var value; 
            for (var i = 0; i < args.selectedRowCellIndex.length; i++) { 
                var cell = this.$refs.grid.ej2Instances.getRowByIndex(args.selectedRowCellIndex[i].rowIndex).cells[args.selectedRowCellIndex[i].cellIndexes[0]]; 
                if (i == 0) { 
                    value = parseInt(cell.innerHTML); 
                    continue; 
                } 
                value = value + 1; 
                var field = this.$refs.grid.ej2Instances.getColumnByIndex(cell.cellIndex).field; 
                this.$refs.grid.ej2Instances.editModule.getBatchChanges().changedRecords[i][field] = value; 
                cell.innerHTML = value.toString(); 
            } 
        } 
        if (args.selectedRowCellIndex.length == 1) {  // Row autoincrement 
            var value; 
            for (var i = 0; i < args.selectedRowCellIndex.length; i++) { 
                var cells = args.selectedRowCellIndex[i].cellIndexes; 
                for (var j = 0; j < cells.length; j++) { 
                    var cell = this.$refs.grid.ej2Instances.getRowByIndex(args.selectedRowCellIndex[i].rowIndex).cells[args.selectedRowCellIndex[0].cellIndexes[j]]; 
                    if (j == 0) { 
                        value = parseInt(cell.innerHTML); 
                        continue; 
                    } 
                    value = value + 1; 
                    var field = this.$refs.grid.ej2Instances.getColumnByIndex(cell.cellIndex).field; 
                    var rowIndex = args.selectedRowCellIndex[0].rowIndex; 
                    this.$refs.grid.ej2Instances.editModule.updateCell(rowIndex, field, value); 
                } 
            } 
        } 
    },
  
    rowDrop(arg){
       let gObj = this.$refs.grid.ej2Instances; 
        if ((gObj).groupSettings.columns.length > 0) {
          let startedRow = arg.rows[0];
          let targetRow = arg.target.closest('tr');
          if (targetRow.classList.contains('e-row') && targetRow.classList.length) {
            targetRow = targetRow;
          } else {
            targetRow = arg.target.closest('tr').nextSibling;
          }
          let startRowIndex = parseInt(startedRow.getAttribute("aria-rowindex"), 10);
          let targetRowIndex = parseInt(targetRow.getAttribute("aria-rowindex"), 10);
          if (startRowIndex === targetRowIndex) {
            return;
          }
          let groupObj = this.currentViewData;
          let groupedCol = gObj.groupSettings.columns;
          for (let i = 0, len = groupedCol.length; i < len; i++) {
            let dataIndex = this.data.indexOf(arg.data);
            let value = gObj.currentViewData["records"][targetRowIndex][groupedCol[i]];
            let dragStartColData = gObj.currentViewData["records"][startRowIndex][groupedCol[i]];
            if (dragStartColData === value) {        
              let dragRow = [].slice.call(gObj.getContentTable().querySelectorAll('tbody .e-row')).filter(x => x.getAttribute("data-uid") == startedRow.getAttribute("data-uid"));
              let cloneRow = dragRow[0].cloneNode(true);
              dragRow[0].remove();
              gObj.getContentTable().querySelector('tbody').insertBefore(cloneRow, targetRow);
              gObj.clearSelection();
              let dragborderEle = cloneRow.querySelector('.e-dragborder');
              if (dragborderEle) {
                dragborderEle.classList.remove("e-dragborder");
              }
            } else {
              gObj.currentViewData["records"][startRowIndex][groupedCol[i]] = value;
              gObj.refreshColumns();
            }
          }

          arg.cancel = true;
        }
    },
//duplicaterow
    contextMenuClick(args) {
      if(args.item.id === 'duplicaterow') {
         this.data.unshift(selectedData);
         this.data = [...this.data];
      }

    },
    columnMenuClick(args) {
      if (args.item.parentObj.id === 'addcolumn') { 
        this.addcolumn(args)
      }

      if (args.item.id === 'duplicateColumn') {
          var customData = [];
       var i = 0;
        this.data.forEach(element => {    
          for (var key in element ) {
             if(key == args.column.field) {
               customData[i]=element[key];
               i++;
             }
          }
        });

        if (args.column.type == 'date') {
           this.$refs.grid.ej2Instances.columns.push({ field: args.column.field,format:"yMd",type:args.column.type,editType:args.column.editType,textAlign: 'Center', headerText: args.column.headerText, width: 100 }) 
           this.$refs.grid.ej2Instances.columns = [...this.$refs.grid.ej2Instances.columns];    
        } else {
            this.$refs.grid.ej2Instances.columns.push({ field: args.column.field,type:args.column.type,editType:args.column.editType,textAlign: 'Center', headerText: args.column.headerText, width: 100 }) 
             this.$refs.grid.ej2Instances.columns = [...this.$refs.grid.ej2Instances.columns];
        }
      }
      if (args.item.parentObj.id === 'aggregate') {
    
        this.aggregates = [{ 
                        columns: [ { type: "Count", field: "groupTitle", groupFooterTemplate: function() { 
                                      return { 
                                        template: Vue.component('countTemplate', { 
                                            template: `<span>Total Tasks: {{data.Count}}</span>`, 
                                            data: function () { 
                                              return {  
                                                data: { 
                                                  data: {} 
                                                } 
                                              }; 
                                            } 
                                        }) 
                                      } 
                                    } }, 
                            { field: 'numbers', type: args.item.text } ] 
                    }]; 
      }

      if(args.item.id === 'gridclearsorting'){
          this.$refs.grid.clearSorting();
      }
    },
    customAggregateStatus : function (data) {
   
         return data.items.filter((item) => item.status === 'Done').length;
      },
    addcolumn(args){
      this.$refs.grid.ej2Instances.columns.push({ field: args.item.text,textAlign: 'Center', headerText: args.item.text, width: 150 }) 
      this.$refs.grid.ej2Instances.columns = [...this.$refs.grid.ej2Instances.columns];
    },
    queryCellInfoEvent: function (args) {
      if(args.column.field === 'status'){
          if (args.data['status'] === 'Done') {
              args.cell.classList.add('done');
          } else if(args.data['status'] === 'Stuck') {
              args.cell.classList.add('stuck');
          }
      }

    },
    clickHandler: function(args) {
      alert("toolbar is clicked")
    },
    // when arrowkeydown is clicked, add new row.
    keyDownHandler(e) {
      if(e.keyCode == 40) {
        let gridIns = this.$refs.grid.ej2Instances;
        gridIns.addRecord();
      }
    },
    dataBound: function() {
      	  // this.$refs.grid.ej2Instances.columns[0].isPrimaryKey = true;
    },
    // cell edit handler
    getCellEdit: function () {
      console.log(this.$refs.grid.ej2Instances.getSelectedRecords())
    },
    addPulse(item) {
       $('#'+item).append(' <li class="drag-item"><div</div></li>');
    },
    onChangeByColumn(args){
      this.value = document.getElementById('fitlerByColumn').value; 
    }
  
  }
}


</script>

<style lang="scss">

@import '../node_modules/@syncfusion/ej2-base/styles/material.css';  
@import '../node_modules/@syncfusion/ej2-buttons/styles/material.css';  
@import '../node_modules/@syncfusion/ej2-dropdowns/styles/material.css';  
@import '../node_modules/@syncfusion/ej2-inputs/styles/material.css';  
@import '../node_modules/@syncfusion/ej2-popups/styles/material.css';
@import '../node_modules/@syncfusion/ej2-splitbuttons/styles/material.css';
@import "../node_modules/@syncfusion/ej2-vue-grids/styles/material.css";
@import "../node_modules/@syncfusion/ej2/material.css";
@import '../node_modules/vue-directive-tooltip/css/index.css';
@import 'node_modules/bootstrap/scss/bootstrap';
@import 'node_modules/bootstrap-vue/src/index.scss';

@import "./style/style.css";   
@font-face {
  font-family: "button-icons";
  src: url(data:application/x-font-ttf;charset=utf-8;base64,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)
    format("truetype");
  font-weight: normal;
  font-style: normal;
}
.kanban {
  color: white
}

#content span {
  color: white
}
#button-control .e-btn-sb-icons {
  font-family: "button-icons";
  line-height: 1;
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
button {
  margin: 25px 5px 20px 20px;
}

.e-plus-icon::before {
  content: '\e823';
}

.control-section {
      overflow: auto;
  }

.done {
  background-color: green
}

.stuck {
  background-color: red
}

.e-headercontent table thead .e-headercell {
  padding: 0px
}

.e-container {
  background-color: transparent;
  border-color: transparent;
  box-shadow: none;
}

/* Tile customization styles */
.e-container .e-palette .e-custom-tile {
  border: 0;
  color: #fff;
  height: 36px;
  font-size: 18px;
  width: 36px;
  line-height: 36px;
  border-radius: 50%;
  margin: 2px 5px;
}

.e-container .e-custom-palette.e-palette-group {
  height: 182px;
}

/* Selected state icon */
.e-container .e-palette .e-custom-tile.e-selected::before {
  content: '\e933';
}

.e-container .e-palette .e-custom-tile.e-selected {
  outline: none;
}
.e-split-btn-wrapper button {
  margin: 0px
}
th .e-headercell{
  padding: 0px
}
.wrap {
  margin: 0 auto;
  height: 29px;
  text-align: center;
}

img {
    height: 25px;
    width: 25px;
    border-radius: 50px;
    box-shadow: inset 0 0 1px #e0e0e0, inset 0 0 14px rgba(0,0,0,0.2);
}

.e-grid-menu .e-menu-parent .e-menu-item .em-status::before {
  content:'\e933';
}
.e-grid-menu .e-menu-parent .e-menu-item .em-text::before {
  content: '\e333';
}
.e-grid-menu .e-menu-parent .e-menu-item .em-person::before {
  content: '\e700';
}
.e-grid-menu .e-menu-parent .e-menu-item .em-date::before {
  content: '\e960';
}
.e-grid-menu .e-menu-parent .e-menu-item .em-tag::before {
  content: '\e353';
}
.e-grid-menu .e-menu-parent .e-menu-item .em-number::before {
  content: '\e21e';
}

$ease-out: all .3s cubic-bezier(0.23, 1, 0.32, 1);
$on-hold: #FB7D44;
$in-progress: #2A92BF;
$needs-review: #F4CE46;
$approved: #00B961;

* {
	box-sizing: border-box;
}

body {
	background: #33363D;
	color: white;
	font-family: 'Lato';
	font-weight: 300;
	line-height: 1.5;
	-webkit-font-smoothing: antialiased;
}

ul {
	list-style-type: none;
	margin: 0;
	padding: 0;
}

.drag-container {
	max-width: 1000px;
	margin: 20px auto;
}

.drag-list {
	display: flex;
	align-items: flex-start;
	
	@media (max-width: 690px) {
		display: block;
	}
}

.drag-column {
	flex: 1;
	margin: 0 10px;
	position: relative;
	background: rgba(black, 0.2);
	overflow: hidden;
	
	@media (max-width: 690px) {
		margin-bottom: 30px;
	}
	
	h2 {
		font-size: 0.8rem;
		margin: 0;
		text-transform: uppercase;
		font-weight: 600;
	}
	
	&-on-hold {
		.drag-column-header,
		.is-moved,
		.drag-options {
			background: $on-hold;
		}
	}
	
	&-in-progress {
		.drag-column-header,
		.is-moved,
		.drag-options {
			background: $in-progress;
		}
	}
	
	&-needs-review {
		.drag-column-header,
		.is-moved,
		.drag-options{
			background: $needs-review;
		}
	}
	
	&-approved {
		.drag-column-header,
		.is-moved,
		.drag-options {
			background: $approved;
		}
	}
}

.drag-column-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 10px;
}

.drag-inner-list {
  min-height: 50px;
  button {
    border: 0px;
    background-color: #292b31;
    color: #fafafa;
  }
}

.drag-item {
	margin: 10px;
  height: 100px;
  background-color: white;
	// background: rgba(black, 0.4);
	transition: $ease-out;
	color: black;
	&.is-moving {
		transform: scale(1.5);
		background: rgba(black, 0.8);
	}
	
}

.drag-header-more {
	cursor: pointer;
}

.drag-options {
	position: absolute;
	top: 44px;
	left: 0;
	width: 100%;
	height: 100%;
	padding: 10px;
	transform: translateX(100%);
	opacity: 0;
	transition: $ease-out;
	
	&.active {
		transform: translateX(0);
		opacity: 1;
	}
	
	&-label {
		display: block;
		margin: 0 0 5px 0;
		
		input {
			opacity: 0.6;
		}
		
		span {
			display: inline-block;
			font-size: 0.9rem;
			font-weight: 400;
			margin-left: 5px;
		}
	}
}

/* Dragula CSS  */

.gu-mirror {
  position: fixed !important;
  margin: 0 !important;
  z-index: 9999 !important;
  opacity: 0.8;
	list-style-type: none;
}

.gu-hide {
  display: none !important;
}

.gu-unselectable {
  -webkit-user-select: none !important;
  -moz-user-select: none !important;
  -ms-user-select: none !important;
  user-select: none !important;
}

.gu-transit {
  opacity: 0.2;
}

/* Demo info */

.section {
	padding: 20px;
	text-align: center;
	
	a {
		color: white;
		text-decoration: none;
		font-weight: 300;
	}
	
	h4 {
		font-weight: 400;
		a {
			font-weight: 600;
		}
	}
}
</style>

