# memberPrice为空

1、远程保存 sku 优惠信息，发现报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693549187533-224f8c94-6db5-4783-8162-64630fb73526.png)

2、定位到 gulimall-coupon 的 SkuFullReductionServiceImpl 的 saveSkuReduction() 方法中的 List<MemberPrice> memberPrices = skuReductionTo.getMemberPrice(); 发现 skuReductionTo 中并没有保存 memberPrice

3、回到 gulimall-product 的 SpuInfoServiceImpl 中，发现 saveSkuReduction() 方法中的 BeanUtils.copyProperties(sku, skuReductionTo); 就没有保存 memberPrice

4、单步调试，进入 BeanUtils.copyProperties() 方法

```java
public static void copyProperties(Object source, Object target) throws BeansException {
    copyProperties(source, target, (Class)null, (String[])null);
}

private static void copyProperties(Object source, Object target, @Nullable Class<?> editable, @Nullable String... ignoreProperties) throws BeansException {
    Assert.notNull(source, "Source must not be null");
    Assert.notNull(target, "Target must not be null");
    Class<?> actualEditable = target.getClass();
    if (editable != null) {
        if (!editable.isInstance(target)) {
            throw new IllegalArgumentException("Target class [" + target.getClass().getName() + "] not assignable to Editable class [" + editable.getName() + "]");
        }
        actualEditable = editable;
    }
    PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);
    List<String> ignoreList = ignoreProperties != null ? Arrays.asList(ignoreProperties) : null;
    PropertyDescriptor[] var7 = targetPds;
    int var8 = targetPds.length;
    for(int var9 = 0; var9 < var8; ++var9) {
        PropertyDescriptor targetPd = var7[var9];
        Method writeMethod = targetPd.getWriteMethod();
        if (writeMethod != null && (ignoreList == null || !ignoreList.contains(targetPd.getName()))) {
            PropertyDescriptor sourcePd = getPropertyDescriptor(source.getClass(), targetPd.getName());
            if (sourcePd != null) {
                Method readMethod = sourcePd.getReadMethod();
                if (readMethod != null) {
                    ResolvableType sourceResolvableType = ResolvableType.forMethodReturnType(readMethod);
                    ResolvableType targetResolvableType = ResolvableType.forMethodParameter(writeMethod, 0);
                    boolean isAssignable = !sourceResolvableType.hasUnresolvableGenerics() && !targetResolvableType.hasUnresolvableGenerics() ? targetResolvableType.isAssignableFrom(sourceResolvableType) : ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType());
                    if (isAssignable) {
                        try {
                            if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
                                readMethod.setAccessible(true);
                            }
                            Object value = readMethod.invoke(source);
                            if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
                                writeMethod.setAccessible(true);
                            }
                            writeMethod.invoke(target, value);
                        } catch (Throwable var18) {
                            throw new FatalBeanException("Could not copy property '" + targetPd.getName() + "' from source to target", var18);
                        }
                    }
                }
            }
        }
    }
}
```

5、遍历迭代器，发现其他参数的 isAssignable 都是 true，而 memberPrice 的是 false

6、再次遍历，发现问题出现在 targetResolvableType.isAssignableFrom(sourceResolvableType)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693549192178-58be4715-8116-4394-b0d9-8818c2af15cd.png)

7、参考 https://zhuanlan.zhihu.com/p/317784108，得知这是用来判断 sourceResolvableType 和 targetResolvableType 之间关系的

8、虽然两个 MemberPrice 实体类一模一样，但是因为包路径不同，编译器也会将它们视为不同的类型

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693549195949-151c6c84-4c5f-417a-bb45-2d3ed48cab7f.png)

9、删除 gulimall-product 下的 MemberPrice，再次测试，发现代码可以正常运行