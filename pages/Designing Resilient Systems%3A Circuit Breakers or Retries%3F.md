title:: Designing Resilient Systems: Circuit Breakers or Retries?
author:: [[Grab Tech]]
url:: https://engineering.grab.com/designing-resilient-systems-part-1
tags:: #[[servicemesh]] #[[Readwise]]

- > With our services communicating with numerous external resources, failures can be caused by:
  
  •   Networking issues
  •   System overload
  •   Resource starvation (e.g. out of memory)
  •   Bad deployment/configuration
  •   Bad request (e.g. lack of authentication credentials, missing request data) ([View Highlight](https://read.readwise.io/read/01gtdfpqh2s862v0gtj1qk8yyb))
- > But circuit breakers are not just about being a *good user* and protecting our upstream services. They are also beneficial for our service as we will see in the next sections. ([View Highlight](https://read.readwise.io/read/01gtdggfrhmgtn5f36ac0gvjnj))
- > In fallback processing, using an estimated value instead of the real value not the only option, other common options include:
  
  •   Retrying the request using a different upstream service
  •   Scheduling the request for some later time
  •   Loading potentially *out of date* data from a cache ([View Highlight](https://read.readwise.io/read/01gtdgvqr77yan2vk9sap1a5rs))
- > One Circuit Per Service ([View Highlight](https://read.readwise.io/read/01gte8taftvzb0a79h25p6mqs3))
	- 这个模式应该指的是：同一个Service下的多个 instance 共用一个断路器。同时 LB 和服务发现都是服务端的实现，在断路器初始化时，就只有一个断路器Name。所以，错误率的设置就很关键了。