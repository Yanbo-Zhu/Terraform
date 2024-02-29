

# 1 文件  .terraform-docs.yml 的位置 和执行


The default name of the configuration file is `.terraform-docs.yml`. The path order for locating it is:

1. root of module directory
2. `.config/` folder at root of module directory (since v0.15.0)
3. current directory
4. `.config/` folder at current directory (since v0.15.0)
5. `$HOME/.tfdocs.d/`

优先级： if `.terraform-docs.yml` is found in any of the folders above, that will take precedence and will override the other ones.


```
 tree
.
├── main.tf
├── ...
├── ...
└── .terraform-docs.yml


```

```
$ terraform-docs -c .terraform-docs.yml .

或者 

$ terraform-docs .
```


---


```

$ pwd
/path/to/parent/folder

$ tree
.
├── module-a
│   └── main.tf
├── module-b
│   └── main.tf
├── ...
└── .terraform-docs.yml


```

执行

```

# executing from parent
$ terraform-docs -c .terraform-docs.yml module-a/

# executing from child
$ cd module-a/
$ terraform-docs -c ../.terraform-docs.yml .

# or an absolute path
$ terraform-docs -c /path/to/parent/folder/.terraform-docs.yml .

```


# 2 .terraform-docs.yml 中的内容 

```yaml
formatter: "" # this is required

version: ""

header-from: main.tf
footer-from: ""

recursive:
  enabled: false
  path: modules

sections:
  hide: []
  show: []

  hide-all: false # deprecated in v0.13.0, removed in v0.15.0
  show-all: true  # deprecated in v0.13.0, removed in v0.15.0

content: ""

output:
  file: ""
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->    

output-values:
  enabled: false
  from: ""

sort:
  enabled: true
  by: name

settings:
  anchor: true
  color: true
  default: true
  description: false
  escape: true
  hide-empty: false
  html: true
  indent: 2
  lockfile: true
  read-comments: true
  required: true
  sensitive: true
  type: true
```


## 2.1 output

```yaml
output:
  file: README.md
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->    
```


## 2.2 formatter 

The following options are supported out of the box by terraform-docs and can be used for `FORMATTER_NAME`:

- `asciidoc` [reference](https://terraform-docs.io/reference/asciidoc/)
- `asciidoc document` [reference](https://terraform-docs.io/reference/asciidoc-document/)
- `asciidoc table` [reference](https://terraform-docs.io/reference/asciidoc-table/)
- `json` [reference](https://terraform-docs.io/reference/json/)
- `markdown` [reference](https://terraform-docs.io/reference/markdown/)
- `markdown document` [reference](https://terraform-docs.io/reference/markdown-document/)
- `markdown table` [reference](https://terraform-docs.io/reference/markdown-table/)
- `pretty` [reference](https://terraform-docs.io/reference/pretty/)
- `tfvars hcl` [reference](https://terraform-docs.io/reference/tfvars-hcl/)
- `tfvars json` [reference](https://terraform-docs.io/reference/tfvars-json/)
- `toml` [reference](https://terraform-docs.io/reference/toml/)
- `xml` [reference](https://terraform-docs.io/reference/xml/)
- `yaml` [reference](https://terraform-docs.io/reference/yaml/)



常用 markdown 


## 2.3 Content 

https://terraform-docs.io/user-guide/configuration/content/

普通情况下  .terraform-docs.yml 中 不需要写出 content entry 

```
---
formatter: "markdown"

output:
  file: README.md
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->
    
```


-----
Generated content can be customized further away with `content` in configuration. If the `content` is empty the default order of sections is used.
Compatible formatters for customized content are `asciidoc` and `markdown`. `content` will be ignored for other formatters.

`content` is a Go template with following additional variables:

- `{{ .Header }}`
- `{{ .Footer }}`
- `{{ .Inputs }}`
- `{{ .Modules }}`
- `{{ .Outputs }}`
- `{{ .Providers }}`
- `{{ .Requirements }}`
- `{{ .Resources }}`

and following functions:

- `{{ include "relative/path/to/file" }}`

These variables are the generated output of individual sections in the selected formatter. For example `{{ .Inputs }}` is Markdown Table representation of _inputs_ when formatter is set to `markdown table`.

Note that sections visibility (i.e. `sections.show` and `sections.hide`) takes precedence over the `content`.

Additionally there's also one extra special variable avaialble to the `content`:

- `{{ .Module }}`

As opposed to the other variables mentioned above, which are generated sections based on a selected formatter, the `{{ .Module }}` variable is just a `struct` representing a [Terraform module](https://pkg.go.dev/github.com/terraform-docs/terraform-docs/terraform#Module).

```yaml
content: |-
  Any arbitrary text can be placed anywhere in the content

  {{ .Header }}

  and even in between sections

  {{ .Providers }}

  and they don't even need to be in the default order

  {{ .Outputs }}

  include any relative files

  {{ include "relative/path/to/file" }}

  {{ .Inputs }}

  # Examples

  `` `hcl
  {{ include "examples/foo/main.tf" }}
  `` `

    ## Resources

  {{ range .Module.Resources }}
  - {{ .GetMode }}.{{ .Spec }} ({{ .Position.Filename }}#{{ .Position.Line }})
  {{- end }}
```



----

In the following example, although `{{ .Providers }}` is defined it won’t be rendered because `providers` is not set to be shown in `sections.show`:

```yaml
sections:
  show:
    - header
    - inputs
    - outputs

content: |-
  {{ .Header }}

  Some more information can go here.

  {{ .Providers }}

  {{ .Inputs }}

  {{ .Outputs }}  
```

