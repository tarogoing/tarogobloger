

# 使用Lua脚本扩展PowerDNS功能

十一月 28th, 2012 | 使用Lua脚本扩展PowerDNS功能已关闭评论

PowerDNS是一个高性能的域名服务器，在无需重新编译源代码的情况下，我们可以利用Lua脚本对其进行扩展，使其满足我们复杂的业务需求。

## DNS功能扩展的需求

当我们直接使用一个域名服务器提供递归查询服务的时候，域名服务器所提供的正常服务是完全能够满足我们的需求的，域名服务器的任务就是把域名转化为正确的IP地址，除此之外，我们还能奢求什么呢？

但非正常的请求或非正常的相应就会使我们对域名服务器提出额外的需求，比如当用户输入一个错误的域名，我们希望可以友好地提醒用户他输入的域名存在问题，而不是让浏览器出现一个错误提示页。

## PowerDNS提供的hook

在PowerDNS中，对域名递归查询提供了了4个查询钩子用于对DNS功能进行扩展

### 1\. preresolve

preresolve钩子在DNS查询前被调用，他可以直接返回一条DNS记录给客户端，也可以封禁掉某些域名解析，还可以用来对付针对DNS进行攻击的工具。

### 2\. postresolve

postresolve钩子在递归查询结束之后，结果返回给客户端之前被调用，这个钩子一般用来修改DNS的查询结果。

### 3\. nxdomain

当你查询的某个域名是不存在的时候，nxdomain钩子就开始起作用了，这个钩子的可应用范围也很广泛。

### 4\. nodata

nxdomain钩子和nxdomain钩子有点类似，但nxdomain钩子是在域名存在，但没有这个查询类型的时候生效。

## Lua示例代码

官方对具体Lua脚本的API说明不是很清楚，通过这个来源于源代码的示例代码可以了解一些的功能与用法，具体配置时候只要在服务器上新建一个lua脚本，在配置文件lua-dns-script中指定这个文件的路径，然后启动服务即可。

[http://wiki.powerdns.com/trac/browser/trunk/pdns/pdns/powerdns-example-script.lua](https://wiki.powerdns.com/trac/browser/trunk/pdns/pdns/powerdns-example-script.lua)

```
function preresolve ( remoteip, domain, qtype )
  print ("prequery handler called for: ", remoteip, getlocaladdress(), domain, qtype)
  pdnslog("a test message.. received query from "..remoteip.." on "..getlocaladdress());
 
  if domain == "www.donotcache.org."
  then
      print("making sure www.donotcache.org will never end up in the cache")
      setvariable()
      return -1, {}
  end
 
  if domain == "www.powerdns.org."
  then
      ret={}
      ret[1]= {qtype=pdns.A, content="85.17.220.215", ttl=86400}
      print "dealing!"
      return 0, ret
  elseif domain == "www.baddomain.com."
  then
      print "dealing - faking nx"
          return pdns.NXDOMAIN, {}
  elseif domain == "echo."
  then
      print "dealing with echo!"
      return 0, 
  elseif domain == "echo6."
  then
      print "dealing with echo6!"
      return 0, 
  else
      print "not dealing!"
      return -1, {}
  end
end
 
function nxdomain ( remoteip, domain, qtype )
  print ("nxhandler called for: ", remoteip, getlocaladdress(), domain, qtype, pdns.AAAA)
  if qtype ~= pdns.A then return -1, {} end  --  only A records
  if not string.find(domain, "^www%.") then return -1, {} end  -- only things that start with www.
 
  setvariable()
  if matchnetmask(remoteip, {"127.0.0.1/32", "10.1.0.0/16"})
  then
      print "dealing"
      ret={}
      ret[1]={qtype=pdns.CNAME, content="www.webserver.com", ttl=3602}
      ret[2]={qname="www.webserver.com", qtype=pdns.A, content="1.2.3.4", ttl=3602}
      ret[3]={qname="webserver.com", qtype=pdns.NS, content="ns1.webserver.com", place=2}
--       ret[1]={15, "25 ds9a.nl", 3602}
      return 0, ret
  else
      print "not dealing"
      return -1, ret
  end
end
 
function axfrfilter(remoteip, zone, qname, qtype, ttl, priority, content)
  if qtype ~= pdns.SOA or zone ~= "secured-by-gost.org"
  then
      ret = {}
      return -1, ret
  end
 
  print "got soa!"
  ret={}
  ret[1]={qname=qname, qtype=qtype, content=content, ttl=ttl}
  ret[2]={qname=qname, qtype=pdns.TXT, content=os.date("Retrieved at %Y-%m-%d %H:%M"), ttl=ttl}
  return 0, ret
end
 
function nodata ( remoteip, domain, qtype, records )
  print ("nodata called for: ", remoteip, getlocaladdress(), domain, qtype)
  if qtype ~= pdns.AAAA then return -1, {} end  --  only AAAA records
 
  setvariable()
    return "getFakeAAAARecords", domain, "fe80::21b:77ff:0:0"
end    
 
-- records contains the entire packet, ready for your modifying pleasure
function postresolve ( remoteip, domain, qtype, records, origrcode )
  print ("postresolve called for: ", remoteip, getlocaladdress(), domain, qtype, origrcode)
 
  for key,val in ipairs(records)
  do
      if(val.content == '173.201.188.46' and val.qtype == pdns.A)
      then
          val.content = '127.0.0.1'
          setvariable()
      end
      if val.qtype == pdns.A and matchnetmask(remoteip, "192.168.0.0/16") and matchnetmask(val.content, "85.17.219.0/24")
      then
          val.content = string.gsub(val.content, "^85.17.219.", "192.168.219.", 1)
          setvariable()
      end
 
  -- print(val.content)
  end
  return origrcode, records
end    
 
function prequery ( dnspacket )
  -- pdnslog ("prequery called for ".. tostring(dnspacket) )
  qname, qtype = dnspacket:getQuestion()
  pdnslog ("q: ".. qname.." "..qtype)
  if qtype == pdns.A and qname == "www.domain.com"
  then
      pdnslog ("calling dnspacket:setRcode")
      dnspacket:setRcode(pdns.NXDOMAIN)
      pdnslog ("called dnspacket:setRcode")
      pdnslog ("adding records")
      ret = {}
      ret[1] = {qname=qname, qtype=qtype, content="1.2.3.4", place=2}
      ret[2] = {qname=qname, qtype=pdns.TXT, content=os.date("Retrieved at %Y-%m-%d %H:%M"), ttl=ttl}
      dnspacket:addRecords(ret)
      pdnslog ("returning true")
      return true
  end
  pdnslog ("returning false")
  return false
end
```


Copyright © 2011 - 猫言猫语 - Powered by [Wordpress](https://wordpress.org)