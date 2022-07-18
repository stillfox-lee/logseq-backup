- 特性
	- DSL——Gomega
	- BDD *Behavior Driven Development*
	- 高效的 CLI
	- 满足单元测试和集成测试
	- 可并行化集成测试
	-
- 一些概念：
	- Spec——单个测试，它们相互之间应该是独立的。
	- Suite——由 spec 组成的一套测试
- ### 编写 Spec
  
  ```go
  var _ = Describe("Books", func() {
    var foxInSocks, lesMis *books.Book
  
    BeforeEach(func() {
      lesMis = &books.Book{
        Title:  "Les Miserables",
        Author: "Victor Hugo",
        Pages:  2783,
      }
  
      foxInSocks = &books.Book{
        Title:  "Fox In Socks",
        Author: "Dr. Seuss",
        Pages:  24,
      }
    })
  
    Describe("Categorizing books", func() {
      Context("with more than 300 pages", func() {
        It("should be a novel", func() {
          Expect(lesMis.Category()).To(Equal(books.CategoryNovel))
        })
      })
  
      Context("with fewer than 300 pages", func() {
        It("should be a short story", func() {
          Expect(foxInSocks.Category()).To(Equal(books.CategoryShortStory))
        })
      })
    })
  })
  ```
- Describe：
	- Ginkgo DSL 的一部分，由一个 description 和闭包函数构成。`Describe("Books", func() {})`会创建一个**container**，它包含着对于`Books`行为的描述。
	- Describe 由多个 Spec 构成。Ginkgo 使用闭包函数来建立 Spec 的层级关系，这种层级关系由三种不同的 `nodes`构成。
- nodes：
	- container nodes——`Describe` `Context` `When`
	- setup nodes—— `BeforeEach`
	- subject nodes—— `It`
	- Container nodes
	  通过使用`Describe` `Context`这样的**Container node**来组织测试代码的层级关系。在这个例子中，Describe 下的两个 Context 描述了在两种不同的背景下book 应该具有的能力。
	- Setup nodes
	  通过使用`BeforeEach`这样的 **setup node** 来为 Spec 设置一些前置条件。
	- Subject nodes
	  通过使用`It`这样的 **subject node** 来编写一个规范。该规范对被测试的 subject 做出 assert。
	- > container, setup和 subject node 共同构成了一个 spec tree。每个 spec 可以有多个 setup nodes，但只能有一个 subject node。
- Setup&Teardown
	- `BeforeEach`
		- 在每个 `It` 之前会运行，用于做一些测试之前的准备工作。
	- `JustBeforeEach`
		- 在`It`之前，但是在`BeforeEach`之后运行
		- 示例代码：
		  collapsed:: true
			- ```go
			  Describe("some JSON decoding edge cases", func() {
			    var book *books.Book
			    var err error
			    var json string
			    JustBeforeEach(func() {
			      book, err = NewBookFromJSON(json)
			      Expect(book).To(BeNil())
			    })
			  
			    When("the JSON fails to parse", func() {
			      BeforeEach(func() {
			        json = `{
			          "title":"Les Miserables",
			          "author":"Victor Hugo",
			          "pages":2783oops
			        }`
			      })
			  
			      It("errors", func() {
			        Expect(err).To(MatchError(books.ErrInvalidJSON))
			      })
			    })
			  
			    When("the JSON is incomplete", func() {
			      BeforeEach(func() {
			        json = `{
			          "title":"Les Miserables",
			          "author":"Victor Hugo",
			        }`
			      })
			      
			      It("errors", func() {
			        Expect(err).To(MatchError(books.ErrIncompleteJSON))
			      })
			    })      
			  })
			  ```
	- `AfterEach`
		- 在每个`It`之后，做一些清理工作。
	- `JustAfterEach`
	-
	- `DeferCleanup`
	-
- 组织 container node
	- `Describe` ——
	- `Context` —— 在`Describe`下的场景
	- `When` —— 支持更细的粒度
	- 三者的范围内都可以支持`setup node`
	-