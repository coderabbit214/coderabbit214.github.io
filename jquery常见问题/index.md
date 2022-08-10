# JQuery常见问题


# JQuery常见问题

### 选择 select 框中选中的项

> **$("#categoryItem option:selected")**

```html
<select class="form-control" id="payType" name="payType.id">
  <option value="">请选择结算类型</option>
  <#list zhifuleixing as tmp>
  <option value="${tmp.id}">${tmp.title}</option>
  </#list>
</select>
<#-- select选择狂数据回显 -->
<script>
    $("select[name='business.id']").val(${consumption.business.id})
</script>

<script>
     var categoryItem_val = $("#categoryItem option:selected").val();
</script>

```



### 选择列表前面 选中的项

> **$(".itemdeletecbclass:checked")**

```html
<div class="table-responsive mailbox-messages">
    <table class="table table-hover table-striped">
        <thead>
        <tr>
            <th><input type="checkbox"></th>
            <th>业务大类</th>
            <th>业务小类</th>
            <th>结算类型</th>
            <th>消费金额(元)</th>
            <th>优惠金额(元)</th>
            <th>实收金额(元)</th>
        </tr>
        </thead>
        <tbody id="titembody">
        <#--                                    <tr>-->
        <#--                                        <td><input class='itemdeletecbclass' type="checkbox"></td>-->
        <#--                                        <td>售卖</td>-->
        <#--                                        <td>二手车</td>-->
        <#--                                        <td>服务费</td>-->
        <#--                                        <td>5,480.00</td>-->
        <#--                                        <td>159.86</td>-->
        <#--                                        <td>5,320.14</td>-->
        <#--                                    </tr>-->

        </tbody>
        <tfoot>
        <tr>
            <th></th>
            <th></th>
            <th></th>
            <th></th>
            <th>总消费金额:<span id="amountspan">0</span>元</th>
            <th>总优惠金额:<span id="disaccountamountspan">0</span>元</th>
            <th>总实收金额:<span id="abc">0</span>元</th>
        </tr>
        </tfoot>
    </table>
</div>

<script>
$(".itemdeletecbclass:checked").parent().parent().remove();

</script>
```

### 将字符串转换为JSON对象

```js
JSON.parse('${groupItemListJSON}')
```

### 从一个select移到另一个select（相互移动）

```html
<script>
     function moveSelected(src, dest){
            $("."+dest).append( $("."+src+">option:selected"));
        }
        function moveAll(src, dest){
            $("."+dest).append( $("."+src+">option"));
        }
</script>


            <select multiple class="form-control allPermissions" size="15">
                <#list permissions as itm>
                    <option value="${itm.id}">${itm.name}</option>
                </#list>
            </select>


        <div class="col-sm-1" style="margin-top: 60px;" align="center">
            <div>
                <a type="button" class="btn btn-primary" style="margin-top: 10px" title="右移动"
                   onclick="moveSelected('allPermissions', 'selfPermissions')">
                    <span class="glyphicon glyphicon-menu-right"></span>
                </a>
            </div>
            <div>
                <a type="button" class="btn btn-primary " style="margin-top: 10px" title="左移动"
                   onclick="moveSelected('selfPermissions', 'allPermissions')">
                    <span class="glyphicon glyphicon-menu-left"></span>
                </a>
            </div>
            <div>
                <a type="button" class="btn btn-primary " style="margin-top: 10px" title="全右移动"
                   onclick="moveAll('allPermissions', 'selfPermissions')">
                    <span class="glyphicon glyphicon-forward"></span>
                </a>
            </div>
            <div>
                <a type="button" class="btn btn-primary " style="margin-top: 10px" title="全左移动"
                   onclick="moveAll('selfPermissions', 'allPermissions')">
                    <span class="glyphicon glyphicon-backward"></span>
                </a>
            </div>
        </div>


            <select multiple class="form-control selfPermissions" size="15" name="ids">
                <#list permissions1 as itm>
                    <option value="${itm.id}">${itm.name}</option>
                </#list>
            </select>

```

#### 去重

```js
            var $options = $(".selfPermissions>option");
            var arr= [];
            $options.each(function (i,that) {
                arr.push($(that).val());
            })
            $(".allPermissions>option").each(function (i,that) {
                if (arr.indexOf($(that).val())>=0) {
                    $(that).remove();
                }
            })
        })
```


