
# 1 Container basic 

![](image/Pasted%20image%2020231121223301.png)

最底下的 Server 为 Physical Server 
Guest OS 通常是 Ubuntu Linux or Amazon Linux 

![](image/Pasted%20image%2020231121223501.png)


![](image/Pasted%20image%2020231121224504.png)

Build Contain image via Jenkins
ECR: Amazon EC2 Container Registry , push the Container Image  into ECR
ECS: Amazon EC2 Container Service, Run the Container image via ECS


----

![](image/Pasted%20image%2020231121224529.png)




# 2 Building Docker images 

![](image/Pasted%20image%2020231121225241.png)

 
![](image/Pasted%20image%2020231121225423.png)

![](image/Pasted%20image%2020231121225618.png)


==== Terraform 能做的
1 creation ecr repositoru 


![](image/Pasted%20image%2020231121225657.png)

docker build -t xxxx:1 

-t 为 标记 tag 值为 xxx,   version 为 1 


![](image/Pasted%20image%2020231121230305.png)
# 3 





