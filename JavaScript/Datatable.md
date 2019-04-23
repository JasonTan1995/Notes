### Datatable

​    Datatables是一款jquery表格插件。它是一个高度灵活的工具，可以将任何HTML表格添加高级的交互功能。

​    为什么使用Datatables？

1.  分页，即时搜索和排序
2. 几乎支持任何数据源：DOM，javascript，Ajax和服务器处理
3. 支持不同主题：Datatables，JQueryUI，Bootstrap
4. 有丰富多样的option和强大的API

   Datatables官方网站：http://www.datatables.club/

 #### DEMO

1. 导入JS，CSS静态资源文件

  ```javascript
<link rel="stylesheet" href="/bootstrap/datatable/datatables.min.css">
<script src="/bootstrap/datatable/datatables.min.js"></script>
  ```

2. 指定表格

```javascript
<table id="exampleTable" class="table table-striped table-bordered table-hover">
  <thead class="headings">
      <tr>
         <th>id</th>
         <th>name</th>
         <th>age</th>
      </tr>
  </thead>
  <tbody>
   </tbody> 
</table>
```

3. 使用Ajax初始化表格并为表格提供数据源

```javascript
$(function(){
    $("#exampleTable").DataTable({
    "ajax" : "/list",
    "columns":[
       {"data":"id"},
       {"data":"name"},
       {"data":"age"}
       ]
  })
})  
```

以上就是Ajax简单的使用方法.

#### 配置

```javascript
var table =$('#errorLogTable').DataTable({
         "searching":false, //是否开启本地搜索
         "aLengthMenu" : [20, 40, 60], //指定分页大小
         "bPaginate" : true, //是否开启分页
         "bSort" : true,  //是否开启排序
         "aaSorting": [  //指定某列以某种方式进行排序
           [4,'desc']
         ],
         "bAutoWidth":true, //是否开启自动列宽
         "aoColumnDefs":[   //指定某列的宽度。可以同columnDefs属性
         {sWidth:'30%',aTargets:[3]}
         ]
    });
```



#### 高阶用法 

现在有这么一种情况，假如需要对某一列进行添加点击事件或者是超链接之类的操作，Datatables可以实现吗？答案是当然可以的，下面来看看吧。

1.  首先去官网查看相关的API或者手册

[![iZGCod.md.png](https://s1.ax1x.com/2018/09/17/iZGCod.md.png)](https://imgchr.com/i/iZGCod)

2. 找到关于处理列的配置

[![iZGeOS.md.png](https://s1.ax1x.com/2018/09/17/iZGeOS.md.png)](https://imgchr.com/i/iZGeOS)

3. 因为是处理关于列的数据所以找到columns.data
4. 找到相关的DEMO(这里是使用data的这个函数来进行处理，可以通过row.  来获取数据源中的数据)

```javascript
$('#example').dataTable( {
  "columnDefs": [ {
    "targets": 0,
    "data": function ( row, type, val, meta ) {
      if (type === 'set') {
        row.price = val;
        // Store the computed display and filter values for efficiency
        row.price_display = val=="" ? "" : "$"+numberFormat(val);
        row.price_filter  = val=="" ? "" : "$"+numberFormat(val)+" "+val;
        return;
      }
      else if (type === 'display') {
        return row.price_display;
      }
      else if (type === 'filter') {
        return row.price_filter;
      }
      // 'sort', 'type' and undefined all just use the integer
      return row.price;
    }
  } ]
} );
```

5. 应用到项目中

```javascript
$(function(){
    $("#exampleTable").DataTable({
    "ajax" : "/list",  //发送Ajax请求的路径
    "columns":[
       {"data":"id"},
       {"data":"name"},
       {"data":"age"},
       {"data":"status"}
       ],
        "columnDefs":[{
           "targets": 12,
           "data": function ( row, type, full, meta ) {
              var status = '';
if(row.status == '1'){
jobStatus = '<i class="fa fa-play fa-lg" title="Resume" style="color: #7BBF6A; cursor: pointer;" th:onclick="javascript:action(\'resume\',\''+row.id+'\');"></i>'+'&nbsp; <i class="fa fa-play-circle fa-lg" title="Run Now" style="color: #337ab7; cursor: pointer;" th:onclick="javascript:action(\'trigger\',\''+row.id+'\');"></i>';
}else{
 jobStatus = '<i class="fa fa-play fa-lg" title="Resume" style="color: #7BBF6A; cursor: pointer;" th:onclick="javascript:action(\'resume\',\''+row.id+'\');"></i>';
}
    return status;
         }
      }]
   })
})
```

