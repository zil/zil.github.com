---
layout: post
title: No Guava,No Java
description: ""
tags: [guava,java]
---
{% include JB/setup %}

# Guava,Google开源的Java基础功能的辅助包

从零星了解到实际使用，Guava帮助自己(优雅的)解决了不少问题.  
每次翻阅Guava源码或文档，发现新的 数据结构或抽象，都会不禁感叹Guava的设计和实现.  
举几个自己用过的地方，希望能帮助读者了解Guava,减少些DRY.

# Splitter/Joiner 字符 分割器/连接器

    import java.util.Arrays;
    import java.util.List;  
    import com.google.common.base.Splitter;
    import com.google.common.base.Joiner;
    
    class Str{
        public static void main(String[] args){
    	Splitter s =  Splitter.on(',').trimResults().omitEmptyStrings();
    	String splitted = "first, second,third,,";
    	System.out.printf("%12s: %s\n","split.result",s.splitToList(splitted));
    
    	Joiner j = Joiner.on(',').skipNulls();
    	List list = Arrays.asList("hi",null,"joiner");
    	System.out.printf("%12s: %s\n","join.result", j.join(list));
        }
    }

# Multiset 带统计的Set，记录每个元素添加的次数

    import java.util.Set;
    import com.google.common.collect.HashMultiset;
    import com.google.common.collect.Multiset;
    
    class Play {
        public static void main(String[] args){
    	Multiset<Integer> ms = HashMultiset.create();
    	ms.add(3);ms.add(3);
    	ms.add(4);ms.add(5);
    	System.out.println("元素 个数");
    	for(Integer i: ms.elementSet()){
    	    System.out.printf("%d %d\n",i,ms.count(i));
    	}
        }
    }

# Multimap 每个键包含多个值的map,List分组时很handy

    import com.google.common.collect.ArrayListMultimap;
    import com.google.common.collect.Multimap;
    
    class Maps{
        public static void main(String[] args){
    	Multimap<String,Integer> mm = ArrayListMultimap.create();
    	mm.put("one",10);
    	mm.put("two",20);mm.put("two",21);
    	mm.put("three",30);mm.put("three",31);mm.put("three",32);
    	//mm.keys() => Multiset
    	for(String k:mm.keySet()){
    	    System.out.printf("%-5s %s\n",k,mm.get(k));
    	}
        }
    }

# 缓存 配置丰富灵活/加载器多样

    import com.google.common.cache.CacheBuilder;
    import com.google.common.cache.CacheLoader;
    import com.google.common.cache.LoadingCache;
    import com.google.common.base.Stopwatch;  
    import java.util.concurrent.ExecutionException;
    import java.util.concurrent.TimeUnit;
    
    class Caches{
        static void loadIt(LoadingCache<Integer,Long> cache) throws ExecutionException{
    	for(int i=0;i<950;i++){
    	    cache.get(i);
    	}
        }
    
        public static void main(String[] args){
    	final LoadingCache<Integer,Long> cache = CacheBuilder.newBuilder()
    		.maximumSize(1000).build(new CacheLoader<Integer,Long>(){
    		    public Long load(Integer n) throws Exception {
    			TimeUnit.MILLISECONDS.sleep(1);
    			return Long.valueOf(n);
    		    }
    		});
    	try{
    	    Stopwatch watch = Stopwatch.createStarted();
    
    	    loadIt(cache);
    	    System.out.println("no.cache-耗时(ms):" + watch.elapsed(TimeUnit.MILLISECONDS));
    
    	    watch.reset();
    	    watch.start();
    
    	    loadIt(cache);
    	    System.out.println("cache.hit-耗时(ms):" + watch.elapsed(TimeUnit.MILLISECONDS));
    	}catch (ExecutionException e){}
        }
    }
