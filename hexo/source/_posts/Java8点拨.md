---
title: Java8点拨
date: 2018-08-04 10:30:52
tags: [Java]
categories: [Java]
---

java8的一些知识点的记录，写的比较糙，能理解就ok<!--more-->

#### Lambda表达式

* lambda表达式可以有零个，一个或者多个参数
* 可以显式声明参数的类型，也可以从上下中推断参数类型
* 多个参数必须包含在括号中，并用逗号分隔，空括号用于表示零个参数
* 当只有一个参数时，如果推断它的类型，可以不用括号
* lambda表达式的函数体可以包含零个，一个或者多个语句
* 如果lambda表达式函数提只有一行，那么就可以不用大括号，匿名函数的返回类型与函数体表达式的返回类型相同。当函数提中大于一行代码则需要用大括号包含。
* 谓词复合，用于多条件判断。
  * negate  Predicate的非
  * and Predicate的与
  * or predicate的或
* 函数复合,用于Fuction接口所代表的Lambda表达式的符合
  * andThen 很明了的函数，aExpression.andThen(bExpression）相当于 数学上的 b(a(x))
  * compose aExpression.compose(bExpression) 相当于数学上的  a（b(x)）

***

#### 函数式编程中的方法引用。

| 方法引用                                     | 描述                                   | 示例                                     |
| -------------------------------------------- | -------------------------------------- | ---------------------------------------- |
| 静态方法引用                                 | 用于引用类的静态方法                   | Math::max等于原先的Math.max（）          |
| 实例对象引用方法                             | 使用实例对象引用来调用实例方法         | System.out::println=System.out.println() |
| 从类中引用实例方法（其实还是实例调用方法吧） | 在上下文提供的对象的医用上调用实例方法 | String::length相当于str.length           |
| 引用构造函数                                 | 引用构造函数，创建实例对象             | HashMap：：new 等于 new  HashMap()       |

这些方法引用其实在函数式编程语言 Rust 中都已经存在，lisp中也有。

***

#### 默认方法与静态方法：

接口新增了default方法和static方法。

* 静态方法，只能通过接口调用，不可以通过实现类的类名或实现类的实例进行调用。最简单的提供工厂的实现功能。

  ```
  package com.space.default_method;
  
  public interface Caculate {
  	int add(int a, int b);
  
  	int subtract(int a, int b);
  
  	int divide(int a, int b);
  
  	int muti(int a, int b);
  	
  	default int mod (int a , int b ) {
  		return a%b;
  	}
  	public static Caculate getCaculateImplInstance() {
  		return new CaculateImpl();
  	}
  }
  ```

  

* 默认方法，只能通过接口的实现类来调用,可以避免新的功能添加導致对现有功能的困扰。方便api的演进

  ```
  package com.space.default_method;
  
  public interface Caculate {
  	int add(int a, int b);
  
  	int subtract(int a, int b);
  
  	int divide(int a, int b);
  
  	int muti(int a, int b);
  	
  	default int mod (int a , int b ) {
  		return a%b;
  	}
  }
  ```

  ```
  package com.space.default_method;
  
  public class CaculateImpl implements Caculate {
  
  	@Override
  	public int add(int a, int b) {
  		return a+b;
  	}
  
  	@Override
  	public int subtract(int a, int b) {
  		// TODO Auto-generated method stub
  		return a-b;	}
  
  	@Override
  	public int divide(int a, int b) {
  		// TODO Auto-generated method stub
  		return a/b;
  	}
  
  	@Override
  	public int muti(int a, int b) {
  		return a*b;
  	}
  	public static void main(String[] args) {
  		Caculate caculate  = new CaculateImpl();
  		int mod = caculate.mod(100, 3);
  		System.out.println(mod);
  	}
  
  }
  ```

  **PS:** 接口冲突，方法冲突 的解决：

  * 类中的方法优先级别最高，类或者父类中的声明的方法优先级高于任何声明为默认的方法的优先级
  * 如果无法通过上述条件进行判断，那么子类接口的优先级更高，函数签名相同时，优先选择拥有最具体实现的默认方法的接口，
  * 如果上述条件还是无法判断的话，继承多个接口的类必须通过显式覆盖和调用期望的方法，显式的选择使用哪一个默认方法的实现。

#### 函数式接口

* 函数式接口也称作单例抽象方法接口（SAM接口），**接口中只允许一个抽象方法**,@FunctionalInterface注解会在你注解的接口违反函数式几口的约定时提示一个编译错误。

```
@FunctionalInterface
public interface  MyFunctionalInterface {
    public void firstFunction（）；
}
```

***

* 常用函数式接口

  * Predicate函数式接口:功能判断输入的对象是否符合某个条件。官方文档解释到：``Determines if the input object matches some criteria``

    原始类型特化：IntPredicate,LongPredicate, DoublePredicate 

    ```
    /*
     * Copyright (c) 2010, 2013, Oracle and/or its affiliates. All rights reserved.
     * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
     */
    package java.util.function;
    
    import java.util.Objects;
    
    /**
     * Represents a predicate (boolean-valued function) of one argument.
     *
     * <p>This is a <a href="package-summary.html">functional interface</a>
     * whose functional method is {@link #test(Object)}.
     *
     * @param <T> the type of the input to the predicate
     *
     * @since 1.8
     */
    @FunctionalInterface
    public interface Predicate<T> {
    
        /**
         * Evaluates this predicate on the given argument.
         *
         * @param t the input argument
         * @return {@code true} if the input argument matches the predicate,
         * otherwise {@code false}
         */
        boolean test(T t);
    
        /**
         * Returns a composed predicate that represents a short-circuiting logical
         * AND of this predicate and another.  When evaluating the composed
         * predicate, if this predicate is {@code false}, then the {@code other}
         * predicate is not evaluated.
         *
         * <p>Any exceptions thrown during evaluation of either predicate are relayed
         * to the caller; if evaluation of this predicate throws an exception, the
         * {@code other} predicate will not be evaluated.
         *
         * @param other a predicate that will be logically-ANDed with this
         *              predicate
         * @return a composed predicate that represents the short-circuiting logical
         * AND of this predicate and the {@code other} predicate
         * @throws NullPointerException if other is null
         */
        default Predicate<T> and(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) && other.test(t);
        }
    
        /**
         * Returns a predicate that represents the logical negation of this
         * predicate.
         *
         * @return a predicate that represents the logical negation of this
         * predicate
         */
        default Predicate<T> negate() {
            return (t) -> !test(t);
        }
    
        /**
         * Returns a composed predicate that represents a short-circuiting logical
         * OR of this predicate and another.  When evaluating the composed
         * predicate, if this predicate is {@code true}, then the {@code other}
         * predicate is not evaluated.
         *
         * <p>Any exceptions thrown during evaluation of either predicate are relayed
         * to the caller; if evaluation of this predicate throws an exception, the
         * {@code other} predicate will not be evaluated.
         *
         * @param other a predicate that will be logically-ORed with this
         *              predicate
         * @return a composed predicate that represents the short-circuiting logical
         * OR of this predicate and the {@code other} predicate
         * @throws NullPointerException if other is null
         */
        default Predicate<T> or(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) || other.test(t);
        }
    
        /**
         * Returns a predicate that tests if two arguments are equal according
         * to {@link Objects#equals(Object, Object)}.
         *
         * @param <T> the type of arguments to the predicate
         * @param targetRef the object reference with which to compare for equality,
         *               which may be {@code null}
         * @return a predicate that tests if two arguments are equal according
         * to {@link Objects#equals(Object, Object)}
         */
        static <T> Predicate<T> isEqual(Object targetRef) {
            return (null == targetRef)
                    ? Objects::isNull
                    : object -> targetRef.equals(object);
        }
    }
    
    ```

  * Consumer函数式接口： **表示一个接受单个输入参数并且没有返回值的操作**。不像其它函数式接口，Consumer 接口期望执行带有副作用的操作 (Consumer 的操作可能会更改输入参数的内部状态)。官方文档：``An operation which accepts a single input argument and returns no result. Unlike most other functional interfaces, Consumer is expected to operate via side-effects.`` 也就是void。

    原始类型特化:

    IntConsumer,LongConsumer, DoubleConsumer 

    ````
    /*
     * Copyright (c) 2010, 2013, Oracle and/or its affiliates. All rights reserved.
     * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
     */
    package java.util.function;
    
    import java.util.Objects;
    
    /**
     * Represents an operation that accepts a single input argument and returns no
     * result. Unlike most other functional interfaces, {@code Consumer} is expected
     * to operate via side-effects.
     *
     * <p>This is a <a href="package-summary.html">functional interface</a>
     * whose functional method is {@link #accept(Object)}.
     *
     * @param <T> the type of the input to the operation
     *
     * @since 1.8
     */
    @FunctionalInterface
    public interface Consumer<T> {
    
        /**
         * Performs this operation on the given argument.
         *
         * @param t the input argument
         */
        void accept(T t);
    
        /**
         * Returns a composed {@code Consumer} that performs, in sequence, this
         * operation followed by the {@code after} operation. If performing either
         * operation throws an exception, it is relayed to the caller of the
         * composed operation.  If performing this operation throws an exception,
         * the {@code after} operation will not be performed.
         *
         * @param after the operation to perform after this operation
         * @return a composed {@code Consumer} that performs in sequence this
         * operation followed by the {@code after} operation
         * @throws NullPointerException if {@code after} is null
         */
        default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNonNull(after);
            return (T t) -> { accept(t); after.accept(t); };
        }
    }
    
    ````

  * Supplier 函数式接口：顾名思义，就是提供者的意思，可以简单理解为工厂，**不接收参数，只提供返回值**

    ````
    /*
     * Copyright (c) 2012, 2013, Oracle and/or its affiliates. All rights reserved.
     * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
     */
    package java.util.function;
    
    /**
     * Represents a supplier of results.
     *
     * <p>There is no requirement that a new or distinct result be returned each
     * time the supplier is invoked.
     *
     * <p>This is a <a href="package-summary.html">functional interface</a>
     * whose functional method is {@link #get()}.
     *
     * @param <T> the type of results supplied by this supplier
     *
     * @since 1.8
     */
    @FunctionalInterface
    public interface Supplier<T> {
    
        /**
         * Gets a result.
         *
         * @return a result
         */
        T get();
    }
    
    ````

  * Function函数式接口：**接受一定数量的参数 同时提供一个返回结果**  ,

    原始类型特化：

    `IntFunction<R>, IntToDoubleFunction,`  `IntToLongFunction, LongFunction<R>,`  `LongToDoubleFunction, LongToIntFunction,`  `DoubleFunction<R>, ToIntFunction<T>,`  `ToDoubleFunction<T>, ToLongFunction<T>` 

    ```
    /*
     * Copyright (c) 2010, 2013, Oracle and/or its affiliates. All rights reserved.
     * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
     */
    package java.util.function;
    
    import java.util.Objects;
    
    /**
     * Represents a function that accepts one argument and produces a result.
     *
     * <p>This is a <a href="package-summary.html">functional interface</a>
     * whose functional method is {@link #apply(Object)}.
     *
     * @param <T> the type of the input to the function
     * @param <R> the type of the result of the function
     *
     * @since 1.8
     */
    @FunctionalInterface
    public interface Function<T, R> {
    
        /**
         * Applies this function to the given argument.
         *
         * @param t the function argument
         * @return the function result
         */
        R apply(T t);
    
        /**
         * Returns a composed function that first applies the {@code before}
         * function to its input, and then applies this function to the result.
         * If evaluation of either function throws an exception, it is relayed to
         * the caller of the composed function.
         *
         * @param <V> the type of input to the {@code before} function, and to the
         *           composed function
         * @param before the function to apply before this function is applied
         * @return a composed function that first applies the {@code before}
         * function and then applies this function
         * @throws NullPointerException if before is null
         *
         * @see #andThen(Function)
         */
        default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
            Objects.requireNonNull(before);
            return (V v) -> apply(before.apply(v));
        }
    
        /**
         * Returns a composed function that first applies this function to
         * its input, and then applies the {@code after} function to the result.
         * If evaluation of either function throws an exception, it is relayed to
         * the caller of the composed function.
         *
         * @param <V> the type of output of the {@code after} function, and of the
         *           composed function
         * @param after the function to apply after this function is applied
         * @return a composed function that first applies this function and then
         * applies the {@code after} function
         * @throws NullPointerException if after is null
         *
         * @see #compose(Function)
         */
        default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requireNonNull(after);
            return (T t) -> after.apply(apply(t));
        }
    
        /**
         * Returns a function that always returns its input argument.
         *
         * @param <T> the type of the input and output objects to the function
         * @return a function that always returns its input argument
         */
        static <T> Function<T, T> identity() {
            return t -> t;
        }
    }
    
    ```

  * Operator函数式接口:  运算符函数接口，有``UnaryOperator<T>和``BinaryOperator<T>``这两个接口。只有一个泛型，那么意思就是说 入参和返回只有一种类型。

    ```
    /**
    * UnaryOperator 接收一个参数 返回同类型结果。
    */
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
        UnaryOperator<Integer> unaryOpt = i -> i * i;
        unaryOperatorFun(unaryOpt, list).forEach(x -> {
        System.out.println(x);
    });
    private static List<Integer> unaryOperatorFun(UnaryOperator<Integer> unaryOpt, List<Integer> list) {
        List<Integer> uniList = new ArrayList<>();
        list.forEach(i -> uniList.add(unaryOpt.apply(i)));
        return uniList;
    }
    /**
     * BinaryOperator
     */
    Map<String, String> map = new HashMap();
    map.put("1", "23");
    map.put("12", "2365");
    map.put("13", "231");
    map.put("14", "232");
    BinaryOperator<String> bo = (s1, s2) -> s1 + s2;
    binaryOperatorFun(bo, map).forEach(x->System.out.println(x));
    
    private static List<String> binaryOperatorFun(BinaryOperator<String> bo, Map<String, String> map) {
        List<String> result = new ArrayList<>();
        map.forEach((s1,s2)->result.add(bo.apply(s1, s2)));
        return result;
    }
    ```

