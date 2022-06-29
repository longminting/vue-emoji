<template>
  <div>

    <div class="biaoqingContent" :style="{width:width}">
      <!--      文本框内容-->
      <div :style="{height:height}" contenteditable="true" @click.stop="contentClick" @blur="onBlur" @change="onBlur" class='contentBox' :id="id" >
      </div>
      <img class="expression" @click.stop="show=!show" src="./expression.png" />
      <!--      表情选择框-->
      <div class='box' :style="{width:width}" v-if="show" @click.stop>
        <div class='left'>
          <ul>
            <li v-for="(item,index) in biaoqingList" :key="index" :class='["cursor",biaoqingActive===index?"active":""]'>
              <div @click="bqNameChange(index,item.iconArr)">{{item.name}}</div>
            </li>
          </ul>
        </div>
        <div class='right'>
          <button v-for='(i,index) in rightList' :key="index" class='emoji' @click.stop="insertHtmlAtCaret(i.className,i.icon)">
            <img src="http://tkeasyemoji.oss-cn-shanghai.aliyuncs.com/images/placeholder.png" :class='i.className' :alt="i.icon">
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  // 定义最后光标对象

  import {
    biaoqingArr
  } from "./biaoqing.js";
  export default {
    props:{
      //表情框宽度
      width:{
        type:String,
        default:'50%',
      },
      //表情框高度
      height:{
        type:String,
        default:'200px',
      },
      //表情框ID
      id:{
        type:String,
        default:'text',
      },
      //内容
      value:{
        type:String,
        default:'',
      },
    },
    data() {
      return {
        lastEditRange:'',
        content: "",
        show: false,
        biaoqingList: biaoqingArr,
        biaoqingActive: 0,
        rightList: [],
        allList:[],
      };
    },
    watch:{
      value(){
        this.valueChange();
      },
      content(){
        // 内容变更重新获取光标位置
        this.contentClick();
      }
    },
    methods: {
      valueChange(){
        if(!this.value)return;
        let str = this.value;
        let newStr =this.forArr(this.forArr(str),'className');
        document.getElementById(this.id).innerHTML = newStr;
      },
      //替换emoji 表情图片
      forArr(str,_type){
        for (let j = 0; j < this.allList.length; j++) {
          let val = this.allList[j];
          let reg =_type==='className'?`alt${val.className}`:val.icon;
          if(_type==='className'){
            str = str.replace(new RegExp(reg,'g'),val.icon)
          }else{
            str= str.replace(new RegExp(reg,'g'),`<img style="vertical-align: sub;" src="http://tkeasyemoji.oss-cn-shanghai.aliyuncs.com/images/placeholder.png" class="${val.className}" alt="alt${val.className}">`)
          }
        }
        return str;
      },
      //失去焦点 内容返回
      onBlur(){
        let text = document.getElementById(this.id).innerHTML;
        this.content= text.replace(/<img style="vertical-align: sub;" src="http:\/\/tkeasyemoji\.oss-cn-shanghai\.aliyuncs\.com\/images\/placeholder\.png" class="/g,'').replace(/alt="/g,'').replace(/">/g,'').replace(/bg-.{7}/g,'');
        this.$emit('contentChange',{
          id:this.id,
          value:this.content
        })
      },
      // 表情title点击事件
      bqNameChange(index, arr) {
        this.biaoqingActive = index;
        this.rightList = arr;

      },
      contentClick(){
        // 获取选定对象
        var selection = getSelection()
        // 设置最后光标对象
        this.lastEditRange = selection.getRangeAt(0)
      },
      //表情插入文本框
      insertHtmlAtCaret(className,icon) {
        var edit = document.getElementById(this.id);
        // 编辑框设置焦点
        edit.focus();
        var sel = getSelection();
        // 判断是否有最后光标对象存在
        if (this.lastEditRange) {
          // 存在最后光标对象，选定对象清除所有光标并添加最后光标还原之前的状态
          sel.removeAllRanges()
          sel.addRange(this.lastEditRange)
        }
        var html = `<img style="vertical-align: sub;" src="http://tkeasyemoji.oss-cn-shanghai.aliyuncs.com/images/placeholder.png" class="${className}" alt="${icon}">`;
        var  range;
          if (window.getSelection) {
            // IE9 and non-IE
            sel = window.getSelection();
            if (sel.getRangeAt && sel.rangeCount) {
              range = sel.getRangeAt(0);
              range.deleteContents();
              var el = document.createElement("div");
              el.innerHTML = html;
              var frag = document.createDocumentFragment(), node, lastNode;
              while ((node = el.firstChild)) {
                lastNode = frag.appendChild(node);
              }
              range.insertNode(frag);
              if (lastNode) {
                range = range.cloneRange();
                range.setStartAfter(lastNode);
                range.collapse(true);
                sel.removeAllRanges();
                sel.addRange(range);
              }
            }
          } else if (document.selection && document.selection.type != "Control") {
            // IE < 9
            document.selection.createRange().pasteHTML(html);
          }
        // 无论如何都要记录最后光标对象
        this.lastEditRange = sel.getRangeAt(0)
        },

    },

    created() {
      this.rightList = biaoqingArr[0].iconArr;
      biaoqingArr.forEach(e=>{
        this.allList =this.allList.concat(e.iconArr)
      });

    },
    mounted() {
      this.valueChange();
      let that = this;
      document.body.onclick=function () {
        that.show = false;
      }
    },
    components: {}
  };
</script>
<style>
  @import "./emoji_sorites.css";
</style>
<style lang='less' scoped>
  .contentBox {
    width: 100%;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 10px;
    font-size: 16px;
    position: relative;

  }
  .biaoqingContent {
    padding: 5px;
    margin: 5px auto;
    .expression{
      width: 30px;
      margin: 5px 0;

      cursor: pointer;
    }

    .box {
      border: 1px solid #73a8f9;
      min-height: 400px;
      box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.2);
      background: #fff;
      z-index: 10;
      position: absolute;
      width: 100%;
      display: flex;
    }
    .left {
      width: 90px;
      height: 100%;
      display: table-cell;
      border-right: 1px solid #ebebeb;

      padding: 4px;

      ul {
        padding: 0;
        margin: 0;
        text-align: center;
        list-style-type: none;

        li {
          height: 30px;
          line-height: 30px;
          cursor: pointer;
          font-size: 14px;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;

          button {
            border: none;
            background: none;
            width: 100%;
            height: 100%;
          }
        }
      }

      .active {
        background-color: #3171d1;
        color: #ffffff;
        border-radius: 4px;

        button {
          color: #ffffff;
        }
      }
    }
    .right {
      flex: 1;
      padding-left: 10px;
      padding-top: 15px;
      overflow: auto;

      button {
        border: none;
        padding: 0;
        cursor: pointer;
      }

      .emoji {
        display: inline-block;
        padding: 3px;
        border: 1px solid transparent;
        cursor: pointer;
        background: #fff;

        &:hover {
          height: 32px;
          background-color: #ddded8;
          border: 1px solid #b3c1fd;
          border-radius: 4px;
        }
      }
    }
    //样式一
    .box0{


    }

  }

</style>
