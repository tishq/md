



# mapreduce实操

- ### 序列化

  ```java
  //（1）必须实现Writable接口
  //（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造
  public FlowBean() {
  	super();
  }
  //（3）重写序列化方法
  @Override
  public void write(DataOutput out) throws IOException {
  	out.writeLong(upFlow);
  	out.writeLong(downFlow);
  	out.writeLong(sumFlow);
  }
  //（4）重写反序列化方法
  @Override
  public void readFields(DataInput in) throws IOException {
  	upFlow = in.readLong();
  	downFlow = in.readLong();
  	sumFlow = in.readLong();
  }
  //（5）注意反序列化的顺序和序列化的顺序完全一致
  //（6）要想把结果显示在文件中，需要重写toString()，可用”\t”分开，方便后续用。
  //（7）如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。详见后面排序案例。
  @Override
  public int compareTo(FlowBean o) {
  	// 倒序排列，从大到小
  	return this.sumFlow > o.getSumFlow() ? -1 : 1;
  }
  ```

  

- ### Reduce Join

  /home/hq/TISHQ/md/imgs/2019-07-08_001.png
  
  

- ### Map Join

  ```java
  1．使用场景
  Map Join适用于一张表十分小、一张表很大的场景。
  2．优点
  思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？
  在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。
  3．具体办法：采用DistributedCache
  	（1）在Mapper的setup阶段，将文件读取到缓存集合中。
  	（2）在驱动函数中加载缓存。
  // 缓存普通文件到Task运行节点。
  job.addCacheFile(new URI("file://e:/cache/pd.txt"));
  ```

  

- ### 倒排索引

  ```java
  k-v反转
  ```

  

- ### 找共同好友

  ```java
  倒排索引+双重for循环
  ```

  

- ### TopN

  ```java
  // 定义一个TreeMap作为存储数据的容器（天然按key排序）
  	private TreeMap<FlowBean, Text> flowMap = new TreeMap<FlowBean, Text>();
// 4向TreeMap中添加数据
  	flowMap.put(kBean, v);
  		
  // 5 限制TreeMap的数据量，超过10条就删除掉流量最小的一条数据
  	if (flowMap.size() > 10) {
      //		flowMap.remove(flowMap.firstKey());
      flowMap.remove(flowMap.lastKey());		
  }
  ```
  
  