---
title: "jquery的serialize()不提交未选中的checkbox的问题"
slug: "Jquery_serialize_checkbox_no_checked_config"
description: "jquery的$(form).serialize()不会提交未选中的checkbox，解决方法是未选中时向页面插入一个同name的隐藏input。"
date: "2018-09-17T14:18:54+08:00"
thumbnail: ""
categories:
  - "develop"
tags:
  - "jquery"
  - "serialize"
  - "checkbox"
---
jquery的$(form).serialize()不会提交未选中的checkbox，解决方法是未选中时向页面插入一个同name的隐藏input。

```javascript
$('input[type="checkbox"]').on("click", function () {
    var name = $(this).attr("name");
    if ($(this).checked) {
        $(this).val(1);
        $(this).parent().find('input[type="hidden"][name="' + name + '"]').remove();
    } else {
        $(this).val(0);
        $(this).parent().append('<input type="hidden" name="' + name + '" value="0">');
    }
});
```

经过此方法设置后，checkbox序列被完整提交，原本被用户选定的checkbox的value为1，未被选定的value为0。

