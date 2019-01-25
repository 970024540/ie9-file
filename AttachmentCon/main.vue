// IE9上传附件
<template>
  <div>
    <label v-if="!destoryFileInput" :for="fileId" class="label-file">
      <span>上传附件</span>
      <form ref="form">
        <input ref="fileInput" name="file" @change="changeFile" :id="fileId" :multiple="multiple" type="file" />
      </form>
    </label>
    <span class='beizhu'>(50MB以内，最多上传1011个)</span>
    <ul>
      <li v-for="item in successList" class='el-upload-list__item is-success'>
        <a class='el-upload-list__item-name'><i class='el-icon-document'></i>{{item.name}}</a>
        <label class="el-upload-list__item-status-label"><i class="el-icon-upload-success el-icon-circle-check"></i></label>
        <i @click='closeFlie(item)' class="el-icon-close"></i>
      </li>
    </ul>
  </div>
</template>
<script>
import { mipDocumentFile2 } from "../../api/api";
  export default {
    data(){
      return {
        fileId:'mainFileId',
        successList:[],
        docIds:'',
        destoryFileInput:false,
        langMsg: 'flowManagePage.processform',
      }
    },
    props:['ListData','fdAuditeId'],
    created(){
      this.$set(this,'successList',this.ListData);
      // this.successList=this.ListData;
    },
    methods:{
        closeFlie(val){//{name:'111.jpg',response:{docId:'aa',downloadUrl:'bb'...}}
          this.successList=this.successList.filter(item => item.name!=val.name);
          this.$message.success(this.$t(this.langMsg + '.delSuccess')); 
          this.filterIds();
        },
        filterIds(){
          let listIds=[];
          this.successList.forEach(item=>{
            if(item.response&&item.response.data){
              listIds.push(item.response.data.docId);
            }
            if(item.status==='success'&&item.docId){
              listIds.push(item.docId);
            }
          })
          this.docIds=listIds.join(';');
          this.$emit('onModifyTheIds',this.docIds,this.successList);
        },
        //接受文件函数
        changeFile(ev){
          let target=ev.target;
          // if (window.navigator.userAgent.indexOf('MSIE 9.0') > -1) {//ie9 不支持多文件上传
              target.select();
              //确保IE9下，不会出现因为安全问题导致无法访问
              target.blur();
              this.uploadHtml4(target,this.fileId);
          // }
        },
        uploadHtml4 (el,fileId) {//ie9的上传
          this.destoryFileInput=true;
          setTimeout(()=>{
              this.destoryFileInput=false;
          },10)
          let file={
              name:el.value.replace(/^.*?([^\/\\\r\n]+)$/, '$1'),
              // size:el.files[0].size
          }
          let isFile=false;
          if(file.name){
            this.successList.forEach(item=>{
              if(item.name===file.name){
                isFile=true;
              }
            })
          }
          if(isFile){//判断重复文件
            this.$message.error(this.$t(this.langMsg + '.noUploaded'));
            return ;
          }
          if(this.successList.length>9){//判断文件个数
            this.$message.error(this.$t(this.langMsg + '.AtMostUploaded'));
            return ;
          }
          // if(_.findWhere(this.fileList,{name:file.name,langType:langType})) return;//过滤同名
          let iframe = document.createElement('iframe')
          iframe.id = 'upload-iframe-' + fileId
          iframe.name = 'upload-iframe-' + fileId
          iframe.src = 'about:blank' 
          iframe.setAttribute('style', 'width:1px;height:1px;top:-999em;position:absolute; margin-top:-999em;')

          let form =document.createElement('form');
          form.appendChild(el);
          let data={
              systemId:'MIP',
              moduleName:'MBPM-AUDIT-NOTE',
              moduleId:this.fdAuditeId,
              sectionId:this.$lang
          }
          let value
          let input
          for (let key in data) {
              value = data[key]
              if (value && typeof value === 'object' && typeof value.toString !== 'function') {
              value = JSON.stringify(value)
              }
              if (value !== null && value !== undefined) {
                  input = document.createElement('input')
                  input.type = 'hidden'
                  input.name = key
                  input.value = value
                  form.appendChild(input)
              }
          }

          form.action =mipDocumentFile2;//上传地址
          form.name = 'upload-form-' + fileId;
          form.setAttribute('method', 'POST');
          form.setAttribute('target', 'upload-iframe-' + fileId);
          form.setAttribute('enctype', 'multipart/form-data');

          iframe.onreadystatechange =(e)=>{
              var readyState=iframe.readyState;
              if(readyState=='loading'){
              }else if (readyState == 'complete') {
                  handler(e);
              }
          };
          let getResponseData = function () {
              let body = iframe.contentDocument.body || iframe.contentWindow.document.body;
              return body.innerHTML || body.textContent;
          }
          let handler=(e)=>{
              let response = getResponseData();
              if(response){
                  response=response.replace('<pre>', '').replace('</pre>', '');
                  response=JSON.parse(response);
                  this.handleFileListSuccess(response||{},file);
                  if(iframe.parentNode&&response.success){//成功之后再删除
                      iframe.parentNode.removeChild(iframe);
                  }
              }
          }
          document.body.appendChild(iframe).appendChild(form);
          // 提交
          form.submit();
      },
      handleFileListSuccess(res,file){//获取到接口返回的数据和上传的名字
        if(res.success){
          this.successList.push({name:file.name,response:res});
          this.filterIds();
        }else{
          if(res.msg){
            this.$message.error(res.msg);
          }
        }
      }
    }
  }
</script>
<style lang="less" scoped>
    .beizhu{
      line-height: 30px;
      vertical-align: top;
      margin-left: 6px;
      font-size: 12px;
      color: #8391a5;
    }
    .label-file{
        line-height: 28px;
        padding:0 9px;
        cursor: pointer;
        display: inline-block;
        position: relative;
        overflow: hidden;
        font-size: 13px;
        border:1px solid #c4c4c4;
        border-radius: 3px;
        color: #1f2d3d;
        text-align: center;
        &:hover{
            border-color:#148ef5;
            color:#148ef5;
        }
        &.languageInput{
            color: #a6adc2;
            min-width:203px;
            &:hover{
                color:#1f2d3d;
            }
        }
    }
    .label-file input[type="file"]{
        position: absolute;
        right: 10000px;
        top:0px;
        opacity: 0;
        height:100%;
    }
</style>

