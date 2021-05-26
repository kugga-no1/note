



## String table

![image-20210526105522633](../assert/image-20210526105522633.png)

s1: "a"放入常量池

s2:"b"放入常量池

s3:"ab"放入常量池  （编译优化，直接在编译阶段把”a“+"b"编译为"ab"）

s4: new Stringbuffer,用append方法拼接”a“ ”b“，最后toString，相当于重新new了一个String对象，放在堆中

s5:常量池已经有"ab" 直接引用

s6:尝试把s4（”ab“）放入常量池，常量池中已有"ab"，所以放入失败，直接返回常量池中对象给s6



s3==s4? :  s3是常量池中“ab”，s4是new的string对象，结果为false

s3==s5? : s3是常量池中“ab”，s5企图把"ab"放入常量池，但常量池已有，直接引用常量池中对象，所以结果为true

s3==s6? :  s3是常量池中“ab”;尝试试把s4（”ab“）放入常量池，常量池中已有"ab"，所以放入失败，直接返回常量池中对象给s6  ;所以为true



x1==x2?: 当前的话是new个x2，然后成功把x2放入常量池，然后x1直接引用常量池对象，结果为true

​				如果x2.intern();和String x1="cd"；交换位置  ，那么在String x1="cd"时把cd放入常量池，而x2.intern();由于常量池中已有cd，不会成功把x2放入常量池，所以x1==x2为false；



**jdk1.6和jdk1.8的intern的区别：**（String s6=s4.intern() 区别在于若常量池中没有，1.8会把s4放入池中，而1.6是复制一份放入池中，真正的s4没有入池）![image-20210526110735792](../assert/image-20210526110735792.png)

