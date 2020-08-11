---
title: "EKS"
linkTitle: "EKS"
description: >
  Installation instructions to deploy KRE on EKS.
weight: 30
---
# EKS deployment

The flavor of Kubernetes on AWS is called EKS (Elastic Kubernetes Service) which allow to deploy a cluster managed by Amazon. This means that Amazon will manage the lifecyle of the Master nodes of our cluster. 

Currently there are two ways of run loads on top of EKS, using EC2 instances as compute nodes that are added to to the cluster or using the Fargate mode, where AWS also manage these compute nodes. In this guide is just described the first one, adding our own compute nodes with EC2 instances.

Deploy an EKS cluster is not the goal of this guide, only the specific configuration needed to run KRE on top of this, but we recomend to use IaaC approach using Terraform to automate the creation of your cluster, [here](https://learn.hashicorp.com/tutorials/terraform/eks) you can find a usefull resource on how to deploy EKS with Terraform, also you can follow the instructions from the official [AWS site](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html). 

# Storage

# Kubernetes required components

# Helm deployment
