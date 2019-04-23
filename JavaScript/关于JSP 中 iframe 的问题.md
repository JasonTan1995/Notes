####  关于JSP 中 iframe 的问题

- 摘要:最近在工作中的一个小困难.特此记录下来以往后提醒自己.

***

#### 1.  发现问题

##### 1.1  环境

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzqx0liwkpj20ld07w0t2.jpg)

 以上是一个JSP页面,简称c.jsp. c.jsp 中包含了两个页面menu.jsp,footer.jsp,此外还有一个iframe.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fzqzjneqrjj209k07za9w.jpg)

##### 1.2 问题

 现在需要test.jsp中的某个方法被触发时把Name传给menu.jsp.这个Name的值是动态的.

 我当时能想到的方案:

- 在该方法被触发时会发送请求到后台.在后台中把该值放到session域中然后menu.jsp再通过el表达式获取该值.

> 然而令到该方法不可行的原因是:页面加载顺序的问题.所有的请求与响应都是发生在test.jsp中的.那就意味着menu.jsp和footer.jsp并不会刷新或者重新加载.el表达式的值只会在JSP第一次加载的时候填充.所以在menu.jsp中通过el表达式是取不到值的.

##### 1.3 思考

- 一个父JSP中包含了两个子JSP与一个iframe.

- 我们可以把JSP页面看作一个容器,那么父JSP就是一个父容器.一个父容器中包含了两个子容器,而容器中装载的是页面中所有的元素.

- 在test.jsp页面中先获取父容器,再通过父容器定位到子容器中的a标签.

- 定位后就可以动态的修改a标签中的href属性值了

需要一些JS中的方法帮助我们完成以上所说的步骤.

***

#### 2. 查找资料

##### 2.1 **iframe子页面调用父页面js函数**

   ```javascript
window.parent.a();
   ```

##### 2.2 子页面取父页面中的标签中的值，比如该标签的id为“test”

```javascript
window.parent.document.getElementById("test").value; 

jQuery方法为： 

$(window.parent.document).contents().find("test").val(); 
```

##### 2.3 多个iframe(子iframe获取另一个子iframe的任一元素)

```javascript
// 首先先获取当前页面的父容器,再获取父容器中所有的iframe标签
window.parent.document.getElementsByTagName('iframe')
```

##### 2.4 **iframe父页面调用子页面js函数** 

```javascript
document.getElementById('ifrtest').contentWindow.b(); 
```

以上的这个4个方法足以解决平时在实际项目中关于iframe的问题.

***

#### 3. 解决问题

##### 3.1 先在必须要触发的方法中定义一个方法.

```javascript
function changeMenuLanguageUrl(formId,language){
            jQuery.ajax({
                url: '../form/GetAllFormName.action',
                type: 'get',
                data: {formId:formId},
                dataType: 'json',
                success: function(data){
                    var formNameCN = data.data[0].formName;
                    var formNameEN = data.data[0].formNameCN;
                    if (language == 'cn') {
                        var basePathEN = (window.parent.document.getElementById("changeEN")).href;
                        if (formNameEN) {
                            (window.parent.document.getElementById("changeEN")).href = basePathEN + '&formName=' + formNameEN;
                        }
					}
                    if (language == 'en') {
                        var basePathCN = (window.parent.document.getElementById("changeCN")).href;
                        if (formNameCN) {
                            (window.parent.document.getElementById("changeCN")).href = basePathCN + '&formName=' + formNameCN;
                        }
					}
                }
            });
		}
```

##### 3.2 方法中所做的事情:

- 利用提供的Id.发起Ajax请求到后台获取表单的名字.
- 在当前页面中获取父容器并定位到所需要修改的标签
- 根据需要修改标签中属性的值.