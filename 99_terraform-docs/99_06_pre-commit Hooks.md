
ensure your Terraform module documentation is kept up-to-date each time you make a commit,

https://github.com/antonbabenko/pre-commit-terraform

# 1 pre-commit Hooks

This is very basic and higly simplified version of [pre-commit-terraform](https://github.com/antonbabenko/pre-commit-terraform). Please refer to it for complete examples and guides.
https://github.com/antonbabenko/pre-commit-terraform

With [`pre-commit`](https://pre-commit.com/), you can ensure your Terraform module documentation is kept up-to-date each time you make a commit.

1. simply create or update a `.pre-commit-config.yaml` in the root of your Git repo with at least the following content:
    
    ```yaml
    repos:
      - repo: https://github.com/terraform-docs/terraform-docs
        rev: "<VERSION, TAG, OR SHA TO USE>"             # e.g. "v0.11.2"
        hooks:
          - id: terraform-docs-go
            args: ["ARGS", "TO PASS", "INCLUDING PATH"]  # e.g. ["--output-file", "README.md", "./mymodule/path"]
    ```

    1. You can also include more than one entry under `hooks:` to update multiple docs. Just be sure to adjust the `args:` to pass the path you want terraform-docs to scan.
    
2. install [`pre-commit`](https://pre-commit.com/) and run `pre-commit` ( or `pre-commit install`  and  `pre-commit install-hooks` ) to activate the hooks.
3. make a Terraform change, `git add` and `git commit`. pre-commit will regenerate your Terraform docs, after which you can rerun `git add` and `git commit` to commit the code and doc changes together.

You can also regenerate the docs manually by running
pre-commit run terraform-docs-go
pre-commit run -a 



---

我用的 `.pre-commit-config.yaml`

```yaml
fail_fast: false

repos:
-   repo: https://github.com/terraform-docs/terraform-docs
    rev: v0.17.0
    hooks:
    -   id: terraform-docs-go
        args: ["."]

```


> 进过我的验证， 上面的这种 方法， 并不能在 执行 `git add` or `git commit` 的时候 ， 连动的 执行 其他的， 使得 terraform documanetaion 一起被更新 




# 2 使用 docker 

 pre-commit via Docker[](https://terraform-docs.io/how-to/pre-commit-hooks/#pre-commit-via-docker)

The pre-commit hook can also be run via Docker, for those who don’t have Go installed. Just use `id: terraform-docs-docker` in the previous example.

This will build the Docker image from the repo, which can be quite slow. To download the pre-built image instead, change your `.pre-commit-config.yaml` to:

```yaml
repos:
  - repo: local
    hooks:
      - id: terraform-docs
        name: terraform-docs
        language: docker_image
        entry: quay.io/terraform-docs/terraform-docs:latest  # or, change latest to pin to a specific version
        args: ["ARGS", "TO PASS", "INCLUDING PATH"]          # e.g. ["--output-file", "README.md", "./mymodule/path"]
        pass_filenames: false
```

# 3 使用 Git Hook[](https://terraform-docs.io/how-to/pre-commit-hooks/#git-hook)

A simple git hook (`.git/hooks/pre-commit`) added to your local terraform repository can keep your Terraform module documentation up to date whenever you make a commit. See also [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) documentation.


```sh
#!/bin/sh

# Keep module docs up to date
for d in modules/*; do
  if terraform-docs md "$d" > "$d/README.md"; then
    git add "./$d/README.md"
  fi
done
```


0
这种方法 ， 在 powershell in windows (not WSL2) 也是有效的 ， 不需要另外 安装 bash.exe, 不需要 Ensure your PATH environment variable looks for `bash.exe` in `C:\Program Files\Git\bin` (the one present in `C:\Windows\System32\bash.exe` does not work with `pre-commit.exe`)

0 我自己使用的   (`.git/hooks/pre-commit`)

```
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.

echo "Keep Terraform module documentation README.md file up to date whenever you make a commit"

# Keep module docs up to date
if terraform-docs . ; then
  git add "./README.md"
fi

```


我的文件结果
![](image/Pasted%20image%2020240302164412.png)

1
`if terraform-docs md "$d" > "$d/README.md"` 会使得 `terraform-docs md <Path>` 产生的 README.md 完全 覆盖  "$d/README.md"


2
.terraform-docs.yml

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

content: content: |-
  Any arbitrary text can be placed anywhere in the content

```

README.md

```md

# terraform-mymodule

Describe in one sentence what this module does

## Description

Put description here


<!-- BEGIN_TF_DOCS -->
<!-- END_TF_DOCS -->

## Development

### Development Requirements

- access to git.ivu-ag.com and the Terraform repositories
- access to what you want to manage with Terraform (like, AWS, Vault, Artifactory, ...)
- terraform ([-> installation instructions](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli))
- terraform-docs ([-> installation instructions](https://terraform-docs.io/user-guide/installation))
- git ([-> installation instruction](https://github.com/git-guides/install-git))


```


上这种情况  使用  `terraform-docs . `  产生出的内容，可以使得 正确的被 填入  README.md 中我们想要的位置中 。 

如使用  `terraform-docs md <Path>`， 原来 整个README.md  都会被覆盖， 产生的内容 不会填入到 我们想要的位置上 

