---
layout: post
title: springboot支持跨域访问
author: andy
tags:  springboot
categories:  spring
excerpt: springboot支持跨域访问
---

* TOC
{:toc}

# springboot支持跨域访问
springmvc提供了corsFilter解决跨域问题，在springboot中需要在Application类中注册corsFilter。
需要注意的是，corsFilter和自定义filter的执行顺序很重要，一定要corsFilter在前执行才可以，因此
在如下代码中，设置了filter的执行顺序。

    @Bean
    public FilterRegistrationBean corsFilter() {
      UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
      CorsConfiguration config = new CorsConfiguration();
      config.setAllowCredentials(true);
      config.addAllowedOrigin("*");
      config.addAllowedHeader("*");
      config.addAllowedMethod("*");
      source.registerCorsConfiguration("/**", config);
      FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
      bean.setOrder(0);
      return bean;
    }



