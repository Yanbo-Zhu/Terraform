
# 1 command autocomplete 

## 1.1 For Bash or zsh: 
https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#enable-tab-completion

run terraform -install-autocomplete
然后在 touch ~/.bashrc 中 加入了 
    complete -C C:\ProgramData\chocolatey\lib\terraform\tools\terraform.exe terraform.exe
上面的 text 

## 1.2 For Powershell 

https://github.com/shellwhale/terraform-target-autocompletion

需要结合 --target 一起使用 
Press tab after --target and get autocomplete suggestions for your resources and modules.

![](image/Pasted%20image%2020231025140642.png)


# 2 tfenv
https://github.com/tfutils/tfenv

Terraform version manager inspired by rbenv 

使得 同一个 机器上 可以使用 不同的  Terraform version 

安装教程见 https://www.bilibili.com/video/BV1Yf4y1c77Y?p=6&vd_source=55e5cc2f534c16c73bbeb684e98c4195

