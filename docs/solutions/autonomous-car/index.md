

## Physical Deployment

![](./diagrams/acr/acr-maptoaws.drawio.png)


### CarRideManager

The component is a Java Quarkus Application running as container inside a ECS cluster with Fargate runtime. The following figure illustrates the main components defined in the [cdk definition]().

![](./diagrams/carride-deployment.drawio.png)

The RDS Postgresql database is in the private subnet. Database admin user is created in AWS Secret Manager.