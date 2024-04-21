

# 1 user data 的作用



![](image/Pasted%20image%2020231124135236.png)


user_data attribute
- be used to initial data when launch the instance 
- the content in user_data will be passed to cloud provider 

Terraform only provision the ec instance. once the ec2 instance is created by terraform, then terraform get a state of ec2 instance.  Terraform will not wait the VM to be initialized. 
Then the cloud provider (not terraform ) will execute the user_data on ec2 instnace.  这部分 Terraform 根本不管


---


![](image/Pasted%20image%2020231124133345.png)



or reference a file 

![](image/Pasted%20image%2020231124133452.png)

# 2 Provisioner: remote-exec, file, local-exec


![](image/Pasted%20image%2020231124140414.png)
![](image/Pasted%20image%2020231124142810.png)

![](image/Pasted%20image%2020231124143235.png)

## 2.1 Connection Block

connection 必须要在 provisioner 块前面 , 用来先确保 连接上了

self 指的是  当前所在的 resource 块 的 resource 本身 

![](image/Pasted%20image%2020231124141721.png)



## 2.2 Connection Block inside the Provisioner Block


这样就可以定义 这个 Provisioner block 到底是 要在那个 server 上执行了. 因为 Connnection block 中 顶一个 要连接到 那个 Server


![](image/Pasted%20image%2020231124143146.png)

## 2.3 remote-exec 

connection types are supported by the remote-exec provisioner: ssh and winrm 

"The remote-exec provisioner invokes a script on a remote resource after it is created."
The remote-exec provisioner invokes a script on a remote resource created by Terraform. It connects to the resource using SSH or WinRM and run the provided inline or script commands.

![](image/Pasted%20image%2020231124142718.png)


## 2.4 file 

The file rovisioner is used to co files or directories from the machine executin Terraform to the newl created resource.

![](image/Pasted%20image%2020231124142917.png)


## 2.5 local-exec

The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource. (this is from the terraform web site)
![](image/Pasted%20image%2020231124143555.png)


# 3 User Data 和 Provisioner 的区别

![](image/Pasted%20image%2020231124141949.png)

user_data 是 terraform 直接交给 aws provider. aws provider 去执行 user_data. terrafrom 根本就不管  aws provider 执行 user_data 的结果是如何.  Terraform 在 create ec2 instance 后 , 就不管了. 把后续的任务交给了 aws provider 



# 4 Provisioner is not commanded 


![](image/Pasted%20image%2020231124144031.png)



![](image/Pasted%20image%2020231124144209.png)

![](image/Pasted%20image%2020231124144310.png)


![](image/Pasted%20image%2020231124144357.png)

