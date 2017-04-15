---
layout: post
categories: fabric8
title: fabric8 初始化
date: 2017-4-15 16:36:51 +0800
description: 开源微服务管理平台 fabric8 安装配置
keywords: fabric8 Kubernetes
---






### Fabric8简介

fabric8是一个开源集成开发平台，为基于Kubernetes和Jenkins的微服务提供持续发布。

使用fabric可以很方便的通过Continuous Delivery pipelines创建、编译、部署和测试微服务，然后通过Continuous Improvement和ChatOps运行和管理他们。

Fabric8微服务平台提供：

	- •	Developer Console，是一个富web应用，提供一个单页面来创建、编辑、编译、部署和测试微服务。
	- •	Continuous Integration and Continous Delivery，使用 Jenkins with a Jenkins Workflow Library更快和更可靠的交付软件。
	- •	Management，集中式管理Logging、Metrics, ChatOps、Chaos Monkey，使用Hawtio和Jolokia管理Java Containers。
	- •	Integration Integration Platform As A Service with deep visualisation of your Apache Camel integration services, an API Registry to view of all your RESTful and SOAP APIs and Fabric8 MQ provides Messaging As A Service based on Apache ActiveMQ。
	- •	Java Tools 帮助Java应用使用Kubernetes:
	
	◦	Maven Plugin for working with Kubernetes ，这真是极好的
	◦	Integration and System Testing of Kubernetes resources easily inside JUnit with Arquillian
	◦	Java Libraries and support for CDI extensions for working with Kubernetes.
	
#### Fabric8微服务平台

Fabric8提供了一个完全集成的开源微服务平台，可在任何的Kubernetes和OpenShift环境中开箱即用。
整个平台是基于微服务而且是模块化的，你可以按照微服务的方式来使用它。
微服务平台提供的服务有：

	- •	开发者控制台，这是一个富Web应用程序，它提供了一个单一的页面来创建、编辑、编译、部署和测试微服务。
	•	持续集成和持续交付，帮助团队以更快更可靠的方式交付软件，可以使用以下开源软件：
	◦	Jenkins：CI／CD pipeline
	◦	Nexus： 组件库
	◦	Gogs：git代码库
	◦	SonarQube：代码质量维护平台
	◦	Jenkins Workflow Library：在不同的项目中复用Jenkins Workflow scripts
	◦	Fabric8.yml：为每个项目、存储库、聊天室、工作流脚本和问题跟踪器提供一个配置文件
	•	ChatOps：通过使用hubot来开发和管理，能够让你的团队拥抱DevOps，通过聊天和系统通知的方式来approval of release promotion
	•	Chaos Monkey：通过干掉pods来测试系统健壮性和可靠性
	•	管理
	◦	日志 统一集群日志和可视化查看状态
	◦	metris 可查看历史metrics和可视化

