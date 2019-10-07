#### 广播变量的引入
+ 不同算子的计算数据之间不能像Java数组之间一样互相访问，而广播变量Broadcast便是解决这种情况的。
+ 广播变量可以理解为是一个公共的共享变量
+ 把一个dataset 数据集广播出去，然后不同的task在节点上都能够获取到，这个数据在每个节点上只会存在一份

>
    1：初始化数据
    DataSet<Integer> num = env.fromElements(1, 2, 3)
    2：广播数据
    .withBroadcastSet(toBroadcast, "num");
    3：获取数据
    Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("num");

    注意：
    1：广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大。因为广播出去的数据，会常驻内存，除非程序执行结束
    2：广播变量在初始化广播出去以后不支持修改，这样才能保证每个节点的数据都是一致的。

+ 在checkpoint时，所有的 task 都会 checkpoint 下他们的广播状态
+ 广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大，百兆左右可以接受，Gb不能接受

#### 案例
>
    public class BroadCastTest {

    public static void main(String[] args) throws Exception{
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //1.封装一个DataSet
        DataSet<Integer> broadcast = env.fromElements(1, 2, 3);
        DataSet<String> data = env.fromElements("a", "b");
        data.map(new RichMapFunction<String, String>() {
            private List list = new ArrayList();
            @Override
            public void open(Configuration parameters) throws Exception {
                // 3. 获取广播的DataSet数据 作为一个Collection
                Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("number");
                list.addAll(broadcastSet);
            }

            @Override
            public String map(String value) throws Exception {
                return value + ": "+ list;
            }
        }).withBroadcastSet(broadcast, "number") 
            // 2. 广播的broadcast
          .printToErr();//打印到err方便查看
        }
    }