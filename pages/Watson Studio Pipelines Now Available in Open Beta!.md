title:: Watson Studio Pipelines Now Available in Open Beta!
author:: [[IBM Developer]]
url:: https://developer.ibm.com/blogs/kubeflow-pipelines-and-tekton-advances-data-workloads/
tags:: #[[devops]] #[[Readwise]] #[[mlops]] #[[Readwise]]

- > The KFP-Tekton team also worked on a new way to enable users to plug their own Tekton custom task into a Kubeflow Pipeline. For example, a user might want to calculate an expression without creating a new worker pod. In this case, the user can plug in the Common Expression Language (CEL) custom task from Tekton to calculate the expression inside a shared controller without creating a new worker pod. ([View Highlight](https://read.readwise.io/read/01hakgez9ssqsdsm6c4dht3kkr))
- > Kubeflow Pipelines caching provides task-level output caching. Unlike Argo, by design, Tekton doesn’t generate the task template in the annotations to perform caching. To support caching on Tekton, we enhanced the KubeFlow Pipeline cache server to auto-generate the task template for Tekton as the hash code which caches all the identical workloads with the same inputs. ([View Highlight](https://read.readwise.io/read/01hakgn3x97ywfy93pv5mzhxq7))
	- 在pipeline 之间也提供了task级别的缓存支持。