
### nextInt
Random.nextInt()方法，是生成一个随机的int值，该值介于[0,n)的区间，也就是0到n之间的随机int值，包含0而不包含n。

> int nextInt()            //随机返回一个int型整数
>
> int nextInt(int num)         //随机返回一个值在[0,num)的int类型的整数,包括0不包括num

例如：Random.nextInt(4)将产生0,1,2,3这4个数字中的任何一个数字，注意这里不是0-4，而是0-3

```java
    /**
     * 生成[0,10)区间的整数
     */
    @Test
    public void RandomNextIntDemo2(){
        Random r = new Random();
        int n2 = r.nextInt(10);
        int n3 = Math.abs(r.nextInt() % 10);
        System.out.println("n2:"+n2);
        System.out.println("n3:"+n3);
    }

```



ThreadLocalRandom.current().nextInt();

Utils.toPositive();