#### Optional

很像Rust中的Option

````
/*
 * Copyright (c) 2012, 2013, Oracle and/or its affiliates. All rights reserved.
 * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
 */
package java.util;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * A container object which may or may not contain a non-null value.
 * If a value is present, {@code isPresent()} will return {@code true} and
 * {@code get()} will return the value.
 *
 * <p>Additional methods that depend on the presence or absence of a contained
 * value are provided, such as {@link #orElse(java.lang.Object) orElse()}
 * (return a default value if value not present) and
 * {@link #ifPresent(java.util.function.Consumer) ifPresent()} (execute a block
 * of code if the value is present).
 *
 * <p>This is a <a href="../lang/doc-files/ValueBased.html">value-based</a>
 * class; use of identity-sensitive operations (including reference equality
 * ({@code ==}), identity hash code, or synchronization) on instances of
 * {@code Optional} may have unpredictable results and should be avoided.
 *
 * @since 1.8
 */
public final class Optional<T> {
    /**
     * Common instance for {@code empty()}.
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;

    /**
     * Constructs an empty instance.
     *
     * @implNote Generally only one empty instance, {@link Optional#EMPTY},
     * should exist per VM.
     */
    private Optional() {
        this.value = null;
    }

    /**
     * Returns an empty {@code Optional} instance.  No value is present for this
     * Optional.
     *
     * @apiNote Though it may be tempting to do so, avoid testing if an object
     * is empty by comparing with {@code ==} against instances returned by
     * {@code Option.empty()}. There is no guarantee that it is a singleton.
     * Instead, use {@link #isPresent()}.
     *
     * @param <T> Type of the non-existent value
     * @return an empty {@code Optional}
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    /**
     * Constructs an instance with the value present.
     *
     * @param value the non-null value to be present
     * @throws NullPointerException if value is null
     */
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    /**
     * Returns an {@code Optional} with the specified present non-null value.
     *
     * @param <T> the class of the value
     * @param value the value to be present, which must be non-null
     * @return an {@code Optional} with the value present
     * @throws NullPointerException if value is null
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    /**
     * Returns an {@code Optional} describing the specified value, if non-null,
     * otherwise returns an empty {@code Optional}.
     *
     * @param <T> the class of the value
     * @param value the possibly-null value to describe
     * @return an {@code Optional} with a present value if the specified value
     * is non-null, otherwise an empty {@code Optional}
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    /**
     * If a value is present in this {@code Optional}, returns the value,
     * otherwise throws {@code NoSuchElementException}.
     *
     * @return the non-null value held by this {@code Optional}
     * @throws NoSuchElementException if there is no value present
     *
     * @see Optional#isPresent()
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    /**
     * Return {@code true} if there is a value present, otherwise {@code false}.
     *
     * @return {@code true} if there is a value present, otherwise {@code false}
     */
    public boolean isPresent() {
        return value != null;
    }

    /**
     * If a value is present, invoke the specified consumer with the value,
     * otherwise do nothing.
     *
     * @param consumer block to be executed if a value is present
     * @throws NullPointerException if value is present and {@code consumer} is
     * null
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    /**
     * If a value is present, and the value matches the given predicate,
     * return an {@code Optional} describing the value, otherwise return an
     * empty {@code Optional}.
     *
     * @param predicate a predicate to apply to the value, if present
     * @return an {@code Optional} describing the value of this {@code Optional}
     * if a value is present and the value matches the given predicate,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the predicate is null
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    /**
     * If a value is present, apply the provided mapping function to it,
     * and if the result is non-null, return an {@code Optional} describing the
     * result.  Otherwise return an empty {@code Optional}.
     *
     * @apiNote This method supports post-processing on optional values, without
     * the need to explicitly check for a return status.  For example, the
     * following code traverses a stream of file names, selects one that has
     * not yet been processed, and then opens that file, returning an
     * {@code Optional<FileInputStream>}:
     *
     * <pre>{@code
     *     Optional<FileInputStream> fis =
     *         names.stream().filter(name -> !isProcessedYet(name))
     *                       .findFirst()
     *                       .map(name -> new FileInputStream(name));
     * }</pre>
     *
     * Here, {@code findFirst} returns an {@code Optional<String>}, and then
     * {@code map} returns an {@code Optional<FileInputStream>} for the desired
     * file if one exists.
     *
     * @param <U> The type of the result of the mapping function
     * @param mapper a mapping function to apply to the value, if present
     * @return an {@code Optional} describing the result of applying a mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    /**
     * If a value is present, apply the provided {@code Optional}-bearing
     * mapping function to it, return that result, otherwise return an empty
     * {@code Optional}.  This method is similar to {@link #map(Function)},
     * but the provided mapper is one whose result is already an {@code Optional},
     * and if invoked, {@code flatMap} does not wrap it with an additional
     * {@code Optional}.
     *
     * @param <U> The type parameter to the {@code Optional} returned by
     * @param mapper a mapping function to apply to the value, if present
     *           the mapping function
     * @return the result of applying an {@code Optional}-bearing mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null or returns
     * a null result
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    /**
     * Return the value if present, otherwise return {@code other}.
     *
     * @param other the value to be returned if there is no value present, may
     * be null
     * @return the value, if present, otherwise {@code other}
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    /**
     * Return the value if present, otherwise invoke {@code other} and return
     * the result of that invocation.
     *
     * @param other a {@code Supplier} whose result is returned if no value
     * is present
     * @return the value if present otherwise the result of {@code other.get()}
     * @throws NullPointerException if value is not present and {@code other} is
     * null
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     *
     * @apiNote A method reference to the exception constructor with an empty
     * argument list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier The supplier which will return the exception to
     * be thrown
     * @return the present value
     * @throws X if there is no value present
     * @throws NullPointerException if no value is present and
     * {@code exceptionSupplier} is null
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

    /**
     * Indicates whether some other object is "equal to" this Optional. The
     * other object is considered equal if:
     * <ul>
     * <li>it is also an {@code Optional} and;
     * <li>both instances have no value present or;
     * <li>the present values are "equal to" each other via {@code equals()}.
     * </ul>
     *
     * @param obj an object to be tested for equality
     * @return {code true} if the other object is "equal to" this object
     * otherwise {@code false}
     */
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Optional)) {
            return false;
        }

        Optional<?> other = (Optional<?>) obj;
        return Objects.equals(value, other.value);
    }

    /**
     * Returns the hash code value of the present value, if any, or 0 (zero) if
     * no value is present.
     *
     * @return hash code value of the present value or 0 if no value is present
     */
    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    /**
     * Returns a non-empty string representation of this Optional suitable for
     * debugging. The exact presentation format is unspecified and may vary
     * between implementations and versions.
     *
     * @implSpec If a value is present the result must include its string
     * representation in the result. Empty and present Optionals must be
     * unambiguously differentiable.
     *
     * @return the string representation of this instance
     */
    @Override
    public String toString() {
        return value != null
            ? String.format("Optional[%s]", value)
            : "Optional.empty";
    }
}    	
````

