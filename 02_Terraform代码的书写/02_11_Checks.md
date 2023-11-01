
# 1 Checks

`check` 块是 Terraform 1.5 开始引入的新功能。

过去我们可以在 `resource` 块里的 `lifecycle` 块中验证基础设施的状态。`check` 块填补了在 Terraform `apply` 后验证基础设施状态这一功能中的一块空白。

`check` 块允许我们定义在每次 `plan` 以及 `apply` 操作后执行的自定义的验证。`check` 块定义的验证逻辑是作为 `plan` 和 `apply` 操作的最后一步执行的。

## 1.1 语法

你可以定义一个包含本地名称的 `check` 块，其中可以定义一个 [有限作用范围的 `data` 块](https://lonegunmanb.github.io/introduction-terraform/3.11.checks.html#有限作用范围的数据源)，以及至少一个的[断言](https://lonegunmanb.github.io/introduction-terraform/3.11.checks.html#断言)。

下面的例子演示了加载 Terraform 官网并验证 HTTP 返回状态码为 `200`。

```
check "health_check" {
  data "http" "terraform_io" {
    url = "https://www.terraform.io"
  }

  assert {
    condition = data.http.terraform_io.status_code == 200
    error_message = "${data.http.terraform_io.url} returned an unhealthy status code"
  }
}
```

### 1.1.1 有限作用范围的数据源

我们可以在 `check` 块使用任意 Provider 提供的任意数据源作为一个有限作用范围的数据源。

一个 `check` 块可以配一个可选的内嵌（也叫有限作用范围）数据源。该 `data` 块和普通的 `data` 块行为类似，但你不能在定义它的 `check` 块以外引用它。另外，如果一个有限作用范围的数据源运行时触发了任意错误，这些错误将被标记为警告，不会阻止 Terraform 继续执行操作。

你可以使用有限作用范围的数据源在 `resource` 的 `lifecycle` 外验证相关基础设施片段的状态。在上面的例子里，如果 `terraform_io` 数据源在加载时发生错误，那么我们将会收到一个警告而不是中断执行的错误。

#### 1.1.1.1 元参数

有限作用域的数据源支持 `depends_on` 和 `provider` [元参数](https://lonegunmanb.github.io/introduction-terraform/3.6.资源.html#元参数)，但不支持 `count` 或 `for_each` 元参数。

`depends_on`

`depends_on` 元参数配合有限作用域数据源可以提供非常强大的能力。

假设上述例子中的 Terraform 网站是我们即将用同一目录下的 Terraform 代码部署的，在第一次创建 Plan 时因为网站还没有被创建，所以验证会失败，Terraform 总是会在一开始显示一条让人分心的警告信息。

我们可以给该内嵌数据源添加 [`depends_on`](https://lonegunmanb.github.io/introduction-terraform/3.6.资源.html#depends_on) 来确保该数据源依赖于某项组成基础设施的必要资源，例如负载均衡器。这样对该数据源的检查结果将保持 `known after apply` 直到依赖项创建完成。该策略避免了在配置阶段产生无意义的警告信息，直到在 `plan` 和 `apply` 操作的合适阶段执行检查。

该策略的一个问题是如果有限作用域数据源所依赖的资源发生了变化，那么 `check` 块将返回 `known after apply` 直到 Terraform 完成了对被依赖资源的更新。在某些情况下，这种行为将会引发一些问题。

我们推荐只有在内嵌数据源依赖于某项资源，但又没有显式的引用其数据时使用 `depends_on` 元参数。

### 1.1.2 断言

我们在 `check` 块中使用 `assert` 块定义自定义的断言条件。每个 `check` 块必须声明至少一个或更多的 `assert` 块。每个 `assert` 块都包含了一个 `condition` 属性与一个 `error_message` 属性。

与其他自定义检查（`variable` 中的 `validation` 以及 `lifecycle` 中的 `precondition` 和 `postcondition`）不同，`assert` 的断言不会影响 Terraform 执行操作。失败的断言将以警告信息的形式输出而不会中断后续的操作。这与其他诸如 `postcondition` 这样的自定义检查形成了对比，因为它们的检查失败会立即终止后续的 `plan` 以及 `apply` 操作，返回错误信息。

`assert` 块中的断言条件表达式可以引用同一 `check` 块里的内嵌数据源数据，以及同一模块中的任意输入参数、资源、数据源、模块的输出值。

### 1.1.3 check 块的元参数

`check` 块目前不支持元参数。Terraform 团队目前正在[收集](https://github.com/hashicorp/terraform/issues/new/choose)有关这一功能的反馈。

## 1.2 是使用 check 块还是其他自定义条件检查

`check` 块提供了 Terraform 中最灵活的验证功能。我们可以在其中引用输出值、输入参数、资源以及数据源的值。我们的确可以使用 `check` 块取代所有其他的自定义条件检查，但这并不意味着我们应该要这么做。

`check` 与其他检查最大的区别在于 `check` 块不会中断 Terraform 的执行。我们需要将这种非阻塞性的行为特点计入考量来决定采取何种检查。

### 1.2.1 输出值与输入参数

[输出值的 `precondition`](https://lonegunmanb.github.io/introduction-terraform/3.4.输出值.html#precondition) 以及 [输入变量的 `validation`](https://lonegunmanb.github.io/introduction-terraform/3.3.输入变量.html#断言)都可以对输入输出值进行断言。

这些检查是用来阻止 Terraform 在数据有问题时继续执行的。

举例来说，如果输入参数的值是无效的, 那么任由 Terraform 执行整个配置文件并没有什么意义. 这种情况下，`check` 块只会输出有关无效输入参数的警告，不会打断 Terraform 的执行，而 `validation` 块则会警告输入参数值非法，并终止 Terraform 执行 `plan` 或 `apply` 操作。

### 1.2.2 resource 块的 precondition 与 postcondition

`check` 块与 [`precondition` 和 `postcondition`](https://lonegunmanb.github.io/introduction-terraform/3.6.资源.html#precondition-与-postcondition) 的区别更加微妙。

`precondition` 是自定义条件检查中最特殊的，因为它们是在资源的变更被计算或应用之前执行的检查。决定使用 `precondition` 还是 `postcondition` 的考量也适用于选择是使用 `precondition` 还是 `check` 块。

我们可以在 `postcondition` 与 `check` 块之间互换来验证资源和数据源。例如，我们可以把上述例子中的 `check` 块改写成 `postcondition`，以下的 `postcondition` 块将会验证对 Terraform 网站的请求是否返回了状态码 `200`：

```
data "http" "terraform_io" {
  url = "https://www.terraform.io"

  lifecycle {
    postcondition {
        condition = self.status_code == 200
        error_message = "${self.url} returned an unhealthy status code"
    }
  }
}
```

`check` 和 `postcondition` 块都在 `plan` 或 `apply` 操作中验证了 Terraform 网站是否返回 `200` 状态码，它们的区别是发生错误时的行为。

如果是 `postcondition` 失败，那么将无法继续执行。Terraform 会阻止任意后续的 `plan` 或 `apply` 操作。

==我们推荐使用 `check` 块来验证基础设施的整体状态，仅在希望确保单一资源状态符合预期时使用 `postcondition`。==



