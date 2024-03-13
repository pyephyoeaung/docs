---
layout: post
title: "Setup AWS APP Mesh on EKS"
date: 2024-01-07 09:00:00 -0500
categories: aws
tags: aws eks servicemesh
image:
  path: /assets/img/headers/headset-black.webp
  lqip: data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAf/AABEIAAUACgMBEQACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5+gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4+Tl5ufo6ery8/T19vf4+fr/2gAMAwEAAhEDEQA/AP6Q/wBt/wAGfGpP2kfgH4l+Fnxzh+HGhz2lnpuqeG7z4exeMJZb/TvHdp4j13WNF1iTxdoK+HZ/GWit/wAIh4ljstKle/0qOLzriSDz7C4TV76tSaet9Lu1ny7aW6W3Gnrrqrq68lfS/S+n3H3imiabcItxPbl551WaZ1nuo1eWUB5GWNbjais7EhF+VQdo4FdKEf/Z

---

To install AWS App Mesh in your Amazon EKS cluster, you can follow these step-by-step instructions:

1. **Connect with AWS Account**:
   - Use the AWS CLI to configure your AWS access key, secret access key, and region:
     ```
     aws configure
     ```

2. **Create an EKS Cluster**:
   - If you don't have an existing EKS cluster, create one using `eksctl create cluster`[1].

3. **Connect with EKS Cluster**:
   - Update the Kubeconfig to connect with the EKS cluster:
     ```
     aws eks update-kubeconfig --name mesh-test-cluster
     ```

4. **Add eks-charts Repository to Helm**:
   - Install the App Mesh Kubernetes custom resource definitions (CRD) using `kubectl apply`:
     ```
     kubectl apply -k "https://github.com/aws/eks-charts/stable/appmesh-controller/crds?ref=master"
     ```

5. **Create appmesh-system Namespace**:
   - Create a namespace for the App Mesh controller:
     ```
     kubectl create ns appmesh-system
     ```

6. **Create an OpenID Connect (OIDC) Identity Provider**:
   - Create an OIDC identity provider for your cluster using `eksctl utils associate-iam-oidc-provider`[1].

7. **Create an IAM Role**:
   - Create an IAM role, attach the AWSAppMeshFullAccess and AWSCloudMapFullAccess policies, and bind it to the appmesh-controller Kubernetes service account:
     ```
     eksctl create iamserviceaccount \
       --cluster mesh-test-cluster \
       --namespace appmesh-system \
       --name appmesh-controller \
       --attach-policy-arn arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
       --override-existing-serviceaccounts \
       --approve
     ```

8. **Deploy the App Mesh Controller**:
   - Deploy the App Mesh controller using Helm:
     ```
     helm upgrade -i appmesh-controller eks/appmesh-controller \
       --namespace appmesh-system \
       --set region=ap-south-1 \
       --set serviceAccount.create=false \
       --set serviceAccount.name=appmesh-controller
     ```

9. **Verify Controller Pod Running**:
   - Verify that the Controller pod is running:
     ```
     kubectl get pods -n appmesh-system
     ```

10. **Create Mesh**:
    - Create a mesh using the App Mesh controller. You can create a mesh from AWS CLI, AWS Console, or Kubernetes native YAML[1].

By following these steps, you can successfully install AWS App Mesh in your Amazon EKS cluster.

Citations:
[1] https://blog.knoldus.com/how-to-setup-aws-app-mesh-with-eks/
[2] https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-kubernetes.html
[3] https://aws.amazon.com/blogs/compute/learning-aws-app-mesh/
[4] https://aws.amazon.com/blogs/containers/getting-started-with-app-mesh-and-eks/
[5] https://archive.eksworkshop.com/intermediate/330_app_mesh/