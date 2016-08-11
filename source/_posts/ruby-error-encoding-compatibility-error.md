---
title: Ruby Error:"in `match':incompatible encoding regexp match (UTF-8 regexp with ASCII-8BIT string) (Encoding::CompatibilityError)"
category: Ruby
tags:
  - Ruby
date: 2016-08-10 23:42:43
---
我在抓取一个中文网页时遇到该问题.
出现该问题的原因是字符编码不一致导致的.查看获取到的网页内容编码是 ASCII-8BIT的,所以需要强制转换为UTF-8.
```ruby
    code_html = HTTP.get('http://www.stats.gov.cn/tjsj/tjbz/xzqhdm/201401/t20140116_501070.html').to_s
    puts code_html.encoding.name
    tmp_result = code_html.force_encoding("UTF-8").scan(/\d{6}&nbsp;&nbsp;&nbsp; [\u4e00-\u9fa5]+/)
    puts tmp_result
```