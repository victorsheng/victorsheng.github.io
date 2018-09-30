---
title: aop-jersey-name-binding
abbrlink: 2820051264
tags:
categories:
---

# 处理流程
## ApplicationHandler

## ProcessingProvidersConfigurator
* Configurator初始化并将{@link ProcessingProviders}实例注册到{@link BootstrapBag}。
* 使用此配置程序处理，配置和提供这些接口的实例：

###  postInit方法
```
@Override
    public void postInit(InjectionManager injectionManager, BootstrapBag bootstrapBag) {
        ServerBootstrapBag serverBag = (ServerBootstrapBag) bootstrapBag;

        ComponentBag componentBag = serverBag.getRuntimeConfig().getComponentBag();
        // scan for NameBinding annotations attached to the application class
        Collection<Class<? extends Annotation>> applicationNameBindings = ReflectionHelper.getAnnotationTypes(
                ResourceConfig.unwrapApplication(serverBag.getRuntimeConfig()).getClass(), NameBinding.class);

        MultivaluedMap<RankedProvider<ContainerResponseFilter>, Class<? extends Annotation>> nameBoundRespFiltersInverse =
                new MultivaluedHashMap<>();
        MultivaluedMap<RankedProvider<ContainerRequestFilter>, Class<? extends Annotation>> nameBoundReqFiltersInverse =
                new MultivaluedHashMap<>();
        MultivaluedMap<RankedProvider<ReaderInterceptor>, Class<? extends Annotation>> nameBoundReaderInterceptorsInverse =
                new MultivaluedHashMap<>();
        MultivaluedMap<RankedProvider<WriterInterceptor>, Class<? extends Annotation>> nameBoundWriterInterceptorsInverse =
                new MultivaluedHashMap<>();

        Iterable<RankedProvider<ContainerResponseFilter>> responseFilters =
                Providers.getAllRankedProviders(injectionManager, ContainerResponseFilter.class);

        MultivaluedMap<Class<? extends Annotation>, RankedProvider<ContainerResponseFilter>> nameBoundResponseFilters =
                filterNameBound(responseFilters, null, componentBag, applicationNameBindings, nameBoundRespFiltersInverse);

        Iterable<RankedProvider<ContainerRequestFilter>> requestFilters =
                Providers.getAllRankedProviders(injectionManager, ContainerRequestFilter.class);

        List<RankedProvider<ContainerRequestFilter>> preMatchFilters = new ArrayList<>();

        MultivaluedMap<Class<? extends Annotation>, RankedProvider<ContainerRequestFilter>> nameBoundReqFilters =
                filterNameBound(requestFilters, preMatchFilters, componentBag, applicationNameBindings,
                        nameBoundReqFiltersInverse);

        Iterable<RankedProvider<ReaderInterceptor>> readerInterceptors =
                Providers.getAllRankedProviders(injectionManager, ReaderInterceptor.class);

        MultivaluedMap<Class<? extends Annotation>, RankedProvider<ReaderInterceptor>> nameBoundReaderInterceptors =
                filterNameBound(readerInterceptors, null, componentBag, applicationNameBindings,
                        nameBoundReaderInterceptorsInverse);

        Iterable<RankedProvider<WriterInterceptor>> writerInterceptors =
                Providers.getAllRankedProviders(injectionManager, WriterInterceptor.class);

        MultivaluedMap<Class<? extends Annotation>, RankedProvider<WriterInterceptor>> nameBoundWriterInterceptors =
                filterNameBound(writerInterceptors, null, componentBag, applicationNameBindings,
                        nameBoundWriterInterceptorsInverse);

        Iterable<DynamicFeature> dynamicFeatures = Providers.getAllProviders(injectionManager, DynamicFeature.class);

        ProcessingProviders processingProviders = new ProcessingProviders(nameBoundReqFilters, nameBoundReqFiltersInverse,
                nameBoundResponseFilters, nameBoundRespFiltersInverse, nameBoundReaderInterceptors,
                nameBoundReaderInterceptorsInverse, nameBoundWriterInterceptors, nameBoundWriterInterceptorsInverse,
                requestFilters, preMatchFilters, responseFilters, readerInterceptors, writerInterceptors, dynamicFeatures);

        serverBag.setProcessingProviders(processingProviders);
    }
```

## ProcessingProviders
可注入的封装类，包含过滤器，拦截器，名称绑定提供程序，动态功能等处理提供程序。