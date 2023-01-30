category:: article
url:: https://gateway-api.sigs.k8s.io/references/policy-attachment/
tags:: #Reading #[[k8s network]] #[[gateway api]]

-
- **超时时间**、**重试**、**健康检测**这些特性，缺乏事实标准。但是 GatewayAPI 的愿景是希望能够建立一个通用的、具有可迁移性的 API。所以，就引入了插件化机制来实现这部分的需求。
- > A standard approach for policy attachment allows implements to create their own custom policy resources that can essentially extend Gateway API.
- ## Policy Attachment for Ingress
	- https://gateway-api.sigs.k8s.io/images/policy/ingress-simple.png
	-
	- 如上，就可以为 Gateway 中的 Route 引入一个 TimeoutPolicy。其下的所有 Route 都可以具有这个 Policy。
	- 一个更加具体的例子：
	  https://gateway-api.sigs.k8s.io/images/policy/ingress-complex.png
	- 一些限制条件
		-
- ## Policy Attachment for Mesh
	- 在 Mesh 中，我们会有更加复杂的使用场景。例如：在**Consumer** namespace 中定制一个访问**Producer** namespace 的超时策略。
	  https://gateway-api.sigs.k8s.io/images/policy/mesh-simple.png
	- 关于 Policy Attachment 的限制：虽然 Policy Attachment 可以在任何的 namespace 中使用。但是，如果涉及到上面的跨 namespace 的引用的话，**就只能对于出方向的 request 定制策略**。
	- > 核心的规则可以理解为：Policy Attachment 只能对本 namespace 生效。
- ## 关于 Traget Reference API
	- 每个 Policy 资源**必须**设置一个`targetRef`字段，这个字段一次只能指定一个资源。它可以指定`Gateway`或者`Namespace`，也可以指定其他子资源。
	-