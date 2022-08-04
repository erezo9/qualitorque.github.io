---
sidebar_position: 3
title: Blueprint Quickstart Guide
---

Now that you have seen how Torque works, it’s time to link your assets to Torque and see what you can do with them.

## Prerequisites

* You have created your own space
* Asset repository associated to the space
* Execution host associated to the space

## Let Torque autogenerate blueprints from your assets

Torque launches environments out of blueprints, which are YAML files that represent the environments. So the first step is to create blueprints out of your existing assets.

1. In your space, go to __Settings > Repositories__ and discover your assets.

> ![Locale Dropdown](/img/discover-assets.png)

2. Select the assets you want Torque to discover and click __Generate Blueprints__.
  
  Torque creates a blueprint YAML for each asset, and lists the blueprints in your space’s __Blueprints__ page. You can click the blueprint to see the YAML file.

> ![Locale Dropdown](/img/new-assets.png)

3. __Publish__ the blueprints and you’re good to go.

> ![Locale Dropdown](/img/publish-blueprint.png)
  
  You and your space’s users can now launch these environments from the space.

  In some cases, you may need to adjust your autogenerated blueprints. For details on what you can do, see [Autogenerated Blueprints](/blueprint-designer-guide/autogenerated%20blueprints).

## Create a multi-asset blueprint
So far, we’ve learned how to create single-asset blueprints. But what if you want to create an application-stack environment? This is easily done by nesting single-asset blueprints within a master blueprint as grains. Each grain represents a single-asset blueprint, which can be an application or cloud service deployed via an asset, like a Terraform module, Helm chart, or CloudFormation template, to name a few. 

For example, the __Helm Application with MySQL and S3 Deployed by Terraform__ sample blueprint (available in the Sample space [here](https://portal.qtorque.io/Sample/blueprints/[Sample]Helm%20Application%20with%20MySql%20and%20S3%20Deployed%20by%20Terraform), which deploys 2 Terraform modules and a Helm chart:

```jsx title=
spec_version: 2
description: Robotshot microservices application deployed on K8S with Helm and RDS deployed with TF

outputs:
  WebsiteUrl:
    kind: link
    value: 'https://portal.qtorque.io/static/demo-quick-links/stans-robot-shop.html'


grains:
  mySqlDB:
    kind: terraform
    spec:
      source:
        path: github.com/QualiTorque/samples.git//terraform/rds
      host:
        name: eks-demo
      inputs:
        - sandbox_id: '{{ sandboxid | downcase }}'
        - size: small 
        - allocated_storage: 20
        - db_name: demo_db
        - engine_version: 8.0.26
        - engine: MySQL
        - username: adminuser
        - vpc_id: vpc-02e3bca90b081cd0f
        - region: us-east-1
      outputs:
        - hostname
        - connection_string

  s3Bucket:
    kind: terraform
    spec: 
      source:
        path: github.com/QualiTorque/samples.git//terraform/s3
      host:
        name: eks-demo
      inputs:
        - region: eu-west-1
        - acl: public-read
        - name: 'robotshop-s3-{{ sandboxid | downcase }}'
      outputs:
        - s3_bucket_arn

  robotShopMicroservices:
    kind: helm
    depends-on: mySqlDB, s3Bucket
    spec:
      source:
        path: https://github.com/QualiTorque/samples.git//helm/robotshop
      host:
        name: eks-demo
      inputs:
        - hostname: 'robotshop-{{ sandboxid | downcase }}'
        - version: 0.4.3
        - connectionString: '{{ .grains.mySqlDB.outputs.connection_string }}'
        - objectStore.s3BucketArn: '{{ .grains.s3Bucket.outputs.s3_bucket_arn }}'
        - redis.storageClassName: gp2
```

To learn more, see [Blueprint YAML](/blueprint-designer-guide/blueprints).