* 创建Optional对象

  * 声明一个空的Optional对象 

    ```
    Optional<Car> optCar = Optional.empty()；
    ```

  * 创建一个有值的Optional如果你赋值为 null，则会抛出空指针异常。 如果值是 null，则目标 Optional 对象将会是空 (记得值是 absent，不是 null)。 

    ```
    Optional<Car> optCar = Optional.of(car)；
    ```

  * 可接受null值的Optional的创建

    ```
    Optional<Car> optCar = Optional.ofNullable(car);
    ```

* 使用map从Optional中提取和转换值

       Optional<Project> ppp = Optional.of(pp);
       Optional<Optional<String>> map2 = ppp.map(Project::getAuth);
       Optional<String> autss = map2.get();
   ````
   Optional<Person> optPerson = Optional.ofNullable(person）;
   
   Optional<Car> optcar=optPerson.map(Person::getCar);
   ````

* 使用flatMap链接Optional对象,用于处理嵌套的optional,流

  ```
  OptionalPerson> optPerson = Optional.ofNullable(person）;
  Optional<Wheel> optw=optPerson.flatMap（Person::getCar）.map(Car::getWheel());
  ```

* 默认行为及解引用Optional对象。

  * ``get()``获取封装的变量，或者抛出NoSuchElementException();
  * ``orElse(T other)``如果Optional对象中不包含值时可以进行提供一个默认值
  * ``orElseGet（Supplier<? extends T> other）``是orElse的延迟嗲用版，Supplier只有在Optional中不含值的时候才执行调用
  * ``orElseThrow（Supplier<? extends X> exceptionSupplier）``和get方法相似，只不过在遭遇对象元素为空的时候会抛出异常，可以定制异常。
  * ``ifPresent(Consumer <? Super T>)`` 在变量值存在时执行一个作为参数传入的方法。否则不做任何操作。

* 使用filter剔除特定的值

   ```
   Optional<String> optt = Optional.of("123");
   optt.filter(s1->"123".equals(s1)).ifPresent(x->System.out.println(x));
   ```

* 以下情况不使用Optional:

   * in the domain model layer (it’s not serializable) 
   * in DTOs (it’s not serializable) 
   * in input parameters of methods 
   * in constructor parameters 

* 何时使用Optional

   **当函数的返回值有可能是 null 时，我们应该尽可能地去使用它.**

####  流

```

package java.util.stream;

import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Arrays;
import java.util.Collection;
import java.util.Comparator;
import java.util.Iterator;
import java.util.Objects;
import java.util.Optional;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.BinaryOperator;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.IntFunction;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.function.ToDoubleFunction;
import java.util.function.ToIntFunction;
import java.util.function.ToLongFunction;
import java.util.function.UnaryOperator;

/**
 * A sequence of elements supporting sequential and parallel aggregate
 * operations.  The following example illustrates an aggregate operation using
 * {@link Stream} and {@link IntStream}:
 *
 * <pre>{@code
 *     int sum = widgets.stream()
 *                      .filter(w -> w.getColor() == RED)
 *                      .mapToInt(w -> w.getWeight())
 *                      .sum();
 * }</pre>
 *
 * In this example, {@code widgets} is a {@code Collection<Widget>}.  We create
 * a stream of {@code Widget} objects via {@link Collection#stream Collection.stream()},
 * filter it to produce a stream containing only the red widgets, and then
 * transform it into a stream of {@code int} values representing the weight of
 * each red widget. Then this stream is summed to produce a total weight.
 *
 * <p>In addition to {@code Stream}, which is a stream of object references,
 * there are primitive specializations for {@link IntStream}, {@link LongStream},
 * and {@link DoubleStream}, all of which are referred to as "streams" and
 * conform to the characteristics and restrictions described here.
 *
 * <p>To perform a computation, stream
 * <a href="package-summary.html#StreamOps">operations</a> are composed into a
 * <em>stream pipeline</em>.  A stream pipeline consists of a source (which
 * might be an array, a collection, a generator function, an I/O channel,
 * etc), zero or more <em>intermediate operations</em> (which transform a
 * stream into another stream, such as {@link Stream#filter(Predicate)}), and a
 * <em>terminal operation</em> (which produces a result or side-effect, such
 * as {@link Stream#count()} or {@link Stream#forEach(Consumer)}).
 * Streams are lazy; computation on the source data is only performed when the
 * terminal operation is initiated, and source elements are consumed only
 * as needed.
 *
 * <p>Collections and streams, while bearing some superficial similarities,
 * have different goals.  Collections are primarily concerned with the efficient
 * management of, and access to, their elements.  By contrast, streams do not
 * provide a means to directly access or manipulate their elements, and are
 * instead concerned with declaratively describing their source and the
 * computational operations which will be performed in aggregate on that source.
 * However, if the provided stream operations do not offer the desired
 * functionality, the {@link #iterator()} and {@link #spliterator()} operations
 * can be used to perform a controlled traversal.
 *
 * <p>A stream pipeline, like the "widgets" example above, can be viewed as
 * a <em>query</em> on the stream source.  Unless the source was explicitly
 * designed for concurrent modification (such as a {@link ConcurrentHashMap}),
 * unpredictable or erroneous behavior may result from modifying the stream
 * source while it is being queried.
 *
 * <p>Most stream operations accept parameters that describe user-specified
 * behavior, such as the lambda expression {@code w -> w.getWeight()} passed to
 * {@code mapToInt} in the example above.  To preserve correct behavior,
 * these <em>behavioral parameters</em>:
 * <ul>
 * <li>must be <a href="package-summary.html#NonInterference">non-interfering</a>
 * (they do not modify the stream source); and</li>
 * <li>in most cases must be <a href="package-summary.html#Statelessness">stateless</a>
 * (their result should not depend on any state that might change during execution
 * of the stream pipeline).</li>
 * </ul>
 *
 * <p>Such parameters are always instances of a
 * <a href="../function/package-summary.html">functional interface</a> such
 * as {@link java.util.function.Function}, and are often lambda expressions or
 * method references.  Unless otherwise specified these parameters must be
 * <em>non-null</em>.
 *
 * <p>A stream should be operated on (invoking an intermediate or terminal stream
 * operation) only once.  This rules out, for example, "forked" streams, where
 * the same source feeds two or more pipelines, or multiple traversals of the
 * same stream.  A stream implementation may throw {@link IllegalStateException}
 * if it detects that the stream is being reused. However, since some stream
 * operations may return their receiver rather than a new stream object, it may
 * not be possible to detect reuse in all cases.
 *
 * <p>Streams have a {@link #close()} method and implement {@link AutoCloseable},
 * but nearly all stream instances do not actually need to be closed after use.
 * Generally, only streams whose source is an IO channel (such as those returned
 * by {@link Files#lines(Path, Charset)}) will require closing.  Most streams
 * are backed by collections, arrays, or generating functions, which require no
 * special resource management.  (If a stream does require closing, it can be
 * declared as a resource in a {@code try}-with-resources statement.)
 *
 * <p>Stream pipelines may execute either sequentially or in
 * <a href="package-summary.html#Parallelism">parallel</a>.  This
 * execution mode is a property of the stream.  Streams are created
 * with an initial choice of sequential or parallel execution.  (For example,
 * {@link Collection#stream() Collection.stream()} creates a sequential stream,
 * and {@link Collection#parallelStream() Collection.parallelStream()} creates
 * a parallel one.)  This choice of execution mode may be modified by the
 * {@link #sequential()} or {@link #parallel()} methods, and may be queried with
 * the {@link #isParallel()} method.
 *
 * @param <T> the type of the stream elements
 * @since 1.8
 * @see IntStream
 * @see LongStream
 * @see DoubleStream
 * @see <a href="package-summary.html">java.util.stream</a>
 */
public interface Stream<T> extends BaseStream<T, Stream<T>> {

    /**
     * Returns a stream consisting of the elements of this stream that match
     * the given predicate.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param predicate a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                  <a href="package-summary.html#Statelessness">stateless</a>
     *                  predicate to apply to each element to determine if it
     *                  should be included
     * @return the new stream
     */
    Stream<T> filter(Predicate<? super T> predicate);

    /**
     * Returns a stream consisting of the results of applying the given
     * function to the elements of this stream.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param <R> The element type of the new stream
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element
     * @return the new stream
     */
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

    /**
     * Returns an {@code IntStream} consisting of the results of applying the
     * given function to the elements of this stream.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">
     *     intermediate operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element
     * @return the new stream
     */
    IntStream mapToInt(ToIntFunction<? super T> mapper);

    /**
     * Returns a {@code LongStream} consisting of the results of applying the
     * given function to the elements of this stream.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element
     * @return the new stream
     */
    LongStream mapToLong(ToLongFunction<? super T> mapper);

    /**
     * Returns a {@code DoubleStream} consisting of the results of applying the
     * given function to the elements of this stream.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element
     * @return the new stream
     */
    DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

    /**
     * Returns a stream consisting of the results of replacing each element of
     * this stream with the contents of a mapped stream produced by applying
     * the provided mapping function to each element.  Each mapped stream is
     * {@link java.util.stream.BaseStream#close() closed} after its contents
     * have been placed into this stream.  (If a mapped stream is {@code null}
     * an empty stream is used, instead.)
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @apiNote
     * The {@code flatMap()} operation has the effect of applying a one-to-many
     * transformation to the elements of the stream, and then flattening the
     * resulting elements into a new stream.
     *
     * <p><b>Examples.</b>
     *
     * <p>If {@code orders} is a stream of purchase orders, and each purchase
     * order contains a collection of line items, then the following produces a
     * stream containing all the line items in all the orders:
     * <pre>{@code
     *     orders.flatMap(order -> order.getLineItems().stream())...
     * }</pre>
     *
     * <p>If {@code path} is the path to a file, then the following produces a
     * stream of the {@code words} contained in that file:
     * <pre>{@code
     *     Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8);
     *     Stream<String> words = lines.flatMap(line -> Stream.of(line.split(" +")));
     * }</pre>
     * The {@code mapper} function passed to {@code flatMap} splits a line,
     * using a simple regular expression, into an array of words, and then
     * creates a stream of words from that array.
     *
     * @param <R> The element type of the new stream
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element which produces a stream
     *               of new values
     * @return the new stream
     */
    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);

    /**
     * Returns an {@code IntStream} consisting of the results of replacing each
     * element of this stream with the contents of a mapped stream produced by
     * applying the provided mapping function to each element.  Each mapped
     * stream is {@link java.util.stream.BaseStream#close() closed} after its
     * contents have been placed into this stream.  (If a mapped stream is
     * {@code null} an empty stream is used, instead.)
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element which produces a stream
     *               of new values
     * @return the new stream
     * @see #flatMap(Function)
     */
    IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);

    /**
     * Returns an {@code LongStream} consisting of the results of replacing each
     * element of this stream with the contents of a mapped stream produced by
     * applying the provided mapping function to each element.  Each mapped
     * stream is {@link java.util.stream.BaseStream#close() closed} after its
     * contents have been placed into this stream.  (If a mapped stream is
     * {@code null} an empty stream is used, instead.)
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element which produces a stream
     *               of new values
     * @return the new stream
     * @see #flatMap(Function)
     */
    LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);

    /**
     * Returns an {@code DoubleStream} consisting of the results of replacing
     * each element of this stream with the contents of a mapped stream produced
     * by applying the provided mapping function to each element.  Each mapped
     * stream is {@link java.util.stream.BaseStream#close() closed} after its
     * contents have placed been into this stream.  (If a mapped stream is
     * {@code null} an empty stream is used, instead.)
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * @param mapper a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *               <a href="package-summary.html#Statelessness">stateless</a>
     *               function to apply to each element which produces a stream
     *               of new values
     * @return the new stream
     * @see #flatMap(Function)
     */
    DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);

    /**
     * Returns a stream consisting of the distinct elements (according to
     * {@link Object#equals(Object)}) of this stream.
     *
     * <p>For ordered streams, the selection of distinct elements is stable
     * (for duplicated elements, the element appearing first in the encounter
     * order is preserved.)  For unordered streams, no stability guarantees
     * are made.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">stateful
     * intermediate operation</a>.
     *
     * @apiNote
     * Preserving stability for {@code distinct()} in parallel pipelines is
     * relatively expensive (requires that the operation act as a full barrier,
     * with substantial buffering overhead), and stability is often not needed.
     * Using an unordered stream source (such as {@link #generate(Supplier)})
     * or removing the ordering constraint with {@link #unordered()} may result
     * in significantly more efficient execution for {@code distinct()} in parallel
     * pipelines, if the semantics of your situation permit.  If consistency
     * with encounter order is required, and you are experiencing poor performance
     * or memory utilization with {@code distinct()} in parallel pipelines,
     * switching to sequential execution with {@link #sequential()} may improve
     * performance.
     *
     * @return the new stream
     */
    Stream<T> distinct();

    /**
     * Returns a stream consisting of the elements of this stream, sorted
     * according to natural order.  If the elements of this stream are not
     * {@code Comparable}, a {@code java.lang.ClassCastException} may be thrown
     * when the terminal operation is executed.
     *
     * <p>For ordered streams, the sort is stable.  For unordered streams, no
     * stability guarantees are made.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">stateful
     * intermediate operation</a>.
     *
     * @return the new stream
     */
    Stream<T> sorted();

    /**
     * Returns a stream consisting of the elements of this stream, sorted
     * according to the provided {@code Comparator}.
     *
     * <p>For ordered streams, the sort is stable.  For unordered streams, no
     * stability guarantees are made.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">stateful
     * intermediate operation</a>.
     *
     * @param comparator a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                   <a href="package-summary.html#Statelessness">stateless</a>
     *                   {@code Comparator} to be used to compare stream elements
     * @return the new stream
     */
    Stream<T> sorted(Comparator<? super T> comparator);

    /**
     * Returns a stream consisting of the elements of this stream, additionally
     * performing the provided action on each element as elements are consumed
     * from the resulting stream.
     *
     * <p>This is an <a href="package-summary.html#StreamOps">intermediate
     * operation</a>.
     *
     * <p>For parallel stream pipelines, the action may be called at
     * whatever time and in whatever thread the element is made available by the
     * upstream operation.  If the action modifies shared state,
     * it is responsible for providing the required synchronization.
     *
     * @apiNote This method exists mainly to support debugging, where you want
     * to see the elements as they flow past a certain point in a pipeline:
     * <pre>{@code
     *     Stream.of("one", "two", "three", "four")
     *         .filter(e -> e.length() > 3)
     *         .peek(e -> System.out.println("Filtered value: " + e))
     *         .map(String::toUpperCase)
     *         .peek(e -> System.out.println("Mapped value: " + e))
     *         .collect(Collectors.toList());
     * }</pre>
     *
     * @param action a <a href="package-summary.html#NonInterference">
     *                 non-interfering</a> action to perform on the elements as
     *                 they are consumed from the stream
     * @return the new stream
     */
    Stream<T> peek(Consumer<? super T> action);

    /**
     * Returns a stream consisting of the elements of this stream, truncated
     * to be no longer than {@code maxSize} in length.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * stateful intermediate operation</a>.
     *
     * @apiNote
     * While {@code limit()} is generally a cheap operation on sequential
     * stream pipelines, it can be quite expensive on ordered parallel pipelines,
     * especially for large values of {@code maxSize}, since {@code limit(n)}
     * is constrained to return not just any <em>n</em> elements, but the
     * <em>first n</em> elements in the encounter order.  Using an unordered
     * stream source (such as {@link #generate(Supplier)}) or removing the
     * ordering constraint with {@link #unordered()} may result in significant
     * speedups of {@code limit()} in parallel pipelines, if the semantics of
     * your situation permit.  If consistency with encounter order is required,
     * and you are experiencing poor performance or memory utilization with
     * {@code limit()} in parallel pipelines, switching to sequential execution
     * with {@link #sequential()} may improve performance.
     *
     * @param maxSize the number of elements the stream should be limited to
     * @return the new stream
     * @throws IllegalArgumentException if {@code maxSize} is negative
     */
    Stream<T> limit(long maxSize);

    /**
     * Returns a stream consisting of the remaining elements of this stream
     * after discarding the first {@code n} elements of the stream.
     * If this stream contains fewer than {@code n} elements then an
     * empty stream will be returned.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">stateful
     * intermediate operation</a>.
     *
     * @apiNote
     * While {@code skip()} is generally a cheap operation on sequential
     * stream pipelines, it can be quite expensive on ordered parallel pipelines,
     * especially for large values of {@code n}, since {@code skip(n)}
     * is constrained to skip not just any <em>n</em> elements, but the
     * <em>first n</em> elements in the encounter order.  Using an unordered
     * stream source (such as {@link #generate(Supplier)}) or removing the
     * ordering constraint with {@link #unordered()} may result in significant
     * speedups of {@code skip()} in parallel pipelines, if the semantics of
     * your situation permit.  If consistency with encounter order is required,
     * and you are experiencing poor performance or memory utilization with
     * {@code skip()} in parallel pipelines, switching to sequential execution
     * with {@link #sequential()} may improve performance.
     *
     * @param n the number of leading elements to skip
     * @return the new stream
     * @throws IllegalArgumentException if {@code n} is negative
     */
    Stream<T> skip(long n);

    /**
     * Performs an action for each element of this stream.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * <p>The behavior of this operation is explicitly nondeterministic.
     * For parallel stream pipelines, this operation does <em>not</em>
     * guarantee to respect the encounter order of the stream, as doing so
     * would sacrifice the benefit of parallelism.  For any given element, the
     * action may be performed at whatever time and in whatever thread the
     * library chooses.  If the action accesses shared state, it is
     * responsible for providing the required synchronization.
     *
     * @param action a <a href="package-summary.html#NonInterference">
     *               non-interfering</a> action to perform on the elements
     */
    void forEach(Consumer<? super T> action);

    /**
     * Performs an action for each element of this stream, in the encounter
     * order of the stream if the stream has a defined encounter order.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * <p>This operation processes the elements one at a time, in encounter
     * order if one exists.  Performing the action for one element
     * <a href="../concurrent/package-summary.html#MemoryVisibility"><i>happens-before</i></a>
     * performing the action for subsequent elements, but for any given element,
     * the action may be performed in whatever thread the library chooses.
     *
     * @param action a <a href="package-summary.html#NonInterference">
     *               non-interfering</a> action to perform on the elements
     * @see #forEach(Consumer)
     */
    void forEachOrdered(Consumer<? super T> action);

    /**
     * Returns an array containing the elements of this stream.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @return an array containing the elements of this stream
     */
    Object[] toArray();

    /**
     * Returns an array containing the elements of this stream, using the
     * provided {@code generator} function to allocate the returned array, as
     * well as any additional arrays that might be required for a partitioned
     * execution or for resizing.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @apiNote
     * The generator function takes an integer, which is the size of the
     * desired array, and produces an array of the desired size.  This can be
     * concisely expressed with an array constructor reference:
     * <pre>{@code
     *     Person[] men = people.stream()
     *                          .filter(p -> p.getGender() == MALE)
     *                          .toArray(Person[]::new);
     * }</pre>
     *
     * @param <A> the element type of the resulting array
     * @param generator a function which produces a new array of the desired
     *                  type and the provided length
     * @return an array containing the elements in this stream
     * @throws ArrayStoreException if the runtime type of the array returned
     * from the array generator is not a supertype of the runtime type of every
     * element in this stream
     */
    <A> A[] toArray(IntFunction<A[]> generator);

    /**
     * Performs a <a href="package-summary.html#Reduction">reduction</a> on the
     * elements of this stream, using the provided identity value and an
     * <a href="package-summary.html#Associativity">associative</a>
     * accumulation function, and returns the reduced value.  This is equivalent
     * to:
     * <pre>{@code
     *     T result = identity;
     *     for (T element : this stream)
     *         result = accumulator.apply(result, element)
     *     return result;
     * }</pre>
     *
     * but is not constrained to execute sequentially.
     *
     * <p>The {@code identity} value must be an identity for the accumulator
     * function. This means that for all {@code t},
     * {@code accumulator.apply(identity, t)} is equal to {@code t}.
     * The {@code accumulator} function must be an
     * <a href="package-summary.html#Associativity">associative</a> function.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @apiNote Sum, min, max, average, and string concatenation are all special
     * cases of reduction. Summing a stream of numbers can be expressed as:
     *
     * <pre>{@code
     *     Integer sum = integers.reduce(0, (a, b) -> a+b);
     * }</pre>
     *
     * or:
     *
     * <pre>{@code
     *     Integer sum = integers.reduce(0, Integer::sum);
     * }</pre>
     *
     * <p>While this may seem a more roundabout way to perform an aggregation
     * compared to simply mutating a running total in a loop, reduction
     * operations parallelize more gracefully, without needing additional
     * synchronization and with greatly reduced risk of data races.
     *
     * @param identity the identity value for the accumulating function
     * @param accumulator an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for combining two values
     * @return the result of the reduction
     */
    T reduce(T identity, BinaryOperator<T> accumulator);

    /**
     * Performs a <a href="package-summary.html#Reduction">reduction</a> on the
     * elements of this stream, using an
     * <a href="package-summary.html#Associativity">associative</a> accumulation
     * function, and returns an {@code Optional} describing the reduced value,
     * if any. This is equivalent to:
     * <pre>{@code
     *     boolean foundAny = false;
     *     T result = null;
     *     for (T element : this stream) {
     *         if (!foundAny) {
     *             foundAny = true;
     *             result = element;
     *         }
     *         else
     *             result = accumulator.apply(result, element);
     *     }
     *     return foundAny ? Optional.of(result) : Optional.empty();
     * }</pre>
     *
     * but is not constrained to execute sequentially.
     *
     * <p>The {@code accumulator} function must be an
     * <a href="package-summary.html#Associativity">associative</a> function.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @param accumulator an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for combining two values
     * @return an {@link Optional} describing the result of the reduction
     * @throws NullPointerException if the result of the reduction is null
     * @see #reduce(Object, BinaryOperator)
     * @see #min(Comparator)
     * @see #max(Comparator)
     */
    Optional<T> reduce(BinaryOperator<T> accumulator);

    /**
     * Performs a <a href="package-summary.html#Reduction">reduction</a> on the
     * elements of this stream, using the provided identity, accumulation and
     * combining functions.  This is equivalent to:
     * <pre>{@code
     *     U result = identity;
     *     for (T element : this stream)
     *         result = accumulator.apply(result, element)
     *     return result;
     * }</pre>
     *
     * but is not constrained to execute sequentially.
     *
     * <p>The {@code identity} value must be an identity for the combiner
     * function.  This means that for all {@code u}, {@code combiner(identity, u)}
     * is equal to {@code u}.  Additionally, the {@code combiner} function
     * must be compatible with the {@code accumulator} function; for all
     * {@code u} and {@code t}, the following must hold:
     * <pre>{@code
     *     combiner.apply(u, accumulator.apply(identity, t)) == accumulator.apply(u, t)
     * }</pre>
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @apiNote Many reductions using this form can be represented more simply
     * by an explicit combination of {@code map} and {@code reduce} operations.
     * The {@code accumulator} function acts as a fused mapper and accumulator,
     * which can sometimes be more efficient than separate mapping and reduction,
     * such as when knowing the previously reduced value allows you to avoid
     * some computation.
     *
     * @param <U> The type of the result
     * @param identity the identity value for the combiner function
     * @param accumulator an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for incorporating an additional element into a result
     * @param combiner an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for combining two values, which must be
     *                    compatible with the accumulator function
     * @return the result of the reduction
     * @see #reduce(BinaryOperator)
     * @see #reduce(Object, BinaryOperator)
     */
    <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);

    /**
     * Performs a <a href="package-summary.html#MutableReduction">mutable
     * reduction</a> operation on the elements of this stream.  A mutable
     * reduction is one in which the reduced value is a mutable result container,
     * such as an {@code ArrayList}, and elements are incorporated by updating
     * the state of the result rather than by replacing the result.  This
     * produces a result equivalent to:
     * <pre>{@code
     *     R result = supplier.get();
     *     for (T element : this stream)
     *         accumulator.accept(result, element);
     *     return result;
     * }</pre>
     *
     * <p>Like {@link #reduce(Object, BinaryOperator)}, {@code collect} operations
     * can be parallelized without requiring additional synchronization.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @apiNote There are many existing classes in the JDK whose signatures are
     * well-suited for use with method references as arguments to {@code collect()}.
     * For example, the following will accumulate strings into an {@code ArrayList}:
     * <pre>{@code
     *     List<String> asList = stringStream.collect(ArrayList::new, ArrayList::add,
     *                                                ArrayList::addAll);
     * }</pre>
     *
     * <p>The following will take a stream of strings and concatenates them into a
     * single string:
     * <pre>{@code
     *     String concat = stringStream.collect(StringBuilder::new, StringBuilder::append,
     *                                          StringBuilder::append)
     *                                 .toString();
     * }</pre>
     *
     * @param <R> type of the result
     * @param supplier a function that creates a new result container. For a
     *                 parallel execution, this function may be called
     *                 multiple times and must return a fresh value each time.
     * @param accumulator an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for incorporating an additional element into a result
     * @param combiner an <a href="package-summary.html#Associativity">associative</a>,
     *                    <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                    <a href="package-summary.html#Statelessness">stateless</a>
     *                    function for combining two values, which must be
     *                    compatible with the accumulator function
     * @return the result of the reduction
     */
    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);

    /**
     * Performs a <a href="package-summary.html#MutableReduction">mutable
     * reduction</a> operation on the elements of this stream using a
     * {@code Collector}.  A {@code Collector}
     * encapsulates the functions used as arguments to
     * {@link #collect(Supplier, BiConsumer, BiConsumer)}, allowing for reuse of
     * collection strategies and composition of collect operations such as
     * multiple-level grouping or partitioning.
     *
     * <p>If the stream is parallel, and the {@code Collector}
     * is {@link Collector.Characteristics#CONCURRENT concurrent}, and
     * either the stream is unordered or the collector is
     * {@link Collector.Characteristics#UNORDERED unordered},
     * then a concurrent reduction will be performed (see {@link Collector} for
     * details on concurrent reduction.)
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * <p>When executed in parallel, multiple intermediate results may be
     * instantiated, populated, and merged so as to maintain isolation of
     * mutable data structures.  Therefore, even when executed in parallel
     * with non-thread-safe data structures (such as {@code ArrayList}), no
     * additional synchronization is needed for a parallel reduction.
     *
     * @apiNote
     * The following will accumulate strings into an ArrayList:
     * <pre>{@code
     *     List<String> asList = stringStream.collect(Collectors.toList());
     * }</pre>
     *
     * <p>The following will classify {@code Person} objects by city:
     * <pre>{@code
     *     Map<String, List<Person>> peopleByCity
     *         = personStream.collect(Collectors.groupingBy(Person::getCity));
     * }</pre>
     *
     * <p>The following will classify {@code Person} objects by state and city,
     * cascading two {@code Collector}s together:
     * <pre>{@code
     *     Map<String, Map<String, List<Person>>> peopleByStateAndCity
     *         = personStream.collect(Collectors.groupingBy(Person::getState,
     *                                                      Collectors.groupingBy(Person::getCity)));
     * }</pre>
     *
     * @param <R> the type of the result
     * @param <A> the intermediate accumulation type of the {@code Collector}
     * @param collector the {@code Collector} describing the reduction
     * @return the result of the reduction
     * @see #collect(Supplier, BiConsumer, BiConsumer)
     * @see Collectors
     */
    <R, A> R collect(Collector<? super T, A, R> collector);

    /**
     * Returns the minimum element of this stream according to the provided
     * {@code Comparator}.  This is a special case of a
     * <a href="package-summary.html#Reduction">reduction</a>.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal operation</a>.
     *
     * @param comparator a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                   <a href="package-summary.html#Statelessness">stateless</a>
     *                   {@code Comparator} to compare elements of this stream
     * @return an {@code Optional} describing the minimum element of this stream,
     * or an empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the minimum element is null
     */
    Optional<T> min(Comparator<? super T> comparator);

    /**
     * Returns the maximum element of this stream according to the provided
     * {@code Comparator}.  This is a special case of a
     * <a href="package-summary.html#Reduction">reduction</a>.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @param comparator a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                   <a href="package-summary.html#Statelessness">stateless</a>
     *                   {@code Comparator} to compare elements of this stream
     * @return an {@code Optional} describing the maximum element of this stream,
     * or an empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the maximum element is null
     */
    Optional<T> max(Comparator<? super T> comparator);

    /**
     * Returns the count of elements in this stream.  This is a special case of
     * a <a href="package-summary.html#Reduction">reduction</a> and is
     * equivalent to:
     * <pre>{@code
     *     return mapToLong(e -> 1L).sum();
     * }</pre>
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal operation</a>.
     *
     * @return the count of elements in this stream
     */
    long count();

    /**
     * Returns whether any elements of this stream match the provided
     * predicate.  May not evaluate the predicate on all elements if not
     * necessary for determining the result.  If the stream is empty then
     * {@code false} is returned and the predicate is not evaluated.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * @apiNote
     * This method evaluates the <em>existential quantification</em> of the
     * predicate over the elements of the stream (for some x P(x)).
     *
     * @param predicate a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                  <a href="package-summary.html#Statelessness">stateless</a>
     *                  predicate to apply to elements of this stream
     * @return {@code true} if any elements of the stream match the provided
     * predicate, otherwise {@code false}
     */
    boolean anyMatch(Predicate<? super T> predicate);

    /**
     * Returns whether all elements of this stream match the provided predicate.
     * May not evaluate the predicate on all elements if not necessary for
     * determining the result.  If the stream is empty then {@code true} is
     * returned and the predicate is not evaluated.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * @apiNote
     * This method evaluates the <em>universal quantification</em> of the
     * predicate over the elements of the stream (for all x P(x)).  If the
     * stream is empty, the quantification is said to be <em>vacuously
     * satisfied</em> and is always {@code true} (regardless of P(x)).
     *
     * @param predicate a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                  <a href="package-summary.html#Statelessness">stateless</a>
     *                  predicate to apply to elements of this stream
     * @return {@code true} if either all elements of the stream match the
     * provided predicate or the stream is empty, otherwise {@code false}
     */
    boolean allMatch(Predicate<? super T> predicate);

    /**
     * Returns whether no elements of this stream match the provided predicate.
     * May not evaluate the predicate on all elements if not necessary for
     * determining the result.  If the stream is empty then {@code true} is
     * returned and the predicate is not evaluated.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * @apiNote
     * This method evaluates the <em>universal quantification</em> of the
     * negated predicate over the elements of the stream (for all x ~P(x)).  If
     * the stream is empty, the quantification is said to be vacuously satisfied
     * and is always {@code true}, regardless of P(x).
     *
     * @param predicate a <a href="package-summary.html#NonInterference">non-interfering</a>,
     *                  <a href="package-summary.html#Statelessness">stateless</a>
     *                  predicate to apply to elements of this stream
     * @return {@code true} if either no elements of the stream match the
     * provided predicate or the stream is empty, otherwise {@code false}
     */
    boolean noneMatch(Predicate<? super T> predicate);

    /**
     * Returns an {@link Optional} describing the first element of this stream,
     * or an empty {@code Optional} if the stream is empty.  If the stream has
     * no encounter order, then any element may be returned.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * @return an {@code Optional} describing the first element of this stream,
     * or an empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the element selected is null
     */
    Optional<T> findFirst();

    /**
     * Returns an {@link Optional} describing some element of the stream, or an
     * empty {@code Optional} if the stream is empty.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * <p>The behavior of this operation is explicitly nondeterministic; it is
     * free to select any element in the stream.  This is to allow for maximal
     * performance in parallel operations; the cost is that multiple invocations
     * on the same source may not return the same result.  (If a stable result
     * is desired, use {@link #findFirst()} instead.)
     *
     * @return an {@code Optional} describing some element of this stream, or an
     * empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the element selected is null
     * @see #findFirst()
     */
    Optional<T> findAny();

    // Static factories

    /**
     * Returns a builder for a {@code Stream}.
     *
     * @param <T> type of elements
     * @return a stream builder
     */
    public static<T> Builder<T> builder() {
        return new Streams.StreamBuilderImpl<>();
    }

    /**
     * Returns an empty sequential {@code Stream}.
     *
     * @param <T> the type of stream elements
     * @return an empty sequential stream
     */
    public static<T> Stream<T> empty() {
        return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
    }

    /**
     * Returns a sequential {@code Stream} containing a single element.
     *
     * @param t the single element
     * @param <T> the type of stream elements
     * @return a singleton sequential stream
     */
    public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }

    /**
     * Returns a sequential ordered stream whose elements are the specified values.
     *
     * @param <T> the type of stream elements
     * @param values the elements of the new stream
     * @return the new stream
     */
    @SafeVarargs
    @SuppressWarnings("varargs") // Creating a stream from an array is safe
    public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }

    /**
     * Returns an infinite sequential ordered {@code Stream} produced by iterative
     * application of a function {@code f} to an initial element {@code seed},
     * producing a {@code Stream} consisting of {@code seed}, {@code f(seed)},
     * {@code f(f(seed))}, etc.
     *
     * <p>The first element (position {@code 0}) in the {@code Stream} will be
     * the provided {@code seed}.  For {@code n > 0}, the element at position
     * {@code n}, will be the result of applying the function {@code f} to the
     * element at position {@code n - 1}.
     *
     * @param <T> the type of stream elements
     * @param seed the initial element
     * @param f a function to be applied to to the previous element to produce
     *          a new element
     * @return a new sequential {@code Stream}
     */
    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
        Objects.requireNonNull(f);
        final Iterator<T> iterator = new Iterator<T>() {
            @SuppressWarnings("unchecked")
            T t = (T) Streams.NONE;

            @Override
            public boolean hasNext() {
                return true;
            }

            @Override
            public T next() {
                return t = (t == Streams.NONE) ? seed : f.apply(t);
            }
        };
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
                iterator,
                Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
    }

    /**
     * Returns an infinite sequential unordered stream where each element is
     * generated by the provided {@code Supplier}.  This is suitable for
     * generating constant streams, streams of random elements, etc.
     *
     * @param <T> the type of stream elements
     * @param s the {@code Supplier} of generated elements
     * @return a new infinite sequential unordered {@code Stream}
     */
    public static<T> Stream<T> generate(Supplier<T> s) {
        Objects.requireNonNull(s);
        return StreamSupport.stream(
                new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
    }

    /**
     * Creates a lazily concatenated stream whose elements are all the
     * elements of the first stream followed by all the elements of the
     * second stream.  The resulting stream is ordered if both
     * of the input streams are ordered, and parallel if either of the input
     * streams is parallel.  When the resulting stream is closed, the close
     * handlers for both input streams are invoked.
     *
     * @implNote
     * Use caution when constructing streams from repeated concatenation.
     * Accessing an element of a deeply concatenated stream can result in deep
     * call chains, or even {@code StackOverflowException}.
     *
     * @param <T> The type of stream elements
     * @param a the first stream
     * @param b the second stream
     * @return the concatenation of the two input streams
     */
    public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
        Objects.requireNonNull(a);
        Objects.requireNonNull(b);

        @SuppressWarnings("unchecked")
        Spliterator<T> split = new Streams.ConcatSpliterator.OfRef<>(
                (Spliterator<T>) a.spliterator(), (Spliterator<T>) b.spliterator());
        Stream<T> stream = StreamSupport.stream(split, a.isParallel() || b.isParallel());
        return stream.onClose(Streams.composedClose(a, b));
    }

    /**
     * A mutable builder for a {@code Stream}.  This allows the creation of a
     * {@code Stream} by generating elements individually and adding them to the
     * {@code Builder} (without the copying overhead that comes from using
     * an {@code ArrayList} as a temporary buffer.)
     *
     * <p>A stream builder has a lifecycle, which starts in a building
     * phase, during which elements can be added, and then transitions to a built
     * phase, after which elements may not be added.  The built phase begins
     * when the {@link #build()} method is called, which creates an ordered
     * {@code Stream} whose elements are the elements that were added to the stream
     * builder, in the order they were added.
     *
     * @param <T> the type of stream elements
     * @see Stream#builder()
     * @since 1.8
     */
    public interface Builder<T> extends Consumer<T> {

        /**
         * Adds an element to the stream being built.
         *
         * @throws IllegalStateException if the builder has already transitioned to
         * the built state
         */
        @Override
        void accept(T t);

        /**
         * Adds an element to the stream being built.
         *
         * @implSpec
         * The default implementation behaves as if:
         * <pre>{@code
         *     accept(t)
         *     return this;
         * }</pre>
         *
         * @param t the element to add
         * @return {@code this} builder
         * @throws IllegalStateException if the builder has already transitioned to
         * the built state
         */
        default Builder<T> add(T t) {
            accept(t);
            return this;
        }

        /**
         * Builds the stream, transitioning this builder to the built state.
         * An {@code IllegalStateException} is thrown if there are further attempts
         * to operate on the builder after it has entered the built state.
         *
         * @return the built stream
         * @throws IllegalStateException if the builder has already transitioned to
         * the built state
         */
        Stream<T> build();

    }
}
```

* 特点
  * 元素序列。可以访问特定元素类型的一组有序值，
  * 源，使用一个提供数据的源，如集合，数组或者输入/输出资源，从有序集合生成流时会保留原有的顺序
  * 数据处理操作，支持类似数据库的操作，及函数式编程中的常用操作，如filter，map,reduce.sort，find,match
  * **流水线** 流操作大多数本身会返回一个流，这样多个操作可以链接起来，那么可以看作类似操作数据库的流水线操作
  * **内部迭代** 流的迭代操作，是在内部进行迭代的。

* 流与集合的区别：

  在于什么时候计算

  * 集合是一个内存中的数据结构，它包含数据结构中目前的所有的值，集合中的每个元素都得先算出来才能添加到集合，集合是我需要就创建，无需考虑需求。
  * 流是在概念上亭的数据结构，你不能添加或者删除元素，元素是按需求进行计算的，input->output,典型的生产者-消费者关系
  * 流只能遍历一次,只能消费一次，而集合会一直存在内存中。
  * 外部迭代与内部迭代，集合使用foreach进行外部迭代，流使用内部迭代，

* **流操作**

  * 中间操作  返回一个流对象的操作都是中间操作：如filter,map，limit, sort .etc

    * filter()  接受一个断言条件去过滤流的所有元素。 这个流的操作是中间的， 这使得我们能够对结果调用另外一个流操作（例如 forEach） 
    * map() 通过给定的功能将每个元素转换成另一个对象。 以下示例将每个字符串转换成大写字符串。当然你也可以使用 map 将每个对象转换成另一种类型。 
    * sorted()  返回流的排序视图。元素按照自然顺序进行排序，除非传递了一个自定义的比较器。 

  * 终端操作 如collect  forEach count 即不会返回流对象的操作。

    *  forEach()  该方法有助于迭代流的所有元素，并对他们中的每一个执行一些操作。 改操作作为 lambda 表达式参数传递 

    * collect()  collect() 方法用于从一个 stream 中接收元素，并将他们存储在一个集合中，并通过参数分类。 

    * match()  各种匹配操作检查某个断言条件是否与流匹配。所有这些操作都是终端并返回一个 Boolean 结果 

    * findAny() 将会返回当前流中的任意元素

    * count 返回符合给定谓词条件的总数

    * reduce（）**归约** 将一个流中的元素进行组合起来。函数式编程语言的术语来讲就是折叠。

      ```
      List<Integer> intList= new ArrayList<>();
      /**
      * 归约
      */
      for (int k = 0; k < 20; k++) {
      intList.add(k);
      }
      int as=intList.stream().reduce(0,(a,b)->a+b);
      System.out.println(as);
      Optional<Integer> as1=intList.stream().reduce(Integer::min);
      System.out.println(as1.get());
      Optional<Integer> as2=intList.stream().reduce(Integer::max);
      System.out.println(as2.get());
      ```

      

  * 短路操作 foreach中的continue或者break的效果

    * anyMatch()  一旦满足断言条件，该方法将返回 true。它不会处理任何更多的元素。  括号里面放一个predicate
    * allMatch（）查看流中的元素是否都能匹配给定的条件
    * noneMatch（）确保流中 的元素没有和给定的谓词相匹配
    * findFirst()  该方法将从流返回第一个元素，然后不再处理任何元素。 括号里面放一个predicate

  * 额外的一些操作 **怎么感觉越来越像sql**

    * distinct() 去重
    * limit（） 截断长度 也可用在无序流set上
    * skip（）跳过元素
    * 

* 流的使用

  * 一个数据源
  * 一个中间操作链
  * 一个终端操作

* **流的构建 **

  * 值构建 使用 Stream.of（T）

  * 使用数组Stream.of(val1, val2, val3….)

    ```
        @SafeVarargs
        @SuppressWarnings("varargs") // Creating a stream from an array is safe
        public static<T> Stream<T> of(T... values) {
            return Arrays.stream(values);
        }
    ```

  * 使用数组 Stream.of(arrayOfElements) 和上面的其实是同一个api

    ```
        @SafeVarargs
        @SuppressWarnings("varargs") // Creating a stream from an array is safe
        public static<T> Stream<T> of(T... values) {
            return Arrays.stream(values);
        }
    ```

  * 使用 someList.stream()

  * 由文件流生成

  * 由函数生成流  使用 Stream.generate(functions ) 或者 Stream.iterate(functions ) 这个生成的是个无限流，函数放在generate()或iterate（）里面

    ```
        /**
         * Returns an infinite sequential unordered stream where each element is
         * generated by the provided {@code Supplier}.  This is suitable for
         * generating constant streams, streams of random elements, etc.
         *
         * @param <T> the type of stream elements
         * @param s the {@code Supplier} of generated elements
         * @return a new infinite sequential unordered {@code Stream}
         */
        public static<T> Stream<T> generate(Supplier<T> s) {
            Objects.requireNonNull(s);
            return StreamSupport.stream(
                    new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
        }
    ```

    ```
        /**
         * Returns an infinite sequential ordered {@code Stream} produced by iterative
         * application of a function {@code f} to an initial element {@code seed},
         * producing a {@code Stream} consisting of {@code seed}, {@code f(seed)},
         * {@code f(f(seed))}, etc.
         *
         * <p>The first element (position {@code 0}) in the {@code Stream} will be
         * the provided {@code seed}.  For {@code n > 0}, the element at position
         * {@code n}, will be the result of applying the function {@code f} to the
         * element at position {@code n - 1}.
         *
         * @param <T> the type of stream elements
         * @param seed the initial element
         * @param f a function to be applied to to the previous element to produce
         *          a new element
         * @return a new sequential {@code Stream}
         */
        public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
            Objects.requireNonNull(f);
            final Iterator<T> iterator = new Iterator<T>() {
                @SuppressWarnings("unchecked")
                T t = (T) Streams.NONE;
    
                @Override
                public boolean hasNext() {
                    return true;
                }
    
                @Override
                public T next() {
                    return t = (t == Streams.NONE) ? seed : f.apply(t);
                }
            };
            return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
                    iterator,
                    Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
        }
    ```

  * 將Streams轉換為其他數據結構

    * 使用 stream.collect(Collectors.xxx())  或 stream.toArray将 Stream 转换成 set list array

      ```
           	List<Project> l1=datas.stream().filter(p12->p12.getStar()>3).collect(Collectors.toList());
      		System.out.println(l1);
      		Set<Project> s1=datas.stream().filter(p12->p12.getStar()>3).collect(Collectors.toSet());
      		System.out.println(s1);
      		Project[] a1=datas.stream().filter(p12->p12.getStar()>3).toArray(Project[]::new);
      ```

    * 其他一些數據結構的轉換在 stream.collect（Collectors.xxx）裏

      ```
      Map<String, Project> map1 = datas.stream().filter(p12 -> p12.getStar() > 3)
      .collect(Collectors.toMap(Project::getLang, Function.identity()));
      System.out.println(map1);
      Map<String, String> map3 =	datas.stream().filter(p12 -> p12.getStar() > 3)
      .collect(Collectors.toMap(Project::getLang, Project::getName));
      ```


         * 用流收集数据
    
           * Collectors.groupingBy分组 二级分组就是将groupBy（）.groupBy（）
    
             ```
             Map<Integer, List<Project>> collect = datas.stream().collect(Collectors.groupingBy(Project::getStar));
             		collect.forEach((k, v) -> {
             			System.out.println(k);
             			System.out.println(v);
             		});
             ```
    
           * Collectors.groupingBy分组与mapping同时使用
    
             ```
             datas.stream().collect(Collectors.groupingBy(Project::getStar, Collectors.mapping(item-> {
                 if (item.getStar() == 1) {
                 return  item.getStar();
                 }
                 return 1l;
             		}, Collectors.toSet())));
             ```
    
           * 流中的最大值，最小值 maxBy,minBy
    
             ```
             Optional<Project> popt = datas.stream().collect(Collectors.maxBy(Comparator.comparingInt(Project::getStar)));
             		Project project3 = popt.get();
             		System.out.println(project3);
             		Optional<Project> popt1 = datas.stream().collect(Collectors.minBy(Comparator.comparingInt(Project::getStar)));
             		System.out.println(popt1.get());
             ```
    
           * 汇总 summingInt
    
             ```
             Integer count2 = datas.stream().collect(Collectors.summingInt(Project::getStar));
                 System.out.println(count2);
             ```
    
           * 链接字符串 需要先用map将元素对象的某个属性提取出来。 joining
    
             ```
             String str = datas.stream().map(Project::getLang).collect(Collectors.joining(","));
             System.out.println(str);
             ```
    
           * 分区 多级分区 使用链式编程 partitioningBy（）.partitioningBy()
    
             ```
             Map<Boolean, List<Project>> collect3 = datas.stream().collect(Collectors.partitioningBy((Project pro)-> pro.getStar()>3));
             		collect3.forEach((k,v)->{
             			System.out.println(k);
             			System.out.println(v.toString());
             		});
             ```
    
             **TODO:并行流，组合异步，Fork/Join分治框架，时间新api**