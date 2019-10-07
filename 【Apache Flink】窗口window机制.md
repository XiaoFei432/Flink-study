#### 窗口类型
+ 窗口划分
    - time-window 按时间划分
    - count-window 按数据划分
+ 窗口属性
    - size 窗口大小
    - interval Task时间间隔
    - 如果size=interval,那么就会形成tumbling-window(无重叠数据)
    - 如果size>interval,那么就会形成sliding-window(有重叠数据)
    - 如果size<interval,那么这种窗口将会丢失数据。比如每5秒钟，统计过去3秒的通过路口汽车的数据，将会漏掉2秒钟的数据。
