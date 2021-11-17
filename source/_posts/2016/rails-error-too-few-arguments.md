---
title: Rails Error:too few arguments
permalink: rails-error-too-few-arguments
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2016-07-12 17:53:19
---
报错描述:
我有一个ajax请求
```javascript
@get_sites = (data) ->
  $.getJSON window.urls.get_sites,{brandsid:data},(result) ->
    console.log result
```
请求的action
```ruby
  def get_sites
    brandsid = params[:brandsid]
    @sites = Site.find_by_brandsid brandsid
    respond_to do |foramt|
      format.json {render json: @sites }
    end
  end
```
然后报错信息为:

```html
ArgumentError in Backend::ExportPvUvsController#get_sites
too few arguments

Rails.root: D:/work/rails_obj/topinsight_query

Application Trace | Framework Trace | Full Trace
app/controllers/backend/export_pv_uvs_controller.rb:20:in `format'
app/controllers/backend/export_pv_uvs_controller.rb:20:in `block in get_sites'
app/controllers/backend/export_pv_uvs_controller.rb:18:in `get_sites'
Request

Parameters:

{"brandsid"=>"271ee3565366688560fc60eab0f7a5c8"}
Toggle session dump
Toggle env dump
Response

Headers:

None
```
让我郁闷了...我查了一整天的资料都没找到解决方法.所有的问题都是说缺少 respond_to 块.但是我的有啊...
实在憋不住了,上ruby china发帖问了一下.帖子发完没几分钟就被关小黑屋了.
我就纳闷了,我累死查了一整天的问题会这么简单?!不死心的我一边问管理员为啥关小黑屋,一边继续查.
在纠结中等到了李华顺大神的回复.他直接指出是 "format"打错了...
得知原因的我真想找个坑跳进去.怎么说我也是一个老程序员了,怎么能这么粗心.
总结:写C#一直用编译器能大大降低这种低级错误,但是在编辑器上一定要多注意、多注意、多注意！
写此篇文章的目的是警醒自己.