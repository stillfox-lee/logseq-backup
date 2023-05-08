- #golang #CleanCode
- ## 区分required和optional 参数
	- 使用基础类型作为 required参数，结构体作为可选参数。以 http client 为例子：
	- ```go
	  type Request struct {
	    Url string
	    Method string
	    Headers map[string]string
	    Body io.Reader
	    SkipSecureVerify bool
	  } // 有些是 required，有些是 optional
	  
	  func NewRequest(url string, method string, headers map[string]string, body io.Reader, isVerify bool) Request {
	    
	  }
	  
	  func main() {
	    // caller
	    requeset := NewRequest("http://", "GET", nil, nil, false) 
	  }
	  ```
	- 这种 nil 很丑陋，可以使用一个 struct作为可选参数：
	- ```go
	  type Request struct {
	    Url string
	    Method string
	  }
	  
	  type Option struct {
	    Headers map[string]string
	    Body io.Reader
	    SkipSecureVerify bool
	  }
	  
	  func NewRequest(url string, method string, opt Option) Request {}
	  
	  func main() {
	    opt := Option{SkipSecureVerify: false}
	    request := NewRequest("http://test.com", "GET", opt)
	  }
	  
	  ```
- ## 使用 Struct 作为可选参数的问题
	- *struct 会为缺省的元素初始化为默认值*
	- 上面的例子，考虑一个场景：默认会开启 HTTPS 的校验，但是在调用`NewRequest`的时候，没有显示指定：
	  ```go
	  func main() {
	    opt := Option{Headers: map[string]string{"Content-Type": "application/json"}}
	    request := NewRequest("https://test.com", "GET", opt)
	  }
	  
	  ```
	- 这样原本需要做 HTTPS 校验，就变成了跳过校验。
- ## 使用指针解决零值问题
	- struct 中所有的成员都替换成对应的指针值。这样的问题就是：在判断的逻辑写得比较复杂。
	- ```go
	  type Option struct {
	    Headers map[string]string
	    Body io.Reader
	    SkipSecureVerify *bool
	  }
	  
	  func NewRequest(url string, method string, opt Option) Request {
	    if opt.Header != nil {
	      
	    }
	    if opt.Body != nil {
	      
	    }
	    if opt.SkipSecureVerify != nil {
	      
	    }
	  }
	  
	  func main() {
	    skip := false
	    opt := Option{SkipSecureVerify: &skip}
	    // ...
	  }
	  ```
	- 还有一个问题就是：每次都需要手动地创建一个临时变量，然后再取指针。写起来比较繁琐。
- ## 使用函数参数解决
	- ```go
	  type Option struct {
	    Headers map[string]string
	    Body io.Reader
	    SkipSecureVerify *bool
	  }
	  
	  type ReqOption interface {
	    ApplyToOpt(*Option)
	  }
	  
	  type WithHeader map[string]string
	  
	  func (o WithHeader) ApplyToOpt(opt *Option) {
	    opt.Headers = o
	  }
	  
	  type WithVerify bool
	  
	  func (o WithVerify) ApplyToOpt(opt *Option) {
	  	skip := bool(o)
	  	opt.SkipSecureVerify = &skip
	  }
	  
	  func NewRequest(url, method string, opts ...ReqOption) Request {
	    defaultOpt := Option{SkipSecureVerify: true}
	  
	    for _, o := range opts {
	      o.ApplyToOpt(&defaultOpt)
	    }
	  
	    request := Request{Url: url, Method: method, defaultOpt}
	  }
	  
	  // caller
	  func main() {
	    header := map[string]string{
	      "Content-Type": "application/json"
	    }
	    req1 := NewRequest("https://test.com", "GET", WithHeader(header))
	    req2 := NewRequest("https://test.com", "GET", WithHeader(header), WithVerify(true))
	  }
	  
	  ```
- TODO:
	- function params
	- builder pattern
	- error handle in two ways
		- New function & Build() function
		- 防御式编程
			- 基础库和对外提供的接口，需要有防御式的意识。
- [ref](https://www.piglei.com/articles/go-func-argument-patterns/)