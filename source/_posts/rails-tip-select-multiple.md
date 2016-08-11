---
title: Rails Tip:select多选后,后台如何接收参数数组
category: Ruby On Rails
tags:
  - Ruby On Rails
date: 2016-07-13 17:53:19
---
在rails中,当下拉框select 可以多选时,如果需要在后台接收到数组参数,需要进行如下设置.
1,html中 name属性需要设置为数组形式,如下
```html
<select name="site_event[]" id="site_event" v-model='site_event_selected' multiple>
  <option value="-1" selected>--选择站点事件--</option>
  <option v-for='option in site_event_options' v-bind="{value: option.eventid}" >{{option.actionname}}</option>
</select>
```
2,后台接收时如下
```ruby
def query_pv_uv_by_day
  site_events = params[:site_event].flatten
  puts site_events
end
```
